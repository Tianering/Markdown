## mmdetection
### 配置文件
https://mmdetection.readthedocs.io/zh_CN/latest/tutorials/config.html  

8 gpus、imgs_per_gpu = 2：lr = 0.02；
2 gpus、imgs_per_gpu = 2 或 4 gpus、imgs_per_gpu = 1：lr = 0.005；
4 gpus、imgs_per_gpu = 2：lr = 0.01  

训练的batch量与学习率成正比，按照这个比例 lr=0.01 for 4 GPUs * 2 img/gpu 修改  
原学习率按照八张显卡设置 ：lr = (base_lr / 8) x num_gpus x (img_per_gpu/2) 
samples_per_gpu (int): Number of training samples on each GPU, i.e.,batch size of each GPU.  
workers_per_gpu (int): How many subprocesses to use for data loading for each GPU.  

单个模型的配置文件序列等级高于._base_文件夹中的配置文件中的配置  
workflow : runner 的工作流程，[('train', 1)] 表示只有一个工作流且工作流仅执行一次。根据 total_epochs 工作流训练 12个回合。
```
Workflow is a list of (phase, epochs) to specify the running order and epochs. By default it is set to be

workflow = [('train', 1)]
which means running 1 epoch for training. Sometimes user may want to check some metrics (e.g. loss, accuracy) about the model on the validate set. In such case, we can set the workflow as

[('train', 1), ('val', 1)]
so that 1 epoch for training and 1 epoch for validation will be run iteratively.

Note:

The parameters of model will not be updated during val epoch.
Keyword total_epochs in the config only controls the number of training epochs and will not affect the validation workflow.
Workflows [('train', 1), ('val', 1)] and [('train', 1)] will not change the behavior of EvalHook because EvalHook is called by after_train_epoch and validation workflow only affect hooks that are called through after_val_epoch. Therefore, the only difference between [('train', 1), ('val', 1)] and [('train', 1)] is that the runner will calculate losses on validation set after each training epoch.
```
### ATSS_目标检测的自适应正负anchor选择(Adaptive Training Sample Selection)   
选取retinaNet与FCOS进行对比，主要对比正负样本定义和回归开始状态的差异  

	RetinaNet使用IoU阈值来区分正负anchor bbox，处于中间的全部忽略
	FCOS使用空间尺寸和尺寸限制来区分正负anchor point，正样本首先必须在GT box内，其次需要是GT尺寸对应的层，其余均为负样本  
	RetinaNet预测4个偏移值对anchor box进行调整输出，而FCOS则预测4个相对于anchor point值对anchor box进行调整输出    

经过交叉实验发现，在相同的正负样本定义下RetinaNet和FCOS性能几乎一样，不同的定义方法性能差异较大，而回归初始状态对性能影响不大，由此可知正负样本的确定方法是影响性能的重要一环  
ATSS方法：

	该方法根据目标的相关统计特征自动进行正负样本的选择，具体逻辑如算法1所示
	对于每个GT box，首先在每个特征层找到中心点最近的k个候选anchor boxes(非预测结果)，计算候选box与GT间的IoU，计算IoU的均值和标准差，得到IoU阈值，最后选择阈值大于均值与标准差之和的box作为最后的输出。如果anchor box对应多个GT，则选择IoU最大的GT  

![](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Atss.jpg)  
resnet101 23.3 task/s 
2*2
7335MiB+8973MiB 3080*2
resnet50   29.0 task/s
4*4 
8415+8207 3080*2
### RetinaNet_Focal Loss  
针对One-Stage算法采样时导致的正负样本不平衡问题，提出loss函数解决类别标签不平衡问题  

	Two-Stage算法在生成框阶段使用Selective Search, EdgeBoxes, RPN的结构极大的减少了背景框的数量，使其大约为1k~2k。在分类阶段，使用一些策略，如使前景背景的比例为1:3或者OHEM算法，这样就使得正负样本达到了一个平衡  
	One-Stage算法在进行将采样的同时产生预选框，在实际中经常会产生很多框，造成了样本间的极度不平衡，虽然有时会使用bootstrapping和hard example mining，但是效率很低  

Focal loss在cross entropy loss的基础上增加一个调节因子![](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Focalloss.svg)  

### FCOS  
在anchor-based目标检测算法中，anchor的大小、数量、长宽比等对检测性能影响很大，针对不同的任务需要重新进行设置才能保证检测器的效果；同时在匹配真实框时会生成大量anchor，容易造成样本件的不平衡，并且训练过程中对所有anchor计算IOU会消耗大量内存和时间  
FCOS相较于早期其他的AnchorFree算法解决了两个问题：  
1. 物体重叠问题  
	在FCOS不再只是学习预测中心点，而是对于每个位置，预测他到GTbox上下左右的四个距离的前提情况下，一旦两个GTbox发生重叠，将会发生样本冲突  
	为了解决此问题，FCOS参考FPN的工作，使用基于FPN的多尺度检测可以很好的减少这种情况  
2. center-ness  
	按照FOCS的思路，不包含目标的像素也会成为正例，同时FCOS也认为没有目标的预测应该贡献小，相反包含物体的像素则贡献大  
	作者提出Center-ness来解决此问题，并使用BCE loss对center-cess进行优化，当loss越小，center-ness就越接近1，即回归框的中心越接近真实框   
	![](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Centerness.jpg)  

resnet50_caffe 1x	4*4  
10001+7815 3080*2
resnet50_caffe 2x mstrain img_scale=[(1333, 640), (1333, 800) 多尺度策略 (multi scale strategy)
10001+9229  

#### NAS-FCOS  
搜索空间中加入了deformable卷积  
将head分成两个部分，不共享的权重和共享权重，前者实际可以合并到特征融合中，后者可以类比RetinaNet中不同尺度权重共享的部分  
相比NAS-FPN，没有太大的创新点，性能提升去除deformable卷积的影响，涨点相对不是很明显  
resnet50_caffe 4*4  
9491+9297 3080*2  
### SSD_Single Shot MultiBox Detector  
从YOLO中继承了将detection转化为regression的思路，一次完成目标定位与分类  
基于Faster RCNN中的Anchor，提出了相似的Prior box  

	Prior Box 按照一定规则生成
	SSD使用感受野小的feature map检测小目标，使用感受野大的feature map检测更大目标
	要人工设置prior box的min_size，max_size和aspect_ratio值

加入基于特征金字塔（Pyramidal Feature Hierarchy）的检测方式，即在不同感受野的feature map上预测目标  

	SSD采用金字塔结构，即利用了conv4-3/conv-7/conv6-2/conv7-2/conv8_2/conv9_2这些大小不同的feature maps，在多个feature maps上同时进行softmax分类和位置回归
	虽然采用了pyramdial feature hierarchy的思路，但是对小目标的recall依然一般，并没有达到碾压Faster RCNN的级别
	作者认为，这是由于SSD使用conv4_3低级feature去检测小目标，而低级特征卷积层数少，存在特征提取不充分的问题

ssd300	32*3
8029+8029 3080*2   

120.3 task/s 

ssd512 16*3  
9653*2	3080*2  
75.5 task/s  

### DETR_End To End Object Detection with Transformer  
将Transformer运用到object detection邻域，取代某些模型需要手工设计的工作（非极大值抑制等），其整体结构与Transformer类似：  
首先通过Backbone得到的特征铺平，加上Position信息之后送到Encoder中  
之后得到一些candidates的特征，这100个candidates被Decoder并行解码，以得到最后的检测框  
![DETR结构](https://raw.githubusercontent.com/Tianering/Markdown/master/images/DETR.jpg)
1. Transformer Encoder
由图可知DETR并不是一个完全由Transformer处理的架构，还是需要依赖于CNN作为backbone提取特征，将CNN backbone输出的feature map转化为能够被Transformer Encoder处理的序列化数据的过程包括：
	+ 维度压缩：将CNN backbone输出的 CxHxW 维的feature map先用 1x1 convolution处理，将channels数量从 C 压缩到 d ，即得到 dxHxW 维的新feature map
	+ 序列化数据：将空间的维度（高和宽）压缩为一个维度，即把上一步得到的 dxHxW 维的feature map通过reshape成 dxHW 维的feature map；
	+ positoin encoding: 由于transformer模型是顺序无关的，而 dxHW 维feature map中 HW 维度显然与原图的位置有关，所以需要加上position encoding反映位置信息

2. Transformer Decoder


### 使用COCO公共数据集进行模型训练评估  

|   Network    | Dataset  | Backbone  |  style  |  Lr schd  |  Mem  |  mAP  | AP50  |  APs  |  APm  | APl   |
| :----------: | :------: | :-------: | :-----: | :-------: | :---: | :---: | :---: | :---: | :---: | ----- |
|     ATSS     | COCO2014 | Resnet101 | Pytorch | 0.0025_1x | 410.2 | 0.395 | 0.577 | 0.217 | 0.423 | 0.491 |
|     ATSS     | COCO2014 | Resnet50  | Pytorch | 0.0025_1x | 257.9 | 0.361 | 0.546 | 0.199 | 0.386 | 0.447 |
|     FCOS     | COCO2014 | Resnet50  |  Caffe  | 0.0025_1x | 257.7 | 0.331 | 0.529 | 0.173 | 0.357 | 0.415 |
| FCOS_mstrain | COCO2014 | Resnet50  |  Caffe  | 0.0025_2x | 257.7 | 0.356 | 0.554 | 0.196 | 0.381 | 0.443 |
|   FCOS_NAS   | COCO2014 | Resnet50  |  Caffe  | 0.0025_1x |       |       |       |       |       |       |
|    SSD300    | COCO2014 |   VGG16   |  Caffe  | 0.002_2x  | 274.5 | 0.242 | 0.427 | 0.065 | 0.254 | 0.384 |
|    SSD512    | COCO2014 |   VGG16   |  Caffe  | 0.001_2x  | 288.4 | 0.276 | 0.478 | 0.109 | 0.307 | 0.406 |


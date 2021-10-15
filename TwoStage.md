# Two Stage

## RCNN
### 候选框搜索阶段  
通过Selective Search方法搜索出所有可能是物体的区域,通过该方法搜索出2000个候选框

```
Selective Search  
使用一种过分割手段，将图像分割成小区域  
查看现有小区域，按照合并规则合并可能性最高的相邻两个区域，直到整张图像合并成一个区域
输出所有曾经存在过的区域  
    合并规则：  
    	- 颜色（颜色直方图）相近的  
        - 纹理（梯度直方图）相近的 
    
        - 合并后总面积小的： 保证合并操作的尺度较为均匀，避免一个大区域陆续“吃掉”其他小区域 
        - 合并后，总面积在其BBOX中所占比例大的： 保证合并后形状规则
```

之后将所有候选框进行缩放处理  
```
1. 各向异性缩放
2. 各向同性缩放
	- 先扩充后裁剪
	- 先裁剪后扩充
```
### CNN特征提取阶段
1. 网络结构设计阶段
	选择经典的Alexnet或者VGG16
	VGG16相比Alexnet选择了较小的卷积核、选择了较小的跨步，网络精度高，同时计算量为Alexnet的7倍
2. 网络有监督预训练阶段
	参数初始化部分：物体检测的一个难点在于，物体标签训练数据少，如果要直接采用随机初始化CNN参数的方法，那么目前的训练数据量是远远不够的这种情况下，最好的是采用某些方法，把参数初始化了，然后在进行有监督的参数微调，这里文献采用的是有监督的预训练
	所以paper在设计网络结构的时候，是直接用Alexnet的网络，然后连参数也是直接采用它的参数，作为初始的参数值，然后再fine-tuning训练。网络优化求解时采用随机梯度下降法，学习率大小为0.001；
3. fine-tuning阶段
	利用已有模型训练其他数据集，通过对ImageNet上训练出来的模型进行微调，然后应用到自己的数据集上，通常可以得到非常好的结果
	同时在目标任务上达到很高performance所需要的数据的量相对很少
### SVM训练
二分类问题-为每一个物体类训练一个SVM分类器

位置精修： 目标检测问题的衡量标准是重叠面积：许多看似准确的检测结果，往往因为候选框不够准确，重叠面积很小。故需要一个位置精修步骤

回归器：对每一类目标，使用一个线性脊回归器进行精修。正则项λ=10000。 输入为深度网络pool5层的4096维特征，输出为xy方向的缩放和平移

训练样本：判定为本类的候选框中和真值重叠面积大于0.6的候选框

### 测试阶段
使用selective search的方法在测试图片上提取2000个region propasals ，将每个region proposals归一化到227x227，然后再CNN中正向传播，将最后一层得到的特征提取出来
然后对于每一个类别，使用为这一类训练的SVM分类器对提取的特征向量进行打分，得到测试图片中对于所有region proposals的对于这一类的分数，再使用贪心的非极大值抑制（NMS）去除相交的多余的框。再对这些框进行canny边缘检测，就可以得到bounding-box(then B-BoxRegression)

（非极大值抑制（NMS）先计算出每一个bounding box的面积，然后根据score进行排序，把score最大的bounding box作为选定的框，计算其余bounding box与当前最大score与box的IoU，去除IoU大于设定的阈值的bounding box。然后重复上面的过程，直至候选bounding box为空，然后再将score小于一定阈值的选定框删除得到这一类的结果（然后继续进行下一个分类）


## Fast R-CNN

### end-to-end
将原始图片输入卷积网络中得到特征图，再使用建议框对特征图提取特征框
在RCNN中，建议框重合部分非常多，卷积重复计算严重，Fast R-CNN则对每一个位置只计算一次卷积，大大减少了计算量

### Rol层
通过ROL池化层将得到的特征框转化为相同大小
![](https://pic2.zhimg.com/v2-f13f58bfae58fe6ab34f02934e7d21dd_r.jpg)


## Faster RCNN
Faster RCNN是两阶段的目标检测算法，包括阶段一的Region proposal以及阶段二的bounding box回归和分类
Faster RCNN使用CNN提取图像特征，然后使用region proposal network（RPN）去提取出ROI，然后使用ROI pooling将这些ROI全部变成固定尺寸，再喂给全连接层进行Bounding box回归和分类预测。
### Conv layers

### Refion Proposal Networks(RPN)

OpenCV adaboost使用滑动窗口+图像金字塔生成检测框；R-CNN使用SS(Selective Search)方法生成检测框  
而Faster RCNN则抛弃了传统的滑动窗口和SS方法，直接使用RPN生成检测框，可以极大提升检测框的生成速度

### Rol pooling
由SPP（Spatial Pyramid Pooling）发展而来 
https://zhuanlan.zhihu.com/p/79888509
### Classification
### Faster RCNN 训练
![](https://pic2.zhimg.com/80/v2-ddfcf3dc29976e384b047418aec9002d_720w.jpg)

mx-net_coco
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.439
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.797
mx-net_voc
AP for 0.5 = 0.8145
Mean AP = 0.5290

Faster RCNN_resnet101_fpn_1x	8000MBx2	6x2

 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.730
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=1000 ] = 0.962

bbox_mAP_s: 0.5860, bbox_mAP_m: 0.7810, bbox_mAP_l: 0.8440,


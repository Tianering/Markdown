## mmdetection
### 配置文件
https://mmdetection.readthedocs.io/zh_CN/latest/tutorials/config.html  

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
### ATSS——目标检测的自适应正负anchor选择(Adaptive Training Sample Selection)
https://zhuanlan.zhihu.com/p/115407465
7335MiB+8973MiB 3080*2	23.3 task/s  
选取retinaNet与FCOS进行对比，主要对比正负样本定义和回归开始状态的差异  

	RetinaNet使用IoU阈值([公式],[公式])来区分正负anchor bbox，处于中间的全部忽略。FCOS使用空间尺寸和尺寸限制来区分正负anchor point，正样本首先必须在GT box内，其次需要是GT尺寸对应的层，其余均为负样本  
	RetinaNet预测4个偏移值对anchor box进行调整输出，而FCOS则预测4个相对于anchor point值对anchor box进行调整输出    

经过交叉实验发现，在相同的正负样本定义下RetinaNet和FCOS性能几乎一样，不同的定义方法性能差异较大，而回归初始状态对性能影响不大，由此可知正负样本的确定方法是影响性能的重要一环  
ATSS方法：

	该方法根据目标的相关统计特征自动进行正负样本的选择，具体逻辑如算法1所示。对于每个GT box，首先在每个特征层找到中心点最近的k个候选anchor boxes(非预测结果)，计算候选box与GT间的IoU，计算IoU的均值和标准差，得到IoU阈值，最后选择阈值大于均值与标准差之和的box作为最后的输出。如果anchor box对应多个GT，则选择IoU最大的GT

### 使用COCO公共数据集进行模型训练评估  

| Network | Dataset | Backbone | style | Lr schd | Mem | mAP | AP50 | APs | APm | APl |
|:-------------:|:----------:|:-------:|:--------:|:--------------:|:------:|:-------:|:------:|:--------:|:--------:|::--------:|
| ATSS | COCO2014 | Resnet101 | Pytorch | 0.0025 |  |  |  |  |  |  |
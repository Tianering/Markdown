### Decoupled Head

Yolox将原本的Yolo head修改为Decoupled Head  
使用Decoupled Head时收敛速度会更快，精度会更高，但是将检测头解耦，会增加运算的复杂度  
Yolox经过速度和性能的权衡，使用1×1卷积先进行降维，并在之后分支中使用3×3卷积  
![](https://raw.githubusercontent.com/Tianering/Markdown/master/images/DecoupledHead.jpg)

1. 主要对目标框类别、预测分数
2. 主要判断目标框是前景还是背景
3. 主要对目标框的坐标信息进行预测

### Anchor free
Yolo系列例如v3、v4、v5等都采用Anchor Based方式提取目标框，进而和标注的GT进行对比并判断差距

    Anchor:
    物体检测问题通常都被建模成对一些候选区域进行分类和回归的问题
    在单阶段检测器中，这些候选区域就是通过滑窗方式产生的 anchor
    在两阶段检测器中，候选区域是 RPN 生成的 proposal，但是 RPN 本身仍然是对滑窗方式产生的 anchor 进行分类和回归
    根据anchor与ground truth的IoU（交并比）损失对anchor的长宽以及位置进行回归，使其越来越接近ground truth，在回归的同时预测anchor的类别，最终输出这些回归分类好的anchors

而yolox采用的anchor-free方法，这种方式相较于Anchor based来说参数量少，节约时间成本  

    Anchor free：
    一种是首先定位到多个预定义或自学习的关键点，然后约束物体的空间范围，称为Keypoint-based方法
    一种是利用中心点或中心目标区域来定义正样本，然后预测其到目标四个边的距离，称为Center-based方法

## YOLOF
针对FPN进行思考并且引入优化方案
https://zhuanlan.zhihu.com/p/359462538
### FPN  
多尺度融合：提高特征的丰富程度  
分治法：将目标检测任务按照目标尺寸不同，分成若干个检测子任务  
通过对比设计的MiMo、MiSo、SiMo、SiSo四种解码器，得出SiMo仅采用C5特征且不进行特征融合即可取得与MiMo编码器相当的性能  
由此推理得出：  

1. C5包含了可以充分用于不同尺度目标的上下文信息
2. 多尺度融合带来的收益远小于分治法，相反分治法将不同尺度的目标检测进行拆分处理，缓解了优化问题  
![](https://pic2.zhimg.com/80/v2-8f73b7b52bad061d1870de9670826349_720w.jpg)  
选用了SiSo结构，以避免SiMo庞大的计算量  
使用Dilated Encoder模块替代FPN

### Dilated Encoder  
使用孔洞卷积（Dilated Convolution）操作增大C5特征的感受野  
positive anchor不均匀问题
使用SiSoEncoder的情况下，锚点的数量会大量减少，导致稀疏锚点，进一步导致Max-IoU时的不匹配；正锚点的不平衡问题导致检测器更多关注于大目标而忽略了小目标  
提出Uniform Matching策略  
### 网络结构
![](https://pic1.zhimg.com/v2-cb6130ff8ed6e5f48c2eb2b1b9e4d994_r.jpg)

YOLOv5P6  
```
Model Summary: 773 layers, 141821340 parameters, 141821340 gradients, 223.1 GFLOPs  
best ap50_95:79.5 model size:282M  
```
YOLOX
```
YOLOX_x(0.33)   
best ap50_95:78.0 model size:71MB  
YOLOX_x(1)  
best ap50_95:79.42 model size:434MB  
YOLOX_x(1.25) 
best ap50_95:79.65 model size:7MB
```
YOLOF
```
yolof_r50_c5_8x8_1x_coco
10x2 9837x2  1333x800
30：
bbox_mAP: 0.7350, bbox_mAP_50: 0.9700
300：
Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.736
Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=1000 ] = 0.958
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=1000 ] = 0.571
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=1000 ] = 0.793
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=1000 ] = 0.884
```


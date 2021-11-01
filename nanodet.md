### Generalized Focal Loss
选择使用Generalized Focal Loss作为损失函数，其不仅去掉了难以训练的Centerness，而且还省去了这一分支上的大量卷积，减少了检测头的计算开销，非常适合移动端的轻量化部署
附[李翔大白话Generalized Focal Loss](https://zhuanlan.zhihu.com/p/147691786)

### 检测头轻量化  
与FCOS不同的是，对于FPN得到的多尺度Feature Map还是选择每一层特征使用一组卷积，并且在检测头上使用BN替代GN作为归一化方式，其在推理时能够将其归一化的参数直接融合进卷积中，从而节省归一化操作时间  
同时使用深度可分离卷积替换普通卷积，并且将卷积堆叠的数量从4个减少2组  
在通道数上，将256维压缩至96维（通道数保持为8或16的倍数，可以享受大部分推理框架的并行加速）  
最后借鉴yolo系列，将边框回归和分类使用同一组卷积进行计算，然后split成两份  
![NanoDet检测头](https://raw.githubusercontent.com/Tianering/Markdown/master/images/NanodetHead.png)  

### FPN层改进
选择极小版本的PAN进行特征融合  
完全去掉PAN中的所有卷积，只保留从骨干网络特征提取后的1x1卷积来进行特征通道维度的对齐，上采样和下采样均使用插值来完成  
与yolo使用的concatenate操作不同，我选择将多尺度的Feature Map直接相加，使得整个特征融合模块的计算量变得非常非常小  


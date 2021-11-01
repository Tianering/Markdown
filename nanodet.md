### Generalized Focal Loss
选择使用Generalized Focal Loss作为损失函数，其不仅去掉了难以训练的Centerness，而且还省去了这一分支上的大量卷积，减少了检测头的计算开销，非常适合移动端的轻量化部署
附[李翔大白话Generalized Focal Loss](https://zhuanlan.zhihu.com/p/147691786)

### 检测头轻量化  
与FCOS不同的是，对于FPN得到的多尺度Feature Map还是选择每一层特征使用一组卷积，并且在检测头上使用BN替代GN作为归一化方式，其在推理时能够将其归一化的参数直接融合进卷积中，从而节省归一化操作时间  
同时使用深度可分离卷积替换普通卷积，并且将卷积堆叠的数量从4个减少2组；在通道数上，将256维压缩至96维（通道数保持为8或16的倍数，可以享受大部分推理框架的并行加速）；最后借鉴yolo系列，将边框回归和分类使用同一组卷积进行计算，然后split成两份  
![NanoDet检测头]()
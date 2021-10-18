# Mobilenet

## v1

### 1. 深度可分离卷积(Deep-wise Spearable)

MobileNet模型基于深度可分离卷积，深度可分离卷积将普通标准化的卷积分解为深度卷积(Depthwise Convolution)与逐点卷积(Pointwise Convolution)  
普通的卷积不仅完成对输入的滤波，又包括对输出的合并    
与普通卷积相比，深度可分离卷积区别在于将原本一层的操作分为两层:  
在深度操作中，深度卷积一次只作用于单个通道,随后逐点卷积使用1*1卷积来将深度卷积的输出合并

    使用深度可分离卷积的CNN网络与普通CNN网络相比，其计算量与模型更小，更加适合移动端的视觉应用

![深度可分离卷积](https://raw.githubusercontent.com/Tianering/Markdown/master/images/DeepwiseSpearable.png)

### 2. 网络结构和训练

![MobileNetV1网络结构](https://raw.githubusercontent.com/Tianering/Markdown/master/images/net.png)  
如前所述，MobileNet由深度可分离卷积建立，与普通标准卷积相比，MobileNet将深度卷积与逐点卷积计算为单独的层  
其网络结构如上图所示，并且在所有的卷积层后都有BatchNormalization和ReLU非线性激活函数进行处理。  
MobileNet这样的模型结构将几乎所有的计算都放在了密集的1×1卷积中，使得MobileNet在1×1卷积上花费了绝大部分计算时间，同时也包含了大部分参数

### 3. 两个超参数

为了满足不同应用程序对模型及速度的要求，MobileNet引入了两个可以控制模型大小的超参数：宽度乘数和分辨率乘数

+ Width Multiplier  
  宽度乘数通过对输入通道与输出通道的控制(输入输出通道数都乘以α得到新的通道数)，使得每层网络减负，即减少计算复杂度和参数数量
+ Resolution Multiplier 分辨率乘数应用于输入图像，按照输入乘数对输入数据的分辨率进行成比例的变换，随着分辨率的下降，准确率也下降，同时速度就会加快

## v2

### 1. 线性瓶颈 Linear Bottlenecks

为了使兴趣流形贯穿整个空间，在v1中，我们通过宽度乘数减少层维度降低激活空间的维度；然而，经过ReLU可能会使激活空间产生坍塌，不可避免的损失信息  
针对此问题使用linear bottleneck替代原本的非线性激活变换，即在网络结构中，插入linear bottleneck，防止非线性破环太多信息  
在通道数较小的层后，用线性激活代替ReLU；即在v2的残差块中，在降维后的1×1卷积层后接线性激活，其他情况为ReLU  
![线性瓶颈图像](https://raw.githubusercontent.com/Tianering/Markdown/master/images/LinearBottlenecks.png)

### 2. 倒残差结构 Inverted residuals

随着深度学习网络深度的增加，网络性能的退化问题需要利用残差结构进行解决，其解决了梯度问题，同时使网络性能得到提升  
MobileNetv2使用的倒残差结构与之相似  
![MobileNet使用的倒残差结构（图像](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Inverted%20residuals.jpg)  
先升维的目的是在数据进入深度卷积之前拓展数据中的通道数量，丰富特征数量；在深度卷积之后压缩通道数，选择有用的特征，减少参数数量  
![线性瓶颈倒残差块结构](https://raw.githubusercontent.com/Tianering/Markdown/master/images/LinearBottleneckInvertedResidualBlock.png)   
在V2的网络设计中，我们除了继续使用深度可分离结构之外，还使用了Expansion layer和 Projection layer。 这个projection layer也是使用
1×1的网络结构，他的目的是希望把高维特征映射到低维空间去。  
Expansion layer的功能正相反，使用1×1的网络结构，目的是将低维空间映射到高维空间。这里Expansion有一个超参数是维度扩展几倍。可以根据实际情况来做调整的，默认值是6，也就是扩展6倍。

### 3. 网络结构

![网络结构](https://raw.githubusercontent.com/Tianering/Markdown/master/images/MobileNet_v2_net.jpg)

## v3

### 1. 神经网络架构搜索（NAS）

+ Platform-Aware NAS for Block-wise Search  
  使用NAS的目的是发现和优化网络结构，MobileNet采用了平台层次的神经体系结构方法查找全局结构： Platform-Aware NAS  
  将设计问题表述为一个多目标搜索，目标是找到准确率高的、推理延迟低的CNN模型  
  引入全新的分解层次搜索空间，将CNN模型分解为独特的块，然后分别搜索每个块的操作和连接，从而允许在不同块中使用不同的层结构
+ NetAdapt for Layer-wise Search  
  NetAdapt在自适应算法中引入了直接度量、这些直接指标通过经验测量来评估;并自动和渐进的简化预先训练的网络，知道满足资源要求且最大化精度  
  ![NetAdapt](https://raw.githubusercontent.com/Tianering/Markdown/master/images/NetAdapt.png)
+ Choose Number of Filters:  
  这一步的重点是确定有多少过滤器保存在一个特定的层基于经验的测量  
  NetAdapt逐渐减少目标层中的过滤器数量，并测量每个简单网络的资源消耗,将选择能够满足当前资源约束的最大过滤器数量
+ Choose Which Filters:  
  此步骤根据前一步的体系结构选择要保留哪些过滤器
+ Short-/Long-Term Fine-Tune:  
  NetAdapt中的短期调优和长期调优步骤都涉及到端到端的网络明智调优  
  短期调优比长期调优迭代更少。在算法的每次迭代中，我们都用相对较少的迭代次数(即(短期的)恢复精度，以并行或顺序恢复。这一步在适应资源减少较多的小网络时尤为重要，否则精度会下降到零，从而导致算法选择错误的网络方案

### 2. 轻量级注意力模型（SE）

![Squeeze-and-Excitation ](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Squeeze-and-Excitation%20.jpg)  
SE模块主要包括Squeeze和Excitation两个操作:  
首先对卷积得到的特征图进行Squeeze操作，得到channel级的全局特征， 然后对全局特征进行Excitation操作，学习各个channel间的关系，也得到不同channel的权重，最后乘以原来的特征图得到最终特征。
本质上，SE模块是在channel维度上做attention或者gating操作，这种注意力机制让模型可以更加关注信息量最大的channel特征，
而抑制那些不重要的channel特征。另外一点是SE模块是通用的，这意味着其可以嵌入到现有的网络架构中

### 3. 网络结构

![MobileNet v3 large](https://raw.githubusercontent.com/Tianering/Markdown/master/images/MobileNet_v3_Large.jpg)  
MobileNet_v3_Large  
![MobileNet v3 Small](https://raw.githubusercontent.com/Tianering/Markdown/master/images/MobileNet_v3_Small.jpg)  
MobileNet v3 Small

# ShuffleNet

## V1

### 1.通道洗牌（Channel Shuffle）

常用的深度可分离卷积的性能瓶颈主要在Pointwise卷积上，为了解决这个问题ShuffleNetv1提出了仅在分组内进行Pointwise卷积的方法，
组内Pointwise卷积的方式可以非常有效的环节性能瓶颈问题,但是这个策略面临着非常严重的问题：卷积直接的信息沟通不畅网络趋近于一个由多个类似结构构成的模型集成  
为了解决通道之间的沟通问题,ShuffleNetv1提出了其最为核心的操作：通道洗牌Channel Shuffle  
![Channel Shuffle](https://raw.githubusercontent.com/Tianering/Markdown/master/images/ChannelShuffle.jpg)  
Channel Shuffle是一种介于整个通道的Pointwise卷积和组内Pointwise卷积的一种折中方案  
在通道洗牌过程中，首先将Feature Map展开成g×n×w×h的四维矩阵，然后沿着尺寸为g×n×w×h的矩阵的g轴和h轴进行转置，  
最后将g轴和n轴进行平铺后得到新的（洗牌后的）Feature Map并进行组内1×1卷积  

![Shuffle unit](https://raw.githubusercontent.com/Tianering/Markdown/master/images/ShuffleNetunit.jpg)  
 在ShuffleNetv1单元中，使用分组卷积替换深度可分离卷积中的1×1卷积，同时在第一个1×1卷积之后添加Channel Shuffle操作  
如果进行了降采样，为了保证参数数量不骤减，往往需要加倍通道数量  

### 2. 网络结构  
![Shuffle Architecture](https://raw.githubusercontent.com/Tianering/Markdown/master/images/ShuffleNetArchitecture.jpg)

## v2
输入输出具有相同的channel时，内存消耗最小
过多的分组卷积操作会增大MAC，从而使模型速度变慢
模型中的分支数量越少，模型速度越快
Element-wise操作不能被忽略
### 1.通道分割（Channel Split） 
在每个单元（Shuffle unit）的开始，将输入的特征通道分为两支，分别带有 c−c' 和c'个通道。按照准则G3，一个分支的结构仍然保持不变。另一个分支由三个卷积组成， 为满足G1，令输入和输出通道相同  
与 ShuffleNet V1 不同的是，两个 1×1 卷积不再是组卷积(GConv)，因为Channel Split分割操作已经产生了两个组  
卷积之后，把两个分支拼接(Concat)起来，从而通道数量保持不变 (G1)，而且也没有Add操作（element-wise操作)  
然后进行与ShuffleNetV1相同的Channel Shuﬄe操作来保证两个分支间能进行信息交流  
![Channel Split](https://raw.githubusercontent.com/Tianering/Markdown/master/images/ChannelSplit.jpg)  
### 2.网络结构
![ShuffleNetv2 Architecture](https://raw.githubusercontent.com/Tianering/Markdown/master/images/ShuffleNetv2Architecture.png)

## Transformer  
使用attention组成了encoder-decoder的框架，并将其用于机器翻译——Attention in All Tou Need(NIPS2017)  
![Transformer结构](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Transformer.jpg)  
每个句子先经过六个Encoder进行编码，然后经过六个Decoder进行解码，最后得到输出  


### Self-Attention Layer  
使用Self-Attention取代RNN  
![Self-Attention计算过程](https://raw.githubusercontent.com/Tianering/Markdown/master/images/self-attention.jpg)  

    1. 嵌入向量（Embedding）X乘以三个不同的权值矩阵（Wq、Wk、Wv）得到不同的三个向量Query向量、Key向量、Value向量
    2. 为每一个向量计算一个score：score = q·k
    3. 为了梯度的稳定，Transformer使用了score归一化，即除以根号dk
    4. 对score施以softmax激活函数
    5. softmax点乘Value值 v ,得到加权的每个输入向量的评分v
    6. 相加之后得到最终的输出结果

#### Multi-head Self-attention
Multi-head Self-attention相当于多个不同的sele-attention的继承
![Multi-head Self-attention](https://raw.githubusercontent.com/Tianering/Markdown/master/images/Multi-headSelf-attention.jpg)  

	1. 将数据分别输入不同的多个self-attention中，得到多个加权后的特征矩阵
	2. 将所得的特征矩阵按列拼成一个打的特征矩阵
	3. 特征矩阵经过一层全连接后得到输出  


### Positional Encoding  
为了使Transformer拥有捕捉顺序序列的能力，在编码词向量时引入了位置编码（Positional Encoding）的特征；即通过位置编码在词向量加入单词的位置信息，使得Transformer能够区分不同位置的单词  
### Layer Normalization  
Self Attention结果经过了一次Skip Connection，然后被送到了Layer Norm中。与Batch Norm不同的是，Layer Norm并不是沿着Batch方向求均值和方差，而是对Batch中的每一个元素求均值和方差   

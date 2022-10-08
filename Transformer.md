# Transformer理解

## 1. Transformer的总体架构

`Transformer`的整体结构如下：

<img src="https://gitee.com/QishengStudent/figs/raw/master/image-20221004110922779.png" alt="image-20221004110922779" style="zoom:80%;" />

$Output\ Embedding$这个输入是啥？

答：$Output\ Embedding$是$ground\ truth$的`embedding`结果。$ground\ truth$指的是有效的正确的数据。

包括下列四个部分：

- 输入部分
- 输出部分
- 编码器部分
- 解码器部分

### 1.1 输入部分

包括两个部分：

- 输入 / 输出Embedding
- 位置编码

### 1.2 输出部分

包括以下两个部分：

- 一个全连接层Linear
- 一个softmax分类

### 1.3 编码器部分构成

从外到内来看，编码器包括以下四个部分：

- 由$N×$个编码器堆叠而成
- 每个编码器由两个子层连接结构构成
- 第一个子层连接结构包括一个多头自注意力子层 + 规范化子层 + 一个残差连接层
- 第二个子层连接结构包括前馈全连接子层 + 规范化子层 + 一个残差连接层

<img src="https://gitee.com/QishengStudent/figs/raw/master/Encoder.png" style="zoom:67%;" />

### 1.4 解码器部分构成

从外到内来看，解码器包括以下五个部分（比编码器多一个组件）：

- 由$N ×$个解码器堆叠而成
- 每个解码器由三个子层连接结构构成
- 第一个子层连接结构包括一个**多头自注意力子层（有掩码mask）**+ 规范化子层 + 一个残差连接
- 第二个子层连接结构包括一个多头子注意力子层 + 规范化子层 + 一个残差连接
- 第三个子层连接结构包括前馈全连接子层 + 规范化子层 + 一个残差连接层

## 2 输入部分实现

包括以下两方面：

- 原文本的嵌入（Embedding） + 位置嵌入
- 目标文本的嵌入（Embedding） + 位置嵌入

**解释Embedding：**把一个符号形式的数据转换成数值形式，即嵌入到一个数学空间里。以NLP为例，要把词语进行word Embedding后才输入模型中使用。

### 2.1 文本Embedding介绍

此处介绍两种方式：随机初始化、word2vec。

#### 2.1.1 采用随机初始化

```
- 注意： 
vocab：词表的大小
d_model : 词嵌入的维度
```

- code 1

```python
embedding = nn.Embedding(10, 3)  
# 词表的大小10， 词嵌入的维度3
input1 = torch.LongTensor([[1, 2, 4, 5], [4, 3, 2, 9]])
print(input1)
print("***********")
print(embedding(input1))

##########对应的输出############

tensor([[1, 2, 4, 5],
        [4, 3, 2, 9]])
***********
tensor([[[-1.1630,  2.8922, -0.5683],
         [-0.8624,  1.6620,  0.2009],
         [-0.5450, -1.9818, -0.1903],
         [-0.0881, -0.7342,  0.5417]],

        [[-0.5450, -1.9818, -0.1903],
         [ 0.0551, -0.5700, -0.3047],
         [-0.8624,  1.6620,  0.2009],
         [-0.7385, -1.4200,  0.1704]]], grad_fn=<EmbeddingBackward0>)
```

#### 2.1.2 采用word2vec

word2vec的目标是：将一个词表示成一个向量。

word2vec中的两个重要模型：CBOW和Skip-gram模型。

- 如果是拿一个词语的上下文作为输入，来预测这个词语本身，则是**『CBOW 模型』**
- 如果是用一个词语作为输入，来预测它周围的上下文，那这个模型叫做**『Skip-gram 模型』**

#### 2.1.3 随机初始化Embedding的实战代码

	代码介绍：
	1.  输入x： [[100, 2, 421, 508], [491, 998, 1, 221]]
	代表的是两组数据(batch_size=2)
	每组数据都是一个4d的向量表示
	
	2.  期望输出 ：对于两个4d的向量， 通过词嵌入，映射成512d的向量
	最后的shape组成 [batch_size, 4, 512]，也就是[2, 4, 512]
	
	3. Embedding层的介绍
	只有一层nn.Embedding()
	返回的时候。注意 × math.sqrt(self.d_model)

完整代码如下：

```python
import math
from torch.autograd import Variable
from torch import nn
import torch


# 构建 Embedding 类来实现文本嵌入层
class Embedding(nn.Module):
    def __init__(self, vocab, d_model):
        # vocab： 词表的大小
        # d_model : 词嵌入的维度
        super(Embedding, self).__init__()
        # 定义 Embedding 层
        self.lut = nn.Embedding(vocab, d_model)
        # 将参数传入类中
        self.d_model = d_model

    def forward(self, x):
        # x: 代表的是输入进模型的文本，通过词汇映射后的数字张量
        return self.lut(x) * math.sqrt(self.d_model) 


# 词表： 1000*512， 共是1000个词，每一行是一个词，每个词是一个512d的向量表示
vocab = 1000
d_model = 512

x = Variable(torch.LongTensor([[100, 2, 421, 508], [491, 998, 1, 221]]))

emb = Embedding(vocab, d_model)
embr = emb(x)
print("ember:", embr, "\n*********\n", embr.shape)
```

输出的shape是`[2, 4, 512]`，详细输出如下：

```
ember: tensor([[[ -1.5284,   8.9630, -28.1908,  ...,   8.9494, -30.8598,  25.3919],
         [ -6.2146, -12.5481,   6.0884,  ...,  -7.7220,  61.5759,   2.7136],
         [ 16.2398,  -4.2148,   7.3554,  ...,  25.9346, -23.0663,  -3.7351],
         [  2.1319,  50.7419, -17.3968,  ...,   4.0045, -38.6387,  -0.0956]],

        [[-16.8649,   9.5925,   6.0954,  ..., -43.2856, -13.5794,  -2.5090],
         [  2.2715,  10.9802,  17.8983,  ..., -23.2544, -53.0442,   3.8845],
         [ 19.2986, -20.5505,  31.6046,  ...,  21.5987,  -2.3912,  -3.8113],
         [  6.6228, -26.5737,  19.1182,  ...,  18.1940,   9.9820,  35.8124]]],
       grad_fn=<MulBackward0>) 
*********
 torch.Size([2, 4, 512])
```

### 2.2 位置编码器实现

问：为什么要添加位置编码器？

在RNN是串行输入的，而在Transformer中是并行输入的，因此需要在Embedding层之后加入位置编码器，将词汇位置不同可能会产生不同的语义的信息加入到词嵌入的张量之中，以弥补位置信息的缺失。

将`Embedding`层的输出$x$（`[2, 4, 512]`）作为位置编码层的输入。

- 实例化参数

```python
# 词嵌入的维度是512维
d_model = 512

# 置0比率为0.1
dropout = 0.1

# 句子最大长度
max_len = 60
```

- 代码

```python
import math
from torch.autograd import Variable
from torch import nn
import torch
from embedding_layer import Embedding  # 导入文本编码的输出

# 构建位置编码器的类
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, dropout, max_len=5000):
        # d_model : 代表词嵌入的维度
        # dropout : 代表Dropout层的置零比率
        # max_len : 代表每个句子的最大长度
        super(PositionalEncoding, self).__init__()

        # 实例化 Dropout层
        self.dropout = nn.Dropout(p=dropout)

        # 初始化一个位置编码矩阵，大小是 max_len * d_model
        pe = torch.zeros(max_len, d_model)

        # 初始化一个绝对位置矩阵, max_len * 1
        position = torch.arange(0, max_len).unsqueeze(1)
        print(position)

        # 定义一个变化矩阵，div_term, 跳跃式的初始化
        div_term = torch.exp(torch.arange(0, d_model, 2) * -(math.log(10000.0) / d_model))
        print("\ndiv_term", div_term)

        # 将前面定义的变化矩阵 进行奇数，偶数分别赋值
        pe[:, 0::2] = torch.sin(position * div_term)  # 用正弦波给偶数部分赋值
        pe[:, 1::2] = torch.cos(position * div_term)  # 用余弦波给奇数部分赋值

        # 将二维张量，扩充为三维张量
        pe = pe.unsqueeze(0)  # 1 * max_len * d_model

        # 将位置编码矩阵，注册成模型的buffer，这个buffer不是模型中的参数，不跟随优化器同步更新
        # 注册成buffer后，就可以在模型保存后 重新加载的时候，将这个位置编码器和模型参数
        self.register_buffer('pe', pe)

    def forward(self, x):
        # x : 代表文本序列的词嵌入表示
        # 首先明确pe的编码太长了，将第二个维度，就是max_len对应的维度，缩小成x的句子的同等的长度
        x = x + Variable(self.pe[:, : x.size(1)], requires_grad=False)  # 表示位置编码是不参与更新的
        return self.dropout(x)

d_model = 512
dropout = 0.1
max_len = 60
vocab = 1000

x = Variable(torch.LongTensor([[100, 2, 421, 508], [491, 998, 1, 221]]))

emb = Embedding(vocab, d_model)
embr = emb(x)
x = embr  # shape: [2, 4, 512]
pe = PositionalEncoding(d_model, dropout, max_len)
pe_result = pe(x)
print(pe_result)
print("\n*********\n", pe_result.shape)
```

### 2.3 输出位置矩阵

由于这里的x传入的是一个zeros，全零矩阵，那么在`PositionalEncoding` 模型 的`forward` 中，就只保留了位置向量（**相当于展示pe**），而且在调用时，`dropout=0`。

- 代码如下：

```python
import matplotlib.pyplot as plt
from position import PositionalEncoding
import math
from torch.autograd import Variable
from torch import nn
import torch
import numpy as np

plt.figure(figsize=(15, 5))

pe = PositionalEncoding(20, 0)  # 实例化这个模型，d_model = 20, dropout = 0
y = pe(Variable(torch.zeros(1, 100, 20)))  # 相当于只看位置矩阵

plt.plot(np.arange(100), y[0, :, 4:8].data.numpy())

# 在画布上填写维度提示信息
plt.legend(["dim %d" %p for p in [4, 5, 6, 7]])
plt.show()
```

- 输出如下：

<img src="https://gitee.com/QishengStudent/figs/raw/master/image-20221004161145250.png" alt="image-20221004161145250" style="zoom:50%;" />

- 分析：
  - 每条颜色的曲线代表某一个词汇中的特征在不同位置的含义
  - 保证同一词汇随着所在位置不同，它对应位置嵌入向量会发生变化
  - `sin`和`cos`函数使得输入都在 [-1, 1] 之间，方便模型计算

### 2.4 总结

文本嵌入层的作用如下：

- 无论是源文本还是目标文本嵌入，都是为了将文本转换为向量表示，**希望在高维空间捕捉词汇间的关系**。
- 位置编码器：因为 `transformer` 是并行输入的，忽略了位置信息，所以要在Embedding层后加入位置编码器。将词汇位置不同可能会产生的不同语义的信息加入到词嵌入张量中，以弥补位置信息的缺失。

## 3 多头注意力解读

### 3.1 公式





### 3.2 例子





## 4 Layer Normalization



## 5 Decoder介绍





References：

https://blog.csdn.net/weixin_42521185/article/details/124648718

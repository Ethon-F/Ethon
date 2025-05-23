# Transformer面经题目

### 1、Transformer为何使用多头注意力机制？为什么不用一个头？

```
因为单头注意力机制只能学习一种信息关联模型。而多头注意力将输入序列投影到多个子空间【即“头”上】，每个头可以独立学习不同的关注模式。
```

### 2、Transformer为什么Q和K使用不同的权重矩阵生成，为何不能使用同一个值进行自身的点乘？

```
用不同的权重矩阵生成是为了保证在不同的空间进行投影，增强表达能力，提高泛化能力；
点乘是代表两个向量的相似度，如果在同一个向量空间进行点乘，理所应当的是自身和自身的相似度最大，会影响其他向量对自己的作用。
```

### 3、Transformer计算attention的时候为何选择点乘而不是加法？两者计算复杂度和效果上有什么区别？

```
选择点乘是为了计算速度更快，点乘操作可以通过矩阵乘法的方式进行优化，并且可以利用硬件的并行计算能力。虽然矩阵加法的计算量简单，但是在硬件优化上不如点乘高效。

从计算复杂度来看，点乘注意力（Scaled Dot-Product Attention）的计算注意力权重复杂度为O(d)；加法注意力（Additive Attention）复杂度也是O(d)，但由于需要逐项计算相加并引入了激活函数等非线性运算，所以实际计算开销比较大。

从数学解释上来看，点乘是一种衡量两个向量相似度的自然方式。当Q和K的方向相似时，点积会较大，softmax后权重也会较大，这样意味着这种相似性直接影响了注意力权重的大小。
```

![image-20250401213508595](C:\Users\1\AppData\Roaming\Typora\typora-user-images\image-20250401213508595.png)

### 4、为什么在进行softmax之前需要对attention进行scaled？【即为什么除以dk的平方根】，使用公式推导进行详解

```
https://blog.csdn.net/qq_37430422/article/details/105042303
```



```
因为dk维度过大，会导致在计算attention的时候，向量内积部分过大，会导致softmax函数的梯度过小；于是就需要对内积部分做缩放，除以dk 的平方根
```

### 5、在计算attention score的时候如何对padding做mask操作？

```
训练过程中，自然语言数据往往都是batch的形式输入，而一个batch中的每一句话不能保证长度都一样，所以需要用padding操作将所有句子都补全到最长的长度，比如拿0填充，但是0填充的位置信息是毫无意义的，因此给它Mask掉；
Padding Mask在attention的计算过程中处于softmax之前，通过Padding Mask操作，使得补全位置上的值成为一个非常大的负数，这样经过Softmax层的时候，这些位置上的概率=0，这就将补全位置的无用信息给mask掉了。
```

### 6、为什么在进行多头注意力的时候需要对每个head进行降维？

```
为了确保每个头的计算复杂度不会太高，同时能够并行计算。将输入的维度分成多个头，可以让每个头处理更小维度的数据，从而降低单头的计算复杂度，减少参数量，提高效率。并且多头的组合，可以增强模型的表达能力和鲁棒性。
```

### 7、大概讲一下Tranformer的Encoder模块

```
由多个相同的层堆叠而成，每一层包括两个子层，第一个是多头自注意力机制，第二个是前馈神经网络；其中多头自注意力机制是为了获取序列中不同词向量之间的关系，而前馈神经网络是为了进一步处理特征
```

### 8、为何在获取输入词向量后，需要对矩阵乘以embedding size 的开方？意义是什么？

```
避免由于维度过高导致的softmax激活函数梯度消失；有一个先点积后除与先除后点积的区别。
```

### 9、简单介绍一下Transformer的位置编码？有什么意义和优缺点？

```
位置编码，也叫做Positional Encoding，由于Transformer本身不具备顺序信息，因此引入位置编码技术给模型提供序列中各个位置的信息。位置编码的具体实现是通过正弦和余弦函数来完成的，对于不同位置生成不同的编码，易于计算。
```

### 10、你还了解哪些关于位置编码的技术？各自的优缺点是什么？

```
可学习的位置编码：位置编码作为可学习的参数，优点是灵活，缺点是实现复杂度高
相对位置编码：考虑到相对位置的关系，优点是能够捕捉相对位置信息，适用于长序列，缺点是实现复杂度高
混合位置编码：
```

### 11、Transformer的完整流程【从输入到输出】

```
Transformer是一种基于自注意力机制的神经网络模型，其完整流程包括输入嵌入、编码器处理、解码器处理和最终输出。具体来说输入序列首先经过嵌入层embedding转化为向量表示，并添加位置编码Positional Encoding 以保留顺序信息。然后，这些向量通过多层编码器【大概是6层】进行特征提取，编码器的核心是自注意力机制和前馈神经网络。解码器接收编码器的输出并生成目标序列，同样包含自注意力机制和前馈网络，但也引入了编码器-解码器注意力机制。最后解码器输出经过线性变换和softmax函数得到概率分布，用于预测每个位置的词。
```

```
embedding嵌入层：将输入序列中的每个词转化为固定维度的向量表示。
位置编码：由于Transformer不包含循环或卷积结构，无法自然地捕捉序列的顺序信息，因此需要显式地为每个词向量添加位置编码
编码器：由多个相同的层堆叠而成，每一层包括两个子层，第一个是多头自注意力机制，第二个是前馈神经网络
解码器：也由多个相同的层组成，每一层包含三个子层，第一个是掩码多头自注意力机制，第二个是编码器-解码器注意力机制，第三个是前馈神经网络。
输出层：解码器的最后一层输出经过线性变换和softmax函数，生成每个位置上的词概率分布，从而完成序列生成任务。
```

### 12、为什么transformer块使用LayerNorm而不是BatchNorm？LayerNorm 在Transformer的位置是哪里？

```
Transformer中的使用的是LayerNorm诗音为LN在序列模型中表现更好。【个人理解】而且Transformer处理的一般是长文本翻译的功能，不同的句子之间的长度不一样，所以用BN进行处理变长序列会不稳定，因此选择用LN。Tranformer的LayerNorm通常位于每个子层的残差连接之后。
```

##2025/04/19

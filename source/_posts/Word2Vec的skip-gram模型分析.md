---
title: Word2Vec的skip-gram模型分析
date: 2020-07-29 19:34:55
tags: [神经网络, 机器学习]
---

## 什么是Word2Vec模型？

`Word2Vec`是从大量文本语料中以无监督的方式学习语义知识的一种模型，它被大量地用在自然语言处理（NLP）中。那么它是如何帮助我们做自然语言处理呢？`Word2Vec`其实就是通过学习文本来用词向量的方式表征词的语义信息，即通过一个嵌入空间使得语义上相似的单词在该空间内距离很近。

<!-- more -->

## 初步理解Word2Vec

###Skip-Gram和CBOW模型

Word2Vec模型中，主要有Skip-Gram和CBOW两种模型，从直观上理解，Skip-Gram是给定input word来预测上下文。而CBOW是给定上下文，来预测input word。

![v2-35339b4e3efc29326bad70728e2f469c_720w](https://pic2.zhimg.com/80/v2-35339b4e3efc29326bad70728e2f469c_720w.png)

这个模型被称为**Skip-gram模型**（名称源于该模型在训练时会对上下文环境里的word进行采样）。

如果将Skip-gram模型的前向计算过程写成数学形式，我们得到：

𝑝(𝑤𝑜|𝑤𝑖)=𝑒𝑈𝑜⋅𝑉𝑖∑<sub>j</sub>𝑒𝑈𝑗⋅𝑉𝑖

其中，𝑉𝑖是Embedding层矩阵里的列向量，也被称为𝑤𝑖的input vector。𝑈𝑗是softmax层矩阵里的行向量，也被称为𝑤𝑖的output vector。

因此，Skip-gram模型的本质是**计算输入word的input vector与目标word的output vector之间的余弦相似度，并进行softmax归一化**。我们要学习的模型参数正是这两类词向量。

> 下文中所有的Word2Vec都是指Skip-Gram模型

Word2Vec模型实际上分为了两个部分，**第一部分为建立模型，第二部分是通过模型获取嵌入词向量。**Word2Vec的整个建模过程实际上与`自编码器（auto-encoder）`的思想很相似，即先基于训练数据构建一个神经网络，当这个模型训练好以后，我们并不会用这个训练好的模型处理新的任务，我们真正需要的是这个模型通过训练数据所学得的参数，例如隐层的权重矩阵——后面我们将会看到这些权重在Word2Vec中实际上就是我们试图去学习的“word vectors”。基于训练数据建模的过程，我们给它一个名字叫“Fake Task”，意味着建模并不是我们最终的目的。

### The Fake Task

我们在上面提到，训练模型的真正目的是获得模型基于训练数据学得的隐层权重。为了得到这些权重，我们首先要构建一个完整的神经网络作为我们的“Fake Task”，后面再返回来看通过“Fake Task”我们如何间接地得到这些词向量。

接下来来看看如何训练我们的神经网络。假如我们有一个句子**“The dog barked at the mailman”。**

- 首先我们选句子中间的一个词作为我们的输入词，例如我们选取“dog”作为input word；
- 有了input word以后，我们再定义一个叫做skip_window的参数，它代表着我们从当前input word的一侧（左边或右边）选取词的数量。如果我们设置![[公式]](https://www.zhihu.com/equation?tex=skip%5C_window%3D2)，那么我们最终获得窗口中的词（包括input word在内）就是**['The', 'dog'，'barked', 'at']**。![[公式]](https://www.zhihu.com/equation?tex=skip%5C_window%3D2)代表着选取左input word左侧2个词和右侧2个词进入我们的窗口，所以整个窗口大小![[公式]](https://www.zhihu.com/equation?tex=span%3D2%5Ctimes+2%3D4)。另一个参数叫num_skips，它代表着我们从整个窗口中选取多少个不同的词作为我们的output word，当![[公式]](https://www.zhihu.com/equation?tex=skip%5C_window%3D2)，![[公式]](https://www.zhihu.com/equation?tex=num%5C_skips%3D2)时，我们将会得到两组 **(input word, output word)** 形式的训练数据，即 **('dog', 'barked')，('dog', 'the')**。
- 神经网络基于这些训练数据将会输出一个概率分布，这个概率代表着我们的词典中的每个词是output word的可能性。这句话有点绕，我们来看个栗子。第二步中我们在设置skip_window和num_skips=2的情况下获得了两组训练数据。假如我们先拿一组数据 **('dog', 'barked')** 来训练神经网络，那么模型通过学习这个训练样本，会告诉我们词汇表中每个单词是“barked”的概率大小。

![v2-ca21f9b1923e201c4349030a86f6dc1f_720w](https://pic3.zhimg.com/80/v2-ca21f9b1923e201c4349030a86f6dc1f_720w.png)

## 深入了解word2vec模型

### Word2Vec是一个神经网络

word2vec是一个简单的神经网络，只由三个层组成：

- １个输入层
- 1个隐藏层
- 1个输出层

输入层输入的就是·数据对·的数字表示，输出到隐藏层。 隐藏层的神经网络单元的数量，其实就是我们所说的**embedding size**。需要注意的是，我们的隐藏层后面不需要使用激活函数。 输出层，我们使用softmax操作，得到每一个预测结果的概率。

![image-20200727193735893.png](https://ae03.alicdn.com/kf/Hba6d57e88c334277b4c43ded69af0dd51.png)

### 输入层

现在问题来了，我们该如何用数字表示文本数据呢？

好像随便一种方式都可以用来表示我们的文本啊。

看上图，我们发现，它的输入使用的是**one-hot**编码。什么是ont-hot编码呢？如图所示，假设有n个词，则每一个词可以用一个n维的向量来表示，这个n维向量只有一个位置是1，其余位置都是0。

那么为什么要用这样的编码来表示呢？答案后面揭晓。

### 隐藏层

隐藏层的神经单元数量，代表着每一个词用向量表示的维度大小。假设我们的**hidden_size**取300，也就是我们的隐藏层有300个神经元，那么对于每一个词，我们的向量表示就是一个1 * N的向量。 有多少个词，就有多少个这样的向量！

所以对于**输入层**和**隐藏层**之间的权值矩阵W的矩阵，

如果果我们现在想用300个特征来表示一个单词（即每个词可以被表示为300维的向量）。那么隐层的权重矩阵应该为10000行，300列（隐层有300个结点）。

Google在最新发布的基于Google news数据集训练的模型中使用的就是300个特征的词向量。词向量的维度是一个可以调节的超参数（在Python的gensim包中封装的Word2Vec接口默认的词向量大小为100， window_size为5）。

看下面的图片，左右两张图分别从不同角度代表了输入层-隐层的权重矩阵。左图中每一列代表一个10000维的词向量和隐层单个神经元连接的权重向量。从右边的图来看，每一行实际上代表了每个单词的词向量。

![img](https://pic3.zhimg.com/80/v2-c538566f7d627ce7ca40589f15ca8284_720w.png)


所以我们最终的目标就是学习这个隐层的权重矩阵。

我们现在回来接着通过模型的定义来训练我们的这个模型。

上面我们提到，input word和output word都会被我们进行one-hot编码。仔细想一下，我们的输入被one-hot编码以后大多数维度上都是0（实际上仅有一个位置为1），所以这个向量相当稀疏，那么会造成什么结果呢。如果我们将一个1 x 10000的向量和10000 x 300的矩阵相乘，它会消耗相当大的计算资源，为了高效计算，它仅仅会选择矩阵中对应的向量中维度值为1的索引行（这句话很绕），看图就明白。



![img](https://pic4.zhimg.com/80/v2-9de68e5c46e9ea1ea480e295b0cc0b87_720w.png)



我们来看一下上图中的矩阵运算，左边分别是1 x 5和5 x 3的矩阵，结果应该是1 x 3的矩阵，按照矩阵乘法的规则，结果的第一行第一列元素为![[公式]](https://www.zhihu.com/equation?tex=0%5Ctimes+17+%2B+0%5Ctimes+23+%2B+0%5Ctimes+4+%2B+1%5Ctimes+10+%2B+0%5Ctimes+11+%3D+10)，同理可得其余两个元素为12，19。如果10000个维度的矩阵采用这样的计算方式是十分低效的。



为了有效地进行计算，这种稀疏状态下不会进行矩阵乘法计算，可以看到矩阵的计算的结果实际上是矩阵对应的向量中值为1的索引，上面的例子中，左边向量中取值为1的对应维度为3（下标从0开始），那么计算结果就是矩阵的第3行（下标从0开始）—— [10, 12, 19]，这样模型中的隐层权重矩阵便成了一个”查找表“（lookup table），进行矩阵计算时，直接去查输入向量中取值为1的维度下对应的那些权重值。隐层的输出就是每个输入单词的“嵌入词向量”。

### 输出层

那么我们的输出层，应该是什么样子的呢？从上面的图上可以看出来，输出层是一个`[vocab_size]`大小的向量，每一个值代表着输出一个词的概率。

为什么要这样输出？因为我们想要知道，对于一个输入词，它接下来的词最有可能的若干个词是哪些，换句话说，我们需要知道它接下来的词的概率分布。

你可以再看一看上面那张网络结构图。

你会看到一个很常见的函数**softmax**，为什么是softmax而不是其他函数呢？不妨先看一下softmax函数长啥样：

<img src="https://ae04.alicdn.com/kf/H195a5365b8ec49509ca48cf5a6dd29bbG.png" alt="image.png" style="zoom:50%;" />

很显然，它的取值范围在(0,1)，并别所有的值和为1。这不就是天然的概率表示吗？

当然，softmax还有一个性质，因为它函数指数操作，如果**损失函数使用对数函数**，那么可以抵消掉指数计算。

关于更多的softmax，请看斯坦福[Softmax Regression](http://ufldl.stanford.edu/tutorial/supervised/SoftmaxRegression/)



## 整个过程的数学表示

至此，我们已经知道了整个神经网络的结构，那么我们应该怎么用数学表示出来呢？

回顾一下我们的结构图，很显然，三个层之间会有两个权值矩阵$W$，同时，两个偏置项$b$。所以我们的整个网络的构建，可以用下面的伪代码：

```python
import tensorflow as tf

# 假设vocab_size = 1000
VOCAB_SIZE = 1000
# 假设embedding_size = 300
EMBEDDINGS_SIZE = 300

# 输入单词x是一个[1,vocab_size]大小的矩阵。当然实际上我们一般会用一批单词作为输入，那么就是[N, vocab_size]的矩阵了
x = tf.placeholder(tf.float32, shape=(1,VOCAB_SIZE))
# W1是一个[vocab_size, embedding_size]大小的矩阵
W1 = tf.Variable(tf.random_normal([VOCAB_SIZE, EMBEDDING_SIZE]))
# b1是一个[1，embedding_size]大小的矩阵
b1 = tf.Variable(tf.random_normal([EMBEDDING_SIZE]))
# 简单的矩阵乘法和加法
hidden = tf.add(tf.mutmul(x,W1),b1)

W2 = tf.Variable(tf.random_normal([EMBEDDING_SIZE,VOCAB_SIZE]))
b2 = tf.Variable(tf.random_normal([VOCAB_SIZE]))
# 输出是一个vocab_size大小的矩阵，每个值都是一个词的概率值
prediction = tf.nn.softmax(tf.add(tf.mutmul(hidden,w2),b2))
```

### 损失函数

网络定义好了，我们需要选一个损失函数来使用梯度下降算法优化模型。

我们的输出层，实际上就是一个softmax分类器。所以按照常规套路，损失函数就选择**交叉熵损失函数**。

交叉熵如下：

<img src="https://ae01.alicdn.com/kf/H6a3f15ba62a0401d91b805db0b3a90c4s.png" alt="image.png" style="zoom:50%;" />

p,q是真实概率分布和估计概率分布

```python
# 损失函数&emsp;
cross_entropy_loss = tf.reduce_mean(-tf.reduce_sum(y_label * tf.log(prediction), reduction_indices=[1]))
# 训练操作
train_op = tf.train.GradientDescentOptimizer(0.1).minimize(cross_entropy_loss)
```


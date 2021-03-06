# LDA 主题模型

## 什么是LDA主题模型

首先我们简单 了解一下LDA以及LDA可以用来做什么。
LDA(Latent Dirichlet Allocation)是一种文档生成模型。它认为一篇文章是有多个主题的，而每个主题又对应着多个不同的词。一篇文章的构造过程，首先是以一定的概率选择某个主题，然后再在 这个主题下以一定的概率选出某一个词，这样就生成这篇文章的第一个词。不断重复这个过程，就生成整篇文章。当然这里假设词与词之间是没有顺序的。

LDA的使用是上述文档生成的逆过程，它将根据一篇的到的文章，去寻找出这篇文章的主题，以及这些主题对应的词。

现在来看怎么用LDA，LDA会给我们返回什么结果。

LDA是非监督的机器学习模型，并且使用词袋模型。一篇文章将会用词袋模型构造成词向量。LDA需要我们 手动确定要划分 的主题的个数 ，超参数将会在后面讲述，一般超参数对结果无很大影响。
                                                                     
## 算法流程

我们以文档集合D中的文档d为例，文档d中包含单词序列<w<sub>1</sub>,w<sub>2</sub>,...w<sub>n</sub>> ,w<sub>i</sub>表示第i个单词 ，设d有 n个单词；

文档集合D中出现的全部词组成Vocabulary;

首先将文档d作为算法的输入，并输入主题数K，此时d对应到各个主题的概率为：
θ<sub>d</sub>=(p<sub>t1</sub>,p<sub>t2</sub>,...p<sub>tk</sub>)，p<sub>ti</sub>为d对应第i个主题的概率；
此时输入到算法中的只有文档d和主题数K，那么p<sub>t1</sub>,p<sub>t2</sub>,...p<sub>tk</sub>的数值从何而来？

我们首先人为设置文档d中对应主题t<sub>1</sub>,t<sub>2</sub>,...t<sub>k</sub>的词的个数，比如文档d中5个词对应主题t<sub>1</sub>，7个词对应主题t<sub>2</sub>，…，4个词对应主题t<sub>k</sub>，那么此时，我们就人为确定了一个参数向量(5,7,…4)，将这个向量记作α⃗ ，这个我们人为设置的参数向量称为**超参数**。 

那么如何将超参数α⃗ 转化为概率分布θ<sub>d</sub>=(p<sub>t1</sub>,p<sub>t2</sub>,...p<sub>tk</sub>)呢？

<hr />

这里我们引入狄利克雷分布函数：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/algorithm/lda/1.1.png)

它所表达的含义简单来说就是，已知α<sub>1</sub>,α<sub>2</sub>,α<sub>3</sub>的条件下，概率p<sub>1</sub>,p<sub>2</sub>,p<sub>3</sub>的概率分布，也就是概率的概率，分布的分布。再直观点说就是：比如在已知α<sub>1</sub>,α<sub>2</sub>,α<sub>3</sub>为(5,7,4)(5,7,4)的条件下，样本点p<sub>1</sub>,p<sub>2</sub>,p<sub>3</sub>为(0.4,0.5,0.1)(0.4,0.5,0.1)的概率是多少。

那么我们将上述的三维Dirichlet函数扩展为K维，即在已知α⃗的条件下，得到p⃗的分布(α⃗ ,p⃗分别为K维向量)。K维Dirichlet公式如下： 

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/algorithm/lda/1.2.png)

至此，我们通过输入超参数α⃗ 得到了文档d的关于K个主题的狄利克雷分布： 

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/algorithm/lda/1.3.png)

其含义显然，Dirichlet的输入参数为α⃗ ，得到的输出为可以理解为一个矩阵：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/algorithm/lda/1.4.png)

即文档d对应各个主题t<sub>k</sub>的概率分布的分布。

<hr />

同理，我们可以将任一主题t<sub>k</sub>产生各个词的概率表示出来。人为设置主题t<sub>k</sub>产生的各个词的数量，即设置超参数，用向量η⃗来表示。同上所述，将η⃗作为Dirichlet函数的输入参数，得到主题t<sub>k</sub>产生各个词的狄利克雷分布：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/algorithm/lda/1.5.png)

此时我们已经得到了文档d对应各个主题的概率分布的分布（即狄利克雷分布）θ<sub>d</sub>，以及文档t<sub>k</sub>产生各个词的概率分布的分布β<sub>k</sub>，那么接下来，我们要从文档d中取出第i个词，求这个词对应各个主题的分布； 

**换句大家熟悉的话来说就是**：已知第i个词w<sub>i</sub>在文档d中出现n次，且已知它对应各个主题的概率（这里每个词对应各个主题的概率就是文档d对应各个主题的概率，二者同分布），求该词被各个主题产生的次数； 

这就等同于我们熟知的一共有n个球，且已知红球、黄球、绿球的概率分别为p<sub>1</sub>,p<sub>2</sub>,p<sub>3</sub>，求这n个求当中红球、黄球、绿球的个数。

那么如何通过文档d对应各个主题的分布θ<sub>d</sub>得到文档中的每个词被各个主题产生的次数，进而重新得到文档d中对应各个主题的词的个数呢？

首先我们引入十分熟悉的多项式分布：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/algorithm/lda/1.6.png)

> 这个公式的意义总所周知：已知一共n个球，且知道每种颜色球的概率，就可以得到有m<sub>1</sub>个红球,m<sub>2</sub>个黄球,m<sub>3</sub>个绿球的概率。

那么同样将其扩展为K维，将θ<sub>d</sub>作为参数，就可以得到文档d中第i个词w<sub>i</sub>对应的各个主题的多项式分布z<sub>dn</sub>=multi(θ<sub>d</sub>) 

于是，非常值得庆幸，我们通过文档d对应各个主题的概率θ<sub>d</sub>，进而得知文档d中各个词对应各个主题的概率，且知道这个词在文档d中的出现次数，于是求得这个词被各个主题的产生次数，遍历文档d中的每一个词，就可以得到新的文档d中对应各个主题的词的个数。

> **白话举例：** 文档d对应主题t<sub>1</sub>,t<sub>2</sub>的概率分别为p<sub>t1</sub>,p<sub>t2</sub>，于是文档d中词w<sub>1</sub>对应的主题t<sub>1</sub>,t<sub>2</sub>的概率也分别为p<sub>t1</sub>,p<sub>t2</sub>，又得知词w<sub>1</sub>在文档d中出现了15次，于是得到词w<sub>1</sub>由主题t<sub>1</sub>,t<sub>2</sub>产生的次数分别为10次、5次（这个是假设的）; 
>
> 对文档d中的每个词都执行此操作，（假设文档中只有两个词）词w<sub>2</sub>由主题t<sub>1</sub>,t<sub>2</sub>产生的次数分别为13次、2次，于是就能重新得到文档d中对应各个主题的词的数量，即对应主题t<sub>1</sub>,t<sub>2</sub>的词的数量分别为2个、0个（初始的d中对应各个主题的词的数量是人为设定的超参数α⃗）。

于是，我们最终得到了文档d对应各个主题的词的个数的更新值：记作向量n⃗ ,我们将更新后的向量n⃗ 再次作为狄利克雷分布的输入向量，即Dirichlet(θ<sub>d</sub>|n⃗ )
就会又会得到文档d对应各个主题的概率的更新值，即更新的θ<sub>d</sub>，如此反复迭代，最终得到收敛的θ<sub>d</sub>，即为我们要的结果。

有了以上的经验，主题t<sub>k</sub>产生各个词的概率β<sub>k</sub>可以同样处理，对于产生文档d中的第i个词w<sub>i</sub>的各个主题的分布为： 
multi(β<sup>i</sup>),于是用同上面相同的方法，可以得到更新后的各个主题产生各个单词的数量：记作向量m⃗ ，将向量m⃗ 作为新的参数再次带入狄利克雷分布Dirichlet(β<sub>k</sub>|m⃗ ),就又会得到每个主题产生各个词的概率，即更新的β<sub>k</sub>，如此反复迭代，最终得到收敛的β<sub>k</sub>,即所求结果。

得到了收敛的θ<sub>d</sub>和β<sub>k</sub>，算法就大功告成了，这时，该算法就会根据输入的文档d，找到潜在主题下的相关词啦！！！！

## 参考博文

[https://blog.csdn.net/weixin_41090915/article/details/79058768](https://blog.csdn.net/weixin_41090915/article/details/79058768)

[https://www.cnblogs.com/fengsser/p/5836677.html](https://www.cnblogs.com/fengsser/p/5836677.html)
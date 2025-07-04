# 独立同分布

机器学习界的炼丹师们最喜欢的数据有什么特点？窃以为，莫过于"**独立同分布**"了，即independent
and identically distributed，简称为
i.i.d。独立同分布并非所有机器学习模型的必然要求（比如 Naive Bayes
模型就建立在特征彼此独立的基础之上，而Logistic Regression 和 神经网络
则在非独立的特征数据上依然可以训练出很好的模型），但独立同分布的数据可以简化常规机器学习模型的训练、提升机器学习模型的预测能力，已经是一个共识。

因此，在把数据喂给机器学习模型之前，**白化（whitening）**是一个重要的数据预处理步骤。白化一般包含两个目的：

（1）去除特征之间的相关性 ---\> 独立；

（2）使得所有特征具有相同的均值和方差 ---\> 同分布。

# ICS: Internal Covariate Shift（内部协变量移位）

深度神经网络模型的训练为什么会很困难？其中一个重要的原因是，深度神经网络涉及到很多层的叠加，而每一层的参数更新会导致上层的输入数据分布发生变化，通过层层叠加，高层的输入分布变化会非常剧烈，这就使得高层需要不断去重新适应底层的参数更新。为了训好模型，我们需要非常谨慎地去设定学习率、初始化权重、以及尽可能细致的参数更新策略。

# Normalization（归一化）

由于 ICS 问题的存在，
的分布可能相差很大。要解决独立同分布的问题，"理论正确"的方法就是对每一层的数据都进行白化操作。然而标准的白化操作代价高昂，特别是我们还希望白化操作是可微的，保证白化操作可以通过反向传播来更新梯度。

因此，以 BN 为代表的 Normalization
方法退而求其次，进行了简化的白化操作。基本思想是：在将
送给神经元之前，先对其做**平移和伸缩变换**， 将
的分布规范化成在固定区间范围的标准分布。

通用变换框架就如下所示：

我们来看看这个公式中的各个参数。

（1）是**平移参数**（shift parameter）， 是**缩放参数**（scale
parameter）。通过这两个参数进行 shift 和 scale 变换：

得到的数据符合均值为 0、方差为 1 的标准分布。

（2）是**再平移参数**（re-shift parameter）， 是**再缩放参数**（re-scale
parameter）。将 上一步得到的 进一步变换为：

最终得到的数据符合均值为 、方差为 的分布。

第(2)步再平移和再缩放有两个目的：

1.  **为了保证模型的表达能力不因为规范化而下降：**我们可以看到，第一步的变换将输入数据限制到了一个全局统一的确定范围（均值为
    0、方差为
    1）。下层神经元可能很努力地在学习，但不论其如何变化，其输出的结果在交给上层神经元进行处理之前，将被粗暴地重新调整到这一固定范围。所以，为了尊重底层神经网络的学习结果，我们将规范化后的数据进行再平移和再缩放，使得每个神经元对应的输入范围是针对该神经元量身定制的一个确定范围（均值为
    、方差为 ）。rescale 和 reshift 的参数都是可学习的，这就使得
    Normalization 层可以学习如何去尊重底层的学习结果。

2.  **保证获得非线性的表达能力：**Sigmoid
    等激活函数在神经网络中有着重要作用，通过区分饱和区和非饱和区，使得神经网络的数据变换具有了非线性计算能力。而第一步的规范化会将几乎所有数据映射到激活函数的非饱和区（线性区），仅利用到了线性变化能力，从而降低了神经网络的表达能力。而进行再变换，则可以将数据从线性区变换到非线性区，恢复模型的表达能力。

**权重伸缩不变性（weight scale invariance）**指的是，当权重 按照常量
进行伸缩时，得到的规范化后的值保持不变。**权重伸缩不变性可以有效地提高反向传播的效率，还具有参数正则化的效果，可以使用更高的学习率。**

**数据伸缩不变性（data scale invariance）**指的是，当数据 按照常量
进行伸缩时，得到的规范化后的值保持不变。**数据伸缩不变性可以有效地减少梯度弥散，简化对学习率的选择**

## 类型

常见的4种类型：

- BN（Batch
  Normalization，[论文](https://arxiv.org/pdf/1502.03167.pdf)）：对BHW维度进行归一化，适合CV任务，batchsize需要偏大

- LN（Layer
  Normalizaiton，[论文](https://arxiv.org/pdf/1607.06450v1.pdf)）；对CHW维度进行归一化，适合NLP任务

- IN（Instance
  Normalization，[论文](https://arxiv.org/pdf/1607.08022.pdf)）：对HW维度进行归一化，适合CV中生成式任务

- GN（Group
  Normalization，[论文](https://arxiv.org/pdf/1803.08494.pdf)）：对(C/groupnum)HW进行归一化，适合CV任务

![descript](./Whitening（白化）/media/image1.png){width="6.299305555555556in"
height="1.7521248906386702in"}

## BN: Batch Normalization（纵向归一化）

BN的提出主要是要解决内部协变量偏移（internal covariate
shift）的问题：网络训练过程中，参数的变化会让下一层的输入数据分布发生变化，随着网络层数变深，分布变化会越来越大，偏移越严重，让模型训练变得难收敛。BN通过标准化操作，强行将网络输出均值和方差拉回到0和1的正态分布，再通过缩放和偏移操作保留原有特征特性以及防止数据集中在0附近，丢失了后续激活层中非线性特性。

BN层的优势：加快了模型训练收敛速度，缓解了深层网络中"梯度弥散"的问题。

BN的缺点：

通过BN的实现可以看出，均值和方差是在一个batch上计算的，同一个batch的同一通道拥有一样均值和方差。如果bs太小，计算得到的均值方差不能很好代码数据分布。

而且如果对于RNN这种任务，序列长度不固定的情况下，BN就无法计算；而且对于RNN任务，不同文本输入，在batch上做归一化也是不合理的，因为词和词的特征差异是很大的。

Batch Normalization 于2015年由 Google 提出，开 Normalization
之先河。其规范化针对单个神经元进行，利用网络训练时一个 mini-batch
的数据来计算该神经元 的均值和方差,因而称为 Batch Normalization。

其中 是 mini-batch 的大小。BN
是针对单个维度定义的，因此标准公式中的计算均为 element-wise
的（对具有不同索引i的，分别计算均值和方差，分别做规范化）。

BN 独立地规范化每一个输入维度 ，但规范化的参数是一个 mini-batch
的一阶统计量和二阶统计量。这就要求每一个 mini-batch
的统计量是整体统计量的近似估计，或者说每一个 mini-batch
彼此之间，以及和整体数据，都应该是近似同分布的。分布差距较小的
mini-batch
可以看做是为规范化操作和模型训练引入了噪声，可以增加模型的鲁棒性；但如果每个
mini-batch的原始分布差别很大，那么不同 mini-batch
的数据将会进行不一样的数据变换，这就增加了模型训练的难度。

需要注意的问题：

1.  大的Batch Size有利于达到 较好的性能表现

2.  建议将BN层放在卷积层和激活层之间，且卷积层不使用Bias，因为BN结果不受Bias影响

3.  训练时要将traning参数设置为True，验证和推理时要将trainning参数设置为False。在pytorch中可通过model.train()和model.eval()方法设置

![descript](./Whitening（白化）/media/image2.jpg){width="6.299305555555556in"
height="3.2375240594925634in"}

李宏毅老师关于BN的视频讲解：

<https://www.bilibili.com/video/av9770302?p=10>

## LN: Layer Normalization（横向归一化）

层规范化就是针对 BN 的上述不足而提出的。与 BN 不同，LN
是一种横向的规范化，如图所示。它综合考虑一层所有维度的输入，计算该层的平均输入值和输入方差，然后用同一个规范化操作来转换各个维度的输入。

其中 枚举了该层所有的输入神经元。

LN 针对单个训练样本进行，不依赖于其他数据，因此可以避免 BN 中受
mini-batch 数据分布影响的问题。但是，BN
的转换是针对单个神经元可训练的------不同神经元的输入经过再平移和再缩放后分布在不同的区间，而
LN
对于一整层的神经元训练得到同一个转换------所有的输入都在同一个区间范围内。如果不同输入特征不属于相似的类别（比如颜色和大小），那么
LN 的处理可能会降低模型的表达能力。

通常在NLP领域的任务，都会使用LN作为标准化层

LN的优点：

和batchsize大小和输入序列长度无关。可应用到NLP任务以及小bs的训练任务。

## IN（Instance Norm）

优缺点：

IN通常在生成式任务中使用，如果BN的batchsize=1，可以认为和IN结果一致，但是在infer阶段，IN是需要统计输入样本的均值方差的，所以IN的性能会下降。

## GN: Group Normalization

介于LN和IN之间，其首先将channel分为许多组（group），对每一组做归一化，即先将feature的维度由N,
C, H, W reshape为N, G，C//G , H, W，然后对每个组进行BN

优缺点：

GN和BN对比，避开了batchsize对训练的影响，训练开销小；GN的num_group=1就是LN，num_group=C就是IN。

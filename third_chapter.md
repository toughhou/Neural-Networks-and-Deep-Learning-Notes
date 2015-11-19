# 优化


第一章简单介绍了神经网络有关的基本概念，第二章介绍了反向传播算法，至此，对于神经网络应该已经有了较清晰的轮廓。

这一章介绍一些实际应用中的技巧。我把这些技巧统称为对神经网络的“优化”：目的是为了得到一个更加实用的神经网络。

优化的技巧千千万，过去、现在、未来都会吸引大批学者投身其中，本章只介绍其中一小部分：(1)，一个更好的损失函数：交叉熵；(2)，四种正则方法；(3)，一种初始化权重的方法；(4)如何选择超参数。

这些技巧都相当实用且非常典型，还能帮助我们举一反三，学习或提出更好的优化技巧。


##交叉熵(cross-entropy)损失函数


古人云“人非圣贤，孰能无过，过而能改，善莫大焉。”我们希望一个实用的神经网络应该有这样的学习能力：在训练时，输出和正确值偏差越大，学习越快。

也就是说，我们期望的神经网络应该是这个样子的：

![](https://ooo.0o0.ooo/2015/11/17/564be07a15224.png)

可实际情况是，神经网络往往是这个样子滴：

![](https://ooo.0o0.ooo/2015/11/17/564be0cb989ca.png)

一开始损失值很大，可是损失值减小的速度令人捉急。为什么会学习慢？实质上就是$$\partial C/\partial w,\partial C /\partial b$$太小了导致每一次的参数更新太慢了！解决这个问题，可以从两个角度入手：增大学习速率$$\eta$$和增大偏导数。这里咱们从后者入手。

目前使用的是平方损失函数：

![](https://ooo.0o0.ooo/2015/11/17/564be28d44739.png)

其中y是训练实例的label,$$a=\sigma(z),z=wx+b$$。为了简化，我们从单个神经元说起，此时使用的训练样本也只有一个实例(1,0)。

使用链式法则计算偏导数：

![](https://ooo.0o0.ooo/2015/11/17/564be2fe5b695.png)

![](https://ooo.0o0.ooo/2015/11/17/564be3c8e96e0.png)

从上图可以看出，当神经元输出接近1时，sigmoid函数导数非常小，这就导致损失函数对参数的偏导数很小，表现出来就是神经网络学习速率很慢。

### 交叉熵

上面提到的学习速率过慢问题可以通过使用交叉熵损失函数替换平方损失函数解决。

这一小节介绍什么是交叉熵损失函数。

![](https://ooo.0o0.ooo/2015/11/17/564be59692ce2.png)

交叉熵损失函数定义：

![](https://ooo.0o0.ooo/2015/11/17/564befd92153e.png)

交叉熵之所以可以作为损失函数，是因为它满足了两个条件：(1) C>0,因为a$$\in (0,1)$$;(2)当a无限接近y时，C也无限趋向0.

交叉熵损失函数的优点还在于它能避免学习速率过慢的问题。

![](https://ooo.0o0.ooo/2015/11/17/564bf6419e7f0.png)

![](https://ooo.0o0.ooo/2015/11/17/564bf67f3d9cd.png)


![](https://ooo.0o0.ooo/2015/11/17/564bf6d758a52.png)

此时偏导数大小不受$$\sigma '(z)$$影响，而是和$$\sigma(z)-y$$线性相关：输出和真实值偏差越大，偏导数越大，权重参数每一次更新时变化也越大，表现出来就是学习速率快！

同理，

![](https://ooo.0o0.ooo/2015/11/18/564c154509b14.png)


使用交叉熵损失函数，我们终于得到能快速从错误中学习的神经网络：

![](https://ooo.0o0.ooo/2015/11/18/564c15adafffe.png)


现在我们将交叉熵从单个神经元推广到多层神经网络。

![](https://ooo.0o0.ooo/2015/11/18/564c1647614fb.png)

其中$$\sum_{j}$$表示输出层的神经元输出和。


对于损失函数的选择，一般**使用sigmoid神经元**时，交叉熵总是优于平方损失函数。


对于交叉熵的物理含义，学过信息论的同学应该会有点印象，
两个随机分布p(x)和q(x)之间的鉴别信息定义为交叉熵或Kullback-Leibler距离：

![](https://ooo.0o0.ooo/2015/11/18/564c1b237f662.png)

物理意义：(a)观察者对随机变量x的了解由分布q(x)变为p(x)时获得的信息量;(b)当实际分布为p(x)而估计为q(x)时，交叉熵衡量了这种估计的偏差程度。


## Softmax

到目前为止，我们设计神经网络每一层的神经元都采用sigmoid神经元。Softmax层的意思是对于输出层中的神经元，用softmax 函数替代sigmoid函数。以第j个神经元为例，它的输出是：

![](https://ooo.0o0.ooo/2015/11/18/564c1cef46682.png)


为什么要采用softmax ？

其中一个很大的优点是：输出层各个神经元输出结果的和为1，可以把输出层看做一个离散分布！这样就可以利用概率来解释结果啦。


softmax会不会碰到学习速率很慢的问题？只要我们使用log-likelihood损失函数就能避免。

![](https://ooo.0o0.ooo/2015/11/18/564c3cd307cfa.png)


原因是，此时的偏导数为：


![](https://ooo.0o0.ooo/2015/11/18/564c3cfe4dd45.png)


到此咱们总结一下，softmax优点是既能从概率角度解释输出，还能结合log-likelihood避免学习速率过慢问题，简直棒！

如果输出层是sigmoid神经元，建议损失函数选择交叉熵；如果输出层是softmax，建议损失函数选择log-likelihood.


## 过拟合&正则化

"I remember my friend Johnny von Neumann used to say, with four parameters I can fit an elephant, and with five I can make him wiggle his trunk." --- Enrico Fermi.

啥是过拟合？用两幅图来说明，

![](https://ooo.0o0.ooo/2015/11/18/564c7771b3d73.png)

![](https://ooo.0o0.ooo/2015/11/18/564c778f0403b.png)

随着epoch增大，损失值越来越小，可是模型在测试集的分类准确率却不是一直增加，反而有一段是下降！

![](https://ooo.0o0.ooo/2015/11/18/564c797ff27bb.png) 

上图中两条折线之间距离较大，说明模型的泛化能力有限，仅在训练集上效果好，碰到测试集就萎了。

还记得第一章时我们将MNIST数据集分为三部分：训练集、验证集、测试集。现在我们要用验证集来防止过拟合。在每一次epoch结束后，我们计算此时模型在验证集上的分类准确率。一旦准确率达到饱和(saturated),我们就停止训练，这种策略称为早期停止([early stopping](https://en.wikipedia.org/wiki/Early_stopping))。而如何判断准确率是否饱和，则要靠经验了。

使用验证集的目的是想确定使神经网络具有泛化能力的超参数：如epoch数目、网络结构、学习率等。实质上就是对超参数进行寻优。
然后再使用测试集测试模型的分类能力。

当然，现在随着能得到的数据量急剧增多，我们可以使用更大的训练集来学习模型，也能很好的防止过拟合。


### 正则化

除了使用更大的训练集来防止过拟合，我们还可以利用正则化技巧来帮助我们选择具有泛化能力的模型。

这一节，介绍的是最常用的一个正则化技巧：权重衰减(weight decay)/L2正则。L2正则的做法是在损失函数基础上增加正则项。

下面是正则交叉熵损失函数：

![](https://ooo.0o0.ooo/2015/11/18/564c708e9387c.png)

其中$$\lambda >0,被称作正则参数$$。注意正则项中不含有偏置参数b。

L2正则化对损失函数的形式没有要求，对于其他损失函数比如平方损失函数，也可以进行L2正则化，

![](https://ooo.0o0.ooo/2015/11/18/564c7191a9b38.png)

所以，不失一般性，正则损失函数可以写作：

![](https://ooo.0o0.ooo/2015/11/18/564c71e4be655.png)

其中,$$C_{0}$$是损失函数。


L2正则项趋向于学习出绝对值更小的权重参数，所以你可以把L2正则化技巧理解为在学习出更小的权重参数和最小化原始损失函数中作出某种折衷。而这种折衷程度取决于$$\lambda$$的选择：如果$$\lambda$$较小，则趋向于最小化原始损失函数；如果$$\lambda$$较大，则趋向于选择小权重参数。

说了这么多，L2正则化不就是为了选择更小的权重参数吗？怎么能防止模型过拟合？
请往下看，此时，偏导数是这个样子的：


![](https://ooo.0o0.ooo/2015/11/18/564c7429467b3.png)

其中$$\partial C_{0}/\partial w,\partial C_{0}/\partial b$$可以通过反向传播算法进行计算，然后加上$$\frac{\lambda}{n}w$$就得到了正则化损失函数的偏导数。

此时，如果使用梯度下降算法，参数更新过程如下：


![](https://ooo.0o0.ooo/2015/11/18/564c75287791f.png)


![](https://ooo.0o0.ooo/2015/11/18/564c7544add1c.png)

和原始损失函数偏导数的差别在于，更新w时，要先对当前w值进行缩放，这个缩放操作也被称为权重衰减，


下面来看看使用正则化后模型的分类能力，


![](https://ooo.0o0.ooo/2015/11/18/564c786a65dc1.png)

![](https://ooo.0o0.ooo/2015/11/18/564c78828d6b2.png)

![](https://ooo.0o0.ooo/2015/11/18/564c798f69ecc.png)

确实能有效抑制过拟合，并且分类准确率也有所提高,模型也具有了较强的泛化能力。







### 为啥正则化有效果

正则化确实能防止过拟合，可是原因究竟是啥？就因为使w取值更小？

一种解释是：更小的权重参数，在某种程度上意味着模型复杂度更低， 使得模型具有更简单且强大的数据解释能力。

下面这段话来自《统计学习方法》：
**正则化符合奥卡姆剃刀(Occam's razor)原理。奥卡姆剃刀原理应用于模型选择时变为以下想法：在所有可能选择的模型中，能够很好地解释已知数据并且十分简单才是最好的模型，也就是应该选择的模型。从贝叶斯估计的角度来看，正则化项对应于模型的先验概率。可以假设复杂的模型有较大的先验概率，简单的模型有较小的先验概率**。

通过一个例子来解释这段话，

![](https://ooo.0o0.ooo/2015/11/18/564c801c0727d.png)

我们想用多项式来拟合这些点，首先，选用x的九次函数$$y_{1}=a_{0}x^{9}+a_{1}x^{8}+...+a_{9}$$,

![](https://ooo.0o0.ooo/2015/11/18/564c82d29aa10.png)

其次，选用线性函数$$y_{2}=2x$$,

![](https://ooo.0o0.ooo/2015/11/18/564c8318801e6.png)

拟合效果也不错哦，现在问题来了，$$y_{1},y_{2}$$哪个函数更接近数据的内部规律？哪个具有更好的预测能力？在没有先验的前提下，这个问题是无法回答的，有的人会说，$$y_{1}$$应该更好吧，你看每个点都精确地处在函数上。别忘了，真实世界的数据总是伴随着各种噪声，比如测量偏差，而$$y_{1}$$的数据拟合能力实在太强了，我们有理由怀疑它都把噪声拟合进去了！这种情况下，我们宁愿选择$$y_{2}$$。


现在回到神经网络，假设我们使用正则化手段来训练神经网络，得到的权重参数都偏小，这意味着，如果随机改变几个训练样本值，模型并不会改变太多。重点来了，**这也意味着模型对局部数据噪声的模拟能力较差**，换句话说，即使部分数据噪很大，也不会对整个模型有太大影响，因为w小！


注意：并非越简单的模型就越好！否则也就不会有Deep Learing这般复杂的模型了。



### 其他的正则化技巧

**L1正则**:

![](https://ooo.0o0.ooo/2015/11/18/564c88b26758e.png)


![](https://ooo.0o0.ooo/2015/11/18/564c89bbc4acc.png)

L1正则也趋向于绝对值更小的w,只不过和L2使用的方法不同。


**Dropout**
假设我们要训练这样一个神经网络，

![](https://ooo.0o0.ooo/2015/11/18/564c8ac9e6f9d.png)

















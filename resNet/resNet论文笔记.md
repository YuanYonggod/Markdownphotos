《Deep Residual Learning for Image Recognition》是2016年CVPR的最佳论文，也是我Kaiming男神第二次获得CVPR的最佳论文，简直强的一批啊！

#摘要

resNet主要解决一个问题，就是更深的神经网络如何收敛的问题，为了解决这个问题，论文提出了一个残差学习的框架。然后简单跟VGG比较了一下，152层的残差网络，比VGG深了8倍，但是比VGG复杂度更低，当然在ImageNet上的表现肯定比VGG更好，是2015年ILSVRC分类任务的冠军。

另外用resNet作为预训练模型的检测和分割效果也要更好，这个比较好理解，分类效果提升必然带来检测和分割的准确性提升。

#介绍

在resNet之前，随着网络层数的增加，收敛越来越难，大家通常把其原因归结为*梯度消失*或者*梯度爆炸*，这是不对的。另外当训练网络的时候，也会有这样一个问题，当网络层数加深的时候，准确率可能会快速的下降，这当然也不是由*过拟合*导致的。我们可以这样理解，构造一个深度模型，我们把新加的层叫做*identity mapping*（这个mapping实在不知道怎么翻译好，尴尬……），而其他层从学好的浅层模型复制过来。现在我们需要保证这个构造的深度模型并不会比之前的浅层模型产生更高的训练错误，然而目前并没有好的比较方法。

![](https://raw.githubusercontent.com/YuanYonggod/Markdownphotos/master/resNet/resnet1.jpg)

从图上可以看到，层数越多，收敛越慢，且error更高。

在论文中，kaiming大佬提出了一个*深度残差学习*框架来解决网络加深之后准确率下降的问题。用公式来表示，假如我们需要的理想的mapping定义为$H(x)$，那么我们新加的非线性层就是$F(x):=H(x)-x$，原始的mapping就从$x$变成了$F(x)+x$。也就是说，如果我们之前的$x$是最优的，那么新加的*identity mapping* $F(x)$就应该都是0，而不会是其他的值。

![](https://raw.githubusercontent.com/YuanYonggod/Markdownphotos/master/resNet/resnet2.jpg)

这样整个残差网络是端对端（end-to-end）的，可以通过随机梯度下降反向传播，而且实现起来很简单（实际上就是两层求和，在Caffe中用Eltwise层实现）。至于它为什么收敛更快，error更低，我是这么理解的：

我们知道随机梯度下降就是用的链式求导法则，我们对$H(x)$求导，相当于对$F(x)+x$求导，那么这个梯度值就会在1附近(x的导数是1)，相比之前的plain网络，自然收敛更快。

# 深度残差学习

假设多个线性和非线性的组合层可以近似任意复杂函数（这是一个开放性的问题），那么当然也可以逼近残差函数$H(x)-x$（假设输入和输出的维度相同）。

论文中残差模块定义为：

$y=F(x,{w_i})+x$

其中，$x$代表输入，$y$代表输出，$F(x,w_i)$代表需要学习的残差mapping。像上图firgure 2有两层网络，用$F=W_2\sigma(W_1x)$表示，这里$\sigma$表示ReLU激活层。这里$Wx$是卷积操作，是线性的，ReLU是非线性的。

其中$x$和$F$的维度一定要相同，如果不同的话，可以通过一个线性映射$W_s$来匹配维度：

$y=F(x,{W_i})+W_sx$

这里$F$是比较灵活的，可以包含两层或者三层，甚至更多层。但是如果只有一层的话，就变成了$y=W_ix+x$，这就是普通的线性函数了，就没有意义了。

接下来就是按照这个思路将网络结构加深了，下面列出几种结构：

![](https://raw.githubusercontent.com/YuanYonggod/Markdownphotos/master/resNet/resNet3.png)

最后是一个更深的瓶颈结构问题，论文中用三个1x1,3x3,1x1的卷积层代替前面说的两个3x3卷积层，第一个1x1用来降低维度，第三个1x1用来增加维度，这样可以保证中间的3x3卷积层拥有比较小的输入输出维度。

![](https://raw.githubusercontent.com/YuanYonggod/Markdownphotos/master/resNet/resNet4.png)

好了，resNet读到这里基本上差不多了，当然啦，后来又出了resNet的加宽版resNeXt，借鉴了GoogLeNet的思想，以后有机会再细读！
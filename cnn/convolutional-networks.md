## CS231n 卷积神经网络

> 翻译自[斯坦福大学的cs231n课程](http://cs231n.github.io/convolutional-networks/)

### 卷积神经网络(CNNs/ConvNets)

卷积神经网络的普通的神经网络有很多相似之处：他们都由神经元组成，这些神经元都有可学习的weights和biases。每个神经元接受一个输入，进行一次点乘运算（dot product），紧接着一次非线性的运算（也可能没有）。整个网络表达的是一个可微分的评价函数：输入的是图片的像素值，得到一个判断图片类别的评分。网络最后的全连接层也会有一个损失函数（SVM/Softmax）。

卷积网络和寻常的网络有何不同呢？卷积网络的输入必须是图片，因为输入是确定的，我们就可以对整个架构做一些特殊处理，使得前向的计算效率更高，同时能也减少网络的参数。

#### 架构总览

> 回忆：普通的神经网络接受一个输入，然后通过层层隐藏层（hidden layers）计算。每个隐藏层由若干个神经元（neurons）组成，每个神经元与前一个隐藏层的所有神经元连接，同层的神经元之间没有任何连接，也不共享任何参数。最后的全连接层（fully-connected layer）叫做输出层，在分类任务中，输出层负责输出每一个类别的得分。

普通神经网络没法拓展到处理图片。CIFAR-10数据集图片尺寸是32\*32\*3（宽=32，高=32，颜色通道=3），普通网络的第一个全连接层就需要有32\*32\*3=3072个参数。这个数字看起来可以接受，但稍大一些的图片，如200\*200\*3的图片就需要120000个参数，这还只是一个神经元，你肯定还想多要几个神经元，整个网络的参数数量会急剧膨胀！

卷积神经网络利用了输入由图片构成这一点对架构做了改善。和普通网络不用，ConvNet的神经元按照长(height)、宽(width)、深度(depth)三个维度组织。例如：CIFAR-10数据集的输入层的长度=32，宽度=32，深度=32。每一层的神经元都只会和前一层的一小块区域连接。处理CIFAR-10数据集的卷及网络的输出层维度会是1\*10\*10，因为最后一层我们会把原图缩成由类别得分构成的一个向量。

![普通3层神经网络](imgs/neural_net2.jpg)

![卷积神经网络，神经元由三个维度构成，每一层都接受一个三维的输入，返回一个三维的输出，图中：红色的输入层表示原始图片](imgs/cnn.jpg)

#### 搭建卷积网络的层(Layers)

卷积神经网络由层构成，每一层都将一个三维激活值矩阵通过一个可微函数转化成另一个矩阵。我们主要使用三种类型的层：卷积层（Convolutional Layer），池化层（Pooling Layer），和全连接层（Fully-Connected Layer）。我们就用这三种层堆叠出整个卷积网络的架构。

一个简单的、用来处理CIFAR-10数据集的卷积神经网络架构会是这样的：[INPUT -> CONV -> RELU -> POOL ->FC].

INPUT [32\*32\*3] 是原始的图片，图片长宽分别为32，有三个颜色通道 R G B

CONV 层会计算

（CONV layer will compute the output of neurons that are connected to local regions in the input, each computing a dot product between their weights and a small region they are connected to in the input volume. This may result in volume such as [32x32x12] if we decided to use 12 filters.）

RELU 层会执行激活函数，它保持没有改变输入的形状，输入还是[32\*32\*12].

池化层会在沿着水平方向（width和height的方向）执行一次下采样操作，缩小volume的尺寸，输出的形状大概会是 [16\*16\*12]

FC layer（全连接层）会计算类别的得分，输出的形状会是[1\*1\*10]，向量中的每一个值是对应类别的得分（CIFAR-10数据集中的图片分属10个不同的类别）。全连接层的每一个神经元都会和前层的每一个神经元连接。

In this way, ConvNets transform the original image layer by layer from the original pixel values to the final class scores. Note that some layers contain parameters and other don’t. In particular, the CONV/FC layers perform transformations that are a function of not only the activations in the input volume, but also of the parameters (the weights and biases of the neurons). On the other hand, the RELU/POOL layers will implement a fixed function. The parameters in the CONV/FC layers will be trained with gradient descent so that the class scores that the ConvNet computes are consistent with the labels in the training set for each image.




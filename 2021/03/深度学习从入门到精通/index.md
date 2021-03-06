# 深度学习从入门到精通


# 深度学习简介

## 机器学习

根据是否有标注数据，机器学习可以分为监督学习和无监督学习。

- 监督学习：从已标注的训练数据中学习如何判断数据的特征，并将其用于对未标注数据的判断的一种方法。常见的线性回归和逻辑回归就属于典型的监督学习。
- 无监督学习：它的学习算法是从未标注的训练数据中学习数据的特征。因此很多方法是基于数据挖掘的，主要特点就是寻求、总结和解释数据，例如，聚类分析就是典型的无监督学习。

## 深度学习

深度学习具备自动提取抽象特征的能力，机器学习大多是人们手动选取特征和构造特征

深度学习就是一个高度复杂的非线性回归模型。

# 神经网络基础

## 神经网络模型

1. M-P 模型
   不能学习
2. 感知机模型
   可以学习，只对线性问题具有分类能力
3. 多层感知机模型
   解决了线性不可分问题

## 激活函数

- Sigmoid 激活函数
- Tanh 激活函数
- ReLU 激活函数（最好）

## 神经网络的训练

1. 参数初始化。有常数初始化，正态分布初始化，均匀分布初始化。
2. 切分 Batch 数据。
3. 前向传播。
4. 建立损失函数。
5. 反向传播
6. 是否达到迭代次数。如果达到，结束训练，否则重复前面。

### 损失函数

可以使用任意函数，但一般用**均方根误差**和**交叉熵误差**。

### 梯度下降算法

沿着函数值下降变化最快的方向改变参数

### 批量梯度下降算法

- 批量梯度下降算法：把整个样本切分为若干分，然后在每一份样本上实施梯度下降算法进行参数更新。
- 随机梯度下降算法：每个体将只有一个样本。这样很难收敛，效率很低。

### 批量梯度下降算法改进

- 动量梯度下降算法：考虑了历史梯度的加权平均作为速率进行优化。
- 均方根加速算法：对历史梯度加权时，对当前梯度取了平方，并在参数更新时，让当前梯度对历史梯度开根号后的值做了除法运算
- 自适应矩估计(Adam)：将前面两种方法结合起来的优化算法。

### 反向传播算法

是一种高效的在所有参数上使用梯度下降算法的方法。

## 过拟合及处理方法

### 过拟合

指在训练过程中，模型对训练数据学习过度，将数据中包含的噪声和误差也学习了，使得模型在训练集上表现很好，在测试集上表现很差的现象。

### 解决方法

- 正则化：就是在损失函数中加入被称为正则化项的惩罚。常用 L2 范数。
- Dropout 方法：在某一层的单元(不包含输入层)数据随即丢弃一部分。

# 神经网络的 TensorFlow 实现

## 线性回归模型

指利用回归分析来确定两种或两种以上变量间相互依赖的定量关系的一种统计分析方法。

- 因变量：Y
- 自变量：一个或多个，X

## 逻辑回归模型

是一种广义线性回归模型，用于处理因变量是二分类的问题。Y 取值 0,1。

## Softmax 回归模型

当二分类扩展为多分类问题时，逻辑回归就变成 Softmax 回归。

# 卷积神经网络（CNN）基础

卷积运算有 3 种：Full 卷积，Same 卷积，Valid 卷积。
池化：分为最大值池化和平均值池化，又分为 same 池化和 valid 池化，就是说总共 4 种

卷积和池化的区别：

- 卷积核的权重需要在认为设定或计算过程中，通过机器学习算法自动优化得到，是一个为止的参数；而池化仅仅是求最大值，没有未知参数需要估计，也不需要参数优化的过程，因此，对计算机而言，池化是非常简单的操作。
- 不管输入的像素矩阵有多少通道，只要进行卷积运算，一个卷积核参与计算智慧产生一个通道；而池化是分层运算，输出的像素矩阵的通道取决于输入像素矩阵的通道数。

# 经典卷积神经网络

老：

- LeNet-5
- AlexNet
- VGG

新：

- Inception
- ResNet
- DenseNet
- MobileNet


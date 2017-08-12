---
layout: post
author: Ding
title:  "TensorFlow(二)--深层神经网络"
date:   2017-08-04
categories: Python
tags: TensorFlow
markdown:
  image_dir: ../images/Tensorflow
  path: 2017-08-04-Tensorflow(二)_.md
  ignore_from_front_matter: true
  absolute_image_path: false
export_on_save:
  markdown: true
---

* content
{:toc}



##　深层神经网络

深度学习是“一类通过多层非线性变换对高复杂性数据建模算法的合集”，深度神经网络就是最常见的深度学习方法。深度学习两个重要的特征`多层`和`非线性`。

### 激活函数实现非线性

![None linear](/images/Tensorflow/3.png)

通常会将每一个神经元的输出加上偏置项后通过一个非线性函数，使得神经网络不再是线性的。偏置项可以看做输出永远为１，权重为`bias`的节点，常用的激活函数有`tf.nn.relu`、`tf.sigmoid`、`tf.tanh`：

![ReLU](/images/Tensorflow/4.png)

```python
  a = tf.nn.relu(tf.matmul(x, w1) + biases1)
  b = tf.nn.relu(tf.matmul(a, w2) + biases2)
```
### 损失函数

- 交叉熵 cross entropy

对于m类问题，通常用一个m维向量表示模型的预测结果。

对于给定的概率分布p和q，其交叉熵为：

$$ H(p,q)=-\sum_{x}p(x)logq(x)$$

交叉熵表示通过概率分布q来表达概率分布p的困难程度，所以通常在神经网络中，用p代表正确答案，q代表预测值，交叉熵越小，两个概率分布越接近。

通常用过`Softmax回归`将神经网络的前向传播结果变为概率分布以计算交叉熵。

![Softmax](/images/Tensorflow/5.png)

公式为：

$$ softmax(y)_{i}=y'_{i}=\frac{e^{yi}}{\sum_{j=1}^ne^{yj}} $$

在Tensorflow中通过以下方式实现交叉熵：

```python
  cross_entropy = -tf.reduce_mean(y_ * tf.log(tf.clip_by_value(y, 1e-10, 1.0)))
```

其中`y_`表示正确结果，`y`表示神经网络模型的预测结果，`tf.clip_by_value(y, low_bound, up_bound)`表示将小于`low_bound`的整数换成`low_bound`,大于`up_bound`的整数换成`up_bound`，`tf.log()`表示求对数的运算，`×`表示矩阵中对应元素相乘。

如此就完成了$p(x)logq(x)$的计算，得到$n*m$的矩阵,其中n为一个batch中的样本数量，m为分类类别。根据公式，应该受限将每行的m个结果相加得到所有样例的交叉熵，在对n行取平均值得到一个batch的平均交叉熵。

实际上通过`tf.reduce_mean()`对整个矩阵取平均可以得到等效的平均交叉熵。

在TensorFlow中提供了`tf.nn.softmax_cross_entropy_with_logits`函数将交叉熵和softmax回归统一进行了封装，可以直接使用

```python
  cross_entropy = tf.nn.softmax_cross_entropy_with_logits(y, y_)
```
其中y代表原始神经网络的输出结果，y_为标准答案。

- 均方误差 Mean Squared Error

不同于分类问题，回归问题通常对具体的数值尽心预测，预测的结果是一个任意实数，其神经网络一般只有一个输出节点，对于回归问题，常见的损失函数是

$$ MSE(y,y') = \frac{\sum_{i=1}^n(y-y'_{i})^{2}}{n} $$

其中$y_{i}$为一个batch中第i个数据的正确答案，$y'_{i}$为神经网络的预测值，Tensorflow中可用下面方法得出

```python
  mse = tf.reduce_mean(tf.squred(y_ - y))
```

- 自定义损失函数

1. `tf.greater(a,b)`

比较两个输入张量中每一个元素的大小并返回比较结果。

2. `tf.select(tf.greater(a,b), v1, v2)`

当满足一个参数为True时，选择v1，否则选择v2。

3. `tf.reduce_sum()`

返回参数张量的所有元素之和。

### 神经网络优化

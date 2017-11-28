---
layout: post
author: Ding
title: TensorFlow简介
date: 2017-08-03T00:00:00.000Z
categories: python
tags: TensorFlow
---

* content
{:toc}

> 训练神经网络通常有三个步骤：
>
> 1. 定义网络结构和前向传播的输出结果
> 2. 定义损失函数并选择反向传播优化算法
> 3. 生成Session并在训练数据上反复运行反向传播优化算法





## 深度学习工具介绍和对比(2017.07)


|工具名称 |维护者|支持语言|支持系统|
|:-----:|:---:|:-----:|:----:|
|Caffe|加州大学伯克利分校视觉与学习中心|C++、Python、MATLAB|Linux、Mac OS X、 Windows|
|Deeplearning4j|Skymind|Java、Scala、Clojore|Linux、Mac OS X、 Windows、Android|
|Microsoft Congnitive Toolkit(CNTK)|微软研究院|Python、C++、BrainScript|Linux、Windows|
|MXNet|分布式机器学习社区(DMLC)|C++、Python、Julia、MATLAB、Go、R、Scala|Linux、Mac OS X、 Windows、Android、iOS|
|PaddlePaddle|百度|C++、Python|Linux、Mac OS X|
|Theano|蒙特利尔大学|Python|Linux、Mac OS X、Windows|
|TensorFlow|Google|C++、Python|Linux、Mac OS X、Windows、Android、iOS|

## 计算图模型


Tensor就是张量，可以简单理解为多维数组，表示了TensorFlow的数据结构。Flow是“流”，表示TensorFlow的计算模型，体现的是张量之间通过计算相互转化的过程。

TensorFlow是一个通过计算图来描述计算过程的变成系统。TensorFlow计算图上的每一个节点代表一次计算，节点之间的边描述了计算之间的依赖关系。

![Node](/images/Tensorflow/1.png)

每个节点代表一次运算(常量也可以理解为一次计算)，每条边表示计算之间的依赖关系，如果一个运算的输入依赖于另一个运算的输出，那么这两个运算之间有一条依赖关系。

- 在默认计算图中定义计算

  ```python
    import tensorflow as tf
    a = tf.constant([1.0, 2.0], name="a")
    b = tf.constant([2.0, 3.0], name="b")
  ```

  TensorFlow运行时系统会自动维护一个默认的计算图，可以通过`tf.get_default_graph()`获得，定义的计算会自动转化为默认计算图上的节点。

  ```python
  print(a.graph is tf.get_default_graph())
  ```

- 定义新的计算图

  通过`tf.graph()`定义新的计算图，不同计算图上的张量和运算相互独立。

- 指定计算设备

  可以通过`tf.Graph.device()`指定运行计算的设备。

  ```python
    g = tf.Graph()

    with g.device('/gpu:0'):
      result = a + b
  ```

## 张量数据模型


在TensorFlow中所有数据都通过张量的形式表示，张量可以理解为多维数组。其中零阶张量表示标量，即一个数；第一阶张量表示为向量，即一个一维数组；第n阶张量可以理解为一个n维数组。

实际上张量在TensorFlow中的实现不直接使用数组的形式，它只是对TensorFlow中运算结果的引用，张量中并不真正保存数字，它只是保存了得到这些数字的计算过程。

一个张量主要有三个属性：名字`name`，维度`shape`和类型`type`。

- 名字不仅是张量的唯一标识符，也能够表示张量如何得到。张量和计算图上节点表示的计算是对应的，张量的名字通过`node:src_output`给出，`node`为节点的名称，`src_output`是一个数字，表示该张量来自节点的第几个输出。

- 维度描述了张量的维度信息，例如`shaoe=(2,)`表示张量是一个长度为2的一维数组。

- 每个张量会有一个唯一的类型`dtype=float32`，当参与运算的张量类型不匹配时会报错。TensorFlow支持14种不同的类型，主要包括实数(`tf.float32`,`tf.float64`)、整数(`tf.int8`,`tf.in16`,`tf.in32`,`tf.int64`,`tf.unit8`)、布尔型(`tf.bool`)和复数(`tf.complex64`,`tf.complex128`)。

张量主要有两种使用方式：

- 对中间计算结果的引用，提高代码可读性。

```python
  a = tf.constant([1.0, 2.0], name="a")
```

- 当计算图构造完成后通过张量获得计算结果，用`session`可以得到张量中的具体数字。

```python
  tf.Session().run(a + b)
```

## 会话运行模型---Session


### 开始和关闭会话


  会话包含了Tensorflow运行的所有资源，计算完成后需要关闭会话`tf.Session().close()`来进行资源回收。

  - 手动资源回收

```python
  sess = tf.Session()
  sess.run(...)
  sess.close()
```

  - 异常退出时自动释放资源

``` python
  with tf.Session() as sess:
    sess.run(...)
```

  - 生成会话并指定为默认。通过`tf.Tensor.eval()`计算一个张量的取值。

```python
  sess = tf.Session()
  with sess.as_default():
    print(result.eval())
```

  - 下面的两种方式可以获得相同的结果

```python
  sess = tf.Session()

  print(sess.run(result))
  print(result.eval(session = sess))
```

### 通过`ConfigProto`配置会话


通过ConfigProto可以配置并行的线程数、GPU分配策略、运算超时时间等参数。

```python
  config = tf.ConfigProto(allow_soft_placement = True, log_device_placement = True)
  sess1 = tf.Session(config = config)
```

  - `allow_soft_placement = True`当运算无法在GPU上执行时自动调整到CPU上，并适应不同数量GPU的机器。

  - `log_device_placement = True`在日志中记录每个节点被安排在哪个设备上以方便调试，设置为`False`可以减少日志量。


## TensorFlow入门


目前主流的神经网络都是分层结构，同一层的节点不会相互连接，每一层之和相邻的下一层连接，直到最后一层输出层得到计算结果。输入层和输出层之间的叫做隐藏层，隐藏层越多，这个神经网络的深度越“深”。神经网络中节点之间的联系表示了神经网络中的参数，通过对参数进行训练对神经网络进行优化，从而解决问题。

使用神经网络解决分类问题主要分为以下4步：

1. 提取特征向量作为神经网络的输入。
2. 定义神经网络的结构，如何通过前向传播算法由输入得到输出。
3. 通过训练调整神经网络中参数的取值。
4. 使用训练好的神经网络对未知的目标数据进行预测。

### 前向传播算法


神经元是神经网络的最小单元，通常由多个输入得到一个输出，其输入可以由外部输入神经网络，也可以是其他神经元的输出。一个简单的神经元的输出就是其所有输入的加权和，不同输入的权重就是神经元的参数，神经网络的优化过程就是优化神经元中参数的过程。

![Network](/images/Tensorflow/2.png)


对于全连通的网络，其节点的值可以通过如下方式计算

<p align="center"><img src="https://latex.codecogs.com/gif.latex?a_{11}=W_{1,1}^{(1)}x_{1}+W_{2,1}^{(1)}x_{2}"/></p>

前向传播算法可以表示为矩阵惩罚，将输入x1,x2组织为１ｘ２的矩阵x=[x1,x2],权重参数Ｗ组织为２ｘ３的矩阵

<p align="center"><img src="https://latex.codecogs.com/gif.latex?W =&#x5C;begin{bmatrix}W_{1,1}^{(1)} &amp; W_{1,2}^{(1)} &amp; W_{1,3}^{(1)}&#x5C;&#x5C;W_{2,1}^{(1)} &amp; W_{2,2}^{(1)} &amp; W_{2,3}^{(1)}&#x5C;end{bmatrix}"/></p>

通过矩阵乘法可以得到三个隐藏层节点组成的向量的取值：

<p align="center"><img src="https://latex.codecogs.com/gif.latex?a^{(1)}=&#x5C;begin{bmatrix} a_{11} &amp; a_{12} &amp; a_{13} &#x5C;end{bmatrix}=xW^{(1)}=[x_{1},x_{2}]&#x5C;begin{bmatrix}W_{1,1}^{(1)} &amp; W_{1,2}^{(1)} &amp; W_{1,3}^{(1)}&#x5C;&#x5C;W_{2,1}^{(1)} &amp; W_{2,2}^{(1)} &amp; W_{2,3}^{(1)}&#x5C;end{bmatrix}&#x5C;&#x5C;=[W_{1,1}^{(1)}x_{1} +W_{2,1}^{(1)}x_{2}, W_{1,2}^{(1)}x_{1}+W_{2,2}^{(1)}x_{2},W_{1,2}^{(1)}x_{1} +W_{2,3}^{(1)}x{2}]"/></p>

类似的输出层可以表示为：

<p align="center"><img src="https://latex.codecogs.com/gif.latex?[y]=a^{(1)}W^{(2)}=[a_{11},a_{12},a_{13}]&#x5C;begin{bmatrix} W_{1,1}^{(2)} &#x5C;&#x5C;W_{2,1}^{(2)}&#x5C;&#x5C;W_{3,1}^{(2)} &#x5C;end{bmatrix}=[W_{1,1}^{(2)}a_{11}+W_{2,1}^{(2)}a_{12}+W_{3,1}^{(2)}a_{13}]"/></p>

在TensorFlow用`tf.matmul`实现矩阵乘法：

```python
  a = tf.matmul(x, w1)
  y = tf.matmul(a, w2)
```

### 神经网络参数与TensorFlow变量


TensorFlow中用变量`tf.Variable`保存和更新神经网络中的参数。用以下方法初始化变量，得到一个2X3的矩阵，矩阵元素均值为0，标准差为2：

```python
  weights = tf.Variable(tf.random_normal([2,3], mean=0, stddev=2))
```
TensorFlow中的随机数生成函数有：

|      函数名称       | 随机数分布 |                 参数                  |
| ------------------- | ---------- | ------------------------------------- |
| tf.random_normal    | 正态分布   | 平均值、标准差、取值类型              |
| tf.truncated_normal | 正态分布   | 平均值、标准差、取值类型              |
| tf.random_uniform   | 均匀分布   | 最小、最大取值、取值类型              |
| tf.random_gamma     | Gamma分布  | 形状参数alpha、尺度参数beta、取值类型 |

用常数初始化变量：

|  函数名称   |        功能        |         样例          |
| ----------- | ------------------ | --------------------- |
| tf.zeros    | 产生全0数组        | tf.zeros([2,3],int32) |
| tf.ones     | 产生全1数组        | tf.ones([2,3],int32)  |
| tf.fill     | 用给定数字填充数组 | tf.fill([2,3],9)      |
| tf.constant | 产生给定值的常量   | tf.constant([1,2,3])  |

用常数来初始化偏置量(bias)：

```python
  biases = tf.Variable(tf.zeros([3]))
```

当变量被声明后，还需要调用变量，通常有两种方式：

- 在会话中调用

```python
  import tensorflow as tf

  w1 = tf.Variable(tf.random_normal([2, 3], stddev=1, seed=1))
  w2 = tf.Variable(tf.random_normal([2, 3], stddev=1, seed=1))

  x = tf.constant([0.7, 0.9])

  a = tf.matmul(x, w1)
  y = tf.matmul(a, w2)

  sess = tf.Session()

  sess.run(w1.initializer)
  sess.run(w2.initializer)

  print(sess.run(y))
  sess.close()
```

- 用`tf.initialize_all_variables`

```python
  import tensorflow as tf

  w1 = tf.Variable(tf.random_normal([2, 3], stddev=1, seed=1))
  w2 = tf.Variable(tf.random_normal([3, 1], stddev=1, seed=1))

  x = tf.constant([0.7, 0.9])

  a = tf.matmul(x, w1)
  y = tf.matmul(a, w2)

  sess = tf.Session()

  init_op = tf.initialize_all_variables()

  sess.run(init_op)

  print(sess.run(y))
  sess.close()
```

### 反向传播算法


反向传播算法是一个迭代的过程，每次迭代选择一部分数据作为`batch`，通过前向传播算法计算在当前模型下batch的预测结果，通过标注的答案与预测结果的差距对神经网络模型的参数进行修正，使得该batch的预测结果更接近真实答案。

由于Tensorflow会对每一次生成的常亮产生一个计算图节点，所以用`placeholder`来保存迭代中的常亮数据，避免计算图的冗余，用`feed_dict`提供每个`placeholder`数据的字典。

```python
  import tensorflow as tf

  w1 = tf.Variable(tf.random_normal([2, 3], stddev=1, seed=1))
  w2 = tf.Variable(tf.random_normal([3, 1], stddev=1, seed=1))

  x = tf.placeholder（tf.float32, shape=[3, 2], name="input")

  a = tf.matmul(x, w1)
  y = tf.matmul(a, w2)

  sess = tf.Session()

  init_op = tf.initialize_all_variables()

  sess.run(init_op)

  print(sess.run(y, feed_dict={x: [[0.7, 0.9], [0.1, 0.4], [0.5, 0.8]]}))
  sess.close()
```

得到`batch`的前向传播结果之后，定义一个损失函数来记录当前预测值与真实答案之前的差距，并用反向传播算法调整神经网络参数。

```python
  # 损失函数
  cross_entropy = - tf.reduce_mean(y_ * tf.log(tf.clip_by_value(y, 1e-10, 1.0)))

  learning_rate = 0.001

  # 定义反向传播算法
  train_step = tf.train.AdamOptimizer(learning_rate).minimizer(cross_entropy)

  sess.run(train_step)
```

### 完整的神经网络样例程序


训练神经网络通常有三个步骤：
1. 定义网络结构和前向传播的输出结果
2. 定义损失函数并选择反向传播优化算法
3. 生成Session并在训练数据上反复运行反向传播优化算法

```python
  import tensorflow as tf

  # 通过NumPy来生成模拟的随机数据
  from numpy.random import RandomState

  w1 = tf.Variable(tf.random_normal([2, 3], stddev=1, seed=1))
  w2 = tf.Variable(tf.random_normal([3, 1], stddev=1, seed=1))

  batch_size = 8
  # 在shape的一个维度上用None，可以一次读入全部数据，数据较大时，可能导致内存溢出
  x = tf.placeholder（tf.float32, shape=(None, 2), name="x-input")
  y_ = tf.placeholder（tf.float32, shape=(None, 1), name="y-input")

  # 前向传播过程
  a = tf.matmul(x, w1)
  y = tf.matmul(a, w2)

  # 损失函数和反向传播算法
  cross_entropy = - tf.reduce_mean(y_ * tf.log(tf.clip_by_value(y, 1e-10, 1.0)))
  learning_rate = 0.001
  train_step = tf.train.AdamOptimizer(learning_rate).minimizer(cross_entropy)

  sess = tf.Session()

  # 生成随机的模拟数据集
  rdm = RandomState(1)
  dataset_size = 128
  X = rdm.rand(dataset_size, 2)
  # x1+x2<1的数据记录为正样本记为1，其余为负样本记为0
  Y = [[int(x1+x2 < 1) for (x1, x2) in X]]

  with tf.Session() as sess:
    init_op = tf.initialize_all_variables()
    sess.run(init_op)

    # 训练之前的参数
    print sess.run(w1)
    print sess.run(w2)

    # 训练的轮数
    STEPS = 5000
    for i in range(STEPS)：
      # 每次选取batch_size个样本进行训练
      start = (i * batch_size) % dataset_size
      end = min(start + batch_size, dataset_size)

      # 通过选取的样本训练神经网络
      sess.run(train_step, feed_dict={x: X[start:end], y_: Y[start:end]})
      if i % 1000 == 0:
        total_cross_entropy - sess.run(cross_entropy, feed_dict{x: X, y_: Y})

        print("After %d training steps, cross entropy on all data is %g" %(i, total_cross_entropy))

    # 训练之后的参数
    print sess.run(w1)
    print sess.run(w2)
```

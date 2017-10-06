
+ combineByKey

combineByKey是Spark中一个比较核心的高级函数，其他一些高阶键值对函数底层都是用它实现的。诸如 groupByKey,reduceByKey等等.

```java
  def combineByKey[C](  
        createCombiner: V => C,  
        mergeValue: (C, V) => C,  
        mergeCombiners: (C, C) => C,  
        partitioner: Partitioner,  
        mapSideCombine: Boolean = true,  
        serializer: Serializer = null)
```

如下解释下3个重要的函数参数：
  - createCombiner: V => C ，这个函数把当前的值作为参数，此时我们可以对其做些附加操作(类型转换)并把它返回 (这一步类似于初始化操作)
  - mergeValue: (C, V) => C，该函数把元素V合并到之前的元素C(createCombiner)上 (这个操作在每个分区内进行)
  - mergeCombiners: (C, C) => C，该函数把2个元素C合并 (这个操作在不同分区间进行)

如下看一个使用combineByKey来求解平均数的例子

```scala
  val initialScores = Array(("Fred", 88.0), ("Fred", 95.0), ("Fred", 91.0), ("Wilma", 93.0), ("Wilma", 95.0), ("Wilma", 98.0))  
  val d1 = sc.parallelize(initialScores)  
  type MVType = (Int, Double) //定义一个元组类型(科目计数器,分数)  
  d1.combineByKey(  
    score => (1, score),  
    (c1: MVType, newScore) => (c1._1 + 1, c1._2 + newScore),  
    (c1: MVType, c2: MVType) => (c1._1 + c2._1, c1._2 + c2._2)  
  ).map { case (name, (num, socre)) => (name, socre / num) }.collect  
```

参数含义的解释:
  - score => (1, score)，我们把分数作为参数,并返回了附加的元组类型。 以"Fred"为列，当前其分数为88.0 =>(1,88.0)  1表示当前科目的计数器，此时只有一个科目

  - (c1: MVType, newScore) => (c1._1 + 1, c1._2 + newScore)，注意这里的c1就是createCombiner初始化得到的(1,88.0)。在一个分区内，我们又碰到了"Fred"的一个新的分数91.0。当然我们要把之前的科目分数和当前的分数加起来即c1._2 + newScore,然后把科目计算器加1即c1._1 + 1

  - (c1: MVType, c2: MVType) => (c1._1 + c2._1, c1._2 + c2._2)，注意"Fred"可能是个学霸,他选修的科目可能过多而分散在不同的分区中。所有的分区都进行mergeValue后,接下来就是对分区间进行合并了,分区间科目数和科目数相加分数和分数相加就得到了总分和总科目数.

结果如下:

```
  res1: Array[(String, Double)] = Array((Wilma,95.33333333333333), (Fred,91.33333333333333))  
```

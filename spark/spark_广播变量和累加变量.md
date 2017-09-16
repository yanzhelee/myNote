# Spark的广播变量和累加变量

**说明:该文档针对spark2.1.0版本**

通常情况下，当向Spark操作（比如map或者reduce）传递一个函数时，它会在一个远程集群节点上执行，它会使用函数中所有变量的副本。这些变量被复制到所有的机器上，远程机器远程机器上并没有被更新的变量会向驱动程序回传。在任务之间使用通用的，支持读写的共享变量是低效的。尽管如此，Spark提供了两种有限类型的共享变量，广播变量和累加器。

## 广播变量

广播变量允许程序员将一个只读的变量缓存在每台机器上，而不用再任务之间传递变量。广播变量可被用于有效地给每个节点一个大输入数据集的副本。Spark还尝试使用高效地广播算法来分发变量，进而减少通信的开销。

Spark的动作通过一系列的步骤执行，这些步骤由分布式的洗牌操作分开。Spark自动地广播每个步骤每个任务需要的通用数据。这些广播数据被序列化地缓存，在运行任务之前被反序列化出来。这意味着当我们需要在多个阶段的任务之间使用相同的数据，或者以反序列化形式缓存数据是十分重要的时候，显式地创建广播变量才有用。

通过在一个变量v上调用SparkContext.broadcast(v)可以创建广播变量。广播变量是围绕着v的封装，可以通过value方法访问这个变量。举例如下：
```
// scala写法
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)

// java写法
Broadcast<int[]> broadcastVar = sc.broadcast(new int[] {1, 2, 3});

broadcastVar.value();
// returns [1, 2, 3]

// python写法
>>> broadcastVar = sc.broadcast([1, 2, 3])
<pyspark.broadcast.Broadcast object at 0x102789f10>

>>> broadcastVar.value
[1, 2, 3]
```
在创建了广播变量之后，在集群上的所有函数中应该使用它来替代使用v.这样v就不会不止一次地在节点之间传输了。另外，为了确保所有的节点获得相同的变量，对象v在被广播之后就不应该再修改。

## 累加变量

累加变量也称为累加器，它只能进行累加操作，因此它可以在并行中被有效地支持。它们可以用于实现计数器(比如MapReduce)或算数运算。Spark本机支持数字类型的累加器，程序员可以添加对新类型的支持。

用户可以创建有名的或者无名的累加变量，如下图所示，一个命名为counter的累加变量，将在web UI中显示用于修改该累加变量的阶段。

![](http://spark.apache.org/docs/2.1.0/img/spark-webui-accumulators.png)

在UI中跟踪累加器对于了解运行阶段的进展非常有用(注意:这在Python中还没有得到支持)。

数字累加器可以通过调用创建SparkContext.longAccumulator()或SparkContext.doubleAccumulator()累加器。在集群上运行的任务可以使用add方法对累加器进行增加。但是，它们无法读取累加器中的值。只有驱动程序可以使用它的value方法读取累加器的值。
```
scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: Some(My Accumulator), value: 0)

scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

scala> accum.value
res2: Long = 10
```
程序员也可以自己定义累加器，只需要继承AccumulatorV2抽象类，并且覆盖抽象方法。
```
class MyAccumulatorV2 extends AccumulatorV2[String, String] {
  //如果累加器中的值是0值返回true否则返回false。对于一个计数器累加器，0是零值;对于列表累加器，Nil是零值。
  override def isZero: Boolean = ???

  // 通过累加器创建一个副本
  override def copy(): AccumulatorV2[String, String] = ???

  // 重置成0值，所以isZero方法就会返回true
  override def reset(): Unit = ???

  // 增加方法
  override def add(v: String): Unit = ???

  // 对另一个同类型的累加器合并到当前对象中，并更新它的状态
  override def merge(other: AccumulatorV2[String, String]): Unit = ???

  // 获取累加器当前值
  override def value: String = ???

}
```

- isZero: 当AccumulatorV2中存在类似数据不存在这种问题时，是否结束程序。
- copy: 拷贝一个新的AccumulatorV2
- reset: 重置AccumulatorV2中的数据
- add: 操作数据累加方法实现
- merge: 合并数据
- value: AccumulatorV2对外访问的数据结果

使用方式如下:
```
// 创建一个累加器对象
val myVectorAcc = new MyAccumulatorV2
// 注册累加器，如果不注册会抛出序列化异常
sc.register(myVectorAcc, "MyVectorAcc1")
```


## 参考文档

[http://spark.apache.org/docs/2.1.0/programming-guide.html](http://spark.apache.org/docs/2.1.0/programming-guide.html)

# Spark中的Driver和Executor

## 驱动器节点Driver

Spark的驱动器是执行开发程序中的main方法的进程。它负责开发人员编写的用来创建SparkContext、创建RDD，以及进行RDD的转化和行动操作的代码的执行。如果你是用spark shell，那么当你启动spark shell的时候，系统后台自启动一个Spark驱动器程序，就是在Spark shell中预加载的一个sc——SparkContext对象。如果驱动程序终止，那么Spark应用也就结束了。

**Driver在spark作业执行的过程中主要负责一下操作**

1. 把用户程序转换为任务
    > Driver程序负责把用户程序转为多个物理执行单元，这些单元也被称为任务(task).从上层来看，sparkch程序的流程是这样的：
    > 1. 读取或转化数据创建一系列RDD
    > 2. 然后使用转化操作生成新的RDD
    > 3. 最后使用行动操作得到结果或者将数据存储到文件系统中。
    >
    > spark程序其实隐士地创建出了一个由上述操作组成的的逻辑上的有向无环图。
    > 当Driver程序运行时，它会把这个逻辑图转为物理执行计划。
    > <br />
    > spark会对逻辑执行计划作一些优化，比如将连续的映射转为流水线化执行，
    > 将多个操作合并到一个步骤中等。
    > 这样Spark就把逻辑计划转为一系列的Stage。而每个Stage又由多个task组成，
    > 这些task会被打包并送到集群中。task是spark中最小的执行单元，用户程序通常要启动成百上千的独立任务。
2. 跟踪Executor运行状况
    > 有了物理执行计划之后，Driver程序必须在各个Executor进程协调任务的调度。Executor进程启动后，会向Driver进程注册自己。因此Driver进程就可以跟踪应用中所有的Executor节点的运行信息。
3. 为Executor节点调度任务
    > Driver进程会根据当前的Executor节点集合，尝试把所有Task基于数据所在位置分配给合适的Executor进程。当Task执行时，Executor进程会把缓存数据存储起来，而Driver进程同样会跟踪这些缓存数据的位置，并且利用这些位置信息来调度以后的任务，以尽量减少数据的网络传输。
4. UI展示应用运行状况
    > Driver程序会将一些spark应用的运行时的信息通过网页界面呈现出来，默认在端口4040上。比如在本地模式下，访问http://localhost:4040 就可以看到这个网页了。

## 执行器节点Executor

spark Executor节点是一个工作进程，负责在spark作业中运行任务，任务间相互独立。Spark应用启动时，Executor节点被同时启动，并且始终伴随着整个Spark应用的生命周期而存在。如果有Executor节点发生了故障崩溃，spark应用也可以继续执行，会将出错节点上的任务调度到其他Executor节点上继续执行。

**Executor有两大作用**

1. 它们负责运行组成spark应用的任务，并将结果返回给Driver进程
2. 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的RDD提供内存式存储。RDD是直接缓存在Executor进程内的，因此任务可以在运行时充分利用缓存数据加速运算。

## 参考链接

[http://blog.sina.com.cn/s/blog_15fc03d810102wto0.html](http://blog.sina.com.cn/s/blog_15fc03d810102wto0.html)

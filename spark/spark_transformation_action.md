# Spark中的Transformations和Action操作

## Transformation

transformation操作是为了得到一个新的RDD，并且所有的transformation都是采用的懒策略，如果只是将transformation提交是不会执行计算的，计算只有在action被提交的时候才被触发。下表列出了所有的transformation算子操作。

|                         算子                         |                                                       解释                                                        |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| map(fun)                                             | 对原RDD中的每个元素进行func操作时会得到新的元素，那么由新的元素组成一个新的RDD                                    |
| filter(func)                                         | 对原RDD中的每个元素进行func操作，如果返回结果为true那么该元素被过滤出来，再由所有过滤出来的元素组成新的RDD        |
| flatMap(func)                                        | 类似map操作，但是func操作中的每个输入可以对应0到多个输出项（所以func的输出是一个序列）                            |
| mapPartitions(func)                                  | 类似map操作，但是func是对每个分区的操作，所以func的函数形式为`Iterator<T> => Iterator<U>`，T类型为原RDD的数据类型 |
| mapPartitionsWithIndex(func)                         | 和mapPartitions操作类似，只不过func的函数形式为(Int, Iterator<T>) => Iterator<U>，其中Int类型是分区索引           |
| sample(withReplacement, fraction, seed)              | 随机采样，是否放回，采样范围，和随机因子，如果随机因子固定那么每次抽样结果都会一样                                |
| union(otherDataset)                                  | 将两个相同数据类型的RDD进行结合                                                                                   |
| intersection(otherDataset)                           | 取原RDD和参数RDD数据的交集组成新的RDD                                                                             |
| distinct([numTasks]))                                | 对原RDD去重，并且可以指定分区数(可选)                                                                             |
| groupByKey([numTasks])                               | 按照key进行分组，参数可以是Partitioner对象，也可以是分区数，也可以不传参数                                        |
| reduceByKey(func, [numTasks])                        | 按照key进行聚合                                                                                                   |
| aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) | 和recudeByKey类似不过比它更加复杂，这里不解释，下面示例有详解                                                     |
|                                                      |                                                                                                                   |

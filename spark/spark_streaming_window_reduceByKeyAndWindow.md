# Spark Streaming中关于reduceByKeyAndWindow详细剖析

```java
/**
   * Return a new DStream by applying `reduceByKey` over a sliding window. Similar to
   * `DStream.reduceByKey()`, but applies it over a sliding window.
   * @param reduceFunc 聚合变换函数
   * @param windowDuration width of the window; must be a multiple of this DStream's
   *                       batching interval
   * @param slideDuration  sliding interval of the window (i.e., the interval after which
   *                       the new DStream will generate RDDs); must be a multiple of this
   *                       DStream's batching interval
   * @param partitioner    partitioner for controlling the partitioning of each RDD
   *                       in the new DStream.
   */
  def reduceByKeyAndWindow(
      reduceFunc: (V, V) => V,
      windowDuration: Duration,
      slideDuration: Duration,
      partitioner: Partitioner
    ): DStream[(K, V)] = ssc.withScope {
    self.reduceByKey(reduceFunc, partitioner)
        .window(windowDuration, slideDuration)
        .reduceByKey(reduceFunc, partitioner)
  }
```

```java
/**
 * Return a new DStream by applying incremental `reduceByKey` over a sliding window.
 * The reduced value of over a new window is calculated using the old window's reduced value :
 *  1. reduce the new values that entered the window (e.g., adding new counts)
 *  2. "inverse reduce" the old values that left the window (e.g., subtracting old counts)
 * This is more efficient than reduceByKeyAndWindow without "inverse reduce" function.
 * However, it is applicable to only "invertible reduce functions".
 * @param reduceFunc     聚合变换函数
 * @param invReduceFunc  inverse reduce function
 * @param windowDuration width of the window; must be a multiple of this DStream's
 *                       batching interval
 * @param slideDuration  sliding interval of the window (i.e., the interval after which
 *                       the new DStream will generate RDDs); must be a multiple of this
 *                       DStream's batching interval
 * @param partitioner    partitioner for controlling the partitioning of each RDD in the new
 *                       DStream.
 * @param filterFunc     Optional function to filter expired key-value pairs;
 *                       only pairs that satisfy the function are retained
 */
def reduceByKeyAndWindow(
    reduceFunc: (V, V) => V,
    invReduceFunc: (V, V) => V,
    windowDuration: Duration,
    slideDuration: Duration,
    partitioner: Partitioner,
    filterFunc: ((K, V)) => Boolean
  ): DStream[(K, V)] = ssc.withScope {

  val cleanedReduceFunc = ssc.sc.clean(reduceFunc)
  val cleanedInvReduceFunc = ssc.sc.clean(invReduceFunc)
  val cleanedFilterFunc = if (filterFunc != null) Some(ssc.sc.clean(filterFunc)) else None
  new ReducedWindowedDStream[K, V](
    self, cleanedReduceFunc, cleanedInvReduceFunc, cleanedFilterFunc,
    windowDuration, slideDuration, partitioner
  )
}
```

##

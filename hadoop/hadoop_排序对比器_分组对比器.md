# hadoop自定义排序对比器和分组对比器

## 概述

MR作业大致分为两个阶段，具体流程如下：
> 1. map阶段
>	1. 读取输入文件内容，解析成k-v对。对输入文件的每一行，解析成k-v对。
>	2. 执行自定义map函数过程，对输入k-v进行处理，每一个k-v对调用一次map函数，然后输出新的k-v对。
>	3. 对输出的k-v进行分区。
>	4. 对不同分区的数据，按照key进行排序和分组。相同的Key的value放在同一个集合中。
>	5. 本地reduce过程，对分组后的数据进行规约。（该过程不是必需的）
> 2. reduce阶段
>	1. 对多个map任务的输出（或者是多个combin过程的输出），按照不同的分区，通过网络传输到不同的reduce节点上。
>	2. 对map任务的输出进行合并、排序。
>	3. 自定义reduce阶段，对输入的k-v进行处理，转换成新的k-v。
>	4. 把reduce的输出保存到hdfs上。

从上面的流程中可以看出不管是在map阶段还是在reduce阶段都需要对key值进行排序和分组。所以对key值进行排序就得需要排序对比器，对key值进行分组就得需要分组对比器。<br />
在默认情况下hadoop是按照key值的compareTo方法进行排序和分组的，hadoop对常用的java基本数据类型以及对象等都进行了包装，将他们包装成WritableCompatrable对象，并且都实现了compareTo方法。<br/>
如果自定义数据类型作为key的话，必须要实现WritableComparable接口。<br/>
当然hadoop也允许程序员自己定义相应的排序对比器和分组排序对比器来对key值进行灵活的排序和分组。

## 自定义对比器方法

实现步骤：

1. 自定义类MyComparator继承WritableComparator
2. 添加空构造方法
3. 重写`compare(WritableComparable a, WritableComparable b)`方法
4. 将MyComparator类型加入到job的配置文件中

代码部分实现
```java
public class MyComparator extends WritableComparator {
    public KeyComparator(){
        super(DefinedKeyType.class, true);
    }

    public int compare(WritableComparable a, WritableComparable b) {
        ......
    }
}
```
将对比起加入到job中
```java
//设置排序对比器
job.setSortComparatorClass(KeyComparator.class);
//设置分组对比起
job.setGroupingComparatorClass(GroupComparator.class);
```
## WritableComparator类解析

### 构造函数分析
```java
protected WritableComparator() {
    this(null);
  }


/** Construct for a {@link WritableComparable} implementation. */
protected WritableComparator(Class<? extends WritableComparable> keyClass) {
this(keyClass, null, false);
}

protected WritableComparator(Class<? extends WritableComparable> keyClass,
    boolean createInstances) {
  this(keyClass, null, createInstances);
}
//所有的构造方法最终都要调用的根构造
protected WritableComparator(Class<? extends WritableComparable> keyClass,
                               Configuration conf,
                               boolean createInstances) {
    this.keyClass = keyClass;
    this.conf = (conf != null) ? conf : new Configuration();
    if (createInstances) {
      key1 = newKey();
      key2 = newKey();
      buffer = new DataInputBuffer();
    } else {
      key1 = key2 = null;
      buffer = null;
    }
}

```

### compare方法

```java
/**
 * 覆盖的原生比较器中的方法，最终调用compare(WritableComparator,WritableComparator)方法
 * 该方法需要buffer对象，key1和key2对象存在，所以自定义比较器时必须要重写构造函数，并且
 * 将boolean createInstances参数设为true
 */
public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2) {
    try {
      buffer.reset(b1, s1, l1);                   // parse key1
      key1.readFields(buffer);

      buffer.reset(b2, s2, l2);                   // parse key2
      key2.readFields(buffer);

      buffer.reset(null, 0, 0);                   // clean up reference
    } catch (IOException e) {
      throw new RuntimeException(e);
    }

    return compare(key1, key2);                   // compare them
}
//最终调用compare(WritableComparator,WritableComparator)方法
public int compare(Object a, Object b) {
    return compare((WritableComparable)a, (WritableComparable)b);
}
//自定义比较器时只需要覆盖该方法即可，因为其它的所有重载方法最终都是 调用的这个方法
public int compare(WritableComparable a, WritableComparable b) {
    return a.compareTo(b);
}
```

## 总结

自定义Key值对比器和分组对比器的实现方式一样，默认在不定义对比器的情况下，排序和分组都是按照key值对象的compateTo方法进行对比的。<br/>
需要注意的是自定义对比器时一定要重写构造函数，将boolean createInstances的属性设置为true。

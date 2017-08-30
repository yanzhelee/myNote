# HBase CURD之Delete

HTable提供了删除方法，同时与之前的方法一样有一个相应的类为Delete。

## 1 单行删除

delete()方法有许多变体其中一个只需一个Delete实例,下面是示例代码：
```java
@Test
public void testDelete() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    Delete delete = new Delete(Bytes.toBytes("row99"));
    table.delete(delete);
    table.close();
    conn.close();
}

@Test
public void testDelete2() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    Delete delete = new Delete(Bytes.toBytes("row98"));
    delete.addColumn(Bytes.toBytes("f1"),Bytes.toBytes("name"));
    table.delete(delete);
    table.close();
    conn.close();
}
```

用户必须创建一个Delete实例，然后再添加你想要删除的数据的详细信息。构造函数如下：
```java
public Delete(byte [] row)
public Delete(final byte [] rowArray, final int rowOffset, final int rowLength)
public Delete(final byte [] rowArray, final int rowOffset, final int rowLength, long ts)
public Delete(byte [] row, long timestamp)
public Delete(final Delete d)
```

有以下方法可以缩小删除范围。
```java
public Delete addFamily(final byte [] family)
public Delete addFamily(final byte [] family, final long timestamp)
public Delete addColumn(final byte [] family, final byte [] qualifier)
public Delete addColumn(byte [] family, byte [] qualifier, long timestamp)
public Delete addColumns(final byte [] family, final byte [] qualifier)
public Delete addColumns(final byte [] family, final byte [] qualifier, final long timestamp)
public Delete setTimestamp(long timestamp)
```
首先用户可以使用addFamily()方法来删除一整个列族，包括其下的所有列。用户可以设置时间戳，出发针对单元格数据版本的过滤，从所有的列中删除与这个时间戳相匹配的版本和比这个 时间戳版本旧的版本。

另一个是addColumns()，它作用于特定的一列，如果用户没有指定时间戳，这个方法会删除该列的所有版本。如果用户指定了时间戳，这个方法会删除所有与这个版本相匹配的版本和更旧的版本。

最后一个方法是setTimestamp()，这个方法在调用其他方法时经常容易忽略，但是如果不指定列族或者列，则此调用与删除整行不同，它会删除匹配时间戳的或者比给定时间戳之前的所有列。

|     方法     |            无时间戳的删除            |                       有时间戳的删除                       |
| ------------ | ------------------------------------ | ---------------------------------------------------------- |
| none         | 删除整行，即所有列的所有版本         | 从所哟列族的所有列中删除与给定时间戳相同或者更旧的版本     |
| addColumn()  | 只删除给定列的最新版本，保留但旧版本 | 只删除与十几件戳匹配的给定列的指定版本，如果不存在则不删除 |
| addColumns() | 删除给定列的所有版本                 | 从给定列中删除给定时间戳相等的和更旧的版本                 |
| addFamily()  | 删除给定列族的所有列                 | 从给定列族的所有列中删除与给定时间戳相等或者更旧的版本     |

## 2 Delete的列表

基于列表的delete()调用与给予列表的put()调用非常类似，需要创建一个包含Delete实例的列表，对其进行配置，并调用下面的方法：
```java
public void delete(final List<Delete> deletes)
```
**代码示例如下**
```java
@Test
public void testDeletes() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    List<Delete> deletes = new ArrayList<Delete>();
    Delete delete1 = new Delete(Bytes.toBytes("row97"));
    Delete delete2 = new Delete(Bytes.toBytes("row96"));
    Delete delete3 = new Delete(Bytes.toBytes("row95"));

    deletes.add(delete1);
    deletes.add(delete2);
    deletes.add(delete3);

    table.delete(deletes);

    table.close();
    conn.close();

}
```

## 3 原子性操作compare-and-delete

提供了让用户可以在服务器端读取并删除的功能。
```java
public boolean checkAndDelete(final byte [] row,
      final byte [] family, final byte [] qualifier, final byte [] value,
      final Delete delete)
```
用户必须指定行键、列族、列限定符和值来执行操作之前的检查。如果检查失败，则不执行删除操作，调用返回false。如果检查成功，则执行删除操作，调用返回true。代码如下：

```java
Configuration conf = HBaseConfiguration.create();

HTable table = new HTable(conf, "testtable");
List<Delete> deletes = new ArrayList<Delete>();
Delete delete1 = new Delete(Bytes.toBytes("row1"));
delete1.deleteColumns(Bytes.toBytes("colfam1"), Bytes.toBytes("qual3"));
boolean res1 = table
               .checkAndDelete(Bytes.toBytes("row1"),
                               Bytes.toBytes("colfam2"), Bytes.toBytes("qual3"), null,
                               delete1);
System.out.println("Delete successful:" + res1);
Delete delete2 = new Delete(Bytes.toBytes("row1"));
delete2.deleteColumns(Bytes.toBytes("colfam2"), Bytes.toBytes("qual3"));
table.delete(delete2);
boolean res2 = table
               .checkAndDelete(Bytes.toBytes("row1"),
                               Bytes.toBytes("colfam2"), Bytes.toBytes("qual3"), null,
                               delete1);
System.out.println("Delete successful:" + res2);
Delete delete3 = new Delete(Bytes.toBytes("row2"));
delete3.deleteFamily(Bytes.toBytes("colfam1"));
try
{
    boolean res4 = table
                   .checkAndDelete(Bytes.toBytes("row1"),
                                   Bytes.toBytes("colfam1"), Bytes.toBytes("qual1"), Bytes.toBytes("val1"),
                                   delete3);

}
catch (Exception e)
{
    e.printStackTrace();
}
```


## 参考文献
HBase权威指南

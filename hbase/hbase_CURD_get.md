# HBase CURD之Get

下面我们将介绍从客户端API中获取已存数据的方法。HTable类中提供了get()方法，同时还有与之对应的Get类。get方法分为两类：一类是一次获取一行数据；另一类是一次获取多行数据。

## 单行get

下面是示例代码：
```java
@Test
public void testGet() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    Get get = new Get(Bytes.toBytes("row93"));
    Result r = table.get(get);
    List<Cell> cells = r.listCells();
    for (Cell c : cells){
        String rowKey = Bytes.toString(CellUtil.cloneRow(c));
        String family = Bytes.toString(CellUtil.cloneFamily(c));
        String column = Bytes.toString(CellUtil.cloneQualifier(c));
        String value = Bytes.toString(CellUtil.cloneValue(c));

        System.out.println(rowKey + "," + family + "," + column + "," + value);
    }
}
```
从上面代码可以看出，数据的查询是通过Table的get方法获取的，get方法对应着Get对象。下面看一下Get类的构造函数。

### Get构造函数

```java
public Get(byte [] row) // 指定rowKey
public Get(Get get)     // 从其他Get对象创建实例
```
**Tip**
> 虽然一次get操作只能读取一行数据，但不会限制一行当中取多少列或者多少单元格。

与put操作一样，用户有许多方法可用，可以通过多种标准筛选目标数据，也可以指定精确的坐标获取某个单元格的数据：
```java
Get addFamily(byte [] family)                   // 限制get请求只能取得一个制定的列族，要取得多个列族时需要多次调用。
Get addColumn(byte [] family,byte [] qualifier) // 用户通过他可以指定get取得那一列的数据，从而进一步缩小地址空间。
Get setTimeRange(long minStamp,long maxStamp)   // 设置只能在制定的时间戳范围获得列版本。
Get setTimeStamp(long timestamp)                // 获得带有制定时间戳的列的版本。
Get setMaxVersions()                            // 设置最大版本数，设置为最大版本数为Integer的最大值。
Get setMaxVersions(int maxVersions)
```
addFamily()方法限制get请求只能取得一个指定的列族，要取得多次调用。addColumn()方法的使用与之类似，用户通过它可以指定get取得哪一列的数据，从而进一步缩小地址空间。有一些方法允许用户设定要获取的数据的时间戳，或者通过设定一个时间段来取得时间戳属于该时间段内的数据。

最后，如果用户没有设定时间戳的话，也有方法允许用户设定要获取的数据的版本数目。默认情况下，版本数为1，即get方法请求返回最新的匹配版本。如果有疑问，可以使用getMaxVersions()来检查这个Get实例所要取出的最大版本数。不带参数的setMaxVersions()方法会把要取出的最大版本数设为Integer.MAX_VALUE，这是用户在单元格中所有的版本，换句话说，此时系统会返回用户在列族中已配置的最大版本数以内的数据。

### Get类中的其他常用方法

|               方法                |                                                                                                          描述                                                                                                           |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| getRow()                          | 返回创建Get实例时指定的行键                                                                                                                                                                                             |
| getTimeRange()                    | 返回指定的Get实例的时间戳范围                                                                                                                                                                                           |
| setFilter()/getFilter()           | 用户可以使用该一个特定的过滤器实例，通过多种规则和条件来筛选列和单元格。                                                                                                                                                |
| setCacheBlocks()/getCacheBlocks() | 每个HBase的region服务器都有一个块缓存来有效地保存最近存取过的数据，并以此来加速之后的相邻信息的读取。不过在某些情况下，例如完全随机读取时，最好能避免这种机制带动的干扰。这些方法能够控制档次读取的块缓存机制是否起效。 |
| numFamilies()                     | 快捷地获取列族FamilyMap大小的方法，包括用addFamily()方法和addColumn()方法添加的列族。                                                                                                                                   |
| hasFamilies()                     | 检查列族或列是否存在于当前的Get实例中                                                                                                                                                                                   |
| familySet()/getFamilyMap()        | 这些方法能够让用户直接访问addFamily()和addColumn()添加的列族和列。FamilyMap列族中键是列族的名称，键对应的值是指定列族的限定符列表。familySet()方法返回一个所有已存储列族的Set，即一个只包含列族名的集合。               |

## Result类

当用户使用get()方法获取数据时，HBase返回的结果包含所有匹配的单元格数据，这些数据将被封装在一个Result实例中返回给用户。用它提供的方法，可以从服务器端获取匹配指定行的特定返回值，这些值包括列族、列限定符和时间戳等。常用方法如下：
```java
byte[] getValue(byte[] family,byte[] qualifier) // 方法允许用户取得一个HBase中存储的特定单元格的值。因为该方法不能指定时间戳，所以用户只能获取数据最新的版本。
byte [] value()                                 // 返回第一个列对应的最新单元格的值。
byte [] getRow()                                // 返回创建Get对象当前实例中的rowKey
int size()                                      // 返回Cell实例的数目
boolean isEmpty()                               // 查看键值对的数目是否大于0
Cell[] rawCells()                               // 返回的数组已经按照字典序排列；排序规则是先按列族排序，列族内再按列限定符排序，此后再按时间戳排序，最后按类型排序。
List<Cell> listCells()                          // 返回所有Cell对象的集合
List<Cell> getColumnCells(byte [] family, byte [] qualifier)  // 返回特定列的单元格
Cell getColumnLatestCell(byte [] family, byte [] qualifier)   // 返回最新版本的Cell对象
boolean containsColumn(byte [] family, byte [] qualifier)     // 检查指定列的值是否存在
```
**字典排序规则**
> 排序规则是先按列族排序，列族内再按列限定符排序，此后再按时间戳排序，最后按类型排序。

## Get列表

使用列表参数的get()方法与使用列表参数的put方法对应，用户可以用一次请求获取多行数据。它允许用户快速高效的从远程服务器获取相关的或完全随机的多行数据。

```java
@Test
public void testGet2() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    List<Get> list = new ArrayList<Get>();
    Get get1 = new Get(Bytes.toBytes("row93"));
    Get get2 = new Get(Bytes.toBytes("row94"));
    Get get3 = new Get(Bytes.toBytes("row95"));
    list.add(get1);
    list.add(get2);
    list.add(get3);
    Result[] results = table.get(list);
    for (Result r : results){
        List<Cell> cells = r.listCells();
        for (Cell c : cells){
            String rowKey = Bytes.toString(CellUtil.cloneRow(c));
            String family = Bytes.toString(CellUtil.cloneFamily(c));
            String column = Bytes.toString(CellUtil.cloneQualifier(c));
            String value = Bytes.toString(CellUtil.cloneValue(c));

            System.out.println(rowKey + "," + family + "," + column + "," + value);
        }
    }
}

@Test
public void testGet3() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    List<Get> list = new ArrayList<Get>();
    Get get1 = new Get(Bytes.toBytes("row93"));
    Get get2 = new Get(Bytes.toBytes("row94"));
    Get get3 = new Get(Bytes.toBytes("row95"));
    list.add(get1);
    list.add(get2);
    list.add(get3);
    Result[] results = table.get(list);
    for (Result r : results){
        List<Cell> cellsId = r.getColumnCells(Bytes.toBytes("f1"),Bytes.toBytes("id"));
        for (Cell cid: cellsId) {
            int id = Bytes.toInt(CellUtil.cloneValue(cid));
            System.out.println(id);
        }
        List<Cell> cellsAges = r.getColumnCells(Bytes.toBytes("f1"),Bytes.toBytes("age"));
        for (Cell cage: cellsAges) {
            int age = Bytes.toInt(CellUtil.cloneValue(cage));
            System.out.println(age);
        }
        List<Cell> cellsNames = r.getColumnCells(Bytes.toBytes("f1"),Bytes.toBytes("name"));
        for (Cell cname : cellsNames) {
            String name = Bytes.toString(CellUtil.cloneValue(cname));
            System.out.println(name);
        }
    }
}


@Test
public void testGet4() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    List<Get> list = new ArrayList<Get>();
    Get get1 = new Get(Bytes.toBytes("row93"));
    Get get2 = new Get(Bytes.toBytes("row94"));
    Get get3 = new Get(Bytes.toBytes("row95"));
    list.add(get1);
    list.add(get2);
    list.add(get3);
    Result[] results = table.get(list);
    for (Result r : results){
        Cell cellsId = r.getColumnLatestCell(Bytes.toBytes("f1"),Bytes.toBytes("id"));
        int id = Bytes.toInt(CellUtil.cloneValue(cellsId));
        System.out.println(id);

        Cell cellsAge = r.getColumnLatestCell(Bytes.toBytes("f1"),Bytes.toBytes("age"));
        int age = Bytes.toInt(CellUtil.cloneValue(cellsAge));
        System.out.println(age);

        Cell cellsName = r.getColumnLatestCell(Bytes.toBytes("f1"),Bytes.toBytes("name"));
        String name = Bytes.toString(CellUtil.cloneValue(cellsName));
        System.out.println(name);

    }
}
```

## 获取数据的其他方法

```java
boolean exists(Get get)             // 测试表中的列是否存在。如果存在返回true，这是一个服务器端调用，防止任何数据转移到客户端。
boolean[] existsAll(List<Get> gets) // 与上面方法功能类似，只不过返回的是boolean数组
```
**Tip**
> exists()和existsAll()方法会引发regionServer端查询数据的操作，包括加载文件块来检查某行或某列是否存在。用户通过这种方法只能避免网络数据传输的开销，不过在需要检查或频繁检查一个比较大的列时，这种方法还是十分实用的。

## 参考文献

HBase权威指南

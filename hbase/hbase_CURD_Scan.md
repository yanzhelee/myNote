# HBase扫描操作Scan

## 1 介绍

扫描操作的使用和get()方法类似。同样，和其他函数类似，这里也提供了Scan类。但是由于扫描工作方式类似于迭代器，所以用户无需调用san()方法创建实例,只需要调用HTable的getScanner()方法，此方法才是返回真正的扫描器(scanner)实例的同时，用户也可以使用它迭代获取数据，Table中的可用的方法如下：
```java
ResultScanner getScanner(Scan scan)
ResultScanner getScanner(byte[] family)
ResultScanner getScanner(byte[] family, byte[] qualifier)
```
后两个为了方便用户，隐式地帮助用户创建一个Scan实例，逻辑中最后调用getScanner(Scan scan)方法。Scan类拥有以下构造器：
```java
public Scan()
public Scan(byte [] startRow)
public Scan(byte [] startRow, byte [] stopRow)
public Scan(byte [] startRow, Filter filter)
public Scan(Get get)
public Scan(Scan scan)
```
用户可以选择性的提供startRow参数，来定义扫描读取HBase表的起始行键，即行键不是必须指定的。同时可选stopRow参数来限定读取到何处停止。

**其实行包括在内，而终止行不包含在内。一般区间表示为`[startRow,stopRow)`**

扫描操作有一个特点：用户提供的参数不必精确匹配两行。扫描会匹配相等或者大于给定的起始行的行键。如果没有显示地指定起始行，它会从表的起始位置开始获取数据。

当遇到了与设置的终止行相同或者大于终止行的行键时，扫描也会终止。如果没有指定终止键，会扫描到表尾。

另一个可选的参数叫做过滤器(filter),可直接指向Filter实例。尽管Scan实例通常由空白构造器构造，但其所有可选参数都有对应的getter方法和setter方法。

创建Scan实例后，用户可能还要给它增加更多限制条件。这种情况下，用户仍然可以使用空白 参数的扫描，它可以读取整个表格，包括所有列族以及它们的所有列。可以用多种方法限制要读取的数据：
```java
public Scan addFamily(byte [] family)                     // 方法限制返回数据的列族
public Scan addColumn(byte [] family, byte [] qualifier)  // 方法限制返回的列
Scan setTimeRange(long minStamp,long maxStamp)            // 设置时间范围
Scan setTimeStamp(long timestamp)                         // 设置时间戳
Scan setMaxVersions()                                     // 设置最大版本数
Scan setMaxVersions(int maxVersions)                      // 设置最大版本数
```

## 2 示例代码

```java
@Test
public void testScan() throws IOException {
    Connection conn = ConnectionFactory.createConnection();
    Table table = conn.getTable(TableName.valueOf("ns1:t1"));
    Scan scan = new Scan(Bytes.toBytes("row"),Bytes.toBytes("row010"));
    scan.addFamily(Bytes.toBytes("f1"));
    ResultScanner scanner = table.getScanner(scan)
    Iterator<Result> results = scanner.iterator();
    while (results.hasNext()){
        Result r = results.next();
        String rowId = Bytes.toString(r.getRow())
        Cell cId = r.getColumnLatestCell(Bytes.toBytes("f1"),Bytes.toBytes("id"));
        Cell cName = r.getColumnLatestCell(Bytes.toBytes("f1"),Bytes.toBytes("name"));
        Cell cAge = r.getColumnLatestCell(Bytes.toBytes("f1"),Bytes.toBytes("age"))
        int id = Bytes.toInt(CellUtil.cloneValue(cId));
        String name = Bytes.toString(CellUtil.cloneValue(cName));
        int age = Bytes.toInt(CellUtil.cloneValue(cAge))
        System.out.println("-----------------------------");
        System.out.println(rowId + "," + id + "," + age + "," + name);

    }
    scanner.close();
    table.close();
    conn.close();
}
```

上述代码显示结果如下：
```
-----------------------------
row000,0,0,tom0
-----------------------------
row001,1,1,tom1
-----------------------------
row002,2,2,tom2
-----------------------------
row003,3,3,tom3
-----------------------------
row004,4,4,tom4
-----------------------------
row005,5,5,tom5
-----------------------------
row006,6,6,tom6
-----------------------------
row007,7,7,tom7
-----------------------------
row008,8,8,tom8
-----------------------------
row009,9,9,tom9
```

## 3 ResultScanner类

扫描操作不会通过一个RPC请求返回所有匹配的行，而是以行为单位进行返回。很明显，行的数目很大，可能有上千条甚至更多，同时在一次请求中发送大量数据，会占用大量的系统资源并消耗很长时间。

ResultScanner类把扫描操作转换为类似的get操作，它将每一行数据封装成一个Result实例，并将所有的Result实例放入一个迭代器中。ResultScanner的一些方法如下：
```java
Result next()
Result[] next(int nbRows)
void close()
```

### 3.1 扫描器租约

要确保尽早释放扫描器对象，一个打开的扫描器会占用不少的服务端资源，累计多了会占用大量的堆空间。当使用完ResultScanner之后调用它的close()方法，同时当把close()方法放到try/finally块中，以保证其在迭代获取数据过程中出现异常和错误时，仍然能执行close()。


## 4 设置扫描器缓存

每一个next()调用都会为每一行数据生成一个单独的RPC请求，即使使用next(int nbRows)方法也是如此，因为该方法仅仅是在客户端循环地调用next()方法。很显然，当单元格数据较少时，这样做的性能不会很好。因此，如果一次RPC请求可以获取多行数据，这样更有意义。这样的方法可以由扫描器的缓存实现，默认情况下，这个缓存是关闭的。

Scan类中提供了设置缓存的方法如下：
```java
public Scan setCacheBlocks(boolean cacheBlocks) // 设置是否应用缓存块来进行扫描
public boolean getCacheBlocks()                 // 查看是否支持块缓存
public Scan setCaching(int caching)             // 设置扫描器的缓存行数
public int getCaching()                         // 获取扫描器中的缓存行数
```
用户需要少量的RPC请求次数和客户端以及服务器的内存消耗找到平衡点。很多时候，设置扫描器缓存可以提高性能，不过设置的太高就会产生不良的影响：每次调用next()将会占用更长的时间，因为要获取更多的文件并传输到客户端，如果返回给客户端的数据超出了其堆的大小，程序就会终止并抛出OutOfMemoryException异常。

**Tip**
> 当传输和处理数据的时间超过配置的扫描器租约时间时，用户将会收到一个ScannerTimeoutException形式抛出的租约过期错误。

下面是代码示例：

```java
/**
 * 添加扫描
 */
@Test
public void testScanCacheBatch() throws Exception {
	//
	Configuration conf = HBaseConfiguration.create();
	Connection conn = ConnectionFactory.createConnection(conf);
	HTable table = (HTable) conn.getTable(TableName.valueOf("ns1:t2"));
	Scan scan = new Scan();
	System.out.println(scan.getBatch());
	//三行
	scan.setCaching(3) ;
	//2列
	scan.setBatch(2) ;
		ResultScanner scanner = table.getScanner(scan);
		Iterator<Result> it = scanner.iterator();
		while (it.hasNext()) {
			Result r = it.next();
			outResult(r);
		}
		scanner.close();
}
private void outResult(Result r){
	System.out.println("=========================");
	List<Cell> cells = r.listCells();
	for(Cell cell : cells){
		String rowkey = Bytes.toString(CellUtil.cloneRow(cell));
		String f = Bytes.toString(CellUtil.cloneFamily(cell));
		String col = Bytes.toString(CellUtil.cloneQualifier(cell));
		long ts = cell.getTimestamp();
		String value = Bytes.toString(CellUtil.cloneValue(cell));
		System.out.println(rowkey+"/"+f+":"+col+"/"+ts + "=" + value);
	}
}
```

### 4.1 chche

在默认情况下，如果你需要从hbase中查询数据，在获取结果ResultScanner时，hbase会在你每次调用ResultScanner.next（）操作时对返回的每个Row执行一次RPC操作。即使你使用ResultScanner.next(int nbRows)时也只是在客户端循环调用RsultScanner.next()操作，你可以理解为hbase将执行查询请求以迭代器的模式设计，在执行next（）操作时才会真正的执行查询操作，而对每个Row都会执行一次RPC操作。

因此显而易见的就会想如果我对多个Row返回查询结果才执行一次RPC调用，那么就会减少实际的通讯开销。这个就是hbase配置属性“hbase.client.scanner.caching”的由来，设置cache可以在hbase配置文件中显示静态的配置，也可以在程序动态的设置。

cache值得设置并不是越大越好，需要做一个平衡。cache的值越大，则查询的性能就越高，但是与此同时，每一次调用next（）操作都需要花费更长的时间，因为获取的数据更多并且数据量大了传输到客户端需要的时间就越长，一旦你超过了maximum heap the client process 拥有的值，就会报outofmemoryException异常。当传输rows数据到客户端的时候，如果花费时间过长，则会抛出ScannerTimeOutException异常。

### 4.2 batch

在cache的情况下，我们一般讨论的是相对比较小的row，那么如果一个Row特别大的时候应该怎么处理呢？要知道cache的值增加，那么在client process 占用的内存就会随着row的增大而增大。在hbase中同样为解决这种情况提供了类似的操作：Batch。可以这么理解，cache是面向行的优化处理，batch是面向列的优化处理。它用来控制每次调用next（）操作时会返回多少列，比如你设置setBatch（5），那么每一个Result实例就会返回5列，如果你的列数为17的话，那么就会获得四个Result实例，分别含有5,5,5,2个列。

下面会以表格的形式来帮助理解，假设我们拥有10Row，每个row拥有2个family，每个family拥有10个列。（也就是说每个Row含有20列）

| 缓存 | 批量处理 | Result个数 | RPC次数 |                                                                 说明                                                                 |
| ---- | -------- | ---------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | 1        | 200        | 201     | 每个列都作为一个Result实例返回。最后还多一个RPC确认扫描完成                                                                          |
| 200  | 1        | 200        | 2       | 每个Result实例都只包含一列的值，不过它们都被一次RPC请求取回                                                                          |
| 2    | 10       | 20         | 11      | 批量参数是一行所包含的列数的一半，所以200列除以10，需要20个result实例。同时需要10次RPC请求取回。                                     |
| 5    | 100      | 10         | 3       | 对一行来讲，这个批量参数实在是太大了，所以一行的20列都被放入到了一个Result实例中。同时缓存为5，所以10个Result实例被两次RPC请求取回。 |
| 5    | 20       | 10         | 3       | 同上，不过这次的批量值与一行列数正好相同，所以输出与上面一种情况相同                                                                 |
| 10   | 10       | 20         | 3       | 这次把表分成了较小的result实例，但使用了较大的缓存值，所以也是只用了两次RPC请求就返回了数据                                          |


要计算一次扫描操作的RPC请求的次数，用户需要先计算出行数和每行列数的乘积。然后用这个值除以批量大小和每行列数中较小的那个值。最后再用除得的结果除以扫描器缓存值。 用数学公式表示如下:
```
RPC请求的次数=(行数x每行的列数)/Min(每行的列数，批量大小)/扫描器缓存
```

## 参考文献

HBase权威指南

[http://www.cnblogs.com/editice/archive/2013/04/22/3035728.html](http://www.cnblogs.com/editice/archive/2013/04/22/3035728.html)

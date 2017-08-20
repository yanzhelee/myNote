# Hadoop压缩与解压缩

## 压缩简介

Hadoop作为一个通用的海量数据处理平台，每次运算都会需要处理大量数据，我们会在hadoop系统中对数据进行压缩处理来优化磁盘使用率，提高数据在磁盘个网络中的传输速度，从而提高系统处理数据的效率。在使用压缩方式方面主要考虑压缩速度和压缩文件的可分割性。

## 压缩格式

Hadoop对于压缩格式是自动识别的。如果我们压缩的文件有相应的压缩格式的扩展名，hadoop会根据扩展名自动选择相对应的解码器来解压数据，此过程完全是hadoop自动处理，我们只需要确保输入的压缩文件有扩展名。如果压缩文件没有扩展名，则需要在执行MapReduce任务的时候指定输入格式。

压缩格式 | 工具 | 算法 | 扩展名 | 多文件 | 是否支持切割
--------|------|-----|-------|---------|----
DEFLATE | 无 | DEFLATE | .deflate | No | No
gzip | gzip | DEFLATE | .gz | No | No
ZIP | zip | DEFLATE | .zip | YES | YES在文件范围内
bzip2 | bzip2 | bzip2 | .bz2 | No | YES
LZO | lzop | LZO | .lzo | No | YES

## 性能对比

Hadoop下各种压缩算法的压缩比，压缩时间，解压缩时间如下表（下面数据来自IBM）：

压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度
--------|------------|-------------|--------|---------
gzip | 8.3GB | 1.8GB | 17.5MB/s | 58MB/s
bzip2 | 8.3GB | 1.1GB | 2.4MB/s | 9.5MB/s
LZO-best | 8.3GB | 2GB | 4MB/s | 60.6MB/s
LZO | 8.3GB | 2.9GB | 49.3MB/s | 74.6MB/s

因此我们可以得出：

1. Bip2压缩效果明显最好，但是Bzip2压缩速度慢，可分割。
2. Gzip压缩效果不如Bzip2，但是压缩速度比较快，不支持分割。
3. LZO压缩效果不如Bzip2和Gzip，但是压缩速度最快，并且支持分割。

为了支持多种压缩解压缩算法，Hadoop引入了编码/解码器。与Hadoop序列化框架类似，编码/解码器也是使用抽象工厂的设计模式。目前，Hadoop支持的编码/解码器如下表。

压缩格式 | 对应的编码/解码器
--------|-----------------
DEFLATE | org.apache.hadoop.io.compress.DefaultCodec
gzip | org.apache.hadoop.io.compress.GzipCodec
bzip | org.apache.hadoop.io.compress.BZip2Codec
Snappy | org.apache.hadoop.io.compress.SnappyCodec

同一个压缩方法对应的压缩、解压缩相关的工具，都可以在相应的编码/解码器中获得。

## 使用方式

MapReduce可以在三个阶段中使用压缩。

1. 输入压缩文件，如果输入的文件是压缩过的，那么在被MapReduce读取时，它们会被自动解压。
2. MapReduce作业中，对Map输出的中间结果集压缩，实现方式如下：
	1. 可以在core-site.xml文件中配置，代码如下：
		```
	    <property>
			<name>mapred.compress.map.output</name>
			<value>true</value>
		</property>
		```
	2. 使用java代码指定
		```java
		conf.setCompressMapOut(true);
		//这行代码指定Map输出结果的编码器
		conf.setMapOutputCompressorClass(GzipCode.class);
		```
3. MapReduce作业中，对Reduce输出的最终结果集压缩，实现方式如下：
	1. 可以在core-site.xml文件中配置，代码如下：
		```
	    <property>
			<name>mapred.compress.map.output</name>
			<value>true</value>
		</property>
		```
	2. 使用java代码指定
		```java
		conf.setBoolean(“mapred.output.compress”，true);
		//这行代码指定Reduce输出结果的编码器
		conf.setClass(“mapred.output.compression.codec”,GzipCode.class,CompressionCodec.class);
		```

## 压缩解压缩框架

我们前面已经提到过关于压缩的使用方式，其中第一种就是将压缩文件直接作为入口参数交给MapReduce处理，MapReduce会自动根据压缩文件的扩展名来自动选择合适解压器处理数据，那么到底是怎么实现的呢？如下图所示：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_%E5%8E%8B%E7%BC%A9%E5%92%8C%E8%A7%A3%E5%8E%8B%E7%BC%A9_%E5%8E%8B%E7%BC%A9%E5%AE%9E%E7%8E%B0%E6%83%85%E5%BD%A2.png)

我们在配置Job作业的时候，会设置数据输入的格式化方式，使用`conf.setInputFormat()`方法，这里的入口参数是`TextInputFormat.class`。

`TextInputFormat.class`继承于`InputFormat.class`主要用于对数据进行两方面的预处理，一是对输入数据进行切分，生成一组split，一个split会分发给一个Mapper进行处理；二是针对每个split再创建一个RecordReader读取split内的数据，并且按照<key,value>的形式组织成一条record传给map函数进行处理。此类在对数据进行切分之前，会首先初始化压缩解压缩工程类`CompressionCodeFactory.class`,通过工厂获取实例化的编解码器`CompressionCodec`后对数据处理操作。

下图是压缩与解压缩类框架结构图

![压缩解压缩类图](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_%E5%8E%8B%E7%BC%A9%E5%92%8C%E8%A7%A3%E5%8E%8B%E7%BC%A9_%E6%A1%86%E6%9E%B6.png)


### 编码/解码器

----------

CompressionCodec接口实现可编码/解码器，使用的是抽象工厂的设计模式。CompressionCodec提供一系列的方法，用于创建特定压缩算法的相关设置。CompressionCodec中的方法很对称，一个压缩功能总是对应一个解压缩功能。其中与压缩有关的方法如下：

```java
//根据底层输出流，创建对应压缩算法的压缩流
createOutputStream(OutputStream out)
//可使用压缩器创建压缩流
createOutputStream(OutputStream out, Compressor compressor)

//方法用于创建压缩算法对应的压缩器
createCompressor()

//根据底层的输入流，创建对应的解压缩算法的解压缩流
createInputStream(InputStream in)
//可使用解压缩器创建解压缩流
createInputStream(InputStream in,Decompressor decompressor)
//方法用于创建解压缩器
createDecompressor()
```

源代码如下：

```java
public interface CompressionCodec {

  // 在底层输出流 out 的基础上创建对应压缩算法的压缩流 CompressionOutputStream 对象
  CompressionOutputStream createOutputStream(OutputStream out)
  throws IOException;

  // 使用压缩器 compressor， 在底层输出流 out 的基础上创建对应的压缩流
  CompressionOutputStream createOutputStream(OutputStream out,
                                             Compressor compressor)
  throws IOException;

  //得到压缩器类型
  Class<? extends Compressor> getCompressorType();

  // 创建压缩算法对应的压缩器
  Compressor createCompressor();

  //根据底层的输入流，创建对应的解压缩算法的解压缩流
  CompressionInputStream createInputStream(InputStream in) throws IOException;

  //可使用解压缩器创建解压缩流
  CompressionInputStream createInputStream(InputStream in,
                                           Decompressor decompressor)
  throws IOException;


  //获取压缩类型
  Class<? extends Decompressor> getDecompressorType();

  //创建解压缩器
  Decompressor createDecompressor();

  // 获得压缩算法对应的文件扩展名
  String getDefaultExtension();

  static class Util {
    /**
     * Create an output stream with a codec taken from the global CodecPool.
     *
     * @param codec       The codec to use to create the output stream.
     * @param conf        The configuration to use if we need to create a new codec.
     * @param out         The output stream to wrap.
     * @return            The new output stream
     * @throws IOException
     */
    static CompressionOutputStream createOutputStreamWithCodecPool(
        CompressionCodec codec, Configuration conf, OutputStream out)
        throws IOException {
      Compressor compressor = CodecPool.getCompressor(codec, conf);
      CompressionOutputStream stream = null;
      try {
        stream = codec.createOutputStream(out, compressor);
      } finally {
        if (stream == null) {
          CodecPool.returnCompressor(compressor);
        } else {
          stream.setTrackedCompressor(compressor);
        }
      }
      return stream;
    }

    /**
     * Create an input stream with a codec taken from the global CodecPool.
     *
     * @param codec       The codec to use to create the input stream.
     * @param conf        The configuration to use if we need to create a new codec.
     * @param in          The input stream to wrap.
     * @return            The new input stream
     * @throws IOException
     */
    static CompressionInputStream createInputStreamWithCodecPool(
        CompressionCodec codec,  Configuration conf, InputStream in)
          throws IOException {
      Decompressor decompressor = CodecPool.getDecompressor(codec);
      CompressionInputStream stream = null;
      try {
        stream = codec.createInputStream(in, decompressor);
      } finally {
        if (stream == null) {
          CodecPool.returnDecompressor(decompressor);
        } else {
          stream.setTrackedDecompressor(decompressor);
        }
      }
      return stream;
    }
  }
}

```

#### 使用CompressionCodec进行压缩解压缩

压缩：通过createOutputStream(OutputStream out)方法获得CompressionOutputStream对象
解压：通过createInputStream(InputStream in)方法获得CompressionInputStream对象

压缩解压缩实例代码如下：

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.compress.*;
import org.apache.hadoop.util.ReflectionUtils;
import org.junit.Test;

import java.io.*;
import java.util.ArrayList;
import java.util.Collections;


public class Codec {

    public static void main(String[] args)throws Exception{
		//编解码器数组
        Class[] classes = new Class[]{
                DefaultCodec.class,
                DeflateCodec.class,
                GzipCodec.class,
                BZip2Codec.class,
                Lz4Codec.class
        };

        ArrayList<CodecFile> codecFiles = new ArrayList<CodecFile>();
        for (Class clazz : classes){
            codecFiles.add(compress(clazz));
            deCompress(clazz);

        }
        Collections.sort(codecFiles);
        for (CodecFile cf : codecFiles){
            System.out.println(cf.toString());
        }
    }

	//压缩函数，根据对应的编解码器类进行压缩
    private CodecFile compress(Class clazz) throws Exception {
        long start = System.currentTimeMillis();
        Configuration conf = new Configuration();

        //创建压缩编解码器
        CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(clazz, conf);

        //压缩器
        Compressor cpr = codec.createCompressor();

        //得到压缩编解码器的扩展名
        String ext = codec.getDefaultExtension();

        File f = new File("g:java/codec/1"+ext);
        FileOutputStream fos = new FileOutputStream(f) ;

        //压缩输出流
        CompressionOutputStream cout = codec.createOutputStream(fos, cpr);

        FileInputStream fis = new FileInputStream("g:java/codec/cat.jpg");

        byte[] buf = new byte[1024] ;

        int len = -1;
        while ((len = fis.read(buf)) != -1){
            cout.write(buf,0,len);
        }
        fis.close();
        cout.close();
        fos.close();

        //花费时间
        long dur = System.currentTimeMillis() -start ;

        //文件长度
        long fileLen = f.length();

        String simpleName = clazz.getSimpleName();
        //System.out.println(simpleName + "\t:dur=" + dur + "\tfileLen=" +fileLen);
        return new CodecFile(simpleName, fileLen, dur) ;

    }

    //解压缩函数，根据对应的编解码器类进行解压缩
    private void deCompress(Class clazz) throws Exception {
        long start = System.currentTimeMillis();
        Configuration conf = new Configuration();

        //创建压缩编解码器
        CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(clazz, conf);

        //解压缩器
        Decompressor dcp = codec.createDecompressor();
        //获取文件扩展名
        String ext = codec.getDefaultExtension();
        String file = "g:/java/codec/1" + ext;
        FileInputStream fis = new FileInputStream(file) ;

        CompressionInputStream cis = codec.createInputStream(fis, dcp);
        FileOutputStream fos = new FileOutputStream("g:/java/codec/de" + ext + ".jpg");
        byte[] buf = new byte[1024] ;
        int len = -1 ;
        while ((len= cis.read(buf)) != -1){
            fos.write(buf, 0, len);
        }

        fos.close();
        fis.close();


    }

	//用于存储压缩文件信息，可用于比较不同压缩算法的压缩速度和压缩大小
    private class CodecFile implements Comparable<CodecFile>{
        String codecType ;
        private long fileLen ;
        private long dur ;

        public String getCodecType() {
            return codecType;
        }

        public void setCodecType(String codecType) {
            this.codecType = codecType;
        }

        public long getFileLen() {
            return fileLen;
        }

        public void setFileLen(long fileLen) {
            this.fileLen = fileLen;
        }

        public long getDur() {
            return dur;
        }

        public void setDur(long dur) {
            this.dur = dur;
        }

        public CodecFile(String codecType, long fileLen, long dur) {
            this.codecType = codecType;
            this.fileLen = fileLen;
            this.dur = dur;
        }

        @Override
        public String toString() {
            return codecType + "\t:dur=" + dur + "\tfileLen=" + fileLen;
        }

        public int compareTo(CodecFile o) {
            return (this.getDur() - o.getDur() < 0) ? -1 : (this.getDur() == o.getDur()) ? 0 : 1 ;
        }
    }
}
```

### 压缩解压缩工厂类CompressionCodecFactory

----------

压缩解压缩工厂类CompressionCodecFactory.class主要功能就是负责根据不同的文件扩展名来自动获取相对应的压缩解压缩器CompressionCodec.class,是整个压缩框架的核心控制器。

#### CompressionCodecFactory源码分析

```java
public class CompressionCodecFactory {

  /**
   * 所有的解压缩编码类放入 codecs Map图中，CompressionCodec是一个基类，
   * 允许添加上其所继承的子类
   */
  private SortedMap<String, CompressionCodec> codecs = null;

  /**
   * 初始化的时候，可以根据配置加入自己希望的压缩算法种类
   * 根据配置初始化压缩编码工厂，默认添加的是gzip和zip编码类
   */
  public CompressionCodecFactory(Configuration conf) {
    codecs = new TreeMap<String, CompressionCodec>();
    List<Class<? extends CompressionCodec>> codecClasses = getCodecClasses(conf);
    if (codecClasses == null) {
      //如果没有编码类的设置，则加入gzip,defaultCode
      addCodec(new GzipCodec());
      addCodec(new DefaultCodec());
    } else {
      Iterator<Class<? extends CompressionCodec>> itr = codecClasses.iterator();
      while (itr.hasNext()) {
        CompressionCodec codec = ReflectionUtils.newInstance(itr.next(), conf);
        addCodec(codec);
      }
    }
  }

	/**
	 * 通过名字获取，其实这种模式类似于享受模式，达到对象的复用效果了。
	 */
	public CompressionCodec getCodec(Path file) {
	  CompressionCodec result = null;
	  if (codecs != null) {
	    String filename = file.getName();
	    String reversedFilename = new StringBuffer(filename).reverse().toString();
	    SortedMap<String, CompressionCodec> subMap =
	      codecs.headMap(reversedFilename);
	    if (!subMap.isEmpty()) {
	      String potentialSuffix = subMap.lastKey();
	      //根据比对名字，从codecs Map中取出对应的CompressionCodec
	      if (reversedFilename.startsWith(potentialSuffix)) {
	        result = codecs.get(potentialSuffix);
	      }
	    }
	  }
	  return result;
	}
```

上面的getCodec()方法，为某一个压缩文件寻找对应的CompressionCodec。为了分析该方法， 需要了解 CompressionCodec 类中保存文件扩展名和CompressionCodec 映射关系的成员变量 codecs。codecs 是一个有序映射表， 即它本身是一个Map， 同时它对 Map 的键排序， 下面是codecs中保存的一个可能的映射关系：

```java
{
2zb.: org.apache.hadoop.io.compress.BZip2Codec,
etalfed.: org.apache.hadoop.io.compress.DeflateCodec,
yppans.: org.apache.hadoop.io.compress.SnappyCodec,
zg.: org.apache.hadoop.io.compress.GzipCodec
}
```

可以看到， Map 中的键是排序的。getCodec() 方法的输入是 Path 对象， 保存着文件路径， 如实例中的“README.txt.bz2” 。首先通过获取 Path 对象对应的文件名并逆转该字符串得到“2zb.txt.EMDAER” ， 然后通过有序映射 SortedMap 的 headMap() 方法， 查找最接近上述逆转字符串的有序映射的部分视图， 如输入“2zb.txt.EMDAER”的查找结果subMap， 只包含“2zb.”对应的那个键 – 值对，如果输入是“zg.txt.EMDAER” ， 则 subMap 会包含成员变量 codecs 中保存的所有键 – 值对。然后， 简单地获取 subMap 最后一个元素的键， 如果该键是逆转文件名的前缀， 那么就找到了文件对应的编码/解码器， 否则返回空。

#### 使用CompressionCodecFactory解压缩

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.io.compress.CompressionCodecFactory;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.URI;

public class FileDecompressor {
    public static void main(String[] args) throws Exception {
        String uri = args[0];
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(uri), conf);

        Path inputPath = new Path(uri);
        CompressionCodecFactory factory = new CompressionCodecFactory(conf);
        CompressionCodec codec = factory.getCodec(inputPath);
        if (codec == null) {
            System.out.println("No codec found for " + uri);
            System.exit(1);
        }
        String outputUri = CompressionCodecFactory.removeSuffix(uri, codec.getDefaultExtension());

        InputStream in = null;
        OutputStream out = null;

        try {
            in = codec.createInputStream(fs.open(inputPath));
            out = fs.create(new Path(outputUri));
            IOUtils.copyBytes(in,out,conf);
        } finally {
            IOUtils.closeStream(in);
            IOUtils.closeStream(out);
        }
    }
}
```

### 压缩器和解压缩器

压缩器(Compressor)和解压缩器(Decompressor)是Hadoop压缩框架中一对比较重要的概念。Compressor可以插入压缩输出流的实现中，提供具体的压缩功能；相反Decompressor提供具体的解压功能并插入CompressionInputStream中。Compressor和Decompressor的这种设计，最初是在java的zlib压缩程序库中引入的，对应的实现分别是java.util.zip.Deflater和java.util.Inflater。

Compressor通过setInput()方法接收数据到内容缓冲区，自然可以多次调用setInput()方法，但内部缓冲区总是会被填满。如何判断压缩器内部缓冲区是否已满呢?可以通过needsInput()的返回值，如果是false，表明缓冲区已经满，这是必须通过compress()方法获取压缩后的数据，释放缓冲区空间。
为了提高压缩效率，并不是每次用户调用setInput()方法，压缩器就会立即执行，所以，为了通知压缩器所有数据已经写入，必须使用finish()方法。finish()调用结束后，压缩器缓冲区中保持的已经压缩的数据，可以继续通过compress()方法获得。至于要判断压缩器中是否还有未读取的压缩数据，则需要利用finished()方法来判断。

----------

**注意**:finish()和finished()的作用不同，finish()结束数据输入的过程，而finished()返回false，表明压缩器中还有未读取的压缩数据，可以继续通过compress()方法读取。

----------

压缩器(Compressor)和解压缩器(Decompressor)类图如下：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_%E5%8E%8B%E7%BC%A9%E5%92%8C%E8%A7%A3%E5%8E%8B%E7%BC%A9_%E5%8E%8B%E7%BC%A9%E8%A7%A3%E5%8E%8B%E7%BC%A9%E5%99%A8.png)

#### Compressor源码解析

- setInput() 方法接收数据到内部缓冲区，可以多次调用；
- needsInput() 方法用于检查缓冲区是否已满。如果是 false 则说明当前的缓冲区已满
- getBytesRead() 输入未压缩字节的总数；
- getBytesWritten() 输出压缩字节的总数；
- finish() 方法结束数据输入的过程；
- finished() 方法用于检查是否已经读取完所有的等待压缩的数据。如果返回 false，表明压缩器中还有未读取的压缩数据，可以继续通过 compress() 方法读取；
- compress() 方法获取压缩后的数据，释放缓冲区空间；
- reset() 方法用于重置压缩器，以处理新的输入数据集合；
- end() 方法用于关闭解压缩器并放弃所有未处理的输入；
- reinit() 方法更进一步允许使用 Hadoop 的配置系统，重置并重新配置压缩器；

### 压缩流解压缩流




## 参考博文

[https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-compression-analysis/#authorN10017](https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-compression-analysis/#authorN10017)

[http://blog.csdn.net/Androidlushangderen/article/details/41647029](http://blog.csdn.net/Androidlushangderen/article/details/41647029)

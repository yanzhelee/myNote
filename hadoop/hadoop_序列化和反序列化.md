# hadoop序列化和反序列化

## 1 什么是序列化和反序列化

序列化就是将内存中的对象或数据，转换成字节数组，以便于存储（持久化）和网络传输。
反序列化就是将字节数组转换成内存对象。

## 2 JDK中的序列化和反序列化

使用java提供的序列化必须遵循三个条件：
1. 该类必须实现java.io.Serializable接口。
2. 对于该类的所有无法序列化的字段必须使用transient修饰。
3. 加上序列化版本ID serialVersionUID，这个是用来识别序列化的之前的类到底是哪一个。比如希望类的不同版本对序列化兼容，需要确保类的不同版本具有相同的serialVersionUID

在使用JDK提供的序列化机制时需要借助一对I/O流，ObjectOutputStream和ObjectInputStream这两个流分别是进行序列化和反序列化操作，通过ObjectOutputStream类的`writeObject(Object obj)`方法可以将对象写入到输出流中，通过ObjectInputStream类的`readObject()`方法可以从该输入流中反序列化该对象出来。

JDK序列化算法一般会有如下步骤：
1. 将对象实例相关的类元数据输出；
2. 递归输出类的超类描述直到不再有超类；
3. 类元数据完了之后，开始从最顶层的超类开始输出对象实例的实际数据值；
4. 从上至下递归输出实例的数据

## 3 Hadoop序列化和反序列化

在hadoop中，hadoop实现了一套自己的序列化框架，相对于JDK比较简洁，在集群信息的传递上速度更快，容量更小。

### 3.1 Hadoop序列化的特点

1. 数据紧凑
	> 带宽是集群中信息传递的最宝贵的资源，所以我们必须设法缩小传递信息的大小。为了更好的控制序列化整个流程使用Writable对象，java序列化过程中会保存类的所有信息以及依赖等，Hadoop序列化不需要。
2. 对象可重用
	> JDK的反序列化会不断地创建对象，这肯定会造成一定的系统开销，但是在hadoop反序列化中，能重复的利用一个对象的readField方法来重新产生不同的对象。
3. 可扩展性
	> hadoop自己写序列化很容易，可以通过实现hadoop的Writable接口实现序列化，或者实现WritableComparable接口实现可比较大小的序列化对象。

## 4 Hadoop Writable框架

```java
@InterfaceAudience.Public
@InterfaceStability.Stable
public interface Writable {
  /** 
   * 序列化一个对象，将一个对象按照某个数据传输格式写入到out流中
   * Serialize the fields of this object to <code>out</code>.
   * 
   * @param out <code>DataOuput</code> to serialize this object into.
   * @throws IOException
   */
  void write(DataOutput out) throws IOException;

  /** 
   * 反序列化，从in流中读入字节，按照某个数据传输格式读出到一个对象中
   * Deserialize the fields of this object from <code>in</code>.  
   * 
   * <p>For efficiency, implementations should attempt to re-use storage in the 
   * existing object where possible.</p>
   * 
   * @param in <code>DataInput</code> to deseriablize this object from.
   * @throws IOException
   */
  void readFields(DataInput in) throws IOException;
}
```

## 5 代码示例

### 5.1 序列化反序列化工具函数

```java
/** 
 * 将一个实现了Writable接口的对象序列化成字节流 
 * @param writable 
 * @return 
 * @throws IOException 
 */ 
public static byte[] serialize2Bytes(Writable writable) throws IOException{
    ByteArrayOutputStream baos = new ByteArrayOutputStream() ;
    DataOutputStream dos = new DataOutputStream(baos);
    writable.write(dos);
    dos.close();
    baos.close();
    return baos.toByteArray();
}

 /** 
 * 将字节流转化为实现了Writable接口的对象 
 * @param writable 
 * @param bytes 
 * @return 
 * @throws IOException 
 */  
public static void deserializeFromBytes(Writable writable, byte[] bytes) throws IOException{
    ByteArrayInputStream bais = new ByteArrayInputStream(bytes) ;
    DataInputStream dataIn = new DataInputStream(bais) ;
    writable.readFields(dataIn) ;
    dataIn.close();
    bais.close();
}

/** 
 * 将一个实现了Writable接口的对象序列化到文件中
 * @param writable
 * @param file
 * @return 
 * @throws IOException 
 */ 
public static void serialize2File(Writable writable, File file) throws IOException{
    FileOutputStream fos = new FileOutputStream(file) ;
    DataOutputStream dos = new DataOutputStream(fos) ;
    writable.write(dos);
    dos.close();
    fos.close();
}

 /** 
 * 将文件输入流流转化为实现了Writable接口的对象 
 * @param writable 
 * @param file 
 * @return 
 * @throws IOException 
 */  
public static void deserializeFromBytes(Writable writable, File file) throws IOException{
    FileInputStream fis = new FileInputStream(file);
    DataInputStream dataIn = new DataInputStream(fis) ;
    writable.readFields(dataIn) ;
    dataIn.close();
    fis.close();
}
```

### 5.2 自定义类型示例（实现WritableComparable接口）

```java
package com.seriable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.WritableComparable;

public class PeopleWritable implements WritableComparable<PeopleWritable> {
    private IntWritable age;
    private Text name;
    public PeopleWritable(){
    }
    public PeopleWritable(IntWritable age, Text name) {
        super();
        this.age = age;
        this.name = name;
    }
    public IntWritable getAge() {
        return age;
    }
    public void setAge(IntWritable age) {
        this.age = age;
    }
    public Text getName() {
        return name;
    }
    public void setName(Text name) {
        this.name = name;
    }
    //序列化方法
    public void write(DataOutput out) throws IOException {
        age.write(out);
        name.write(out);
    }
    //反序列化方法
    public void readFields(DataInput in) throws IOException {
        age.readFields(in);
        name.readFields(in);
    }
    //比较函数，使得对象可比较大小
    public int compareTo(PeopleWritable o) {
        int cmp = age.compareTo(o.getAge());
        if(0 !=cmp)return cmp;
        return name.compareTo(o.getName());
    }
}
```

## 参考博文

[http://www.cnblogs.com/kxdblog/p/4799282.html](http://www.cnblogs.com/kxdblog/p/4799282.html)

[http://www.cnblogs.com/zl1991/p/6322361.html](http://www.cnblogs.com/zl1991/p/6322361.html)

[http://blog.csdn.net/zhang0558/article/details/53444533](http://blog.csdn.net/zhang0558/article/details/53444533)
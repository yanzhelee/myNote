# Scala隐式转换和隐式参数

在scala语言中，隐式转换是一项强大的语言功能，他不仅能够简化程序设计，也能够使程序具有很强的灵活性。要想更进一步地掌握scala语言，了解其隐式转换的作用和原理是很有必要的，否则很难得以应手的处理日常开发中的问题。

在scala语言中，隐式转换是无处不在的，只不过scala语言为我们隐藏了相应的细节，例如scala中的继承层次结构中：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/scala/scala_implicit_1_1.png)

它们存在固有的隐式转换，不需要人工进行干预，例如Float在必要的情况下自动转换为Double类型。

## 1 隐式转换函数

```scala
// 定义了一个隐式转换函数double2Int，将传入参数从Double类型转换为Int类型
implicit def double2Int(x:Double) = x.toInt
val x:Int = 3.5
// x = 3
```
隐式函数的名称对结构没有影响，即implicit def double2Int(x:Double)=x.toInt函数可以是任何名字，只不过采用source2Target这种方式函数的意思比较明确，阅读代码的人可以见名知义，增加代码的可读性。

隐式转换功能非常强大，可以快速扩展现有类库的功能，例如下面代码：
```scala
package cn.scala.xtwy

import java.io.File
import scala.io.Source
//RichFile类中定义了Read方法
class RichFile(val file:File){
  def read=Source.fromFile(file).getLines().mkString
}

object ImplicitFunction extends App{
  implicit def double2Int(x:Double)=x.toInt
  var x:Int=3.5
  //隐式函数将java.io.File隐式转换为RichFile类
  implicit def file2RichFile(file:File)=new RichFile(file)
  val f=new File("file.log").read
  println(f)
}
```

## 2 隐式转换规则

隐式转换可以定义在目标文件中，例如：
```scala
implicit def double2Int(x:Double)=x.toInt
var x:Int = 3.5
```
隐式转换函数与目标代码在同一文件中，也可以将隐式转换集中放置在某个包中，在使用的时候直接将该包引入即可，例如：
```scala
package cn.scala.xtwy
import java.io.File
import scala.io.Source
// 在cn.scala.xtwy包中定义了子包implicitConversion
// 然后在object ImplicitConversion中定义所有的隐式转换方法
package implicitConversion{
  object ImplicitConversion{
    implicit def double2Int(x:Double) = x.toInt
    implicit def file2RichFile(file:File) = new RichFile(file)
  }
  class RichFile(val file:File){
    def read=Source.fromFile(file).getLines().mkString
  }

  object ImplicitFunction extends App{
    //在使用时引入所有的隐式方法
    import cn.scala.xtwy.implicitConversion.ImplicitConversion._
    var x:Int=3.5


    val f=new File("file.log").read
    println(f)
}
```
这种方式在scala语言中比较常见，在前面我们也提到，scala会默认帮我们引用Predef对象中所有的方法，Predef中定义了很多隐式转换函数，下面是Predef的部分隐式转换源码：
```scala
scala> :implicits -v
/* 78 implicit members imported from scala.Predef */
  /* 48 inherited from scala.Predef */
  implicit def any2ArrowAssoc[A](x: A): ArrowAssoc[A]
  implicit def any2Ensuring[A](x: A): Ensuring[A]
  implicit def any2stringadd(x: Any): runtime.StringAdd
  implicit def any2stringfmt(x: Any): runtime.StringFormat
  implicit def boolean2BooleanConflict(x: Boolean): Object
  implicit def byte2ByteConflict(x: Byte): Object
  implicit def char2CharacterConflict(x: Char): Object
  implicit def double2DoubleConflict(x: Double): Object
  implicit def float2FloatConflict(x: Float): Object
  implicit def int2IntegerConflict(x: Int): Object
  implicit def long2LongConflict(x: Long): Object
  implicit def short2ShortConflict(x: Short): Object

 //....................
```
那么什么时候会发生隐式转换呢?主要有以下几种情况：

1 当方法中参数的类型与实际类型不一致时：
```scala
def f(x:Int)=x
//方法中输入的参数类型与实际类型不一致，此时会发生隐式转换
//double类型会转换为Int类型，再进行方法的执行
f(3.14)
```
2 当调用类中不存在的方法或成员时，会自动将对象进行隐式转换：
```scala
package cn.scala.xtwy

import java.io.File
import scala.io.Source
//RichFile类中定义了Read方法
class RichFile(val file:File){
  def read=Source.fromFile(file).getLines().mkString
}

object ImplicitFunction extends App{
  implicit def double2Int(x:Double)=x.toInt
  var x:Int=3.5
  //隐式函数将java.io.File隐式转换为RichFile类
  implicit def file2RichFile(file:File)=new RichFile(file)
  //File类的对象并不存在read方法，此时便会发生隐式转换
  //将File类转换成RichFile
  val f=new File("file.log").read
  println(f)
}
```

前面讲了什么情况下会发生隐式转换，下面我们讲一讲什么时候不会发生隐式转换：

1 编译器可以不在隐式转换情况下编译通过，则不进行隐式转换，例如：
```scala
// 这里定义隐式转换函数
scala> implicit def double2Int(x:Double) = x.toInt

// 下面几条语句，不需要自己定义隐式转换就可以编译通过
// 因此它们不会发生隐式转换
scala> 3.0*2
res0: Double = 6.0

scala> 2*3.0
res1: Double = 6.0

scala> 2*3.7
res2: Double = 7.4
```

2 如果转换存在二义性，则不会发生隐式转换，例如：
```scala
package implicitConversion{
  object ImplicitConversion{
    implicit def double2Int(x:Double)=x.toInt
    //这里定义了一个隐式转换
    implicit def file2RichFile(file:File)=new RichFile(file)
    //这里又定义了一个隐式转换，目的与前面那个相同
    implicit def file2RichFile2(file:File)=new RichFile(file)
  }

}

class RichFile(val file:File){
  def read=Source.fromFile(file).getLines().mkString
}

object ImplicitFunction extends App{
  import cn.scala.xtwy.implicitConversion.ImplicitConversion._
  var x:Int=3.5

  //下面这条语句在编译时会出错，提示信息如下：
  //type mismatch; found : java.io.File required:
  // ?{def read: ?} Note that implicit conversions
  //are not applicable because they are ambiguous:
  //both method file2RichFile in object
  //ImplicitConversion of type (file:
  //java.io.File)cn.scala.xtwy.RichFile and method
  //file2RichFile2 in object ImplicitConversion of
  //type (file: java.io.File)cn.scala.xtwy.RichFile
  //are possible conversion functions from java.io.File to ?{def read: ?}
value read is not a member of java.io.File

  val f=new File("file.log").read
  println(f)
}
```

3 隐式转换不会嵌套进行，例如：
```scala
package cn.scala.xtwy
import java.io.File
import scala.io.Source

package implicitConversion{
  object ImplicitConversion{
    implicit def double2Int(x:Double)=x.toInt
    implicit def file2RichFile(file:File)=new RichFile(file)
    //implicit def file2RichFile2(file:File)=new RichFile(file)
    implicit def richFile2RichFileAnother(file:RichFile)=new RichFileAnother(file)
  }

}

class RichFile(val file:File){
  def read=Source.fromFile(file).getLines().mkString
}

//RichFileAnother类，里面定义了read2方法
class RichFileAnother(val file:RichFile){
  def read2=file.read
}



object ImplicitFunction extends App{
  import cn.scala.xtwy.implicitConversion.ImplicitConversion._
  var x:Int=3.5

  //隐式转换不会多次进行，下面的语句会报错
  //不能期望会发生File到RichFile，然后RifchFile到
  //RichFileAnthoer的转换
  val f=new File("file.log").read2
  println(f)
}
```

## 3 隐式参数

在一般的函数数据定义过程中，需要明确传入函数的参数，代码如下：
```scala
package cn.scala.xtwy

class Student(val name:String){
  def formatStudent(outputFormat:OutputFormat) = {
    outputFormat.first + " " + this.name + " " + outputFormat.second
  }
}

class OutputFormat(val first:String, val second:String)

object ImplicitParamter {
  def main(args:Array[String]):Unit = {
    val outputFormat = new OutputFormat("<<", ">>")
    println(new  Student("john").formatStudent(outputFormat))
  }
}
// 执行结果
// <<john>>
```
如果给函数定义隐式参数的话，则在使用时可以不带参数，代码如下:
```scala
package cn.scala.xtwy

class Student(val name:String){
  // 利用柯里化的定义方式，将函数的参数利用
  // implicit关键字标识
  // 这样的话，在使用的时候可以不给出implicit对应的参数
  def formatStudent()(implicit outputFormat:OutputFormat) = {
    outputFormat.first + " " + this.name + " " + outputFormat.second
  }
}

class OutputFormat(val first:String, val second:String)

object ImplicitParameter {
  def main(args:Array[String]) : Unit = {
    // 程序中定义的变量outputFormat被称为隐式值
    implicit val outputFormat = new OutputFormat("<<", ">>")
    // 在.formatStudent()方法时，编译器会查找类型为
    // outputFormat的隐式值，本程序定义的隐式值为outputFormat
    println(new Student("john").formatStudent())
  }
}
```

## 4 隐式参数中的隐式转换

```scala
object ImplicitParameter extends App{
  // 指定T的上界为Ordered[T],所有混入特质Ordered
  // 的类都可以直接使用 `<` 符号比较
  def compare[T <: Ordered[T]](first:T,second:T) = {
    if(first < second) first else second
  }
}
```
我们还有一种方法可以达到上面的效果，就是通过隐式参数的隐式转换来实现
```scala
object ImplicitParameter extends App {
  // 下面代码中的(implicit order:T=>Ordered[T])
  // 给函数compare制定了一个隐式参数
  // 该隐式参数是一个饮食转换
  def compare[T](first:T, second:T)(implicit order:T=>Ordered[T]) = {
    if(first > second) first else second
  }

  println(compare("A","B"))
}
```

## 5 函数中隐式参数使用概要

要点一：在定义函数时，如果函数没有柯里化，implicit关键字作用所有参数，例如：
```scala
// implicit 关键字在下面的函数中只能出现一次
// 它作用于两个参数x,y,也即x,y都是隐式参数
def sum(implicit x:Int, y:Int) = x + y
```
另外，值得注意的是，def `sum(implicit x:Int,y:Int)`在使用时，也只能指定一个隐式值，即指定的隐式值分别对应函数中的参数(这里是x,y)
```scala
def sum(implicit x: Int, y: Int) = x + y
//只能指定一个隐式值
//例如下面下定义的x会自动对应maxFunc中的
//参数x,y即x=3,y=3，从而得到的结果是6
 implicit val x:Int=3
//不能定义两个隐式值
//implicit val y:Int=4
  println(sum)
```

要点二：要想使用implicit只作用于某个函数参数，则需要将函数进行柯里化，如：
```scala
def sum(x:Int)(implicit y:Int) = x + y
```
值得注意的是，下面这两种带隐式参数的函数也是不合法的
```scala
def sum(x:Int)(implicit y:Int)(d:Int) = x + y + d
def sum(x:Int)(implicit y:Int)(implicit d:Int) = x + y + d
```
要点三：匿名函数不能使用隐式参数，例如：
```scala
val sum2 = (implicit x :Int)=>x+1
```
要点四：如果函数带有隐式参数，则不能使用其偏函数，例如：
```scala
def sum(x: Int)(implicit y:Int)=x+y
//不能定义sum的偏函数，因为它带有隐式参数
//could not find implicit value for
//parameter y: Int
//not enough arguments for method sum:
// (implicit y: Int)Int. Unspecified value parameter y.
def sum2=sum _
```

## 6 隐式转换问题的梳理

1 多次隐式转换问题

在上一讲中我们提到，隐式转换从源类型转换到目标类型不会多次进行，也即源类到目标类型的转换只会进行一次
```scala
class RichFile(val file:File){
  def read = Source.fromFile(file).getLines().mkString
}
// RichFileAnother，里面定义了read2方法
class RichFileAnother(val file:RichFile){
  def read2 = file.read
}
// 隐式转换不会多次进行，下面的语句会报错
// 不能期望会发生File到RichFile，然后RichFile到RichFileAnother的转换
val f = new File("file.log").read2
println(f)
```
**注意**这里指的是源类到目标类的转换只会进行一次，并不是说不存在多次隐式转换，在一般的方法调用过程中可能会出现多次隐式转换，例如：
```scala
class ClassA {
  override def toString() = "This is Class A"
}
class ClassB {
  override def toString() = "This is Class B"
}
class ClassC {
  override def toString() = "This is  ClassC"
  def printC(c: ClassC) = println(c)
}
class ClassD

object ImplicitWhole extends App {
  implicit def B2C(b:ClassB) = {
    println("B2C")
    new ClassC
  }
  implicit def D2C(d:ClassD) = {
    println("D2C")
    new ClassC
  }
  //下面的代码会进行两次隐式转换
  //因为ClassD中并没有printC方法
  //因为它会隐式转换为ClassC（这是第一次,D2C）
  //然后调用printC方法
  //但是printC方法只接受ClassC类型的参数
  //然而传入的参数类型是ClassB
  //类型不匹配，从而又发生了一次隐式转地换(这是第二次,B2C）
  //从而最终实现了方法的调用
  new ClassD().printC(new ClassB)
}
```
还有一种情况会发生隐式转换，如果给函数定义了隐式参数，在实际执行的过程中可能会发生多次隐式转换，例如：
```scala
object Main extends App {
  class PrintOps() {
    def print(implicit i: Int) = println(i);
  }

  implicit def str2PrintOps(s: String) = {
    println("str2PrintOps")
    new PrintOps
  }

  implicit def str2int(implicit s: String): Int = {
    println("str2int")
    Integer.parseInt(s)
  }

  implicit def getString = {
    println("getString")
    "123"
  }
  //下面的代码会发生三次隐式转换
  //首先编译器发现String类型是没有print方法的
  //尝试隐式转换，利用str2PrintOps方法将String
  //转换成PrintOps（第一次）
  //然后调用print方法，但print方法接受整型的隐式参数
  //此时编译器会搜索隐式值，但程序里面没有给定，此时
  //编译器会尝试调用 str2int方法进行隐式转换，但该方法
  //又接受一个implicit String类型参数，编译器又会尝试
  //查找一个对应的隐式值，此时又没有，因此编译器会尝试调用
  //getString方法对应的字符串（这是第二次隐式转换，
  //获取一个字符串，从无到有的过程）
  //得到该字符串后，再调用str2int方法将String类型字符串
  //转换成Int类型（这是第三次隐式转换）
  "a".print
}
```




## 参考博文

[https://yq.aliyun.com/articles/60376?spm=5176.8251999.569296.18.3d10f3b6mLAnf](https://yq.aliyun.com/articles/60376?spm=5176.8251999.569296.18.3d10f3b6mLAnf)

[https://yq.aliyun.com/articles/60375?spm=5176.8251999.569296.19.3d10f3b6mLAnf](https://yq.aliyun.com/articles/60375?spm=5176.8251999.569296.19.3d10f3b6mLAnf)

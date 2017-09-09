# Scala 高阶函数

Scala混合了面向对象和函数式的特性。在函数式编程语言中，函数是“头等公民”，可以像任何其他数据类型一样被传递和操作。每当你想要给算法传入明细动作时这个特性就会变得非常有用。

## 作为值的函数

在Scala中，函数是“头等公民”，就和数字一样。你可以在变量中存放函数：
```scala
import scala.math._

val num = 3.14
val fun = ceil _
```
这段代码将num设为3.14，fun设为ceil函数。

ceil参数后面的`_`意味着你确定指的是这个函数，而不是碰巧忘记给它送参数。

| **说明**：从技术上讲，`_`将ceil方法转成了函数。在scala中，你无法直接操纵方法，而只能直接操纵函数。 |
| -------------------------------------------------------------------------------------------------- |

## 匿名函数

在Scala中，你不需要给每一个函数命名，正如你不需要给每个数字命名一样。以下是一个匿名函数:
```scala
(x : Double) => 3 * x
```
这个函数将传给它的参数乘以3

你可以将这个函数存放到变量中：
```scala
val triple = (x : Double) => 3 * x
```
这就跟你用def一样：
```scala
def triple(x : Double) = 3 * x
```

## 带函数参数的函数

下面代码是一个函数作为参数的示例：
```scala
def valueAtOneQuarter(f : (Double) => Double) = f(0.25)
```
注意，这里的参数可以是任何接受Double并返回Double的函数。valueAtOneQuarter函数将计算那个函数在0.25位置的值。例如：
```scala
valueAtOneQuarter(ceil _)  // 1.0
valueAtOneQuarter(sqrt _)  // 0.5 (因为 0.5 * 0.5 = 0.25)
```

## 参数类型推断

当你将一个匿名函数传递给另一个函数或方法时，Scala会尽可能帮助你推断类型信息。举例来说，你不需要将代码写成：
```scala
valueAtOneQuarter((x:Double) => 3 * x)  // 0.75
```
由于valueAtOneQuarter方法知道你会传入一个类型为`(Double)=>Double`的函数，你可以简单写成：
```scala
valueAtOneQuarter((x) => 3 * x)
```
对于只有一个参数的函数，你可以略去参数外围的():
```scala
valueAtOneQuarter(x => x * 3)
```
如果参数在 => 右侧只出现一次，你可以用_替换掉它：
```scala
valueAtOneQuarter(3 * _)
```
请注意这些简写方式仅在参数类型已知的情况下有效。
```scala
val fun = 3 * _ // 错误：无法推断出类型
val fun = 3 * (_ : Double) // OK
val fun : (Double)=>Double = 3 * _ // OK,因为我们给出了fun的类型
```

## 柯里化

柯里化指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的 函数返回一个以原有第二个参数作为参数的函数。下面看一段代码：
```scala
def mul(x:Int) = (y:Int) => x * y
// 要计算两个数的乘积则调用mul(4)(5)
```
严格地讲，mul(4)的结果是函数`(y:Int) => 4 *y`。而这个函数又被应用到5，因此结果为20 。

Scala支持如下简写来定义柯里化：
```scala
def mul(x:Int)(y:Int) = x * y
```
下面写几个逆天的柯里化函数：
```scala
// 这个还比较简单，就是计算三个数连乘
def fun(x:Int)(y:Int)(z:Int) = x * y * z

// 下面这个就比较变态，其实和上面的用法一样
val fun = (a:Int) => (b:Int) => (c:Int) => a * b * c
// 和上面的代码一样，只不过定义了fun1的类型
val fun1:Int=>(Int=>(Int=>Int)) = (a:Int)=>(b:Int)=>(c:Int) => a * b * c

// 直线函数，通过f1(a)计算系数，通过f2(b)计算截距
def line(f1:Int=>Int)(a:Int)(f2:Int=>Int)(b:Int)(c:Int) = f1(a) * c + f2(b)
// 与上述功能一样
val line = (f1:Int=>Int) => (a:Int) => (f2:Int => Int) => (b:Int) => (c:Int) => f1(a) * c + f2(b)
```

## 控制抽象

在Scala中，我们可以将一系列语句归组成不带参数也没有返回值的函数。举例来说，如下函数在线程中执行某段代码：
```scala
def runInThread(block:() => Unit){
  new Thread{
    override def run(){
      block()
    }
  }.start()
}
```
这段代码以类型为`()=>Unit`的函数的形式给出。不过，当你调用该函数时，需要写一段不美观的`()=>`
```scala
runInThread{() => println("Hi");println("world")}
```
要想省略`()=>`,可以使用换名调用表示法，在参数声明和调用该函数参数的地方省略(),但是保留=>
```scala
def runInThread(block: => Unit){
  new Thread{
    override def run(){
      block
    }
  }.start()
}
```
调用就可以这样：
```scala
runInThread{println("Hi");println("world")}
```
Scala程序员可以构建控制抽象：看上去像是变成语言的关键字的函数。看下面例子：
```scala
def until(condition : => Boolean)(block : => Unit){
  if(!condition){
    block
    until(condition){block}
  }
}

// 调用until

var x = 10
until(x == 0){
  x -= 1
  println(x)
}
```
这样的函数参数有一个专业术语叫做换名调用参数。和一个常规(或者说换值调用)的参数不同，函数在被调用时，参数表达式不会求值。毕竟，在调用until时，我们不希望 x == 0被求值得到false。与之相反，表达式成为无参数的函数体，而函数被当做参数传递下去。

## Scala中常用的高阶函数

### 1 map函数

所有的集合类型都存在map函数，例如Array的map函数的API具有如下形式：
```scala
//这里面采用的是匿名函数的形式，字符串*n得到的是重复的n个字符串，这是scala中String操作的一个特点
scala> Array("spark","hive","hadoop").map((x:String)=>x*2)
res3: Array[String] = Array(sparkspark, hivehive, hadoophadoop)

//省略匿名函数参数类型
scala> Array("spark","hive","hadoop").map((x)=>x*2)
res4: Array[String] = Array(sparkspark, hivehive, hadoophadoop)

//单个参数，还可以省去括号
scala> Array("spark","hive","hadoop").map(x=>x*2)
res5: Array[String] = Array(sparkspark, hivehive, hadoophadoop)

//参数在右边只出现一次的话，还可以用占位符的表示方式
scala> Array("spark","hive","hadoop").map(_*2)
res6: Array[String] = Array(sparkspark, hivehive, hadoophadoop)
```

**List类型**

```scala
scala> val list=List("Spark"->1,"hive"->2,"hadoop"->2)
list: List[(String, Int)] = List((Spark,1), (hive,2), (hadoop,2))

//写法1
scala> list.map(x=>x._1)
res20: List[String] = List(Spark, hive, hadoop)
//写法2
scala> list.map(_._1)
res21: List[String] = List(Spark, hive, hadoop)

scala> list.map(_._2)
res22: List[Int] = List(1, 2, 2)
```

**Map类型**

```scala
//写法1
scala> Map("spark"->1,"hive"->2,"hadoop"->3).map(_._1)
res23: scala.collection.immutable.Iterable[String] = List(spark, hive, hadoop)

scala> Map("spark"->1,"hive"->2,"hadoop"->3).map(_._2)
res24: scala.collection.immutable.Iterable[Int] = List(1, 2, 3)

//写法2
scala> Map("spark"->1,"hive"->2,"hadoop"->3).map(x=>x._2)
res25: scala.collection.immutable.Iterable[Int] = List(1, 2, 3)

scala> Map("spark"->1,"hive"->2,"hadoop"->3).map(x=>x._1)
res26: scala.collection.immutable.Iterable[String] = List(spark, hive, hadoop)
```

### 2 flatMap函数

```scala
//写法1
scala> List(List(1,2,3),List(2,3,4)).flatMap(x=>x)
res40: List[Int] = List(1, 2, 3, 2, 3, 4)

//写法2
scala> List(List(1,2,3),List(2,3,4)).flatMap(x=>x.map(y=>y))
res41: List[Int] = List(1, 2, 3, 2, 3, 4)
```

### 3 fitler函数

filter函数是对集合中的每个元素进行过滤，符合条件的所有元素组成新的集合。

```scala
scala> Array(1,2,4,3,5).filter(_>3)
res48: Array[Int] = Array(4, 5)

scala> List("List","Set","Array").filter(_.length>3)
res49: List[String] = List(List, Array)

scala> Map("List"->3,"Set"->5,"Array"->7).filter(_._2>3)
res50: scala.collection.immutable.Map[String,Int] = Map(Set -> 5, Array -> 7)
```

### 4 reduce函数

使用reduce我们可以处理列表的每个元素并返回一个值。通过使用reduceLeft和reduceRight我们可以强制处理元素的方向。（使用reduce方向是不被保证的）
```scala
//写法1
scala> Array(1,2,4,3,5).reduce(_+_)
res51: Int = 15

scala> List("Spark","Hive","Hadoop").reduce(_+_)
res52: String = SparkHiveHadoop

//写法2
scala> Array(1,2,4,3,5).reduce((x:Int,y:Int)=>{println(x,y);x+y})
(1,2)
(3,4)
(7,3)
(10,5)
res60: Int = 15

scala> Array(1,2,4,3,5).reduceLeft((x:Int,y:Int)=>{println(x,y);x+y})
(1,2)
(3,4)
(7,3)
(10,5)
res61: Int = 15

scala> Array(1,2,4,3,5).reduceRight((x:Int,y:Int)=>{println(x,y);x+y})
(3,5)
(4,8)
(2,12)
(1,14)
res62: Int = 15
```

### 5 fold函数

```scala
scala> Array(1,2,4,3,5).foldLeft(0)((x:Int,y:Int)=>{println(x,y);x+y})
(0,1)
(1,2)
(3,4)
(7,3)
(10,5)
res66: Int = 15

scala> Array(1,2,4,3,5).foldRight(0)((x:Int,y:Int)=>{println(x,y);x+y})
(5,0)
(3,5)
(4,8)
(2,12)
(1,14)
res67: Int = 15

scala> Array(1,2,4,3,5).foldLeft(0)(_+_)
res68: Int = 15

scala> Array(1,2,4,3,5).foldRight(10)(_+_)
res69: Int = 25

// /:相当于foldLeft
scala> (0 /: Array(1,2,4,3,5)) (_+_)
res70: Int = 15


scala> (0 /: Array(1,2,4,3,5)) ((x:Int,y:Int)=>{println(x,y);x+y})
(0,1)
(1,2)
(3,4)
(7,3)
(10,5)
res72: Int = 15
```

### 6 scan函数

```scala
//从左扫描，每步的结果都保存起来，执行完成后生成数组
scala> Array(1,2,4,3,5).scanLeft(0)((x:Int,y:Int)=>{println(x,y);x+y})
(0,1)
(1,2)
(3,4)
(7,3)
(10,5)
res73: Array[Int] = Array(0, 1, 3, 7, 10, 15)

//从右扫描，每步的结果都保存起来，执行完成后生成数组
scala> Array(1,2,4,3,5).scanRight(0)((x:Int,y:Int)=>{println(x,y);x+y})
(5,0)
(3,5)
(4,8)
(2,12)
(1,14)
res74: Array[Int] = Array(15, 14, 12, 8, 5, 0)
```

## 参考博文

[http://blog.csdn.net/lovehuangjiaju/article/details/47079383](http://blog.csdn.net/lovehuangjiaju/article/details/47079383)

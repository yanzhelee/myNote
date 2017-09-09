# apply方法和unapply方法

## appply方法

通常，在一个类的伴生对象中定义apply方法，在生成这个类的对象时，就省去了new关键字。

请看下面代码：

```
class Foo(foo:String){}

object Foo{
  def apply(foo:String) : Foo = {
    new Foo(foo)
  }
}
```
定义一个Foo类，并且在这个类中，有一个伴生对象Foo，里面定义了apply方法。有了这个apply方法以后，在调用这个类的时候用函数的方式来调用：
```scala
object Client{
  def main(args:Array[String]):Unit = {
    val foo = Foo("hello")
  }
}
```
我们用`Foo("hello")`的方式，就得到了一个Foo类型的对象，这一切就是apply方法的功劳。如果没有apply方法，我们将需要使用new关键字来得到Foo对象。

## unapply方法

可以认为unapply方法是apply方法的反向操作，apply方法接受构造参数变成对象，而unapply方法接受一个对象从中提取值。

请看下面的代码：
```scala
class Money(val value : Double, val country : String){}

object Money{
  def apply(value : Double, country : String) : Money = new Money(value, country)

  def unapply(Money : Money) : Option[(Double, String)] = {
    if(money = null) None else Some(money.value, money.country)
  }
}
```
客户端实现：
```scala
def testUnapply  {
  val money = Money(10.1, "RMB")

  money match{
    case Money(num, "RMB") => println("RMB:" + num)
    case _ => println("Not RMB!")
  }
}
//最后输出
RMB:10.1
```

## 参考博文

[http://blog.csdn.net/bitcarmanlee/article/details/76736252](http://blog.csdn.net/bitcarmanlee/article/details/76736252)

# Scala Option(选项)

Scala Option(选项)类型表示一个值得可选的(有值或者无值)。

Option[T] 是一个类型为T的可选值得容器：如果值存在，Option[T]就是一个Some[T],如果不存在，Option[T]就是对象None。

接下来看一段代码：
```scala
val myMap:Map[Int,String] = Map(1 -> "tom")
val v1:Option[String] = myMap.get(1)
val v2:Option[String] = myMap.get(2)

println(v1)    // 打印Some("tom")
println(v2)    // 打印None
```

在上面代码中，myMap是一个Key类型为Int，Value类型为String，的Map，但不一样的是它的get()方法的返回值是一个类型为Option[String]的类别。

Scala使用Option[String]来告诉你：*我会想办法回传一个String，但是也有可能没有String给你*

myMap里面没有key为2的值，所以get方法返回None。

Option有两个子类别，一个是Some，一个是None，当和回传Some的时候，代笔这个函数成功的回传给你一个String，而你可以通过get()函数拿到那个String，如果它返回的是None，则代表没有字符串可以给你。


## getOrElse()方法

乜可以使用getOrElse()方法来获取元组中存在的元素或则使用其默认的值，示例如下：
```scala
object Test {
   def main(args: Array[String]) {
      val a:Option[Int] = Some(5)
      val b:Option[Int] = None

      println("a.getOrElse(0): " + a.getOrElse(0) )
      println("b.getOrElse(10): " + b.getOrElse(10) )
   }
}

//执行结果如下
a.getOrElse(0): 5
b.getOrElse(10): 10
```

## isEmpty()方法

你可以使用 isEmpty() 方法来检测元组中的元素是否为 None，实例如下：
```scala
object Test {
   def main(args: Array[String]) {
      val a:Option[Int] = Some(5)
      val b:Option[Int] = None

      println("a.isEmpty: " + a.isEmpty )
      println("b.isEmpty: " + b.isEmpty )
   }
}
//上述代码执行结果如下
a.isEmpty: false
b.isEmpty: true
```

## Scala Option常用方法

|                          方法名                          |                                                   描述                                                   |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| def get : A                                              | 获取可选值                                                                                               |
| def isEmpty : Boolean                                    | 检测可选类型值是否为None，是返回true，否则为false                                                        |
| def productArity : Int                                   | 返回元素个数                                                                                             |
| def productElement(n : Int) : Any                        | 获取指定的可选项，以0为起始。                                                                            |
| def exists(p: (A) => Boolean):Boolean                    | 如果可选项中指定条件的元素是否存在，如果存在返回true，否则为false                                        |
| def filter(p: (A) => Boolean): Option[A]                 | 如果选项包含有值，而且传递给 filter 的条件函数返回 true， filter 会返回 Some 实例。 否则，返回值为 None  |
| def filterNot(p: (A) => Boolean): Option[A]              | 如果选项包含有值，而且传递给 filter 的条件函数返回 false， filter 会返回 Some 实例。 否则，返回值为 None |
| def flatMap[B](f: (A) => Option[B]): Option[B]           | 如果选项包含有值，则传递给函数 f 处理后返回，否则返回 None                                               |
| def foreach[U](f: (A) => U): Unit                        | 如果选项包含有值，则将每个值传递给函数 f， 否则不处理。                                                  |
| def getOrElse[B >: A](default: => B): B                  | 如果选项包含有值，返回选项值，否则返回设定的默认值                                                       |
| def isDefined: Boolean                                   | 如果可选值是 Some 的实例返回 true，否则返回 false                                                        |
| def iterator: Iterator[A]                                | 如果选项包含有值，迭代出可选值。如果可选值为空则返回空迭代器。                                           |
| def map[B](f: (A) => B): Option[B]                       | 如果选项包含有值， 返回由函数 f 处理后的 Some，否则返回 None                                             |
| def orElse[B >: A](alternative: => Option[B]): Option[B] | 如果一个 Option 是 None ， orElse 方法会返回传名参数的值，否则，就直接返回这个 Option。                  |
| def orNull                                               | 如果选项包含有值返回选项值，否则返回 null                                                                |


## 参考博文

[http://www.runoob.com/scala/scala-options.html](http://www.runoob.com/scala/scala-options.html)

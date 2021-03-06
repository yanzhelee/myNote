# scala基础语法

在Java或C++中，我们把表达式和语句看做两样不同的东西。表达式有值，而语句执行动作。在Scala中，几乎所有构造出来的语法结构都有值。这个特性使得程序更加精简，也更易读。

在Scala中，{}块包含一系列表达式，其结果也是一个表达式。块中最后一个表达式的值就是块的值。

## 条件表达式

在Scala中if/else表达式有值，这个值就是更在if或else之后的表达式的值。例如：
```scala
if(x > 0) 1 else -1
```
上述表达式的值是1或者-1，具体是哪个取决于x的值。你可以将if/else表达式的值赋值给变量：
```java
val s = if(x > 0) 1 else -1
```
与下面的语句效果一样
```java
var s = 0
if (x > 0) s = 1 else s = -1
```
不过第一种写法更好，因为它可以用来初始化一个val。而第二种写法当中，s必须是var。

## 高级for循环和for推导式

多个生成器
```java
// 打印 9X9 乘法表
for (i <- 1 to 9 ; j <- 1 to i){
  printf("%d X %d = %d\t",i,j,i*j)
  if (j == i) print("\r\n")
}
```
每个生成器都可以带一个守卫，if之前没有分号
```java
for(i <- 1 to 3; j <- 1 to 3 if i != j) print(10*i + j + " ")
//将打印 12 13 21 23 31 32
```

----------------

**说明**：也可以将生成器、守卫和定义包含在花括号当中，并以换行的方式而不是分号来分割他们：
```java
for{
  i <- 1 to 3
  from = 4 - i
  j <- from to 3
}
```

----------------

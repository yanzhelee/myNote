# TensorFlow 之 graph session和op

graph即`tf.Graph()` ,session即`tf.Session()`,很多人经常将两者混淆，其实两者完全不是同一个东西。

- graph定义了计算方式，是一些加减乘除等运算的组合，类似于一个函数。它本身不会进行任何计算，也不保存任何中间计算结果。
- session用来运行graph，或者运行graph的一部分。它类似于一个执行者，给graph灌入输入数据，得到输出，并保存中间的计算结果。同时它也给graph分配计算资源(如内存、显卡等)。

Tensorflow是一种符号编程框架，首先要构造一个图(graph),然后在这个图上做运算。打个比方，graph就像一条生产线，session就像生产者。生产线具有一系列的加工步骤(加减乘除等运算)，生产者把原料投进去，就能得到产品。不同生产者都可以使用这条生产线，只要他们的加工步骤是一样的就行。同样的，一个graph可以供多个session使用，而一个session不一定需要使用graph的全部，可以只使用其中的一部分。

## 关于graph

### 定义一个图：graph

```python
g = tf.Graph()
a = tf.constant(2)
b = tf.constant(3)
x = tf.add(a, b)
```

上面就定义了一个graph。tensorflow会默认给我们建立一个graph。所以`g = tf.Graph()` 这句其实是可以省略的。上面的graph包含3个操作，即op，但凡是op，都需要通过session运行之后，才能得到结果。如果你直接执行`print(a)`,那么输出结果是:

```python
Tensor('a:0', shape=(), dtype=int32)
```

是一个张量(Tensor)。如果你执行`print(tf.Session().run(a))`，才能得到2。

### 关于子图:subgraph

你可以定义多个graph，例如一个graph实现z = x + y, 另一个graph实现 u = 2 * v

```python
g1 = tf.Graph()
g2 = tf.Graph()
with g1.as_default():
    x = tf.constant(2)
    y = tf.constant(3)
    z = tf.add(x, y)
with g2.as_default():
    v = tf.constant(4)
    u = tf.mul(2, v)
```

但通常不建议这么做，原因如下:

- 运行多个graph需要多个session，而每个session会试图耗尽所有的计算资源，开销太大；
- graph之间没有数据通道，要认为通过python/numpy传数据；

事实上，你可以把所有的op都定义在一个graph中:

```python
x = tf.constant(2)
y = tf.constant(3)
z = tf.add(x, y)
v = tf.constant(4)
u = tf.mul(2, v)
```

从上面graph的定义可以看到，x/y/z是一波，u/v是另一波，二者没有任何交集。这相当于在一个graph里有两个独立的subgraph。当你要计算 z = x + y时，执行`tf.Session().run(z)`；当你想计算 u = 2 * v, 就执行`tf.Session().run(u)`, 二者完全独立。但更重要的是，二者同一个session上运行，系统会均衡地给两个subgraph分配合适的计算资源。

## 关于session

通常我们会显示地定义一个session来运行graph：

```python
x = tf.constant(2)
y = tf.constant(3)
z = tf.add(x, y)

with tf.Session() as sess:
    result = sess.run(z)
    print(result)
```

输出结果是5.

## 关于op

tensorflow是一个符号式编程框架，首先要定义一个graph，然后用一个session来运行这个graph得到结果。graph就是由一系列op构成的。上面的`tf.constant()`,`tf.add()`,`tf.mul()`都是op，都要现用session运行，才能得到结果。

很多人会以为`tf.Variable()`也是op，其实不是的。tensorflow里，首字母大写的类，首字母小写的才是op。`tf.Variable()`就是一个类，不过它包含了各种op。比如你定义了`x = tf.Variable([2,3], name='vector')`,那么x就具有如下op:

- x.initializer 对x做初始化，即赋值为初始值[2,3]
- x.value() 获取x的值
- x.assign(...) 赋值操作
- x.assign_add(...) 加法操作

`tf.Variable()`必须先初始化，再做运算，否则会报错。下面的写法就不是很安全，容易导致错误：

```python
W = tf.Variable(tf.truncated_normal([700, 100]))
U = tf.Variable(2 * W)
```

### 一个特殊的op: tf.placeholder()

palceholder,翻译过来就是占位符。其实它类似于函数里的自变量。比如 z = x + y, 那么x和y就可以定义成占位符。占位符，顾名思义，就这是占一个位子，平时不用关心它们的值，当你做运算的时候，你再把你的数据灌进去就行了。是不是和自变量很像？看下面的代码：

```python
a = tf.placeholder(tf.float32, shape=[3]) # a是一个3维向量
b = tf.constant([5, 5, 5], tf.float32)
c = a + b
with tf.Session() as sess:
    print sess.run(c, feed_dict = {a: [1, 2, 3]}) # 把[1, 2, 3]灌到a里去
```

输出结果是[6,7,8]。上面代码中出现了`feed_dict`的概念，其实就是用[1, 2, 3]代替a的意思。相当于在本轮计算中，自变量a的取值为[1, 2, 3]。其实不仅仅是`tf.placeholder`才可以用`feed_dict`,很多op都可以。只要 `tf.Graph.is_feedable(tensor)` 返回值是True，那么这个tensor就可用用feed_dict来灌入数据。

`tf.constant()`是直接定义的graph里的，它是graph的一部分，会随着graph一起加载。如果通过`tf.constant()`定义了一个维度很高的张量，那么graph占用的内存就会变大，加载会变慢。而`tf.placeholder`就没有这个问题，所以如果数据维度很高的话，定义成`tf.palceholder()`是更好的选择。
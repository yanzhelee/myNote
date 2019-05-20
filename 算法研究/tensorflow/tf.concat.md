# tf.concat用法

tensorflow中用来拼接张量的函数为tf.concat(),用法：

```python
tf.concat([tensor1, tensor2, tensor3,...], axis)
```

tensorflow 源码中的示例：

  ```python
  t1 = [[1, 2, 3], [4, 5, 6]]
  t2 = [[7, 8, 9], [10, 11, 12]]
  tf.concat([t1, t2], 0)  # [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]
  tf.concat([t1, t2], 1)  # [[1, 2, 3, 7, 8, 9], [4, 5, 6, 10, 11, 12]]

  # tensor t3 with shape [2, 3]
  # tensor t4 with shape [2, 3]
  tf.shape(tf.concat([t3, t4], 0))  # [4, 3]
  tf.shape(tf.concat([t3, t4], 1))  # [2, 6]
 ```
  
 这里解释了当axis=0和axis=1的情况，怎么理解这个axis呢？其实这与numpy中的np.concatenate()用法是一样的。
  
 - axis=0 代表在第0维拼接
 - axis=1 代表在第1维拼接
  
 对于一个二维矩阵，第0个维度代表最外层方括号所框下的子集，第1个维度代表内部方括号所框下的子集。维度越高，括号越小。
  
 比如两个shape为[2,3]的矩阵拼接，要么通过axis=0变成[4,3]，要么通过axis=1变成[2,6]。改变的维度索引对应axis的值。
  
 对于三维矩阵的拼接，自然axis取值范围是[0, 1, 2]。
 
 **对于axis等于负数的情况**
 
 负数在数组索引里面表示倒数(countdown)。比如，对于列表ls = [1,2,3]而言，ls[-1] = 3，表示读取倒数第一个索引对应值。
 
 
xis=-1表示倒数第一个维度，对于三维矩阵拼接来说，axis=-1等价于axis=2。同理，axis=-2代表倒数第二个维度，对于三维矩阵拼接来说，axis=-2等价于axis=1。

一般在维度非常高的情况下，我们想在最'高'的维度进行拼接，一般就直接用countdown机制，直接axis=-1就搞定了。

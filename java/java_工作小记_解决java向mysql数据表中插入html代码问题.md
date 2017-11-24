# 解决java向mysql数据表中插入html代码问题

最近在写一个爬虫程序，其中要把网页中的html代码插入到mysql数据库中，结果程序一直提示报错，错误信息提示我的sql语句有错，但是我检查了半天都没发现程序有问题。之后我将要插入的html字符串内容换成一个简单的字符串(比如"hello world")再次进行测试，结果程序运行成功。所以推断造成程序报错的根本原因是html代码中含有特殊字符，如果不对特殊字符进行处理就会报错,比如，网页上的一段链接是这样写的:`<a href="http://csdn.net/zh">xxx</a>;`

我们只要处理其中的这一段就可以了：`"http://csdn.net/zh";`,这一段用字符串表示就是这样：`String str = "\"http://csdn.net/zh\"";`

下面是解决问题的核心代码：

```java
// 下面的代码就是将单引号和双引号进行转义
String arg1 = Character.toString('\"');
String arg2 = "\\\\"+'"';

String ret = str.replaceAll(arg1,arg2);

String arg3 = Character.toString('\'');
String arg4 = "\\\\'";

ret = ret.replaceAll(arg3,arg4);
```

**Tip**：有时候还需要在链接数据库的url中添加如下的参数，根据数据库中的编码而变 : `useUnicode=true&characterEncoding=utf-8`

## 参考博文

[http://blog.csdn.net/laozhaokun/article/details/22034787](http://blog.csdn.net/laozhaokun/article/details/22034787)

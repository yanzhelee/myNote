# JAVA套接字之UDP编程

## 1 UDP协议

用户数据报协议UDP是无连接的服务。在无连接的情况下，两个实体之间的通信不需要建立好一个连接，因此其下层的有关资源不需要事先进行预订的保留。这些资源在数据传输时动态的进行分配。

无连接服务的另一个特征就是他不需要通信的两个实体同时是活跃的（即处于激活状态）。当发送端的实体正在进行发送时，它才是活跃的。

无连接服务的优点就是灵活方便并且比较迅速。但是无连接服务不能防止报文的丢失、重复或失序。无连接服务特别适合传输少量零星的报文。

在java中操纵UDP使用JDK中java.net包下的DatagramSocket和DatagramPacket类，可以方便的控制用户数据报文。DatagramPacket类将数据字节填充到UDP包中，这称为数据报。 DatagramSocket来发送这个包。要接受数据，可以从DatagramSocket中接受一个 DatagramPack对象，然后从该包中读取数据的内容。

使用UDP传输数据是有大小限制的，每个被传输的数据报必须限定在64KB之内。

## 2 DatagramSocket类

构造函数

构造函数 | 说明
-----|--------
`DatagramSocket()` | 创建实例，通常用于客户端编程，他并没有特定的监听端口，仅仅使用一个临时的。
`DatagramSocket(int port)` | 创建实例，并固定监听Port端口的报文。
`DatagramSocket(int port, InetAddress laddr)` | 这是个非常有用的构建器，当一台机器拥有多于一个IP地址的时候，由它创建的实例仅仅接收来自LocalAddr的报文。
`DatagramSocket(SocketAddress bindaddr)` | bindaddr对象中指定了端口和地址。

常用函数

常用函数 | 说明
--------|---------
`receive(DatagramPacket p)` | 接收数据报文到p中。**注意：**receive方法是阻塞的，如果没有接收到数据报包的话就会阻塞在哪里。
`send(DatagramPacket p)` | 发送报文p到目的地。
`setSoTimeout(int timeout)` | 设置超时时间，单位为毫秒。
`close()` | 关闭DatagramSocket。在应用程序退出的时候，通常会主动的释放资源，关闭Socket，但是由于异常的退出可能造成资源无法回收。所以应该在程序完成的时候，主动使用此方法关闭Socket，或在捕获到异常后关闭Socket。

**注意**
> 1. 在创建DatagramSocket类实例时，如果端口已经被使用，会产生一个SocketException的异常抛出，并导致程序非法终止，这个异常应该注意捕获。
> 2. “阻塞”是一个专业名词，它会产生一个内部循环，是程序暂停在这个地方，直到条件被触发。

## 3 DatagramPacket类

用于处理报文，将字节数组、目标地址、目标端口等数据包装成报文或者将报文拆卸成字节数组。

构造函数

构造函数 | 说明
--------|---------
`DatagramPacket(byte[] buf, int length, InetAddress addr, int port)` | 从buf字节数组中取出offset开始的、length长的数据创建数据对象，目标地址是addr，目标端口是port。
`DatagramPacket(byte buf[], int offset, int length, SocketAddress address)` | 从buf字节数组中取出offset开始的、length长的数据创建数据对象，目标地址是address


常用函数

常用函数 | 说明
-------|---------------
getData() byte[] | 从实例中取得报文中的字节数组编码。
setData(byte[] buf, int offset, int length) | 设置数据报包中的数据内容

## 4 基于UDP实现屏幕广播



## 参考博文

[http://blog.csdn.net/redarmy_chen/article/details/12784909](http://blog.csdn.net/redarmy_chen/article/details/12784909)
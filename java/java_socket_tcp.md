# JAVA套接字之TCP编程

## 1 TCP协议

TCP是面向谅解的协议。所谓连接，就是两个对等实体为进行数据通信而进行的一种结合。面向连接服务是在数据交换之前，必须先建立连接。当数据交换结束后，则应终止这个连接。

面向连接服务具有：连接建立、数据传输和连接释放这三个阶段。在传送数据时是按序传送的。

当一台计算机需要与另一台远程计算机连接时，TCP协议会让他们建立一个连接：用于发送和接收数据的虚拟链路。TCP协议负责收集信息包，并将其按适当的次序放好传送，在接收端收到后再将其正确的还原。为了保证数据包在传送中准确无误，TCP使用了重发机制：当一个通信实体发送一个消息给另一个通信实体后需要收到另一个实体的确认信息，如果没有收到确认信息，则会再次重发刚才发送的信息。

TCP通信分为客户端和服务器端，对应的对象是分别是Socket和ServerSocket。

Socket类是java执行客户端TCP操作的基础类，这个类本身使用代码通过主机操作系统的本地TCP栈进行通信。Socket类的方法会建立和销毁连接，设置各种Socket选项。

ServerSocket类是java执行服务器端操作的基础类，该类运行于服务器，监听入栈TCP连接，每个socket服务器监听服务器的某个端口，当远程主机的客户端尝试连接此端口时，服务器就被唤醒，并返回一个表示两台主机之间socket的正常的Socket对象。

ServerSocket和Socket通信流程：

![](http://i.imgur.com/02orEKc.png)

## 2 ServerSocket

### 2.1 构造函数

```java
ServerSocket()throws IOException
ServerSocket(int port)throws IOException
ServerSocket(int port, int backlog)throws IOException
ServerSocket(int port, int backlog, InetAddress bindAddr)throws IOException
```

在以上构造方法中，参数port指定服务器要绑定的端口（服务器要监听的端口），参数backlog指定客户端连接请求队列的长度，参数bindAddr指定服务器要绑定的IP地址。

#### 2.1.1 绑定端口

除了第一个不带参数的构造方法以外，其他构造方法都会使服务器与特定的端口绑定，该端口由参数port指定。例如，`ServerSocket serverSocket = new SererSocket(80);`
如果运行时无法绑定到80端口，以上代码会抛出IOException，更确切地说，是抛出BindException，它是IOException的子类。BindException一般由以下原因造成：

- 端口已经被其他服务器进程占用；
- 在某些操作系统中，如果没有以超级管理员用户身份来运行服务器程序，那么操作系统不允许服务器绑定到1~1023之间的端口。
- 如果把port设置为0，表示由操作系统来为服务器分配一个任意可用的端口。由操作系统分配的端口也称为匿名端口。对于多数服务器，会使用明确的端口，而不会使用匿名端口，因为客户程序需要事先知道服务器的端口，才能方便的访问服务器。在某些场合，匿名端口有着特殊的用途。

#### 2.1.2 设定客户端连接请求队列的长度

当服务器进程运行时，可能会同时监听到多个客户端的连接请求。例如，每当一个客户端进程执行以下代码：
`Socket sock = new Socket("192.168.32.105",80);`
就意味着在远程主机的80端口上，监听到一个客户端的连接请求。管理客户端请求的任务是由操作系统完成的。操作系统把这些请求连接存储在一个先入先出的队列中。许多操作系统限定了队列的最大长度，一般为50.当队列中的连接请求达到队列的最大容量时，服务器进程所在的主机会拒绝新的连接请求。只有当服务器进程通过ServerSocket的accept()方法从队列中取出连接请求，使队列腾出空位时，队列才能继续加入新的连接请求。

对于客户进程，如果它发出的连接请求被加入到服务器的队列中，就意味着客户端与服务器的连接建立成功，客户进程从Socket构造方法中正常返回。如果客户进程发出的连接请求被服务器拒绝，Socket构造方法会抛出ConnectionException。

ServerSocket构造方法的backlog参数用来显式设置连接请求队列的长度，它将覆盖操作系统限定的队列的最大长度。在以下几种情况中，仍然会采用操作系统限定的队列的最大长度：

- backlog参数的值大于操作系统限定的队列的最大长度；
- backlog参数的值小于或者等于0；
- 在ServerSocket构造方法中没有指定backlog。

#### 2.1.3 设定绑定IP地址

如果主机只有一个IP地址，那么默认情况下，服务器程序就与该IP地址绑定。ServerSocket的第4个构造方法ServerSocket(int port, int backlog, InetAddress bindAddr)有一个bindAddr参数，它显式指定服务器要绑定的IP地址，该构造方法适用于具有多个IP地址的主机。假定一个主机有两个网卡，一个网卡用于连接到Internet， IP地址为222.67.5.94，还有一个网卡用于连接到本地局域网，IP地址为192.168.3.4。如果服务器仅仅被本地局域网中的客户访问，那么可以按如下方式创建ServerSocket：
`ServerSocket serverSocket = new ServerSocket(80,10,InetAddress.getByName ("192.168.3.4"));`

#### 2.1.4 默认构造方法的作用

ServerSocket有一个不带参数的默认构造方法。通过该方法创建的ServerSocket不与任何端口绑定，接下来还需要通过bind()方法与特定端口绑定。

这个默认构造方法的用途是，允许服务器在绑定到特定端口之前，先设置ServerSocket的一些选项。因为一旦服务器与特定端口绑定，有些选项就不能再改变了。

在以下代码中，先把ServerSocket的SO_REUSEADDR选项设为true，然后再把它与8000端口绑定：

```java

ServerSocket serverSocket=new ServerSocket();
serverSocket.setReuseAddress(true);      //设置ServerSocket的选项
serverSocket.bind(new InetSocketAddress(8000));   //与8000端口绑定
```

如果把以上程序代码改为：

```java
ServerSocket serverSocket=new ServerSocket(8000);
serverSocket.setReuseAddress(true);      //设置ServerSocket的选项
```

那么serverSocket.setReuseAddress(true)方法就不起任何作用了，因为SO_ REUSEADDR选项必须在服务器绑定端口之前设置才有效。

### 2.2 接收和关闭与客户的连接

ServerSocket的accept()方法从连接请求队列中取出一个客户的连接请求，然后创建与客户连接的Socket对象，并将它返回。如果队列中没有连接请求，accept()方法就会一直等待，直到接收到了连接请求才返回。

接下来，服务器从Socket对象中获得输入流和输出流，就能与客户交换数据。当服务器正在进行发送数据的操作时，如果客户端断开了连接，那么服务器端会抛出一个IOException的子类SocketException异常：
`java.net.SocketException: Connection reset by peer`
这只是服务器与单个客户通信中出现的异常，这种异常应该被捕获，使得服务器能继续与其他客户通信。

### 2.3 关闭ServerSocket

ServerSocket的close()方法是服务器释放占用的端口，并且断开与所有客户端的连接。当一个服务器程序运行结束时，即使没有执行ServerSocket的close()方法，操作系统也会释放这个服务器占用的端口。因此，服务器程序并不一定要在结束之前执行ServerSocket的close方法。

在某些情况下，如果希望及时释放服务器的端口，以便让其他程序占用这个端口，则可以显式调用close()方法。例如以下代码用于扫描1~65535之间的端口号。如果ServerSocket成功创建，意味着该端口未被其他服务器进程绑定，否者说明该端口已经被其他进程占用：

```java
for(int i = 1; i < 65535; i++){
    try{
        ServerSocket serverSocket = new ServerSocket(i);
        serverSocket.close();
    }catch(Exception e){
        System.out.println("端口" + i + "已经被其他服务器进程占用");
    }
}
```

以上程序代码创建了一个ServerSocket对象后，就马上关闭它，以便及时释放它占用的端口，从而避免程序临时占用系统的大多数端口。

ServerSocket的isClose()方法判断ServerSocket是否关闭，只有执行了ServerSocket的close()方法，isClose()方法的返回值为true，即使ServerSocket还没有和特定的端口绑定，isClose()方法的返回值也是false。

ServerSocket的isBound()方法判断ServerSocket是否已经与一个端口绑定，只要ServerSocket已经与一个端口绑定，即使它已经被关闭，isBound()方法也会返回true。

如果要确定一个ServerSocket已经与特定端口绑定，并且还没有被关闭，则可以采用以下方式：
`boolean isOpen = serverSocket.isBound() && !serverSocket.isClosed();`

### 2.4 获取ServerSocket的信息

ServerSocket的以下两个get方法可分别获得服务器绑定的IP地址，以及绑定的端口：

- public InetAddress getInetAddress();
- public int getLocalPort()

前面已经讲到，在构造ServerSocket时，如果把端口设为0，那么将由操作系统为服务器分配一个端口（称为匿名端口），程序只要调用getLocalPort()方法就能获知这个端口号。如例程3-3所示的RandomPort创建了一个ServerSocket，它使用的就是匿名端口。

多数服务器会监听固定的端口，这样才便于客户程序访问服务器。匿名端口一般适用于服务器与客户之间的临时通信，通信结束，就断开连接，并且ServerSocket占用的临时端口也被释放。

## 3 Socket

### 3.1 构造函数

```java
Socket()
Socket(InetAddress address, int port)throws UnknownHostException, IOException
Socket(InetAddress address, int port, InetAddress localAddress, int localPort)throws IOException
Socket(String host, int port)throws UnknownHostException, IOException
Socket(String host, int port, InetAddress localAddress, int localPort)throws IOException
```

除了第一种不带参数以外，其他构造函数会尝试建立与服务器的连接。如果失败会抛出IOException错误，如果成功则返回Socket对象。
InetAddress是一个用于记录主机的类，其静态getHostByName(String msg)可以返回一个实例，其静态方法getLocalHost()也可以获得当前主机的IP地址，并返回一个实例。Socket(String host, int port, InetAddress localAddress, int localPort)构造函数的参数分别为目标IP、目标端口、绑定本地IP、绑定本地端口。

### 3.2 Socket方法

```java
getInetAddress();    　　远程服务端的IP地址
getPort();    　　　　　　远程服务端的端口
getLocalAddress()    　　本地客户端的IP地址
getLocalPort()    　　　　本地客户端的端口
getInputStream();    　获得输入流
getOutStream();    　　获得输出流
```

### 3.3 Socket状态

```java
isClosed();        　　　　//连接是否已关闭，若关闭，返回true；否则返回false
isConnect();　　　　　　//如果曾经连接过，返回true；否则返回false
isBound();        　　　　//如果Socket已经与本地一个端口绑定，返回true；否则返回false
```

如果要确认Socket的状态是否处于连接中，可以通过下面语句。
```java
//判断当前是否处于连接
boolean isConnection = socket.isConnedted() && !socket.isClosed();
```

## 参考博文

[http://www.cnblogs.com/xujian2014/p/4660570.html](http://www.cnblogs.com/xujian2014/p/4660570.html)

[http://www.51cto.com/specbook/11/40196.htm](http://www.51cto.com/specbook/11/40196.htm)

[http://www.cnblogs.com/rond/p/3565113.html](http://www.cnblogs.com/rond/p/3565113.html)
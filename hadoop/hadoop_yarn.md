# Hadoop YARN

## 1 Yarn的组件及架构

Yarn主要由以下几个组件组成：

1. ResourceManager : Global(全局)的进程
2. NodeManager : 运行在每个节点上的进程
3. ApplicationMaster : Application-specific(应用级别)的进程
4. *Scheduler : 是ResourceManager的一个组件
5. *Container : 节点上一组CPU和内存资源

**Container**是Yarn对计算机计算资源的抽象，它其实就是一组CPU和内存资源，所有的应用都会运行在Container中。

**ApplicationMaster**是对运行在Yarn中某个应用的抽象，它其实就是某个类型应用的实例，ApplicationMaster是应用级别的，他的主要功能就是向ResourceManager申请计算资源(Containers)并且和NodeManager交互执行和监控具体的task。Scheduler是ResourceManager专门进行资源管理的一个组件，负责分配NodeManager上的Container资源，NodeManager也会不断发送自己Container使用情况给ResourceManager。

ResourceManager和NodeManager两个进程主要负责系统管理方面的任务。ResourceManager有一个Scheduler，负责各个集群中应用的资源分配。对于每种类型的每个应用，都会对应一个ApplicationMaster实例，ApplicationMaster通过和ResourceManager沟通获得Container资源来运行具体的job，并跟踪这个job的运行状态、监控运行进度。

下面我们看一下整个Yarn的架构图：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_yarn_1.png)

## 2 Yarn组件详解

### 2.1 Container

Container是Yarn框架的计算单元，是具体执行应用task(如map task、reduce task)的具体单位。Container和集群节点的关系是：一个节点会运行多个Container，但一个Container不会跨节点。

一个Container就是一组分配的系统资源，现阶段只包含两种系统资源(之后可能会增加磁盘、网络等资源)：
```
1. CPU core
2. Memory in MB
```

既然一个Container指的是具体节点上的计算资源，这就意味着Container中必定含有计算资源的位置信息：计算资源位于那个机器上。所以我们在请求某个Container时，其实就是向某台机器发起的请求，请求的是这台机器上的CPU和内存资源。

任何一个job或application必须运行在一个或者多个Container中，在Yarn框架中，ResourceManager只负责告诉ApplicationMaster那些Containers可以用，ApplicationMaster还需要去找NodeManager请求分配具体的Container。

### 2.2 NodeManager

NodeManager进程运行在集群中的节点上，每个节点都会有自己的NodeManager。NodeManager是一个slave服务：它负责ResourceManager的资源分配请求，分配具体的Container给应用。同时，它还负责监控并报告Container使用信息给ResourceManager的资源分配请求，分配具体的Container给应用。同时，它还负责监控并报告Container使用信息个ResourceManager。通过和ResourceManager配合，NodeManager负责整个Hadoop集群中的资源分配工作。ResourceManger是一个全局的进程，而NodeManager只是每个节点上的进程，管理这个节点上的资源分配和监控运行节点的健康状态。下面是NodeManager的具体任务列表：

1. 接收ResourceManager的请求，分配Container给应用的某个任务
2. 和ResourceManager交换信息，以确保整个集群平稳运行。ResourceManager就是通过手机每个NodeManager的报告信息来追踪整个集群的健康状态，而NodeManager负责监控自身的健康状态。
3. 管理每个Container的生命周期
4. 管理每个节点上的日志
5. 执行Yarn上面应用的一些额外服务，比如MapReduce的shuffle过程

当一个节点启动时，它会向ResourceManager进行注册并告知ResourceManager自己有多少资源可用。

在运行期间，通过NodeManager和ResourceManager协同工作，这些信息会不断被更新并保障整个集群发挥出最佳状态。

NodeManager只负责管理自身的Container，它并不知道运行在它上面的应用的信息。负责管理应用信息的组件是ApplicationMaster。

### 2.3 ResourceManager

ResourceManager主要有两个重要的组件：Scheduler和ApplicationManager。

Scheduler是一个资源调度器，它主要负责协调集群中的各个应用的资源分配，保障整个集群的运行效率。Scheduler的角色是一个纯调度器，它只负责调度Containers，不会关心应用程序监控及其运行状态等信息。同样，它也不会重启因应用失败或者硬件错误而运行失败的任务。

Scheduler是一个可插拔的插件，它可以调度集群中的各种队列，应用等。在Hadoop的MapReduce框架中主要有两种Scheduler：Capacity Scheduler和Fair Scheduler。

另一个组件ApplicationManager主要负责接收job的提交请求，为应用分配第一个Container来运行ApplicationMaster，还有就是负责监控ApplicationMaster，在遇到失败时重启ApplicationMaster运行的Container。

### 2.4 ApplicationMaster

ApplicationMaster













## 参考博文

[http://blog.csdn.net/suifeng3051/article/details/49486927](http://blog.csdn.net/suifeng3051/article/details/49486927)

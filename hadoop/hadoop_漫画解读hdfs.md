# [漫画解读]HDFS存储原理

博文转自京东大数据([链接](https://mp.weixin.qq.com/s/CIsQYDcAGKtxi4ax_8AwCw))

根据Maneesh Varshney的漫画改编，以简洁易懂的漫画形式讲解HDFS存储机制与运行原理。

## 一、角色出演

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3BmYYHloxphr2IxVeZN4s9mqD0qAZ3yoklMxrk4mE65Tic0PS7D3OnpQ/640?wxfmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

如上图所示，HDFS存储相关角色与功能如下：

**Client：客户端**，系统使用者，调用HDFS API操作文件；与NN交互获取文件元数据；与DN交互进行数据读写。

**Namenode：元数据节点**，是系统唯一的管理者。负责元数据的管理；与client交互进行提供元数据查询；分配数据存储节点等。

**Datanode：数据存储节点**，负责数据块的存储与冗余备份；执行数据块的读写操作等。

## 二、写入数据

### 1、发送写数据请求

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3iaFkhVE1sLVTHBBwfKo8ukCUU8MZLD5HxNel2yznwpq3qg3iaCrWA18Q/640?wxfmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

HDFS中的存储单元是block。文件通常被分成64或128M一块的数据块进行存储。与普通文件系统不同的是，在HDFS中，如果一个文件大小小于一个数据块的大小，它是不需要占用整个数据块的存储空间的。

### 2、文件切分

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3CbKdhZjNoeb3YEXeAjmbEQ4qcHtibUq6ODeiakSLOcVjrhQWxBibmuvWQ/640?wxfmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

### 3、DN分配

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3G3wQgiaTCIETm4bB0EdxjPgdm4ncTUtClLkiaOicf24Ticic5W1eJpGFDWA/640?wxfmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3HbFBVibmVSGeicv8au67ibxTbwPSDrjJxgjejdC31HNzXtHZGPJDa0ib9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 4、数据写入

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3QeEsQW1DTs68GkMgxIcvwyfJp2PhzxlNdw9P8VTg0ztABlAHwPjw9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3E0iaCgoOoYRsxkVOV2a6smOgxflhTleamxuDlQzmZq8hBwsHnHN2tfw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 5、完成写入

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH39W2uiawSn0HlxERkWUfbhcO0Xuw1KqALmjpc6TePyTRicTLhvW4Q2Fiag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3uvRm4TbMrrEXXz9puml0sibTkE8iamrdpP1tTicOWKgFKcdFcCxwhdrcQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3TnGr9W9RuiaibNax44jzsrd9Mh6pbqQcicSbn4bibChc2AHCxA8Fq8ia0nQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 6、角色定位

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkDicNC0OrjMAKjD2PDMibrH3z8eficnzwzsdm3aNFOhyicCfHWXmtLTxJmbmibv6dFrh8UY9dBCSzb8NA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

## 三、HDFS读文件

### 1、用户需求

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAlDoQJf3B7ibn4XzA0UZKtLXyxVicvoicVeUiazZhgBcXxuCMHQjhGHPRrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

HDFS采用的是“一次写入多次读取”的文件访问模型。一个文件经过创建、写入和关闭之后就不需要改变。这一假设简化了数据一致性问题，并且使高吞吐量的数据访问成为可能。

### 2、先联系元数据节点

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAW5gu9wzibbGiaXL3f1TxmzC9A0vsO6OGHRNQ32sYXyaMqFibhG7UmwopA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAM4rdIUPG5DEHmxCmjJn0OIgL3sGFiclkT1tp5sSom38KGXHl6hGYnWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAibuD7hjGe1YzTeJtT6Tthjxv3qIQWXJIoLbSKvzUJ0SeoZarGFhJDSA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 3、下载数据

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAPshWsyPgBrtwGqMdGJLmok7rSnH9BHciaahMrDgPGXbwtrKElfDv60Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

前文提到在写数据过程中，数据存储已经按照客户端与DataNode节点之间的距离进行了排序，距客户端越近的DataNode节点被放在最前面，客户端会优先从本地读取该数据块。

### 4、思考

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZATLCJiaxZE9xM3AzzZIOekxOtOno2fumfZuMCZJvzbqzHzITH2tRwRug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

## 四、HDFS容错机制——第一部分：故障类型及监测方法

### 1、三类故障

(1) 第一类：节点失败

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAX2h8Mt7zFqkrwbylTts8luyZ5KXcWWAqNwfuic70rnvK5McPQmvuszQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

(2) 第二类：网络故障

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAewAVrQ3wzqYsQ94icKrmHlVT06U2CWKArBicdu88afQWN2MJ9mtVMHug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

(3) 第三类：数据损坏（脏数据）

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAKz0QQS6MdiaUjoPR3LTJAq9ePnIdfCMr6RiapDHUrD8gNP5uTNB1X3rA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 2、故障检测机制

(1) 节点失败监测机制

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAms2636ucROPhAMIMib5IBmricDrGCvjqW1xxOmaPGaHLUmib0T5TWcwfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAIMsiafXxktdwAhAzE3mGZiau8g7S9RN8KibTqHyT5mNjUJgBErkicdtcsQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAuw3XZYN91zwejladgblkMLYhY9Q3XqMqwZMzGiaJf3z1c2LqgkmUftw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

(2) 通信故障监测机制

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAn5MEQP7f9L0xJSMGwveBXn6XxDEvEHVHXoW7oSFOQFjIAUNkOsia5ag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

(3) 数据错误监测机制

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZA1FancXVj06OnV4pOQ20QygJaKVMFHq3JKLyfrMEu6RKyELb9b8Pujg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAthbMSbGYSOXqCXYMwq3G4EEpknzCAL6Pib2GLichoUGRoL6FJCw5qbMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAOeKCd4JicddyiaShicLwGPnBD3n06K0C6rKQWiaicO97nVOAibrtsHyybdPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 3、回顾：心跳信息与数据块报告

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvn4bibpOQic6MWL4WInWhanZAz063ygS6oN2R2ibzdzh2D0eibiaKLwEjFVHE5H4e5FfBGXYiciaXqsj8S6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

HDFS存储理念是以最少的钱买最烂的机器并实现最安全、难度高的分布式文件系统（高容错性低成本），从上可以看出，HDFS认为机器故障是种常态，所以在设计时充分考虑到单个机器故障，单个磁盘故障，单个文件丢失等情况。

## 五、容错第二部分：读写容错

### 1、写容错

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkIdguMnn6Z3ic4OXMX3mdmMqLDgsj71ylp7BUYm0TickzXQmI0Wpy5fYXb1GM0R7lvxW17sJ95uEAg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQJcEEoxhCyVeQJ1Iia1Bnno1K4eVY2icibv8bjTfB8FSicicPtVCGyZAs5Gg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQIRadTXjwwicUHkiaaam8RGexOETCXHgrCEt4p3slCXzDYDNic7mWYvdQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQfVmE2jocJCTwI7QB0l4eXxGKvhb5HOZq5hDgicPpaG75ScqXE1Z63icw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 2、读容错

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQB0JL3FTicrdPPywH5StbgdTU6jpzQzibMiamXy5O7IrB0YPeq7Nqro2sA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQMLCn6E5VF6o3t44S0HWNqVOwUjSiaCia6icG5sTtkdcEEh967MUfdSAQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

## 六、容错第三部分：数据节点(DN)失效

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQUjEHvHCEl2icWW5DO9J7jqicau5buUiaTvZ3JNG6xZWghIu2v0VHlWp1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQl7lKwEuSzhkHSE8Wd56qzicBbwepchnTnVLMH5UJx0GHyK5RwsaDzeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkIdguMnn6Z3ic4OXMX3mdmM0gAX0dnnN1y45T0GwHo3AYlXTQlnafaD8Bb2BGK2PDGQUcWPZNaVcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkIdguMnn6Z3ic4OXMX3mdmMwNOI7m0TkVVs0FonHVtK49kGQERK2BfGsib1dFnhsGAjicIOaXUpJVxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQic143suLLE5bU0jjksdI10Xs3njibnF4Kmf5rJfuOa3MeV6RuDNW2Qqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQptH9cypDvmUmAEhOHice07MsJtbjqNt62iaE2uOibEWPLp4Ub7usOKs3w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

## 七、备份规则

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQHuZfzPsZ902SFicHoHbvKTzj6apwO8L8pbKoY8icGTrAtWK9633ysuibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 1、机架与数据节点

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQgoC94aKggoctSlHqhicAdicYyQRtsxPoQj8fnBwqswibH8tuVbpdZLZhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 2、副本放置策略

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQEFT5JcibUKfulZ4ATvzvB32RMZSjJoNCAGVHVJBjQLBWDJuayfQXf4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

数据块的第一个副本优先放在写入数据块的客户端所在的节点上，但是如果这个客户端上的数据节点空间不足或者是当前负载过重，则应该从该数据节点所在的机架中选择一个合适的数据节点作为本地节点。

如果客户端上没有一个数据节点的话，则从整个集群中随机选择一个合适的数据节点作为此时这个数据块的本地节点。

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQaoWSm1GBnJJxSpsnicEXCEoImx4ZNuKoQL6nJnKpttwQ40BYRHx8Z5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

HDFS的存放策略是将一个副本存放在本地机架节点上，另外两个副本放在不同机架的不同节点上。

这样集群可在完全失去某一机架的情况下还能存活。同时，这种策略减少了机架间的数据传输，提高了写操作的效率，因为数据块只存放在两个不同的机架上，减少了读取数据时需要的网络传输总带宽。这样在一定程度上兼顾了数据安全和网络传输的开销。

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQBGSuqMu9CnXxxhK7IzRmmV5FiacXs8NVGnkv9SDqictfo6bsUQqyXj6g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

![](http://mmbiz.qpic.cn/mmbiz/fgsIH6KSdvkE3mQNLpClo0ColuQtRMBQJSMbMNVicdMLibDCMgj6FpdLXC2VtCDZofmAjNxwaib3b5NicFlib8sdAMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

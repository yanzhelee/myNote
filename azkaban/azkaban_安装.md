# Azkaban安装

## 1 Azkaban介绍

Azkaban是由Linkedin开源的一个批量工作流任务调度器。用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban定义了一种kv文件格式来建立任务之间的关系，并提供一个易于使用的web用户界面维护和跟踪你的工作流。它有如下功能特点：

- Web用户界面
- 方便上传工作流
- 方便设置任务之间的关系
- 调度工作流
- 认证/授权（权限的管理）
- 能够杀死并重新启动工作流
- 模块化和可插拔的插件机制
- 项目工作区
- 工作流和任务的日志记录和审计

好了废话不多说，下面开始安装。

## 2 Azkaban安装部署

### 2.1 准备工作
1.Azkaban Web服务器
`azkaban-web-server-2.5.0.tar.gz`
2.Azkaban执行服务器 
`azkaban-executor-server-2.5.0.tar.gz`
3.Msql数据库
目前azkaban只支持 mysql,需安装mysql服务器,本文档中默认已安装好mysql服务器,并建立了 root用户,密码 root.

下载地址：[http://azkaban.github.io/downloads.html](http://azkaban.github.io/downloads.html "软件下载")

将安装文件上传到集群,最好上传到安装 hive、sqoop的机器上。
为了方便命令的执行在当前用户目录下新建azkabantools目录,用于存放源安装文件，新建azkaban目录,用于存放azkaban运行程序。


### 2.2 Azkaban web服务器安装

第一步：
>解压azkaban-web-server-2.5.0.tar.gz
>命令: 
`tar –zxvf azkaban-web-server-2.5.0.tar.gz`

第二步：
>将解压后的azkaban-web-server-2.5.0 移动到 azkaban目录中,并重新命名 webserver
>命令: 
<pre>
mv azkaban-web-server-2.5.0 ../azkaban
cd ../azkaban
mv azkaban-web-server-2.5.0  webserver
</pre>

### 2.3 Azkaban执行服务器安装

第一步：
>解压azkaban-executor-server-2.5.0.tar.gz
>命令:
```tar –zxvf azkaban-executor-server-2.5.0.tar.gz```

第二步：
>将解压后的azkaban-executor-server-2.5.0 移动到 azkaban目录中,并重新命名 executor
>命令:
<pre>
mv azkaban-executor-server-2.5.0  ../azkaban
cd ../azkaban
mv azkaban-executor-server-2.5.0  executor
</pre>

### 2.4 导入Msql数据

第一步：
>解压: azkaban-sql-script-2.5.0.tar.gz
>命令:
```tar –zxvf azkaban-sql-script-2.5.0.tar.gz```

第二步：
>将解压后的mysql 脚本,导入到mysql中:
>进入mysql
<pre>
mysql> create database azkaban;
mysql> use azkaban;
Database changed
mysql> source /home/hadoop/azkaban-2.5.0/create-all-sql-2.5.0.sql;
</pre>

### 2.5 创建SSL配置

参考地址: http://docs.codehaus.org/display/JETTY/How+to+configure+SSL
命令: `keytool -keystore keystore -alias jetty -genkey -keyalg RSA`
运行此命令后,会提示输入当前生成 keystor的密码及相应信息,输入的密码请劳记,信息如下:
 
输入keystore密码： 
再次输入新密码:
您的名字与姓氏是什么？
  [Unknown]： 
您的组织单位名称是什么？
  [Unknown]： 
您的组织名称是什么？
  [Unknown]： 
您所在的城市或区域名称是什么？
  [Unknown]： 
您所在的州或省份名称是什么？
  [Unknown]： 
该单位的两字母国家代码是什么
  [Unknown]：  CN
CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=CN 正确吗？
  [否]：  y
 
输入<jetty>的主密码
        （如果和 keystore 密码相同，按回车）： 
再次输入新密码:
完成上述工作后,将在当前目录生成 keystore 证书文件,将keystore 考贝到 azkaban web服务器根目录中.如:`cp keystore azkaban/webserver`

### 2.6 配置文件

**注**：先配置好服务器节点上的时区
1、先生成时区配置文件Asia/Shanghai，用交互式命令 tzselect 即可
2、拷贝该时区文件，覆盖系统本地时区配置
`cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`

#### azkaban web服务器配置

进入azkaban web服务器安装目录 conf目录

修改azkaban.properties文件
```
#Azkaban Personalization Settings
azkaban.name=Test                                  #服务器UI名称,用于服务器上方显示的名字
azkaban.label=My Local Azkaban                     #描述
azkaban.color=#FF3601                              #UI颜色
azkaban.default.servlet.path=/index
web.resource.dir=web/                              #默认根web目录
default.timezone.id=Asia/Shanghai                  #默认时区,已改为亚洲/上海 默认为美国
 
#Azkaban UserManager class
user.manager.class=azkaban.user.XmlUserManager     #用户权限管理默认类
user.manager.xml.file=conf/azkaban-users.xml       #用户配置,具体配置参加下文
 
#Loader for projects
executor.global.properties=conf/global.properties  #global配置文件所在位置
azkaban.project.dir=projects
 
database.type=mysql                                 #数据库类型
mysql.port=3306                                     #端口号
mysql.host=hadoop03                                 #数据库连接IP
mysql.database=azkaban                              #数据库实例名
mysql.user=root                                     #数据库用户名
mysql.password=root                                 #数据库密码
mysql.numconnections=100                            #最大连接数
 
# Velocity dev mode
velocity.dev.mode=false
# Jetty服务器属性.
jetty.maxThreads=25                                 #最大线程数
jetty.ssl.port=8443                                 #Jetty SSL端口
jetty.port=8081                                     #Jetty端口
jetty.keystore=keystore                             #SSL文件名
jetty.password=123456                               #SSL文件密码
jetty.keypassword=123456                            #Jetty主密码 与 keystore文件相同
jetty.truststore=keystore                           #SSL文件名
jetty.trustpassword=123456                          # SSL文件密码
 
# 执行服务器属性
executor.port=12321                                 #执行服务器端口
 
# 邮件设置
mail.sender=xxxxxxxx@163.com                        #发送邮箱
mail.host=smtp.163.com                              #发送邮箱smtp地址
mail.user=xxxxxxxx                                 # 发送邮件时显示的名称
mail.password=**********                            #邮箱密码
job.failure.email=xxxxxxxx@163.com                  #任务失败时发送邮件的地址
job.success.email=xxxxxxxx@163.com                  #任务成功时发送邮件的地址
lockdown.create.projects=false
cache.directory=cache                               #缓存目录
```
改配置主要的就是配置mysql信息。

#### azkaban 执行服务器配置

进入执行服务器安装目录conf,修改azkaban.properties
```
#Azkaban
default.timezone.id=Asia/Shanghai                 #时区
 
# Azkaban JobTypes 插件配置
azkaban.jobtype.plugin.dir=plugins/jobtypes       #jobtype 插件所在位置
 
#Loader for projects
executor.global.properties=conf/global.properties
azkaban.project.dir=projects
 
#数据库设置
database.type=mysql                               #数据库类型(目前只支持mysql)
mysql.port=3306                                   #数据库端口号
mysql.host=192.168.20.200                         #数据库IP地址
mysql.database=azkaban                            #数据库实例名
mysql.user=azkaban                                #数据库用户名
mysql.password=oracle                             #数据库密码
mysql.numconnections=100                          #最大连接数
 
# 执行服务器配置
executor.maxThreads=50                            #最大线程数
executor.port=12321                               #端口号(如修改,请与web服务中一致)
executor.flow.threads=30                          #线程数
```
#### 用户配置

进入azkaban web服务器conf目录,修改azkaban-users.xml
```
<azkaban-users>
        <user username="azkaban" password="azkaban" roles="admin" groups="azkaban" />
        <user username="metrics" password="metrics" roles="metrics"/>

		<!--添加下面这行语句，登陆Azkaban的用户名和密码-->
        <user username="admin" password="admin" roles="admin,metrics" />


        <role name="admin" permissions="ADMIN" />
        <role name="metrics" permissions="METRICS"/>
</azkaban-users>
```

## 3 启动

### 3.1 web服务器

在azkaban web服务器目录下执行启动命令
`bin/azkaban-web-start.sh`
注:在web服务器根目录运行

### 3.2 执行服务器

在执行服务器目录下执行启动命令
`bin/azkaban-executor-start.sh ./`
注:只能要执行服务器根目录运行
 
启动完成后,在浏览器(建议使用谷歌浏览器)中输入https://服务器IP地址:8443 ,即可访问azkaban服务了.在登录中输入刚才新的户用名及密码,点击 login.

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/azkaban/azkaban%E7%99%BB%E9%99%86%E7%95%8C%E9%9D%A2.png)

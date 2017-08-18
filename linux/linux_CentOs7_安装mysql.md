# CentOs7安装mysql

## 1.安装依赖

```shell
yum search libaio  # 检索相关信息
yum install libaio # 安装依赖包
```

## 2.检查MySQL是否已安装

``` yum list installed | grep mysql ```
如果有就全部卸载，命令如下：
```yum -y remove mysql-libs.x86_64```

*安装下载工具wget*
``` yum -y install wget ```

## 3.下载 MySQL Yum Repository(yum仓库)
地址为 http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

###执行下载

```wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm```
如果提示-bash: wget: 未找到命令，请先执行 ```yum install wget``` 安装 wget

## 4.添加 MySQL Yum Repository

添加 MySQL Yum Repository 到你的系统 repository 列表中，执行
```yum localinstall mysql-community-release-el7-5.noarch.rpm```

## 5.验证下是否添加成功

``` yum repolist enabled | grep "mysql.*-community.*"```

## 6.开始yum安装mysql

``` yum install mysql-community-server``` 

## 7.开启
``` systemctl start  mysqld```
``` systemctl status  mysqld #查看状态```

## 8.验证 mysql
输入```mysql```命令

## 9.创建hive需要的数据库
在mysql上创建hive元数据库，并对hive进行授权
```sql
create database if not exists hive_metadata;
grant all privileges on hive_metadata.* to 'hive'@'%' identified by 'hive';
grant all privileges on hive_metadata.* to 'hive'@'localhost' identified by 'hive';
grant all privileges on hive_metadata.* to 'hive'@'s201' identified by 'hive';
flush privileges;
use hive_metadata;
```

## 参考博文
[http://www.centoscn.com/mysql/2016/0315/6844.html](http://www.centoscn.com/mysql/2016/0315/6844.html)
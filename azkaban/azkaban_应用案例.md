# Azkaban四个应用案例

Azkaba内置的任务类型支持command、java

## 案例一（单个job）

1. 创建job描述文件

<pre>
#command.job
type=command                                                    
command=echo 'hello'
</pre>

2. 将job资源文件打包成zip文件
3. 通过azkaban的web管理平台创建project并上传job压缩包
>首先创建project
> ![](http://i.imgur.com/TfQmWrB.png)
>上传zip包
>![](http://i.imgur.com/taspBYt.png)
4. 启动执行该job
![](http://i.imgur.com/MfNAHF7.png)

## 案例二（多job工作流flow）

1. 创建有依赖关系的多个job描述
第一个job：foo.job
<pre>
#foo.job
type=command
command=echo foo
</pre>

第二个job：bar.job依赖foo.job
<pre>
# bar.job
type=command
dependencies=foo
command=echo bar
</pre>
2. 将所有job资源文件打到一个zip包中
![](http://i.imgur.com/X8aSJ30.png)
3. 在azkaban的web管理界面创建工程并上传zip包

## 案例三（HDFS操作任务）

1. 创建job描述文件
<pre>
# fs.job
type=command
command=/home/hadoop/apps/hadoop-2.6.1/bin/hadoop fs -mkdir /azaz
</pre>
2. 将job资源文件打包成zip文件
![](http://i.imgur.com/k3tOpFv.png)
3. 通过azkaban的web管理平台创建project并上传job压缩包
4. 启动执行该job

## 案例四（MAPREDUCE任务）

MR任务依然可以使用command的job类型来执行

1. 创建job描述文件及mr程序jar包（示例中直接使用hadoop自带的example jar）

<pre>
# mrwc.job
type=command
command=/home/hadoop/apps/hadoop-2.6.1/bin/hadoop jar hadoop-mapreduce-examples-2.6.1.jar wordcount /wordcount/input /wordcount/azout
</pre>

2. 将job资源文件打包成zip文件
> ![](http://i.imgur.com/sQ8VE2Q.png)
3. 通过azkaban的web管理平台创建project并上传job压缩包
4. 启动执行该job

# 多个MapReduce之间的嵌套

在很多实际工作中，单个MR不能满足逻辑需求，而是需要多个MR之间的相互嵌套。很多场景下，一个MR的输入依赖于另一个MR的输出。结合案例实现一下两个MR的嵌套。
** Tip：如果只关心多个MR嵌套的实现，可以直接跳到下面*《多个MR嵌套源码》*章节查看 **

## 案例描述

根据log日志计算log中不同的IP地址数量是多少。测试数据如下图所示：
![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_mapReduce%E4%B9%8B%E9%97%B4%E7%9A%84%E5%B5%8C%E5%A5%97_1.png)
该日志中每个字段都是用Tab建分割的。

## 案例分析

本次任务的目的是计算该日志不同的IP地址一共有多少。实现这个目的的方式有很多种，但是本文的目的是借助改案例对两个MapReduce之间的嵌套进行总结的。

### 实现方法

该任务分为两个MR过程，第一个MR（**命名为MR1**）负责将重复的ip地址去掉，然后将无重复的ip地址进行输出。第二个MR（**命名为MR2**）负责将MR1输出的ip地址文件进行汇总，然后将计算总数输出。

#### MR1阶段

**map过程**
````java
public class IpFilterMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

	@Override
	protected void map(LongWritable key, Text value,
			Mapper<LongWritable, Text, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		String line = value.toString();
		String[] splits = line .split("\t");
		String ip = splits[3];
		context.write(new Text(ip), NullWritable.get());
	}
}
````
输入的key和value是文本的行号和每行的内容。
输出的key是ip地址，输出的value为空类型。

**shuffle过程**

> 主要是针对map阶段输出的key进行排序和分组，将相同的key分为一组，并且将相同key的value放到同一个集合里面，所以不同的组绝对不会出现相同的ip地址，分好组之后将值传递给reduce。**注：该阶段是hadoop系统自动完成的，不需要程序员编程**

**reduce过程**
```` java
 public class IpFilterReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<NullWritable> values, Context context) 
			throws IOException, InterruptedException {
		context.write(key, NullWritable.get());
	}
} 
````
由于经过shuffle阶段之后所有输入的key都是不同的，也就是ip地址是无重复的，所以可以直接输出。

#### MR2阶段
**map过程**

````java
public class IpCountMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		//输出的key为字符串"ip",这个可以随便设置，只要保证每次输出的key都一样就行
		//目的是为了在shuffle阶段分组
		context.write(new Text("ip"), NullWritable.get());
	}
}
````

**shuffle过程**

> 按照相同的key进行分组，由于map阶段所有的key都一样，所以最后只有一组。

**reduce过程**

````java
public class IpCountReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<NullWritable> values,
			Reducer<Text, NullWritable, Text, NullWritable>.Context context) throws IOException, InterruptedException {
		//用于存放ip地址总数量
		int count = 0;
		for (NullWritable v : values) {
			count ++;
		}
		context.write(new Text(count+""), NullWritable.get());
	}
}
````


### 流程图

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_mapReduce%E4%B9%8B%E9%97%B4%E7%9A%84%E5%B5%8C%E5%A5%97_2.png)

## 源码
### MR1 map源码
```` java
//MR1 map源码
package com.ipcount.mrmr;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class IpFilterMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

	@Override
	protected void map(LongWritable key, Text value,
			Mapper<LongWritable, Text, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		String line = value.toString();
		String[] splits = line .split("\t");
		String ip = splits[3];
		context.write(new Text(ip), NullWritable.get());
	}
}

````
### MR1 reduce源码
````java
package com.ipcount.mrmr;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class IpFilterReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<NullWritable> values, Context context) 
			throws IOException, InterruptedException {
		context.write(key, NullWritable.get());
	}
}

````

### MR2 map源码

````java
package com.ipcount.mrmr;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class IpCountMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		context.write(new Text("ip"), NullWritable.get());
	}
}

````

### MR2 reduce源码

````java
package com.ipcount.mrmr;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class IpCountReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<NullWritable> values,
			Reducer<Text, NullWritable, Text, NullWritable>.Context context) throws IOException, InterruptedException {
		int count = 0;
		for (NullWritable v : values) {
			count ++;
		}
		context.write(new Text(count+""), NullWritable.get());
	}
}

````

### 多个MR嵌套源码
````java
package com.ipcount.mrmr;

import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.jobcontrol.ControlledJob;
import org.apache.hadoop.mapreduce.lib.jobcontrol.JobControl;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class Driver {

	public static void main(String[] args) throws Exception {

		JobConf conf = new JobConf(Driver.class);
		
		//job1设置
		Job job1 = new Job(conf, "job1");
		job1.setJarByClass(Driver.class);
		job1.setMapperClass(IpFilterMapper.class);
		job1.setMapOutputKeyClass(Text.class);
		job1.setMapOutputValueClass(NullWritable.class);
		
		job1.setReducerClass(IpFilterReducer.class);
		job1.setOutputKeyClass(Text.class);
		job1.setOutputValueClass(NullWritable.class);
		FileInputFormat.setInputPaths(job1, new Path(args[0]));
		FileOutputFormat.setOutputPath(job1, new Path(args[1]));
		
		//job1加入控制器
		ControlledJob ctrlJob1 = new ControlledJob(conf);
		ctrlJob1.setJob(job1);
		
		//job2设置
		Job job2 = new Job(conf, "job2");
		job2.setJarByClass(Driver.class);
		job2.setMapperClass(IpCountMapper.class);
		job2.setMapOutputKeyClass(Text.class);
		job2.setMapOutputValueClass(NullWritable.class);
		
		job2.setReducerClass(IpCountReducer.class);
		job2.setOutputKeyClass(Text.class);
		job2.setOutputValueClass(NullWritable.class);
		FileInputFormat.setInputPaths(job2, new Path(args[1]));
		FileOutputFormat.setOutputPath(job2, new Path(args[2]));
		
		//job2加入控制器
		ControlledJob ctrlJob2 = new ControlledJob(conf);
		ctrlJob2.setJob(job2);
		
		//设置作业之间的以来关系，job2的输入以来job1的输出
		ctrlJob2.addDependingJob(ctrlJob1);
		
		//设置主控制器，控制job1和job2两个作业
		JobControl jobCtrl = new JobControl("myCtrl");
		//添加到总的JobControl里，进行控制
		jobCtrl.addJob(ctrlJob1);
		jobCtrl.addJob(ctrlJob2);
		
		
		//在线程中启动，记住一定要有这个
		Thread thread = new Thread(jobCtrl);
		thread.start();
		while (true) {
			if (jobCtrl.allFinished()) {
				System.out.println(jobCtrl.getSuccessfulJobList());
				jobCtrl.stop();
				break;
			}
		}
		
	}

}

````

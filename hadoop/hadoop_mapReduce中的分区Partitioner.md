# MapRedece中的分区Partitioner

## 分析

MapReduce中会将map输出的k-v对，按照相同的key进行分组，然后分发给不同的reduceTask中。
默认的分发规则为：根据key的hashcode%reducetask数来分发
所以如果要按照特定的需求进行分组，则需要改写数据分发组件Partitioner。

## 实现
1. 自定义数据分发类CustomPartitioner 继承抽象类Partitioner
2. 重写getPartition方法

### getPartition方法说明
public int getPartition(Text key, LongWritable value, int numPartitions)
- key	: map阶段输出的key值
- value	: map阶段输出的value值
- numPartitions	: 设置的reduce数量（获取的是job.setNumReduceTasks(int num)设置的num）
- 返回值是根据自定义规则得出的分区位置

### 案例
#### 需求
根据归属地输出流量统计数据结果到不同文件，以便于在查询统计结果时可以定位到省级范围进行流量统计。
#### 实现
```java
/**
* 自定义分发规则类CustomPartitioner
* @author:yanzhelee
*/
public class CustomPartitioner extends Partitioner<Text,LongWritable>{
	//用于将手机号的前三个数字和分区块进行对应
	static HashMap<String, Integer> provinceMap = new HashMap<String, Integer>();

	//设置默认的初始值
	static {
		//key为手机号的前三位数字，value为用于表示地区的分区号
		provinceMap.put("135", 0);
		provinceMap.put("136", 1);
		provinceMap.put("137", 2);
		provinceMap.put("138", 3);
		provinceMap.put("139", 4);

	}

	@Override
	public int getPartition(Text key, FlowBean value, int numPartitions) {

		Integer code = provinceMap.get(key.toString().substring(0, 3));
		//如果不存在，则分区号为5
		return code == null ? 5 : code;
	}
}
```
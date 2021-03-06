# 不适合HDFS的领域

1. 低时间延迟的数据访问
	> 要求低时间延迟数据访问的应用，例如几十毫秒范围，不适合在hdfs上运行。记住，hdfs是为了搞数据吞吐量应用优化的，这可能会以提高时间延迟为代价。目前对于低延迟的访问需求，HBase是更好的选择。 
2. 大量的小文件
	> 由于namenode将文件系统的元数据存储在内存中，因此文件系统所能存储的文件总数受限于namenode的内存容量。根据经验，每个文件、目录和数据块的存储信息大约找150个字节。因此，举例来说，如果有一百万个文件，且每个文件占一个数据块，那至少需要300MB的内存。尽管存储上百万个文件是可行的，但是存储数十亿个文件就超出了当前硬件的能力。
3. 多用户写入，任意修改文件
	> hdfs中的文件可能只有一个writer，而且写操作总是将数据添加在文件的末尾。它不支持具有多个写入者的操作，也不支持在文件的任意位置进行修改。可能以后会支持这些操作，但是它们相对比较低效。

## 参考文献
《HADOOP权威指南第3版》
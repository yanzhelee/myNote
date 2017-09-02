# FileStatus类介绍

FileStatus对象封装了文件系统中文件和目录的元数据，包括文件的长度、块大小、备份数、修改时间、所有者以及权限等信息。

FileStatus对象一般由FileSystem的getFileStatus()方法获得，调用该方法的时候要把文件的Path传递进去。

## FileStatus字段解析

```java
private Path path;                  // Path路径
private long length;                // 文件长度
private boolean isdir;              // 是都是目录
private short block_replication;    // 块的复本数
private long blocksize;             // 块大小
private long modification_time;     // 修改时间
private long access_time;           // 访问时间
private FsPermission permission;    // 权限
private String owner;               // 所有者
private String group;               // 所在组
private Path symlink;               // 符号链接,如果isdir为true那么symlink必须为null
```

## FileStatus构造函数

```java
public FileStatus()
public FileStatus(FileStatus other)
public FileStatus(long length, boolean isdir, int block_replication,
                    long blocksize, long modification_time, Path path)
public FileStatus(long length, boolean isdir,
                    int block_replication,
                    long blocksize, long modification_time, long access_time,
                    FsPermission permission, String owner, String group,
                    Path path)
public FileStatus(long length, boolean isdir,
                    int block_replication,
                    long blocksize, long modification_time, long access_time,
                    FsPermission permission, String owner, String group,
                    Path symlink,
                    Path path)

// 上面所有的构造函数最后都是调用的最后一个构造函数，所以下面只列出最后一个构造函数的源码

public FileStatus(long length, boolean isdir,
                    int block_replication,
                    long blocksize, long modification_time, long access_time,
                    FsPermission permission, String owner, String group,
                    Path symlink,
                    Path path) {
    this.length = length;
    this.isdir = isdir;
    this.block_replication = (short)block_replication;
    this.blocksize = blocksize;
    this.modification_time = modification_time;
    this.access_time = access_time;
    if (permission != null) {
      this.permission = permission;
    } else if (isdir) {
      this.permission = FsPermission.getDirDefault();
    } else if (symlink!=null) {
      this.permission = FsPermission.getDefault();
    } else {
      this.permission = FsPermission.getFileDefault();
    }
    this.owner = (owner == null) ? "" : owner;
    this.group = (group == null) ? "" : group;
    this.symlink = symlink;
    this.path = path;
    // 如果isdir为true则symlink必须为null
    // 如果isdir为false，则表示为是一个文件或者符号链接
    // 如果smylink为null，那么它就是一个文件
    assert (isdir && symlink == null) || !isdir;
   }
```

## FileStatus常用函数介绍

```java
public int compareTo(Object o)          // 比较两个对象是否指向相同的路径
public long getAccessTime()             // 得到访问时间
public long getBlockSize()              // 得到块大小
public String getGroup()                // 得到组名
public long getLen()                    // 得到文件大小
public long getModificationTime()       // 得到修改时间
public String getOwner()                // 获取所有者信息
public Path getPath()                   // 获取Path路径
public FsPermission getPermission()     // 获取权限信息
public short getReplication()           // 获取块副本数
public path getsymlink()                // 获取符号链接的Path路径
public boolean isSymlink()              // 是否为符号链接
public void readFields(DataInput in)    // 序列化读取字段
public void setPath(final Path p)       // 设置Path路径
public void setSymlink(final Path p)    // 设置符号链接
public void write(DataOutput out)       // 序列化写入字段
```

## 代码示例

```java
package com.hdfs;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.FileUtil;
import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.util.Progressable;

public class HdfsTest1 {
    //显示文件所有信息
    public static void fileInfo(String path) throws IOException{
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(conf);
        Path p = new Path(path);
        //FileStatus对象封装了文件的和目录的额元数据，包括文件长度、块大小、权限等信息
        FileStatus fileStatus = fs.getFileStatus(p);
        System.out.println("文件路径："+fileStatus.getPath());
        System.out.println("块的大小："+fileStatus.getBlockSize());
        System.out.println("文件所有者："+fileStatus.getOwner()+":"+fileStatus.getGroup());
        System.out.println("文件权限："+fileStatus.getPermission());
        System.out.println("文件长度："+fileStatus.getLen());
        System.out.println("备份数："+fileStatus.getReplication());
        System.out.println("修改时间："+fileStatus.getModificationTime());
    }
    public static void main(String[] args) throws IOException {
        fileInfo("/user/hadoop/aa.mp4");
    }

}
```

输出结果如下：

```
文件路径：hdfs://master:9000/user/hadoop/aa.mp4
块的大小：67108864
文件所有者：hadoop:supergroup
文件权限：rw-r--r--
文件长度：76805248
备份数：3
修改时间：1371484526483
```

## 参考博文

[http://www.cnblogs.com/liuling/p/2013-6-18-01.html](http://www.cnblogs.com/liuling/p/2013-6-18-01.html)

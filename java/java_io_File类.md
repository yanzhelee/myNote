# JAVA File类总结
# *缺少案例*
## 1 概述

File是“文件”和“目录路径名”的抽象表示形式。
File直接继承Object，实现Serializable接口和Comparable接口。实现Serializable意味着File对象可以进行序列化，实现Compareable意味着File对象可以比较。File能直接被存储在有序的集合(如TreeSet、TreeMap中);

## 2 File参数列表

```java
// 静态成员
public static final String     pathSeparator        // 路径分割符":"
public static final char       pathSeparatorChar    // 路径分割符':'
public static final String     separator            // 分隔符"/"
public static final char       separatorChar        // 分隔符'/'

// 构造函数
File(File dir, String name)
File(String path)
File(String dirPath, String name)
File(URI uri)

// 成员函数
boolean    canExecute()    // 测试应用程序是否可以执行此抽象路径名表示的文件。
boolean    canRead()       // 测试应用程序是否可以读取此抽象路径名表示的文件。
boolean    canWrite()      // 测试应用程序是否可以修改此抽象路径名表示的文件。
int    compareTo(File pathname)    // 按字母顺序比较两个抽象路径名。
boolean    createNewFile()         // 当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件。
static File    createTempFile(String prefix, String suffix)    // 在默认临时文件目录中创建一个空文件，使用给定前缀和后缀生成其名称。
static File    createTempFile(String prefix, String suffix, File directory)    // 在指定目录中创建一个新的空文件，使用给定的前缀和后缀字符串生成其名称。
boolean    delete()             // 删除此抽象路径名表示的文件或目录。
void    deleteOnExit()       // 在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。
boolean    equals(Object obj)   // 测试此抽象路径名与给定对象是否相等。
boolean    exists()             // 测试此抽象路径名表示的文件或目录是否存在。
File    getAbsoluteFile()    // 返回此抽象路径名的绝对路径名形式。
String    getAbsolutePath()    // 返回此抽象路径名的绝对路径名字符串。
File    getCanonicalFile()   // 返回此抽象路径名的规范形式。
String    getCanonicalPath()   // 返回此抽象路径名的规范路径名字符串。
long    getFreeSpace()       // 返回此抽象路径名指定的分区中未分配的字节数。
String    getName()            // 返回由此抽象路径名表示的文件或目录的名称。
String    getParent()          // 返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。
File    getParentFile()      // 返回此抽象路径名父目录的抽象路径名；如果此路径名没有指定父目录，则返回 null。
String    getPath()            // 将此抽象路径名转换为一个路径名字符串。
long    getTotalSpace()      // 返回此抽象路径名指定的分区大小。
long    getUsableSpace()     // 返回此抽象路径名指定的分区上可用于此虚拟机的字节数。
int    hashCode()               // 计算此抽象路径名的哈希码。
boolean    isAbsolute()         // 测试此抽象路径名是否为绝对路径名。
boolean    isDirectory()        // 测试此抽象路径名表示的文件是否是一个目录。
boolean    isFile()             // 测试此抽象路径名表示的文件是否是一个标准文件。
boolean    isHidden()           // 测试此抽象路径名指定的文件是否是一个隐藏文件。
long    lastModified()       // 返回此抽象路径名表示的文件最后一次被修改的时间。
long    length()             // 返回由此抽象路径名表示的文件的长度。
String[]    list()           // 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中的文件和目录。
String[]    list(FilenameFilter filter)    // 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录。
File[]    listFiles()                        // 返回一个抽象路径名数组，这些路径名表示此抽象路径名表示的目录中的文件。
File[]    listFiles(FileFilter filter)       // 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。
File[]    listFiles(FilenameFilter filter)   // 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。
static File[]    listRoots()    // 列出可用的文件系统根。
boolean    mkdir()     // 创建此抽象路径名指定的目录。
boolean    mkdirs()    // 创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。
boolean    renameTo(File dest)    // 重新命名此抽象路径名表示的文件。
boolean    setExecutable(boolean executable)    // 设置此抽象路径名所有者执行权限的一个便捷方法。
boolean    setExecutable(boolean executable, boolean ownerOnly)    // 设置此抽象路径名的所有者或所有用户的执行权限。
boolean    setLastModified(long time)       // 设置此抽象路径名指定的文件或目录的最后一次修改时间。
boolean    setReadable(boolean readable)    // 设置此抽象路径名所有者读权限的一个便捷方法。
boolean    setReadable(boolean readable, boolean ownerOnly)    // 设置此抽象路径名的所有者或所有用户的读权限。
boolean    setReadOnly()                    // 标记此抽象路径名指定的文件或目录，从而只能对其进行读操作。
boolean    setWritable(boolean writable)    // 设置此抽象路径名所有者写权限的一个便捷方法。
boolean    setWritable(boolean writable, boolean ownerOnly)    // 设置此抽象路径名的所有者或所有用户的写权限。
String    toString()    // 返回此抽象路径名的路径名字符串。
URI    toURI()    // 构造一个表示此抽象路径名的 file: URI。
URL    toURL()    // 已过时。 此方法不会自动转义 URL 中的非法字符。建议新的代码使用以下方式将抽象路径名转换为 URL：首先通过 toURI 方法将其转换为 URI，然后通过 URI.toURL 方法将 URI 装换为 URL。
```

## 3 新建目录

### 3.1 根据相对路径新建目录

```java
//在当前目录下新建dir文件夹
File dir = new File("dir");
dir.mkdir();
```

### 3.2 根据绝对路径新建目录

```java
//递归创建目录
File dir = new File("d:/1/2/dir");
dir.mkdirs();
```

### 3.3 使用URI创建目录

```java
URI uri = new URI("file:/d:/1/2/dir");
File dir = new File(uir);
dir.mkdirs();
```

## 4 新建子目录的常用方法

### 4.1 方法一

```java
//要求dir目录必须存在
File sub = new File("dir","sub");
sub.mkdir();
```

### 4.2 方法二

```java
//下面的dir是File对象
File sub = new File(dir, "sub");
sub.mkdir();
```

### 4.3 方法三

```java
//递归创建子目录，如果dir不存在那么创建dir目录
File sub = new File("dir/sub");
sub.mkdirs();
```

### 4.4 方法四

```java
File sub = new File(new URI("file:/d:/1/2/dir/sub"));
sub.mkdirs();
```

## 5 新建文件

### 5.1 方法一

```java
try{
    File dir = new File("dir");
    File file = new File(dir, "file.txt");
    file.createNewFile();
}catch(IOException e){
    e.printStackTrace();
}
```

### 5.2 方法二

```java
try{
    File file = new File("d:/1/2/dir/file.txt");
    file.createNewFile();
}catch(IOException e){
    e.printStackTrace();
}
```


## 参考博文

[http://www.cnblogs.com/skywang12345/p/io_08.html](http://www.cnblogs.com/skywang12345/p/io_08.html)

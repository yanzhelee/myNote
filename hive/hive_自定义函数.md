# Hive自定义函数

## 1 UDF

用户自定义函数(user defined function)针对单条记录。

### 1.1 创建函数流程

1. 添加pom依赖
2. 自定义一个java类
3. 继承UDF类
4. 重写evaluate方法
5. 打成jar包
6. 在hive中执行add jar方法
7. 在hive执行创建模板函数

### 1.2 实例一

**第一步、添加依赖**
```
<?xml version="1.0" encoding="UTF-8"?>
	<dependencies>
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>2.1.0</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
		</dependency>
	</dependencies>
</project>
```
**第二步、自定义java类**
```java
package udfdemo;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class UDFTest extends UDF{
  public int evaluate(int a,int b){
    return a + b;
  }

  public int evaluate(int a, int b, int c){
    return a + b + c;
  }
}
```
**第三步、打成jar包**

**第四步、在hive客户端中执行add jar方法**

**第五步、在hive创建的函数，引入的类是UDF.UDFTest**

```
hive>add jar /home/centos/UDFTest.jar
// 创建临时函数
hive>create  temporary function add as 'UDFDemo.UDFTest';
```
**第六步、调用**
```
hive>select add(1,2);
hvie>select add(1,2,3);
```

## 1.3 实例二(自定义处理时间戳函数)

**添加pom文件 略。。。**

**时间处理工具类**
```java
package dateutil;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

public class DateUtil{
  /**
   * 得到指定日期的起始时刻
   */
  public static long getDayBegin(Date date, int offset){
    try{
      // 日历类
      Calendar c = Calendar.getInstance() ;
      c.setTime(date) ;
      // 时间增减
      c.add(Calendar.DAY_OF_MONTH,offset) ;
      Date newDate = c.getTime() ;
      SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM//dd 00:00:00");
      return sdf.parse(sdf.format(newDate)).getTime();
    }catch(Exception e){
      e.printStackTrace();
    }
    return -1 ;
  }

  /**
   * 得到指定日期所在月份的月初零点时刻(即所在月的1号的零点)
   */
   public static long getMonthBegin(Date date, int offset){
     try{
       // 日历类
       Calendar c = Calendar.getInstance() ;
       c.setTime(date) ;
       c.add(Calendar.MONTH, offset) ;

       SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/01 00:00:00") ;
       String ymd = sdf.format(c.getTime());

       return sdf.parse(ymd).getTime();
     }catch(Exception e){
       e.printStackTrace();
     }
     return -1 ;
   }

   /**
    * 得到制定日期的所在周的周日的零时刻
    */
    public static long getWeekBegin(Date date, int offset){
      try{
        Calendar c = Calendar.getInstance() ;
        c.setTime(date) ;
        int n = c.get(Calendar.DAY_OF_WEEK) ;
        c.add(Calendar.DAY_OF_MONTH, -(n-1));
        c.add(Calendar.DAY_OF_MONTH,  offset * 7) ;

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd 00:00:00");
        String ymd = sdf.format(c.getTime());
        return sdf.parse(ymd).getTime();

      }catch(Exception e){
        e.printStackTrace() ;
      }
      return -1 ;
    }
}

```

**自定义函数，获取天的起始时刻**
```java
import dateutil;
import org.apache.hadoop.hive.ql.exec.UDF;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
 * 自定义函数，获取天的起始时刻
 */
public class DayBeginUDF extends UDF{
  /**
	 *今天零时刻
	 */
	public long evaluate() {
		Date date = new Date();
		return evaluate(date, 0);
	}

	public long evaluate(int offset) {
		Date date = new Date();
		return evaluate(date, offset);
	}

	public long evaluate(long timestamp,int offset) {
		Date date = new Date(timestamp) ;
		return evaluate(date , offset) ;
	}

	public long evaluate(Date date,int offset) {
		return DateUtil.getDayBegin(date , offset) ;
	}

	public long evaluate(String date , String fmt ,int offset) {
		try{
			SimpleDateFormat sdf = new SimpleDateFormat(fmt) ;
			return DateUtil.getDayBegin(sdf.parse(date) , offset) ;
		}
		catch(Exception e){
			e.printStackTrace();
		}
		return -1 ;
	}
}
```
**格式化时间串函数**
```java
import dateutil;
import org.apache.hadoop.hive.ql.exec.UDF;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 格式串函数
 */
public class FormatTimeUDF extends UDF{

	public String evaluate(String fmt) {
		SimpleDateFormat sdf = new SimpleDateFormat(fmt) ;
		return sdf.format(new Date()) ;
	}

	public String evaluate(long timestamp,String fmt) {
		SimpleDateFormat sdf = new SimpleDateFormat(fmt);
		return sdf.format(new Date(timestamp));
	}

	public String evaluate(Date date , String fmt) {
		SimpleDateFormat sdf = new SimpleDateFormat(fmt);
		return sdf.format(date);
	}
}
```
**自定义函数,获取月的起始时刻**
```java
import dateutil;
import org.apache.hadoop.hive.ql.exec.UDF;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 自定义函数,获取月的起始时刻
 */
public class MonthBeginUDF extends UDF {
	/**
	 *今天零时刻
	 */
	public long evaluate() {
		Date date = new Date();
		return evaluate(date, 0);
	}

	public long evaluate(int offset) {
		Date date = new Date();
		return evaluate(date, offset);
	}

	public long evaluate(long timestamp,int offset) {
		Date date = new Date(timestamp) ;
		return evaluate(date , offset) ;
	}

	public long evaluate(Date date,int offset) {
		return DateUtil.getMonthBegin(date , offset) ;
	}

	public long evaluate(String date , String fmt ,int offset) {
		try{
			SimpleDateFormat sdf = new SimpleDateFormat(fmt) ;
			return DateUtil.getMonthBegin(sdf.parse(date) , offset) ;
		}
		catch(Exception e){
			e.printStackTrace();
		}
		return -1 ;
	}
}
```
**自定义函数，获取周的起始数据**
```java
import dateutil;
import com.it18zhang.umeng.util.DateUtil;
import org.apache.hadoop.hive.ql.exec.UDF;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 自定义函数,获取周的起始时刻
 */
public class WeekBeginUDF extends UDF {
	/**
	 *今天零时刻
	 */
	public long evaluate() {
		Date date = new Date();
		return evaluate(date, 0);
	}

	public long evaluate(int offset) {
		Date date = new Date();
		return evaluate(date, offset);
	}

	public long evaluate(long timestamp,int offset) {
		Date date = new Date(timestamp) ;
		return evaluate(date , offset) ;
	}

	public long evaluate(Date date,int offset) {
		return DateUtil.getWeekBegin(date , offset) ;
	}

	public long evaluate(String date , String fmt ,int offset) {
		try{
			SimpleDateFormat sdf = new SimpleDateFormat(fmt) ;
			return DateUtil.getWeekBegin(sdf.parse(date) , offset) ;
		}
		catch(Exception e){
			e.printStackTrace();
		}
		return -1 ;
	}
}
```

**启动thriftserver时，通过--jars指定jar包**
```
start-thriftserver.sh --num-executors 12 --executor-cores 4 --conf spark.task.cpus=1 --executor-memory=4g --master yarn-client --jars udf.jar
```

**注册函数**
```
beeline中注册
$beeline>create function getdaybegin as 'hive.udf.DayBeginUDF' ;
$beeline>create function getweekbegin as 'hive.udf.WeekBeginUDF' ;
$beeline>create function getmonthbegin as 'hive.udf.MonthBeginUDF' ;
$beeline>create function formattime as 'hive.udf.FormatTimeUDF' ;
```

## 2 UDAF

UDAF(user defined aggregation function)用户自定义聚合函数，针对记录集合

### 2.1 开发UDAF流程

第一是编写resolver类，resolver负责类型检查，操作符重载。

第二是编写evaluator类，evaluator真正实现UDAF的逻辑。














## 参考博文

[http://blog.csdn.net/scgaliguodong123_/article/details/46993005](http://blog.csdn.net/scgaliguodong123_/article/details/46993005)

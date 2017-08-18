# JAVA线程生命周期

## 摘要

本文详细总结了java线程的五种基本状态，和状态之间的转换关系；介绍了常见了创建线程的两种方法，一种是通过继承Thead类并从写run()函数的方式，另一种是通过实现Runnable接口的方法；最后介绍了常见的线程状态控制函数，比如`sleep()`,`yield()`,`join()`,`setPriority(int newPriority)`,`setDaemon()`方法。

## 1 java线程的五种基本状态

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/java/java_%E5%A4%9A%E7%BA%BF%E7%A8%8B_%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F_1.jpg)


- 新建状态（New） 
	> 当线程对象创建后，即进入新建状态，如：`Thread t = new MyThread();`
	
- 就绪状态（Runnable）
	> 当调用线程对象的`start()`方法时，线程即进入就绪状态。处于就绪状态的线程只是说明此线程已经做好准备，随时等待CPU调度执行，并不是说执行了`start()`方法就立即执行。
	
- 运行状态（Running）
	> 当CPU开始调度处于就绪状态的线程时，此时线程才得以真正执行，即进入到运行状态。
	
- 阻塞状态（Blocked）
	> 处于运行状态中的线程由于某种原因，暂时放弃对CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才有机会再次被CPU调用以进入到运行状态。
	>
	> **阻塞状态分类**
	> 1. 等待阻塞：运行状态中的线程执行wait()方法，使本线程进入到等待阻塞状态；
	> 2. 同步阻塞：线程在获取`synchronized`同步锁失败（因为锁被其它线程占用），它会进入到同步阻塞状态；
	> 3. 其他阻塞：通过调用线程的sleep()或join()或发出I/O请求时，线程会进入到阻塞状态。当`sleep()`状态超时、`join()`等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

- 死亡状态
	> 线程执行完毕或者是异常退出，该线程结束生命周期。
	
## 2 java线程创建方式

### 2.1 继承`Thread`类并覆盖`run()`方法

```java
//定义线程类
public class MyThread extends Thread{
	run(){
		System.out.println("This is myThread");
	}
}

//线程是通过start()方法调用的
new MyThread().start();
```

### 2.2 实现`Runnalbe`接口并重写`run()`方法

```java
//定义线程类
public class MyThread implements Runnable{
	run(){
		System.out.println("This is myThread");
	}
}

//线程是通过start()方法调用的
new MyThread().start();
```

## 3 java多线程的就绪、运行和死亡状态

- 就绪状态转换为运行状态：该线程得到处理器资源；
- 运行状态转换为就绪状态：当此线程主动调用`yield()`方法或在运行过程中失去处理器资源。
- 运行状态转换为死亡状态：当此线程执行执行完毕或者发生了异常。

**注意：**当调用线程中的`yield()`方法时，线程从运行状态转换为就绪状态，但接下来CPU调度就绪状态中的那个线程具有一定的随机性，因此，可能会出现A线程调用了`yield()`方法后，接下来CPU仍然调度了 A线程的情况。

## 4 线程状态控制

### 4.1 线程休眠-sleep()

sleep函数会让当前的线程暂停一段时间，并进入阻塞状态，调用sleep并不会释放锁。
**注意：**
- sleep是静态方法，最好不要用Thread的实例对象调用它，因为它睡眠的始终是当前正在运行的线程，而不是调用它的线程对象。
- 调用sleep方法后当前线程的休眠时间不会完全精确到设置的时间参数，因为只有当睡眠的时间结束，才会重新进入到就绪状态，而就绪状态进入到运行状态，是由系统控制的，程序员没法精准的干预，所以使用`Thread.sleep(100)`,实际结果会大于100毫秒。

```java
public class ThreadTest {  
    public static void main(String[] args) throws InterruptedException {  
        for(int i=0;i<100;i++){  
            System.out.println("main"+i);  
            Thread.sleep(1000);//休眠1秒  
        }  
    }  
} 
```

### 4.2 线程让步-yield()

调用`yield()`方法之后，从运行状态转换到就绪状态，CPU从就绪状态队列中只会选择与该线程优先级相同或者是优先级更高的线程去执行。

**TIP：**sleep()方法和yield()方法的区别
- sleep()方法暂停当前线程后，会进入到阻塞状态，只有睡眠时间到了，才会转入到就绪状态，而yield方法调用后，是直接进入到就绪状态，所以有可能刚进入就绪状态，又被调度到运行状态。
- sleep方法生命抛出InterruptedException，所以调用sleep方法的时候岩捕获异常，或者生命抛出异常，而yield方法是不需要抛出异常。
- sleep方法比yield方法有更好的可移植性，通常不要依靠yield方法控制并发线程的执行。

```java
public class ThreadTest {  
    public static void main(String[] args) throws InterruptedException {  
        new MyThread("低级", 1).start();  
        new MyThread("中级", 5).start();  
        new MyThread("高级", 10).start();  
    }  
}  
  
class MyThread extends Thread {  
    public MyThread(String name, int pro) {  
        super(name);// 设置线程的名称  
        this.setPriority(pro);// 设置优先级  
    }  
  
    @Override  
    public void run() {  
        for (int i = 0; i < 30; i++) {  
            System.out.println(this.getName() + "线程第" + i + "次执行！");  
            if (i % 5 == 0)  
                Thread.yield();  
        }  
    }  
} 
```

### 4.3 线程合并-join()

线程合并就是将几个并发线程合并为一个单一线程执行，应用场景就是当一个线程的执行必须是要等到其他线程执行完毕之后才能执行。
join()方法有三个重载

  方法 | 说明
-------------------------|-----
`void join()`| 调用`myThread.join()；`语句的线程会等到myThread线程执行完毕后它才能执行
`void join(long mills)` | 当前线程等待该线程终止的最大时间为mills毫秒，即使该线程没有执行完毕，那么当前线程也会进入就绪状态，重新等待CPU的调用
`void join(long mills，int nanos)` | 当前线程等待该线程终止的最大时间为mills毫秒+nanos纳秒，即使该线程没有执行完毕，那么当前线程也会进入就绪状态，重新等待CPU的调用

```java
//玩家线程类
class PlayerThread extends Thread{
	public String name ;
	public int time ;
	public MyThread(String name, int time){
		this.name = name ;
		this.time = time ;
	}
	run(){
		try{
			Thread.sleep(time);

		}catch(Exception e){
		}
		System.out.println(name + "用了" + time + "时间到了");
	}
}

//庄家类，庄家需要召集四个玩家然后开始打麻将
public class Banker{
	public static void main(String[] args){
		PlayerThread p1 = new PlayerThread("李小龙",5);
		PlayerThread p2 = new PlayerThread("成龙",10);
		PlayerThread p3 = new PlayerThread("马龙",1);
		PlayerThread p4 = new PlayerThread("狄龙",15);

		p1.start();
		p2.start();
		p3.start();
		p4.start();

		p1.join();
		p2.join();
		p3.join();
		p4.join();

		//下面这段代码需要等到p1,p2,p3,p4线程全部执行完毕之后才会别调用
		System.out.println("人到齐了，开始打麻将！！！");
	}
}

```

### 4.4 线程优先级设置-priority

每个线程执行时都有一个优先级的属性，优先级高的线程可以获得较多的执行机会，而优先级低的线程则获得较少的执行机会。线程的优先级仍然无法保证线程的执行次序。只是优先级高的线程获取CPU资源的概率比较大，优先级比较低的线程也并非没有机会执行。
每个线程默认的优先级都与创建它的父线程具有相同的优先级，在默认的情况下main线程具有普通的优先级。
Thread类提供了`setPriority(int newPriority)`和`getPriority()`方法设置和返回优先级。优先级的范围是1-10；Thread类中提供了三个静态常量：
- MAX_PRIORITY = 10
- MIN_PRIORITY = 1
- NORM_PRIORITY = 5

举例:

```java
public class Test {  
    public static void main(String[] args) throws InterruptedException {  
        new MyThread("高级", 10).start();  
        new MyThread("低级", 1).start();  
    }  
}  
  
class MyThread extends Thread {  
    public MyThread(String name,int pro) {  
        super(name);//设置线程的名称  
        setPriority(pro);//设置线程的优先级  
    }  
    @Override  
    public void run() {  
        for (int i = 0; i < 100; i++) {  
            System.out.println(this.getName() + "线程第" + i + "次执行！");  
        }  
    }  
} 
```

### 4.5 守护线程-Daemon

守护线程是为其他非守护线程提供服务的，比如 JVM中的垃圾回收线程就是守护线程。
生命周期：守护线程的生命周期与非守护线程的生命周期相关。当**所有**的前台线程都进入死亡状态时，守护线程会自动死亡。也就是说守护线程就是为前台线程提供服务的，前台线程都已经不存在了，所以也就没有守护的必要了。

设置守护线程：调用Thead实例的setDaemon(true)方法可以将指定的线程设置为守护线程。
如果正在运行的线程都是守护线程，JVM退出。
**注意：**`setDaemon(true);`语句必须在开启线程前进行调用。

```java
/** 
* Java线程：线程的调度-守护线程 
*/  
public class Test {  
        public static void main(String[] args) {  
                Thread t1 = new MyCommon();  
                Thread t2 = new Thread(new MyDaemon());  
                t2.setDaemon(true);        //设置为守护线程  
  
                t2.start();  
                t1.start();  
        }  
}  
  
class MyCommon extends Thread {  
        public void run() {  
                for (int i = 0; i < 5; i++) {  
                        System.out.println("线程1第" + i + "次执行！");  
                        try {  
                                Thread.sleep(7);  
                        } catch (InterruptedException e) {  
                                e.printStackTrace();  
                        }  
                }  
        }  
}  
  
class MyDaemon implements Runnable {  
        public void run() {  
                for (long i = 0; i < 9999999L; i++) {  
                        System.out.println("后台线程第" + i + "次执行！");  
                        try {  
                                Thread.sleep(7);  
                        } catch (InterruptedException e) {  
                                e.printStackTrace();  
                        }  
                }  
        }  
}
```

## 参考博文
[http://www.cnblogs.com/lwbqqyumidi/p/3804883.html](http://www.cnblogs.com/lwbqqyumidi/p/3804883.html "Java总结篇系列：Java多线程（一）")

[http://blog.csdn.net/lonelyroamer/article/details/7949969](http://blog.csdn.net/lonelyroamer/article/details/7949969)

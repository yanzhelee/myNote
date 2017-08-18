# JAVA线程同步锁和线程间协调操作

由于同一进程的多个线程是共享内存的，当多个线程并发操作同一对象是就会导致数据安全问题。
例如：

```java
public class ThreadTest {

    public static void main(String[] args) {
        Account account = new Account("123456", 1000);
        DrawMoneyRunnable drawMoneyRunnable = new DrawMoneyRunnable(account, 700);
        Thread myThread1 = new Thread(drawMoneyRunnable);
        Thread myThread2 = new Thread(drawMoneyRunnable);
        myThread1.start();
        myThread2.start();
    }

}

class DrawMoneyRunnable implements Runnable {

    private Account account;
    private double drawAmount;

    public DrawMoneyRunnable(Account account, double drawAmount) {
        super();
        this.account = account;
        this.drawAmount = drawAmount;
    }

    public void run() {
        if (account.getBalance() >= drawAmount) {  //1
            System.out.println("取钱成功， 取出钱数为：" + drawAmount);
            double balance = account.getBalance() - drawAmount;
            account.setBalance(balance);
            System.out.println("余额为：" + balance);
        }
    }
}

class Account {

    private String accountNo;
    private double balance;

    public Account() {

    }

    public Account(String accountNo, double balance) {
        this.accountNo = accountNo;
        this.balance = balance;
    }

    public String getAccountNo() {
        return accountNo;
    }

    public void setAccountNo(String accountNo) {
        this.accountNo = accountNo;
    }

    public double getBalance() {
        return balance;
    }

    public void setBalance(double balance) {
        this.balance = balance;
    }

}
```

上面例子很容易理解，有一张银行卡，里面有1000的余额，程序模拟你和你老婆同时在取款机进行取钱操作的场景。多次运行此程序，可能具有多个不同组合的输出结果。其中一种可能的输出为：

```
取钱成功， 取出钱数为：700.0
余额为：300.0
取钱成功， 取出钱数为：700.0
余额为：-400.0
```

也就是说，对于一张只有1000余额的银行卡，你们一共可以取出1400，这显然是有问题的。

经过分析，问题在于Java多线程环境下的执行的不确定性。CPU可能随机的在多个处于就绪状态中的线程中进行切换，因此，很有可能出现如下情况：当thread1执行到//1处代码时，判断条件为true，此时CPU切换到thread2，执行//1处代码，发现依然为真，然后执行完thread2，接着切换到thread1，接着执行完毕。此时，就会出现上述结果。

为了解决上述问题就需要要到 多线程的同步机制。

## 1 同步与锁定

### 1.1 锁的原理

java中的每个对象都有一个内置的锁，当程序运行在非静态`synchronized`修饰的方法上时，自动获得与正在执行的当前实例（this）有关的锁。获得一个对象的锁也称为获取锁、锁定对象、在对象上锁定或在对象上同步。
当程序运行到同步方法或者同步代码块时该对象上的锁才起作用。

一个对象只有一个锁，所以，如果一个线程获得了该对象的锁，那么在同一时刻就不会有其它线程获得该锁，直到第一个线程释放锁。这也意味着任何其它线程都不能进入该对象上的同步方法或者同步代码块，直到该锁被释放。
释放锁是指线程退出了synchronized方法或代码块。

**注意要点**

1. 只能同步方法，不能同步变量和类；
2. 每个对象只有一个锁。
3. 不必同步类中所有的方法，类可以同时拥有同步和非同步方法。
4. 如果两个线程要执行一个类中的synchronized方法，并且两个线程使用相同的实例调用方法，那么同一时刻只能有一个线程能够执行同步方法或同步代码块，另一个需要等待，直到锁被释放。也就是说：如果一个线程在对象上获得了锁，就不可能有任何其它的线程可以进入类中任何一个同步方法。
5. 线程可以获得多个锁。比如，在一个对象的同步方法里面调用另一个对象的同步方法，则获得了两个对象的同步锁。
6. 同步损害并发性，应该尽可能的缩小使用范围。

### 1.2 静态方法同步

前面讲到*非静态同步方法是以当前对象(this)为锁*，那么静态同步方法是以什么作为锁呢？
静态同步方法是以整个类对象为锁，这个对象就是这个类(xxx.class)。
例如：

```java
public static synchronized void setAge(int age){
	Xxx.name = name ;
}

//上面的函数等价于下面
public static void setAge(int age){
	synchronized(Xxx.class){
		Xxx.name = name ;
	}
}
```

## 2 如果线程不能获得锁会怎样

如果线程视图进入同步方法，而其锁已经被占用，则线程在该对象上被阻塞。实质是，线程进入该对象的一种池中，必须在那里等待，直到其锁被释放，该线程再次变为可运行。

当考虑阻塞时，一定要注意那个对象正被用于锁定：
1. 调用同一对象中非静态同步方法的线程彼此阻塞。如果不是同一对象，则每个线程有自己的对象锁，线程间彼此互补干扰。
2. 调用同一类中的静态同步方法的线程彼此阻塞，因为它们都是锁在了类对象上(Xxx.class对象).
3. 静态同步方法和非静态同步方法将永远不会彼此阻塞，因为静态方法锁定在Class对象上。非静态方法锁定在该类的对象上。
4. 对于同步代码块，要看清什么对象已经用于锁定(synchronized后面括号中的内容)。在同一对象上进行同步的线程将彼此阻塞，在不同的 对象上锁定的线程将永远不会彼此阻塞。

代码举例

```java
//在该类中定义了两个同步方法，并且每个方法都是死循环
public class SyncMethod {
    public synchronized void sayHello(){
        while (true){
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

            System.out.println("hello");
        }

    }
    public synchronized void sayHi(){
        while (true) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("hi");
        }
    }
}

//定义一个线程类
public class MyThread extends Thread {
    private int num ;//通过num控制调用那个方法
    private SyncMethod syncMethod;
    private int i = 0 ;

    public MyThread(int num, SyncMethod syncMethod) {
        this.num = num;
        this.syncMethod = syncMethod;
    }

    @Override
    public void run() {
        if (num == 1) {
           syncMethod.sayHello();
        } else {
            syncMethod.sayHi();
        }
    }
}

//主函数用于测试（Tip：用单元测试有问题）
public static void main(String[] args){
        SyncMethod syncMethod = new SyncMethod();
        MyThread m1 = new MyThread(1, syncMethod);
        MyThread m2 = new MyThread(2, syncMethod);
        m1.start();
        m2.start();
    }
```

运行结果
```
hello
hello
hello
hello
hello
hello
hello
hello
全都是hello
```

**上述代码可以说明同一个对象只有一把锁，一旦这把锁被线程获取了，那么其他线程就无法通过任何其他同步方法或是同步代码块去获得该对象的锁。**

## 3 死锁

死锁：多个线程同时被阻塞，他们中的一个或者全部都在等待某个资源的被释放。由于线程被无限期的阻塞，因此程序不能正常运行。或者是线程都调用了wait()方法也可能会导致死锁。
举例1：
```java
public class DeadLock2 {  
    public static void main(String[] args) {  
        Object object1=new Object();  
        Object object2=new Object();  
        new Thread(new T(object1,object2)).start();  
        new Thread(new T(object2,object1)).start();  
    }  
}  
class T implements Runnable{  
    private Object object1;  
    private Object object2;  
    public T(Object object1,Object object2) {  
        this.object1=object1;  
        this.object2=object2;  
    }  
    public void run() {  
        synchronized (object1) {  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            synchronized (object2) {  
                System.out.println("无法执行到这一步");  
            }  
        }  
    };  
}  
```

上面的就是个死锁。第一个线程首先锁住了object1，然后休眠。接着第二个线程锁住了object2，然后休眠。在第一个线程企图在锁住object2，进入阻塞。然后第二个线程企图在锁住object1，进入阻塞。死锁了。

举例2：

```java
import java.util.LinkedList;
import java.util.List;


public class ProducerConsumerDemo {
	public static void main(String[] args) {
		Pool p = new Pool();
		Producer p1 = new Producer("p1",p);
		Producer p2 = new Producer("p2",p);
		Consumer c1 = new Consumer("c1" , p);
		p1.start();
		p2.start();
		c1.start();
	}
}

/**
 * 池子
 */
class Pool{
	//将池子中的最大值设为1
	private int MAX = 1 ;
	private List<Integer> list = new LinkedList<Integer>();

	/**
	 *
	 * @param i
	 */
	public synchronized void add(Integer i){
		while(list.size() >= MAX){
			try {
				this.wait();
				System.out.println("xxx");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		list.add(0, i);
		this.notify();
	}

	/**
	 * 剪切
	 */
	public synchronized Integer remove(){
		while(list.isEmpty()){
			try {
				this.wait();
			} catch (InterruptedException e) {

				e.printStackTrace();
			}
		}
		Integer i = list.remove(0);
		this.notify();
		return i ;
	}
}

/**
 * 生产者
 */
class Producer extends Thread{

	private static int index = 0 ;

	private String name0;
	private Pool pool ;

	public Producer(String name0 ,Pool pool) {
		this.pool = pool;
		this.name0 = name0 ;
	}

	public void run() {
		while(true){
			int tmp = index ;
			index ++ ;
			pool.add(tmp);
			System.out.println(name0 + " produced : " + tmp );
		}
	}
}

/**
 * 消费者
 */
class Consumer extends Thread{

	private String name0 ;

	private static int index = 0 ;
	private Pool pool ;

	public Consumer(String name0 ,Pool pool) {
		this.pool = pool;
		this.name0 = name0 ;
	}

	public void run() {
		while(true){
			Integer i = pool.remove();
			if(i != null){
				System.out.println(name0 + " consumed : " + i);
			}
		}
	}
}
```
上述代码也会导致死锁，这种死锁是由于所有的线程都进入到了等待队列中，而且没有其他线程对他们唤醒。

## 4 线程的协调运行

通过同步和锁的方法可以解决线程安全问题，但是解决不了生产消费问题。比如生产者不断的生产馒头，放入篮子中，消费者不断的从篮子中 取馒头。并且，当篮子满的时候生产者通知消费者过来吃，并且自己等待，不再生成馒头。如果消费者取走馒头后，需要通知生产者继续生产。这样的问题 就需要线程之间的协调运行才能解决。

### 4.1 线程等待-wait()方法

wait方法有三个重载

函数         | 说明
-------------|--------
`wait()` | 在同步中调用`wait()`方法会使当前线程放弃对象的锁定权并且进入等待队列中，直到被其他线程通知，该线程才能继续运行。
`wait(long timeout)`|  在同步中调用`wait(long timeout)`方法会使当前线程放弃对象的锁定权并且进入等待队列中，直到被其他线程通知或者经过了timeout微秒后，该线程才能继续运行。
`wait(long timeout, int nanos)`| 在同步中调用`wait(long timeout)`方法会使当前线程放弃对象的锁定权并且进入等待队列中，直到被其他线程通知或者经过了timeout微秒+nanos纳秒后，该线程才能继续运行。


### 4.2 单个通知-notify()

唤醒在此同步监视器上等待的单个线程。如果所有线程都在此同步监视器上等待，则会选择幻想其中一个线程。选择是任意性。只有当前线程放弃对该同步监视器的锁定后(使用wait()方法)，才可以执行被唤醒的其他线程。

### 4.3 全通知-notifyAll()

唤醒在此同步监视器上等待的所有线程。只有当前线程放弃对该同步监视器的锁定后，才可以执行被唤醒的线程。

**注意：**
wait、notify和notifyAll方法一定要在同步代码中使用，在其它位置不能使用。
1. 如果两个线程是因为都要得到同一个对象的锁，而导致其中一个线程进入阻塞状态。那么只有等获得锁的线程执行完毕，或者它执行了该锁对象的wait方法，阻塞的线程才会有机会得到锁，继续执行同步代码块。
2. 使用wait方法进入等待状态的线程，会释放掉锁。并且只有其他线程调用notify或者notifyAll方法，才会被唤醒。
3. 线程因为锁阻塞和等待是不同的，因为锁进入阻塞状态，会在其他线程释放锁的时候，得到锁在执行。而等待状态必须要靠别人唤醒，并且唤醒了也不一定会立刻执行，有可能因为notifyAll方法使得很多线程被唤醒，多个线程等待同一个锁，而进入阻塞状态。还可能是调用notify的线程依然没有释放掉锁，只有等他执行完了，其他线程才能去争夺这个锁。
4. notify()/notifyAll()方法执行后，将唤醒此同步锁对象上的（任意一个-notify()/所有-notifyAll()）线程对象，但是，此时还并没有释放同步锁对象，也就是说，如果notify()/notifyAll()后面还有代码，还会继续进行，知道当前线程执行完毕才会释放同步锁对象；
5. 当wait线程唤醒后并执行时，是接着上次执行到的wait()方法代码后面继续往下执行的。

### 4.4 区分wait()方法和sleep()方法

sleep 是线程类（Thread）的方法，导致此线程暂停执行指定时间，给执行机会给其他线程，但是监控状态依然保持，到时后会自动恢复。调用sleep 不会释放对象锁。

wait 是Object 类的方法，对此对象调用wait 方法导致本线程放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象发出notify 方法（或notifyAll）后本线程才进入对象锁定池准备获得对象锁进入运行状态。

wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用(使用范围)

## 参考博文

[http://lavasoft.blog.51cto.com/62575/99155/](http://lavasoft.blog.51cto.com/62575/99155/)

[http://blog.csdn.net/LonelyRoamer/article/details/7956097](http://blog.csdn.net/LonelyRoamer/article/details/7956097)

[http://www.cnblogs.com/lwbqqyumidi/p/3821389.html](http://www.cnblogs.com/lwbqqyumidi/p/3821389.html)

[http://blog.csdn.net/congqingbin/article/details/7871862](http://blog.csdn.net/congqingbin/article/details/7871862)
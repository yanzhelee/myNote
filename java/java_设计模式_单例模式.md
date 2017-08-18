# JAVA设计模式-单例模式

单例模式是为了确保某一个类只有一个实例，并且通过类的静态方法获取到唯一的实例。一些管理器和控制器通常被设计成单例模式。

## 1 饿汉模式

```java
public classs Singleton{
	//私有的静态成员直接初始化
	private static Singleton instance = new Singleton();
	//构造函数私有化
	private Singleton(){}

	//静态工厂方法
	public static Singleton getInstance(){
		retrun instance;
	}
}
```
上面的代码就是典型的饿汉式，在类加载的时候静态变量instance就会被初始化，此时类的私有构造函数会被调用。这个时候，单例类的唯一实例就被创建出来了。

饿汉式其实是一种比较形象的称谓。**饿汉式是典型的空间换时间**，当类加载的时候就会创建类的实例，不管你用还是不用，实例对象就在那里，每次调用的时候就不需要判断，节省了运行时间。

## 2 懒汉模式

### 2.1 静态同步方法创建实例
```java
public class Singleton{
	private static Singleton instance = null ;

	//构造函数私有
	private Singleton(){}

	//静态工厂方法
	public static synchronized Singleton getInstance(){
		if(instance == null){
			instance = new Singleton();
		}
		return instance ;
	}
}
```
上面的懒汉模式需要对静态工厂方法加同步，因为如果不加同步，在多线程访问时可能会导致创建出两个实例对象。这样就不是单例模式了。
懒汉式是**典型的时间换空间**就是每次获取实例都会进行判断，看是否需要创建实例。如果没有人用的话就不会创建实例，节约了内存空间。

通过同步方法的方式实现懒汉式是一种比较浪费时间的，因为每次获取实例都会进行判断，降低了访问速度。所以懒汉式还有一种更好的实现方式。

### 2.2 双重检查加锁

可以通过“双重检查加锁”的方式既可以实现线程安全，又能使性能不受很大的影响。

```java
public class Singleton{
	private static Singleton instance = null ;

	//构造函数私有
	private Singleton(){}

	//静态工厂方法
	public static Singleton getInstance(){

		//先判断实例实例是否存在
		if(instance == null){
			//同步代码块，线程安全创建实例
			synchronized(Singleton.class){
				//再次进行判断，如果不存在，那么创建实例
				if(instance == null){
					instance = new Singleton();
				}
			}
		}
		
		return instance ;
	}
}
```

这种实现方式既可以实现线程安全，又对性能不会有太大的影响。它只在第一次创建实例的时候同步，以后就不需要进行同步，从而加快了运行的速度。

## 参考博文

[http://www.cnblogs.com/java-my-life/archive/2012/03/31/2425631.html](http://www.cnblogs.com/java-my-life/archive/2012/03/31/2425631.html)
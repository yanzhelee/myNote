# JAVA设计模式-装饰模式

装饰模式就是添加一些额外的功能（装饰作用）。装饰使得我们可以动态的为对象添加一些功能，而无需事先在类中定义。
装饰模式结构图如下：

![](http://i.imgur.com/JyxvkgO.png)

## 1 概述

**1.设计意图：** 
> 动态的为对象添加一些额外的功能。就增加功能来说，Decorator模式相比生成子类更为灵活。该模式以对客户端透明的方式扩展对象的功能。

**2.适用环境**

> 1. 在不影响其它对象的情况下，以动态、透明的方式给单个对象添加职责。
> 2. 处理那些可以撤销的职责。
> 3. 当不能采用生成子类的方式进行扩展时。一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长。另一种情况可能 是因为类定义被隐藏，或类定义不能用于生成子类。

**3.参与者**

> 1. Component(被装饰对象的接口)
> 2. ConcreteComponent(具体被装饰的对象基类)
> 3. Decorator(装饰者抽象类)
> 4. ConcreteDecotator(具体装饰者)：具体的装饰类，给内部持有的具体被装饰对象，增加具体的职责。

## 2 装饰模式的特点

1. 装饰对象和真实的对象有相同的接口。这样客户端对象就能以真实对象相同的方式和装饰对象交互。
2. 装饰对象包含一个真实的对象引用(reference)。
3. 装饰对象接收所有来自客户端的请求。它把这些请求转发给真实的对象。
4. 装饰对象可以在转发这些请求之前或者之后增加一些额外的附加功能。这样就确保了在运行时，不用修改指定对象的结构就可以在外部增加附加的功能。在面向对象的设计中，通常是通过继承来实现指定类的功能。

## 3 适用场景

1. 需要扩展一个类的功能，或给一个类增加附加的职责。
2. 需要动态的给一个对象添加功能，这些功能可以再动态的撤销。
3. 需要增加由一些基本功能的排列组合而产生的非常大量的功能，从而使继承关系变得不现实。
4. 当不能采用生成子类的方法进行扩充时。一种情况是，可能有大量的独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长。

## 4 装饰模式的优点

1. Decorator模式与继承关系的目的都是要扩展对象的功能，但是Decorator可以提供比继承更加多的灵活性。
2. 通过使用不同的具体装饰模式以及这些装饰类的排列组合，设计师可以创造出很多不同行为的组合。

## 5 实例代码

其中java的IO流设计就是使用的装饰模式。

```java
//定义一个被装饰者--抽象构件组建
interface Plan {
    public void toDoList();
}

//定义被装饰者，被装饰者有具体的实现方法--具体的构建角色
class GrandPlan implements Plan {
    public void toDoList() {
        System.out.println("我的终极目标是成为一名资深程序猿!!!");
    }
}

//定义装饰者
abstract class PlanDecorator implements Plan {
    private Plan plan;

    public PlanDecorator(Plan plan) {
        this.plan = plan;
    }

    public void toDoList() {
        plan.toDoList();
    }
}

//具体构架角色
class DailyPlan extends PlanDecorator {
    public DailyPlan(Plan plan) {
        super(plan);
    }

    public void toDoList() {
        super.toDoList();
        System.out.println("每天敲击代码一万行!!!");
    }
}

//具体构架角色
class WeeklyPlan extends PlanDecorator {
    public WeeklyPlan(Plan plan) {
        super(plan);
    }

    public void toDoList() {
        super.toDoList();
        System.out.println("每周一篇技术blog!!!");
    }
}

//具体构架角色
class MonthlyPlan extends PlanDecorator {
    public MonthlyPlan(Plan plan) {
        super(plan);
    }

    public void toDoList() {
        super.toDoList();
        System.out.println("每月编写一个小项目!!!");
    }
}

//测试类
public class Test {
    public static void main(String[] args) {
        GrandPlan plan = new GrandPlan();
        MonthlyPlan monthlyPlan = new MonthlyPlan(new WeeklyPlan(new DailyPlan(plan)));

        monthlyPlan.toDoList();
    }
}
```

输出结果

```
我的终极目标是成为一名资深程序猿!!!
每天敲击代码一万行!!!
每周一篇技术blog!!!
每月编写一个小项目!!!
```

## 参考博文

[http://blog.csdn.net/bingduanlbd/article/details/29360091](http://blog.csdn.net/bingduanlbd/article/details/29360091)

[http://www.cnblogs.com/chenxing818/p/4705919.html](http://www.cnblogs.com/chenxing818/p/4705919.html)

[http://blog.csdn.net/xhbxhbsq/article/details/54090376?locationNum=2&fps=1](http://blog.csdn.net/xhbxhbsq/article/details/54090376?locationNum=2&fps=1)
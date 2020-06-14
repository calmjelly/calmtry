# 对象的共享

## synchronized 内置锁

 synchronized是可重入的。

一个示例，说明锁是可重入的。

参考：https://my.oschina.net/u/3149614/blog/3032984

```java
public class Widget {
    public synchronized void doSomething() {
        System.out.println("Widget中的this: " + this);
    }
}

public class LoggingWidget extends Widget {
    @Override
    public synchronized void doSomething() {
        super.doSomething();
        System.out.println("LoggingWidget中的super: " + super.toString());
        System.out.println("LoggingWidget中的this: " + this);
    }

    public static void main(String[] args) {
        LoggingWidget loggingWidget = new LoggingWidget();
        loggingWidget.doSomething();
    }
}
```

super、this都是在子类对象中调用的，他们具体的指向都是子类的实例对象，持有的锁都同一个对象锁。如果内置锁是不可重入的，这段代码会导致死锁。



## 发布与逸出



## 线程封闭





### 栈封闭

其实就是指方法内部定义的局部变量，它是线程安全的。

方法的每次调用都会产生一个对应的栈帧，对一个方法的调用过程，就是一个栈帧入栈到出栈的过程。（参考JVM内存模型）

局部变量存放在虚拟机栈中，而虚拟机栈是私有的，不是线程共享的，所以局部变量是线程安全的。



### ThreadLocal

使用示例：

```java
private static final ThreadLocal<SimpleDateFormat> dateFormatter = 
ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy/MM/dd"));
```

一般都使用static修饰，避免重复创建与线程相关的变量。

可以简单理解为 Map<Thread,Object> ，各个线程都有自己独立的副本，所以不会在并发情况下出现线程安全的问题。

### 不可变对象

使用不可变对象，如String，就不必担心线程安全问题。

注意，final修饰的变量并非是不可变的。例如一个final引用指向某个对象，虽然引用指向不变，但是对象本身的属性还是可以被修改的。

Java并发编程实践里面给出的对象不可变定义：

> 1、对象创建后状态不能再修改，需要修改时候，通过保存一个新状态的实例来替换原有的不可变对象
>
> 2、对象的所有域都是final类型
>
> 3、对象是正确创建的，在创建过程中this引用没有逸出

## 安全的发布对象

必要条件：对象的引用一级对象的状态必须同时对其他线程可见。常用的方式：

1、静态初始化函数中初始化一个对象的引用

2、对象引用保存在volatile类型的域中或者 Atomicreferance对象中

3、保存在正确构造的final域中

4、保存在一个由锁保护的域中

最后一条，包括将对象放入线程安全的集合中，例如：ConcurrentMap、SynchronizedHashMap、CopyOnWriteArrayList、ConcurrentLinkedQueue等。以JUC包中的线程安全集合类为主。



# 对象的组合

## 设计线程安全的类

### 收集同步需求

确保类的线程安全性，就是确保它的不变性条件不会再并发访问的情况下被破坏。需要对他的状态进行判断。

对象与变量都有一个状态空间，就是他们所有可能的取值。状态空间越小越容易判断线程的状态。

#### 不可变条件和后验条件

类的方法中往往会有一些不可变的条件，用于判断状态时是有效还是无效的。

操作中可能还会有一些后验条件来判断状态迁移是否是有效的，下一个状态可能会依赖于当前的状态。

#### 先验条件

例如：读-->判断-->更新。

单线程中这种操作是没问题的，但是在多线程下，先验条件可能由其他线程执行操作而改变。

### 实例封闭

将对象封装在实例对象里面，通过线程安全的方法进行访问。



### 监视器模式

对累的方法使用synchronized加锁，建议使用一个私有的锁对象而不是对象的内置锁。
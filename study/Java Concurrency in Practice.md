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



# 基础构建模块

## 同步容器类

Vector ，Stack，HashTable以及Collections提供的同步集合类:

```java
List list = Collections.synchronizedList(new ArrayList());
Set set = Collections.synchronizedSet(new HashSet());
Map map = Collections.synchronizedMap(new HashMap());
```

会严重降低并发性，吞吐量会急剧降低。

## 并发容器类



CopyOnWriteArrayList

这系列容器基本原理都是读写分离，读操作无锁，写操作会另起一个副本。所以叫“写时复制(CopyOnWrite)”。应对读多写少的场景。

在每次修改时候，都会创建并重新发布一个新的容器副本，从而实现读写分离。当然，这种情况下，读取的可能不是最新值。



CopyOnWriteArraySet

待补充



ConcurrentHashMap

JDK8之前，采用分段锁实现，

它不会抛出ConCurrentModificationException，在迭代过程中不需加锁，它具有弱一致性，不具备快速失败机制，可以容忍并发修改。所以例如size() 和 isEmpty() 方法可能返回已经过期的值。

对于一些复合操作，已经实现为原子操作：

仅当没有相应的key才会插入

```java
V putIfAbsent(K key, V value);
```

remove：仅当key被映射到V时候才移除。

```java
boolean remove(Object key, Object value);
```

replace：对于某个key而言，仅当它对应的 value值和预期值相等时才会更新value。

```java
 boolean replace(K key, V oldValue, V newValue);
```

等等。

ConcurrentSkipListMap

待补充

ConcurrentSkipListSet

待补充

ConcurrentLinkedQueue

待补充

ConsurrentLinkedDeque

待补充

ArrayBlockingQueue

阻塞队列BlockingQueue接口，基本方法有4个，put、take以及支持定时的offer、poll。

take从队列中拿走数据，如果队列为空则阻塞自身，一直到队列中有数据可取才会自动唤醒该线程。

put 将数据放入队列，如果队列已满则阻塞自身，一直到队列有空间可以放数据为止。

```java
//BlockingQueue接口的基本方法
void put(E e) throws InterruptedException;
boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;
E take() throws InterruptedException;
E poll(long timeout, TimeUnit unit) throws InterruptedException;
```

很容易想到生产者消费者模型。

生产者与消费者模型-->线程池与工作队列，Executor任务执行框架就是这样的设计思想。

如果队列是无界的，那么put操作不会阻塞，但可能会抛出OOM异常。

阿里巴巴开发手册里面，提到不能使用Executors创建线程池。建议手动创建，示例：

```java
package com.example.demo;
//引入google的guava包
import com.google.common.util.concurrent.ThreadFactoryBuilder;

import java.util.concurrent.*;

public class MyTest {
    public static void main(String[] args) {
        ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
                .setNameFormat("demo-pool-%d").build();
        ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("线程名称-%s").build();
        ExecutorService pool = new ThreadPoolExecutor(5, 200, 0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());
        pool.execute(()-> System.out.println(Thread.currentThread().getName()));
        pool.shutdown();//gracefully shutdown
    }
}
```

方法解释：

```java
public ThreadPoolExecutor(int corePoolSize, //线程池核心线程数量
                          int maximumPoolSize,//线程池最大数量
                          long keepAliveTime,//空闲线程存活时间
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//线程池所使用的缓冲队列
                          ThreadFactory threadFactory,//线程池创建线程使用的工厂
                          RejectedExecutionHandler handler)//线程池对拒绝任务的处理策略
```

Executors.newCachedThreadPool

```java
   public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

核心线程池数量为0，但是线程池最大数量没有限制(Integer.MAX_VALUE),提交一个新任务时，不创建核心线程。由于SynchronousQueue 是一种非常特殊的阻塞队列，他没有实际的容量，甚至连一个队列的容量都没有。（没有数据缓冲），每个put操作必须等待take操作，反过来也一样（和Exchanger提供的作用非常相似）。所以可以理解为这个队列时钟都是满的，所以最终都是创建非核心线程来执行任务，由于线程池最大容量没做限制，所以可以认为线程是可以无限创建的，可能会引发OOM异常。

Executors.newSingleThreadExecutor

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

**注意这里使用了一个无界(Integer.MAX_VALUE)的阻塞队列**，如果不断地的提交任务，超过了核心线程的数量，就会暂时放入阻塞队列中，因为无界，所以可以不限制的放入任务，可能会引发OOM。由于队列是无界限的，所以根本就不会创建非核心线程。

Executors.newFixedThreadPool

```java
  public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

和上一个差不多，使用了无界队列存放任务，所以可能会堆积大量任务从而引发OOM异常。

LinkedBlockingQueue

待补充

LinkedBlockingDeque

待补充

SynchronousQueue

待补充

PriorityBlockingQueue

待补充

LinkedTransferQueue

待补充

DelayQueue

待补充



### 串行线程封闭

生产者-消费者通过阻塞队列，实现了串行线程封闭，对象的所有权从生产者安全地交付给消费者。线程封闭对象只能由单个线程拥有，但是通过这种模式安全地转移了所有权。

这里的关键在于，确保只有一个线程能接受被转移的对象。

使用阻塞队列简化 生产者-消费者设计：

```
//生产者：
...

while(true){
//生产产品
Production production=new Production();
//放入阻塞队列
blockingQueue.put(production);
}
...

//消费者
...
while(true){
	Production production = blockingQueue.take();
	doSomething(production);
}
...
```

生产者消费者模型可以用多种方式实现，例如wait-notify、Semaphore、Lock等：详情参考 [生产者消费者的五种实现方式-Java](https://juejin.im/entry/596343686fb9a06bbd6f888c "With a Title"). 

### 双端队列与工作密取

在生产者-消费者设计中，所有的消费者共享一个工作队列，在工作密取设计中，每个消费者拥有自己的双端队列。

如果一个消费者完成了自己双端队列中的全部工作，可以从其他消费者的双端队列**末尾**秘密的获取工作。

工作密取模式更适合即使生产者又是消费者对的情况->当执行某个工作任务的时候，会出现更多的工作。

参考代码：

```java
//来源：https://houbb.github.io/2019/01/18/jcip-14-deque-workstealing
/**
 * 基本思路是维护一个阻塞队列数组，开始时候消费者按照自己规定的编号消费自己的双端队列，
 * 如果消费完成，那么可以选择数组中另外一个双端队列进行消费。
 */
 public interface WorkStealingEnableChannel<P> extends Chanel<P> {
    P take(BlockingDeque<P> preferredQueue) throws InterruptedException;
}

public class WorkStealingChannel<P> implements WorkStealingEnableChannel<P> {
    private final BlockingDeque<P>[] managedQueues;

    public WorkStealingChannel(BlockingDeque<P>[] managedQueues) {
        super();
        this.managedQueues = managedQueues;
    }

    @Override
    public P take() throws InterruptedException {
        return take(null);
    }

    @Override
    public void put(P product) throws InterruptedException {
        int targetIndex = (product.hashCode() % managedQueues.length);
        BlockingQueue<P> targetQueue = managedQueues[targetIndex];
        targetQueue.put(product);
    }

    @Override
    public P take(BlockingDeque<P> preferredQueue) throws InterruptedException {
        BlockingDeque<P> targetQueue = preferredQueue;
        P product = null;
        if (null != targetQueue) {
            product = targetQueue.poll();
        }
        int queueIndex = -1;
        while (null != product) {
            queueIndex = (queueIndex + 1) % managedQueues.length;
            targetQueue = managedQueues[queueIndex];
            product = targetQueue.pollLast();
            if (preferredQueue == targetQueue) {
                break;
            }
        }
        if (null == product) {
            queueIndex = (int) (System.currentTimeMillis() % managedQueues.length);
            targetQueue = managedQueues[queueIndex];
            product = targetQueue.pollLast();
            System.out.println("stealed from " + queueIndex + ": " + product);
        }
        return product;
    }
}

public class WorkStealingExample {
    private final WorkStealingEnableChannel<String> channel;
    private final TerminationToken token = new TerminationToken();

    public static void main(String[] args) throws InterruptedException {
        WorkStealingExample wse = new WorkStealingExample();
        Thread.sleep(3500);
    }

    public WorkStealingExample() {
        int nCPU = Runtime.getRuntime().availableProcessors();
        int consumerCount = nCPU / 2 + 1;
        BlockingDeque<String>[] managedQueues = new LinkedBlockingDeque[consumerCount];
        channel = new WorkStealingChannel<String>(managedQueues);
        Consumer[] consumers = new Consumer[consumerCount];
        for (int i = 0; i < consumerCount; i++) {
            managedQueues[i] = new LinkedBlockingDeque<String>();
            consumers[i] = new Consumer(token, managedQueues[i]);
        }
        for (int i = 0; i < nCPU; i++) {
            new Producer().start();
        }
        for (int i = 0; i < consumerCount; i++) {
            consumers[i].start();
        }
    }

    private class Producer extends AbstractTerminatableThread {
        private int i = 0;

        @Override
        protected void doRun() throws Exception {
            channel.put(String.valueOf(i++));
            Thread.sleep(10);
            token.reservations.incrementAndGet();
        }
    }

    private class Consumer extends AbstractTerminatableThread {
        private final BlockingDeque<String> workQueue;

        public Consumer(TerminationToken token, BlockingDeque<String> workQueue) {
            super(token);
            this.workQueue = workQueue;
        }

        @Override
        protected void doRun() throws Exception {
            String product = channel.take();
            if (product != null) {
            }
            System.out.println("Processing product:" + product);
            try {
                Thread.sleep(new Random().nextInt(50));
            } catch (Exception e) {
            } finally {
                token.reservations.decrementAndGet();
            }
        }
    }
}
 
```

## 阻塞方法与中断方法
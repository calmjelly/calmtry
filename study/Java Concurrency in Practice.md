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

super、this都是在子类对象中调用的，同门具体的指向都是子类的实例对象，持有的锁都同一个对象锁。如果内置锁是不可重入的，这段代码会导致死锁。


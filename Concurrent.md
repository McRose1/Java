# Java 多线程并发

## Java 线程实现/创建方式

### 继承 Thread 类
Thread 类本质上是实现了 Runnable 接口的一个实例，代表一个线程的实例。

启动线程的唯一方法就是通过 Thread 类的 start() 实例方法。

**start() 方法是一个 native 方法**，它将启动一个新线程，并执行 run() 方法。

```java
public class MyThread extends Thread {
  public void run() {
    System.out.println("MyThread.run()");
  }
}
MyThread myThread1 = new MyThread();
myThread1.start();
```

### 实现 Runnable 接口
如果自己的类已经 extends 另一个类，就无法直接 extends Thread，此时，可以实现一个 Runnable 接口。
```java
public class MyThread extends OtherClass implements Runnable {
  public void run() {
    System.out.println("MyThread.run()");
  }
}
// 启动 MyThread，需要首先实例化一个 Thread，并传入自己的 MyThread 实例：
MyThread myThread = new MyThread();
Thread thread = new Thread(myThread);
thread.start();
// 事实上，当传入一个 Runnable target 参数给 Thread 后，Thread 的 run() 方法就会调用 target.run()
public void run() {
  if (target != null) {
    target.run();
  }
}
```

## Java 锁

### 乐观锁
乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会去修改，所以不会上锁，但是**在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作**（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写操作。

Java 中的乐观锁基本都是通过 CAS 操作实现的，CAS 是一种更新的原子操作，**比较当前值跟传入值是否一样，一样则更新，否则失败**。

### 悲观锁
悲观锁就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会 block 直到拿到锁。Java 中的悲观锁就是 **Synchronized**，AQS 框架下的锁则是先尝试 CAS 乐观锁去获取锁，获取不到，才会转换为悲观锁，如 RetreenLock。



















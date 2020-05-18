# Java 多线程并发

## 进程和线程
进程和线程的由来：
- 串行：初期的计算机智能串行执行任务，并且需要长时间等待用户输入
- 批处理：预先将用户的指令集中成清单，批量串行处理用户指令，仍然无法并发执行
- 进程：进程独占内存空间，保存各自运行状态，相互间不干扰且可以互相切换，为并发处理任务提供了可能
- 线程：共享进程的内存资源，相互间切换更快捷，支持更细粒度的任务控制，使进程内的子任务得以并发执行

### 进程和线程的区别
**进程是资源分配的最小单位，线程是 CPU 调度的最小单位**。
- 所有与进程相关的资源，都被记录在 PCB 中
- 进程是抢占处理机的调度单位；线程属于某个进程，共享其资源
- 线程只由堆栈寄存器、程序计数器和 TCB 组成

总结：
- 线程不能看作独立应用，而进程可看做独立应用
- 进程由独立的地址空间，相互不影响，线程只是进程的不同执行路径
- 线程没有独立的地址空间，多进程的程序比多线程程序健壮
- 进程的切换比线程的切换开销大

### Java 进程和线程的区别
- Java 对操作系统提供的功能进行封装，包括进程和线程
- 运行一个程序会产生一个进程，

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

## J.U.C
java.util.concurrent：提供了并发编程的解决方案
- CAS 是 java.util.concurrent.atomic 包的基础
- AQS 是 java.util.concurrent.locks 包以及一些常用类比如 Semaphore，ReentrantLock 等类的基础

## J.U.C 包的分类
### 线程执行器（executor）

### 锁（locks）

### 原子变量类（atomic）

### 并发工具类（tools）

#### 闭锁（CountDownLatch）
让主线程等待一组事件发生后继续执行
- 事件指的是 CountDownLatch 里的 countDown()方法

#### 栅栏（CyclicBarrier）
阻塞当前线程，等待其他线程
- 等待其他线程，且会阻塞自己当前线程，所有线程必须同时到达栅栏位置后，才能继续执行；
- 所有线程到达栅栏处，可以触发执行另外一个预先设置的线程

#### 信号量（Semaphore）
控制某个资源可被同时访问的线程个数

#### 交换器（Exchanger）
两个线程到达同步点后，相互交换数据

### 并发集合（collections）

#### BlockingQueue
提供可阻塞的入队和出队操作

主要用于生产者-消费者模式，在多线程场景时生产者线程在队列尾部添加元素，而消费者线程则在队列头部消费元素，通过这种方式能够达到将任务的生产和消费进行隔离的目的

实现方式（都是线程安全的）：
1. **ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列（容量有限，初始化需要指定，指定后无法改变）
2. **LinkedBlockingQueue：一个由链表结构组成的有界/无界阻塞队列
3. **PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列
4. DealyQueue：一个使用优先级队列实现的无界阻塞排列
5. SynchronousQueue：一个不存储元素的阻塞队列
6. LinkedTransferQueue：一个由链表结构组成的无界阻塞队列
7. LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列

























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
- 运行一个程序会产生一个进程，进程包含至少一个线程
- 每个进程对应一个 JVM 实例，多个线程共享 JVM 里的堆
- Java 采用单线程编程模型，程序会自动创建主线程
- 主线程可以创建子线程，原则上要后于子线程完成执行

```java
public class CurrentThreadDemo {
  public static void main(String[] args) {
    System.out.println("Current Thread: " + Thread.currentThread().getName());
  }
}
```
Current Thread: main

## Java 线程实现/创建方式

**Thread 中的 start 和 run 方法的区别**:
Thread#start()  ->   JVM_StartThread   ->    thread_entry   ->   Thread#run()
- 调用 start() 方法会创建一个新的子线程并启动
- run()方法只是 Thread 的一个普通方法的调用

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
// 因为 Runnable 接口没有 start() 这个方法，所以需要首先实例化一个 Thread，并传入自己的 MyRunnable 实例：
MyRunnable myRunnable = new MyRunnable();
Thread thread = new Thread(myRunnable);
thread.start();
// 事实上，当传入一个 Runnable target 参数给 Thread 后，Thread 的 run() 方法就会调用 target.run()
public void run() {
  if (target != null) {
    target.run();
  }
}
```

**Thread 和 Runnable 是什么关系**
- Thread 是实现了 Runnable 接口的类，使得 run 支持多线程
- 因类的单一继承原则，推荐多使用 Runnable 接口

**如何给 run() 方法传参**
实现的方式主要有三种：
- 构造函数传参
- 成员变量传参
- 回调函数传参

**如何实现处理线程的返回值**
实现的方式主要有三种：
- 主线程等待法
  - 优点：实现简单
  - 缺点：需要自己实现循环等待的逻辑，等待的变量如果很多，代码会显得异常臃肿；需要循环多久是不确定的，无法做到精准控制
- 使用 Thread 类的 join() 阻塞当前线程以等待子线程处理完毕
  - 优点：能做到比主线程等待法更精准的控制是，实现起来更简单
  - 缺点：粒度不够细   
- 使用 Callable 接口实现：通过 FutureTask 或线程池获取（JDK5 之后实现了多线程可以有返回值）

CycleWait.java
```java
public class CycleWait implements Runnable {
  private String value;
  public void run() {
    try {
      Thread.currentThread().sleep(5000);
    } catch {
      e.printStackTrace();
    }
    value = "we have data now";
  }
}

public static void main(String[] args) throws InterruptedException {
  CycleWait cw = new CycleWait();
  Thread t = new Thread(cw);
  t.start();
  // 1.
  while (cw.value == null) {
    Thread.currentThread().sleep(100);
  }
  // 2.
  t.join();
  System.out.println("value : " + cw.value);
}
```

MyCallable.java
```java
public class MyCallable implements Callable<String> {
  @Override
  public String call() throws Exception {
    String value = "test";
    System.out.println("Ready to work");
    Thread.currentThread().sleep(5000);
    System.out.println("task done");
    return value;
  }
}
```

FutureTaskDemo.java
```java
public class FutureTaskDemo {
  public static void main(String[] args) {
    FutureTask<String> task = new FutureTask<String>(new MyCallable);
    new Thread(task).start();
    if (!task.isDone()) {
      System.out.println("task has not finished, please wait!");
    } 
    System.out.println("task return: " + task.get());
  }
}
```

ThreadPoolDemo.java
```java
public class ThreadPoolDemo {
  public static void main(String[] args) {
    ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
    Future<String> future = newCachedThreadPool.submit(new Callable());
    if (!future.isDone()) {
      System.out.println("task has not finished, please wait!");
    }
    try {
      System.out.println(future.get());
    } catch (InterruptedException e) {
      e.printStackTrace();
    } catch (ExecutionException e) {
      e.printStackTrace();
    } finally {
      newCachedThreadPool.shutdown();
    }
  }
}
```

### 线程的状态
6 个状态：
- 新建（New）：创建后尚未启动的线程的状态
- 运行（Runnable）：包含 Running 和 Ready
- 无限期等待（Waiting）：不会分配 CPU 执行时间，需要显式被唤醒
  - 没有设置 Timeout 参数的 Object.wait() 方法
  - 没有设置 Timeout 参数的 Thread.join() 方法
  - LockSupport.park() 方法
- 限期等待（Time Waiting）：在一定时间后由系统自动唤醒
  - Thread.sleep()方法
  - 设置了 Timeout 参数的 Object.wait() 方法
  - 设置了 Timeout 参数的 Thread.join() 方法
  - LockSupport.parkNanos() 方法
  - LockSupport.parkUntil() 方法
- 阻塞（Blocked）：等待获取排他锁
- 结束（Terminated）：已终止线程的状态，线程已经结束执行

Thread.java
```java
public enum State {
  /**
   *  Thread state for a thread which has not yet started.
   */ 
   NEW,
   
   /**
    * Thread state for a runnable thread. A thread in the runnable state is executing in the JVM 
    * but it may be waiting for other resources from the operating system such as processor.
    */
    RUNNABLE,
    
    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method after calling 
     * {@link Object#wait() Object.wait}.
     */
     BLOCKED,
     
     /**
      * Thread state for a waiting thread.
      * A thread is in the waiting state due to calling one of the following methods:
      * <ul>
      *   <li>{@link Object#wait() Object.wait()} with no timeout</li>
      *   <li>{@link #join() Thread.join} with no timeout</li>
      *   <li>{@link LockSupport#park() LockSupport.park}</li>
      * </ul>
      *
      * <p>A thread in the waiting state is waiting for another thread to perform a particular action.
      *
      * For example, a thread that has called {@code Object.wait()} on an object is waiting for another thread to call
      * {@code Object.notify()} or {@code Object.notifyAll()} on that object 
      * A thread that has called {@code Thread.join()} is waiting for a specified thread to terminate.
      */
      WAITING,
      
      /**
       * Thread state for a waiting thread with a specified waiting time.
       * A thread is in the timed waiting state due to calling one of the following methods 
       * with a specified positive waiting time:
       * <ul>
       *   <li>{@link #sleep Thread.sleep}</li>
       *   <li>{@link Object#wait(long) Object.wait()} with timeout</li>
       *   <li>{@link #join(long) Thread.join} with timeout</li>
       *   <li>{@link LockSupport#parkNanos() LockSupport.parkNanos}</li>
       *   <li>{@link LockSupport#parkUntil() LockSupport.parkUntil}</li>
       * </ul>
       */
       TIME_WAITING,
       
       /**
        * Thread state for a terminated thread.
        * The thread has completed execution.
        TERMINATED;
}
```

### sleep 和 wait 的区别
**基本的差别**：
- sleep 是 Thread 类的方法，wait 是 Object 类中定义的方法
- sleep() 方法可以在任何地方使用
- wait() 方法只能在 synchronized 方法或 synchronized 块中使用

Thread.java
```java
public static native void sleep(long millis) throws InterruptedException
```

Object.java
```java
public final void wait() throws InterruptedException {
  wait(0L);
}

public final native void wait(long timeoutMillis) throws InterruptedException;
```

**最主要的本质区别**：
- Thread.sleep 只会让出 CPU，不会导致锁行为的改变
- Object.wait 不仅让出 CPU，还会释放已经占有的同步资源锁

WaitSleepDemo.java
```java
public class WaitSleepDemo {
  public static void main(String[] args) {
    final Object lock = new Object();
    
    // 执行 wait 逻辑
    new Thread(new Runnable) {
      @Override
      public void run() {
        System.out.println("thread A is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread A get lock");
            Thread.sleep(20);
            System.out.println("thread A do wait method);
            lock.wait(1000);
            System.out.println("thread A is done");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    
    // 执行 sleep 逻辑
    new Thread(new Runnable) {
      @Override
      public void run() {
        System.out.println("thread B is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread B get lock");
            System.out.println("thread B is sleeping 10 ms");
            Thread.sleep(10);
            System.out.println("thread B is done");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
  }
}
```
- thread A is waiting to get lock 
- thread A get lock
- thread B is waiting to get lock 
- thread A do wait method 
- thread B get lock 
- thread B is sleeping 10 ms
- thread B is done
- thread A is done

如果顺序交换：
- thread A is waiting to get lock 
- thread A get lock
- thread B is waiting to get lock 
- thread A do wait method 
- thread A is done
- thread B get lock 
- thread B is sleeping 10 ms
- thread B is done

### notify 和 notifyAll 的区别 
- notifyAll 会让所有处于等待池的线程全部进入锁池去竞争获取锁的机会
- notify 只会随机选取一个处于等待池中的线程进入锁池去竞争获取锁的机会

#### 锁池（EntryList）
假设线程 A 已经拥有了某个对象（不是类）的锁，而其它线程 B、C 想要调用这个对象的某个 synchronized 方法（或者块），由于 B、C 线程在进入对象的 synchronized 方法（或者块）之前必须先获得该对象锁的拥有权，而恰巧该对象的锁目前正被线程 A 所占用，此时 B、C 线程就会被阻塞，进入一个地方去等待锁的释放，这个地方便是该对象的锁池。

#### 等待池（WaitSet）
假设线程 A 调用了某个对象的 wait()方法，线程 A 就会释放该对象的锁，同时线程 A 就进入到了该对象的等待池中，进入到等待池中的线程不会去竞争该对象的锁。

WaitSleepDemo.java
```java
public class WaitSleepDemo {
  public static void main(String[] args) {
    final Object lock = new Object();
    
    // 执行 wait 逻辑
    new Thread(new Runnable) {
      @Override
      public void run() {
        System.out.println("thread A is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread A get lock");
            Thread.sleep(20);
            System.out.println("thread A do wait method);
            lock.wait();                              // 进入无限等待
            System.out.println("thread A is done");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    
    // 执行 sleep 逻辑
    new Thread(new Runnable) {
      @Override
      public void run() {
        System.out.println("thread B is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread B get lock");
            System.out.println("thread B is sleeping 10 ms");
            Thread.sleep(10);
            System.out.println("thread B is done");
            lock.notify();                    // 唤醒线程 A
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
  }
}
```

### yield
当调用 Thread.yield()函数时，会给线程调度器一个当前线程愿意让出 CPU 使用的暗示，但是线程调度器可能会忽略这个暗示。

Thread.java
```java
/**
  * A hint to the scheduler that the current thread is willing to yield its current use of processor.
  * The scheduler is free to ignore this hint.
  *
  * <p> Yield is a heurisitic attempt to improve relative progression between threads that would otherwise over-utilise a CPU.
  * Its use should be combined with detailed profiling and benchmarking to ensure that it actually has the desired effect.
  * 
  * <p> It is rarely appropriate to use this method. 
  * It may be useful for debugging or testing purposes, where it may help to reproduce bugs due to race conditions.
  * It may also be useful when designing concurrency control constructs such as the ones in the 
  * {@links java.util.concurrent.locks} package.
  */
  public static native void yield();
```

### 如何中断线程（Interrupt）
目前使用的方法：
- 调用 interrupt()，通知线程应该中断了
  - 如果线程处于被阻塞状态，那么线程将立即退出被阻塞状态，并抛出一个 InterruptedException 异常
  - 如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true。被设置中断标志的线程将继续正常运行，不受影响。
- 需要被调用的线程配合中断
  - 在正常运行任务时，经常检查本线程的中断标志位，如果被设置了中断标志就自行停止线程。
  - 如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true。被设置中断标志的线程将继续正常运行，不受影响。

已经被抛弃的方法：
- 通过调用 stop()方法停止线程
- 通过调用 suspend()和resume()方法

### 线程状态图
(images/线程状态图.jpeg)

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

























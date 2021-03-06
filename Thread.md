# Thread
There are 4 ways of creating threads
- extends **Thread class** 并重写 run()
- implements **Runnable interface** 并重写 run() 
- implements **Callable interface** 并重写 call()
- 调用 ThreadPool 线程池（应该算使用线程而不是创建线程）

## 继承 Thread 类

优点：
- 实现简单

缺点：
- 不符合里氏替换原则
- 不可以继承其他类



## 实现 Runnable 接口
1. 创建一个 Runnable 接口的实现类
2. 在实现类中重写 Runnable 接口的 run 方法
3. 创建一个 Runnable 接口实现类的对象
4. 创建 Thread 对象，构造方法中传参为：Runnable 接口实现类的对象
5. 调用 Thread 类中的 start 方法，启动多线程

优点：
- 避免了继承 Thread 类方法遇到的类只能单继承的局限性，使编程更加灵活，还可以继承其他的类，实现其他的接口
- 降低了线程对象和线程任务的耦合性，增强了程序的可扩展性（将设置线程任务run()和开启新线程start()进行了分离，实现解耦）
  - 实现类中，重写了 run() 方法来设置线程任务
  - 创建 Thread 类对象，调用 start() 方法来开启新线程
  - 创建 Thread 类对象，构造方法中传递 Runnable 接口的实现类对象，可以传递不同的实现类（可扩展性）
- 将线程单独进行对象的封装，对外暴露细节少，更符合面向对象思想

缺点
- 

## 实现 Callable 接口 
However, one feature lacking in Runnable is that we cannot make a thread return result when it terminates, i.e. when run() completes.

For supporting this feature, the Callable interface is present in Java.

优点：
- 可以获取线程执行结果的返回值，并且可以抛出异常

### Callable vs. Runnable 
- For implementing Runnable, the run() method needs to be implemented which does not return anything, while for a Callable, the call() method needs to be implemented which returns a result on completion. Note that a thread can't be created with a Callable, it can only be created with a Runnable.
- Another difference is that the call() method can throw an exception whereas run() cannot.

```java
class CallableExample implements Callable {
  
  public Object call() throws Exception {
    // Create random number generator 
    Random generator = new Random();
    
    Integer randomNumber = generator.nextInt(5);
    
    // To simulate a heavy computation, we delay the thread for some random time
    Thread.sleep(randomNumber * 1000);
    
    return randomNumber;
  }
}
```

## 线程池（Java 1.5 后引入）

### 作用：
1. 降低资源消耗，复用已创建的线程降低开销、控制最大并发数
2. 隔离线程环境，可以配置独立线程池，将较慢的线程与较快的隔离开，避免相互影响
3. 实现任务线程队列缓冲策略和拒绝机制
4. 实现某些与实践相关的功能，如定时执行、周期执行等

### 当提交一个新任务到线程池时的处理流程：
1. 核心线程池未满，创建一个新的线程执行任务，此时 workCount < corePoolSize，这一步需要获取**全局锁**
2. 如果核心线程池已满，工作队列未满，将线程存储在工作队列，此时 workCount >= corePoolSize
3. 如果工作队列已满，线程数小于最大线程数就创建一个新线程处理任务，此时 workCount < maximumPoolSize，这一步也需要获取**全局锁**
4. 如果超过最大线程数，按照拒绝策略来处理任务，此时 workCount > maximumPoolSize

线程池采取这种设计思路是为了在执行 execute 方法时尽可能地避免获取全局锁，在线程池完成预热之后，即当前运行的线程池大于等于 corePoolSize 时，几乎所有的 execute 方法都是在执行步骤 2，不需要获取全局锁。

线程池创建线程时，会将线程封装成工作线程 Worker。Worker 在执行完任务后还会循环获取工作队列中的任务来执行。

线程池中的线程执行任务分为 2 种情况：
1. 在 execute 方法中创建一个线程时会让这个线程执行当前任务
2. 这个线程执行完任务之后，就会反复从工作队列中获取任务并执行

可以使用 execute 和 sumbit 方法向线程池提交任务。

execute 用于提交**不需要返回值**的任务，所以无法判断任务是否被线程池执行成功了。

submit 用于提交**需要返回值**的任务，线程池会返回一个 **Future** 类型的对象，通过该对象可以判断任务是否执行成功，并且可以通过该对象的 get 方法获取返回值，get 会阻塞当前线程直到任务完成，带超时参数的 get 方法会在阻塞当前线程一段时间后立即返回，这时任务可能还没有完成。

### 创建线程池
1. newFixedThreadPool：**固定大小**的线程池，输入的参数既是**核心线程数**也是**最大线程数**，不存在空闲线程，因此 keepAliveTime 等于 0。该线程池使用的工作队列是**无界阻塞队列**
2. newSingleThreadExecutor：使用**单线程**的线程池，相当于**单线程串行**执行所有任务，适用于需要**保证顺序执行任务**的应用场景
3. newCachedThreadPool：**maximumPoolSize 设置为 Integer.MAX_VALUE**，是高度可伸缩的线程池。该线程池使用的工作队列是没有容量的**SynchronousQueue**，如果主线程提交任务的速度大于线程处理的速度，线程池会不断创建新线程，极端情况下会创建过多线程而耗尽 CPU 和内存资源。适用于**执行很多短期异步任务的小程序或者负载较轻**的服务器


### 线程池的参数
- corePoolSize：常驻核心线程数，如果为 0，当执行完任务没有任何请求时会消耗线程池；如果大于 0，即使本地任务执行完，核心线程也不会被销毁。**该值设置过大会浪费资源，过小会导致线程的频繁创建于销毁**。
- maximumPoolSize：线程池能够容纳同时执行的线程最大数，必须大于等于 1，如果与核心线程数设置相同代表固定大小线程池。
- keepAliveTime：线程空闲时间，线程空闲时间达到该值后会被销毁，直到只剩下 corePoolSize 个线程为止，避免浪费内存资源。
- unit：keepAliveTime 的时间单位。
- workQueue：工作队列，当线程请求数大于等于 corePoolSize 时线程会进入阻塞队列。
- threadFactory：线程工厂，用来生产一组相同任务的线程。可以给线程命名，有利于分析错误。
- handler：拒绝策略，默认策略下使用 AbortPolicy 丢弃任务并抛出异常，CallerRunsPolicy 表示重新尝试提交该任务，DiscardOldestPolicy 表示抛弃队列里等待最久的任务并把当前任务加入队列，DiscardPolicy 表示直接抛弃当前任务但不抛出异常。












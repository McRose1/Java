# Java 的 IO 机制

## BIO、NIO、AIO 之间的对比
|属性\模型 | 阻塞 BIO | 非阻塞 NIO | 异步 AIO |
| --- | ---- | ---- | ---- |
|blocking | 阻塞并同步 | 非阻塞但同步 | 非阻塞但异步 |
| 线程数（server:client）| 1:1 | 1:N | 0:N |
| 复杂度 | 简单 | 较复杂 | 复杂 |
| 吞吐量 | 低 | 高 | 高 |
| 使用场景 | 连接数比较小且固定的架构 | 连接数目多且连接比较短的架构，比如聊天服务器 | 连接数目多且连接比较长的架构，比如相册服务器 |


## 阻塞 IO 模型
Block-IO：InputStream 和 OutputStream，Reader 和 Writer 

最传统的一种 IO 模型，即在读写数据过程中会发生阻塞现象。

当用户线程发出 IO 请求之后，内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出 CPU。

当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除 block 状态。

典型的阻塞 IO 模型的例子为：data = socket.read()；如果数据没有就绪，就会一直阻塞在 read 方法。

```java
public void improvedServe(int port) throws IOException {
  // 将 ServerSocket 绑定到指定的端口里
  final ServerSocket socket = new ServerSocket(port);
  // 创建一个线程池（避免频繁创建和销毁线程这个巨大的开销，对连接数只有几百个的场景线程池比较有效）
  ExecutorService executorService = Executors.newFixedThreadPool(6);
  while (true) {
    // 阻塞直到收到新的客户端连接
    final socket clientSocket = socket.accept();
    System.out.println("Accepted connection from " + clientSocket);
    // 将请求提交给线程池去执行
    executorService.execute(() -> {
      @Override
      public void run() {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(clientSocket))) {
          PrintWriter writer = new PrintWriter(clientSocket.getOutputStream(), true);
          // 从客户端读取数据并原封不动写回去
          while (true) {
            writer.println(reader.readLine());
            writer.flush();
          }
        } catch (IOException e) {
          e.printStackTrace();
        }
      });
    }
  }
}
```

## 非阻塞 IO 模型
NonBlock-IO：构建多路复用的、同步非阻塞的 IO 操作

当用户线程发起一个 read 操作后，并不需要等待，而是马上得到了一个结果。

如果结果是一个 error 时，它就知道数据还没有准备好，于是它可以再次发送 read 操作。

一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。

所以事实上，在非阻塞 IO 模型中，用户线程需要不断地询问内核数据是否就绪，也就是说非阻塞 IO 不会交出 CPU，而会一直占用 CPU。

典型的非阻塞 IO 模型一般如下：
```java
while (true) {
  data = socket.read();
  if (data != error) {
    处理数据
    break;
  }
}
```
但是对于非阻塞 IO 就有一个非常严重的问题，**在 while 循环中需要不断地去询问内核数据是否就绪，这样会导致 CPU 占用率非常高**，因此一般情况下很少使用 while 循环这种方式来获取数据。

```java
public void serve(int port) throws IOException {
  System.out.println("Listening for connections on port " + port);
  ServerSocketChannel serverChannel = ServerSocketChannel.open();
  ServerSocket ss = serverChannel.socket();
  InetSocketAddress address = new InetSocketAddress(port);
  // 将 ServerSocket 绑定到指定的端口里
  ss.bind(address);
  serverChannel.configureBlocking(false);   // 阻塞模式下注册操作是不允许的
  Selector selector = Selector.open();
  // 将 channel 注册到 Selector 里，并说明让 Selector 关注的点，这里是关注建立连接这个事件
  serverChannel.register(selector, SelectionKey.OP_ACCEPT);
  while (true) {
    try {
      // 阻塞等待就绪的 channel，即没有与客户端建立连接前就一直轮询
      selector.select();
    } catch (IOException e) {
        e.printStackTrace();
        break;
    }
    // 获取到 Selector 里所有就绪的 SelectorKey 实例，每将一个 channel 注册到一个 selector 就会产生一个 SelectionKey
    Set<SelectionKey> readyKeys = selector.selectedKeys();
    Iterator<SelectionKey> iterator = readyKeys.iterator();
    while (iterator.hasNext()) {
      SelectionKey key = (SelectionKey) iterator.next();
      // 将就绪的 selectedKey 从 Selector 中移除，因为马上就要去处理它，防止重复执行
      iterator.remove();
      try {
        // 若 SelectedKey 处于 Acceptable 的状态
        if (key.isAcceptable()) {
          ServerSocketChannel server = ()
        }
        ...
      }
    }
  }
}
```

NIO 的核心：
- Channels：Channel <-> Buffer
  - FileChannel 
    - transferTo：把 FileChannel 中的数据拷贝到另外一个 Channel
    - transferFrom：把另外一个 Channel 中的数据拷贝到 FileChannel
    - 避免了两次用户态和内核态间的上下文切换，即“零拷贝”，效率较高
  - DatagramChannel 
  - SocketChannel
  - ServerSocketChannel
- Buffers 
   - ByteBuffer
   - CharBuffer
   - DoubleBuffer
   - FloatBuffer
   - IntBuffer
   - Longbuffer
   - ShortBuffer
   - MappedByteBuffer
- Selectors：允许单线程处理多个 Channel

## 多路复用 IO 模型
IO 多路复用：调用系统级别的 select\poll\epoll

优点在于单线程可以处理多个网络 IO

多路复用 IO 模型是目前使用得比较多的模型。

Java NIO 实际上就是多路复用 IO。

在多路复用 IO 模型中，会有**一个线程不断地去轮询多个 socket 的状态，只有当 socket 真正有读写事件时，才真正调用实际的 IO 读写操作**。

因为在多路复用 IO 模型中，只需要使用一个线程就可以管理多个 socket，系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有 socket 读写事件进行时，才会使用 IO 资源，所以它大大减少了资源占用。

在 Java NIO 中，是通过 selector.select()去查询每个通道是否有到达事件，如果没有事件，则一直阻塞在那里，因此这种方式会导致用户线程的阻塞。

因此，多路复用 IO 比较适合连接数比较多的情况。

**另外多路复用 IO 为何比非阻塞 IO 模型的效率高是因为在非阻塞 IO 模型中，不断地询问 socket 状态是通过用户线程去进行的，而在多路复用 IO 中，轮询每个 socket 状态是内核在进行的，这个效率要比用户线程要高得多**。

不过要注意的是，多路复用 IO 模型是通过轮询的方式来检测是否有事件到达，并且对到达的时间逐一进行响应。

因此对于多路复用 IO 模型来说，**一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响新的事件轮询**。

### select、poll、epoll 的区别：

#### 支持一个进程所能打开的最大连接数：
- select：单个进程所能打开的最大连接数由 FD_SETSIZE 宏定义，其大小是 32 个整数的大小（在 32 位的机器上，大小是 32 * 32， 64 位机器上 FD_SETSIZE 为 32 * 64），我们可以对其进行修改，然后重新编译内核，但是性能无法保证，需要做进一步测试（底层是数组）
- poll：本质上与 select 没有区别，但是它没有最大连接数的限制，原因是它是基于链表来存储的
- epoll：虽然连接数有上限，但是很大，1G 内存的机器上可以打开 10 万左右的连接

#### FD 剧增后带来的 IO 效率问题：
- select：因为每次调用时都会对连接进行线性遍历，所以随着 FD 的增加会造成遍历速度的“线性下降”的性能问题
- poll：同上
- epoll：由于 epoll 是根据每个 fd 上的 callback 函数来实现的，只有活跃的 socket 才会主动调用 callback，所以在活跃 socket 较少的情况下，使用 epoll 不会有“线性下降”的性能问题，但是所有 socket 都很活跃的情况下，可能会有性能问题

#### 消息传递方式
- select：内核需要将消息传递到用户空间，需要内核的拷贝动作
- poll：同上
- epoll：通过内核和用户空间共享一块内存来实现，性能较高

## AIO（NIO 2.0）
Asynchronous IO：基于事件和回调机制

异步模型

AIO 如何进一步加工处理结果？
- 基于回调：实现 CompletionHandler 接口，调用时触发回调函数
- 返回 Future：通过 isDone() 查看是否准备好，通过 get() 等待返回数据































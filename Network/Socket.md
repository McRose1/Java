# Socket 编程
Socket 也是一种数据源，是网络通信的端点，可以把socket理解为一个唯一的IP地址和端口号，例如“127.0.0.1:8080”。

## 模拟TCP通信
- Socket类：是客户端的socket连接，使用步骤：
  1. 创建对象时要提供服务器的主机和端口参数
  2. 数据通信结束后通过close方法关闭连接
- ServerSocket类：是服务器端的socket连接，使用步骤：
  1. 创建对象时要提供端口参数进行绑定，服务器会监听该端口，任何发送到该端口的信息都会被服务器接收并处理
  2. 通过accept方法获取客户端连接，通过该连接与客户端进行数据通信，是一种**阻塞式**调用。
  
Server.java
```java
public class Server {
  
  public static void main(String[] args) {
    // 退出条件
    final String QUIT = "quit";
    // 服务器端口
    final int PORT = 8080;
    // 服务器的socket对象
    ServerSocket serverSocket = null;
    
    try {
      // 绑定监听端口
      serverSocket = new ServerSocket(PORT);
      System.out.println("启动服务器，监听端口：" + PORT);
      
      while (true) {
        // 等待客户端连接
        Socket socket = serverSocket.accept();
        System.out.println("客户端[" + socket.getPort() + "]已连接");
        // 获取和客户端通信的字符输入流和字符输出流
        BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        BufferedWriter bw = new BufferedWriter(new OutputStreamReader(socket.getOutputStream()));
        
        // 读取客户端发送的信息
        String message;
        while ((message = br.readLine()) != null) {
          System.out.println("客户端[" + socket.getPort() + "]发送消息：" + message);
          
          // 回复客户端
          bw.write("服务器已经收到消息：" + message + "\n");
          bw.flush();
          
          // 查看客户端是否关闭
          if (QUIT.equals(message)) {
            System.out.println("客户端[" + socket.getPort() + "]已断开连接");
            break;
          }
        }
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      // 释放资源
      if (serverSocket != null) {
        try {
          serverSocket.close();
          System.out.println("关闭 serverSocket");
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }
  }
}
```

Client.java
```java
public class Client {
    
  public static void main(String[] args) {
    // 退出条件
    final String QUIT = "quit";
    // 服务器主机和端口
    final String HOST = "127.0.0.1";
    final String PORT = 8080;
    // 客户端的socket对象
    Socket socket = null;
    BufferedWriter bw = null;
    
    try {
      // 创建socket
      socket = new Socket(HOST, PORT);
      
      // 获取和服务器通信的字符输入流和字符输出流
      BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
      bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
      
      // 等待用户输入信息
      while (true) {
        System.out.println("请输入要发送的消息：");
        BufferedReader consoleReader = new BufferedReader(new InputStreamReader(System.in));
        String input = consoleReader.readLine();
        
        // 发送消息给服务器
        bw.write(input + "\n");
        bw.flush();
        
        // 读取服务器返回的消息
        String message = br.readLine();
        if (message != null) {
          System.out.println("服务器发送消息：" + message);
        }
        
        // 查看用户是否退出
        if (QUIT.equals(input)) {
          break;
        }
      }
    }
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    // 释放资源
    try {
      if (bw != null) {
        bw.close();
        System.out.println("关闭 socket");
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

# Java 异常分类及处理
异常处理机制主要回答了 3 个问题：
- What：异常类型回答了什么被抛出
- Where：异常堆栈跟踪回答了在哪抛出
- Why：异常信息回答了为什么被抛出

如果某个方法不能按照正常的途径完成任务，就可以通过另一种路径退出方法。

在这种情况下会抛出一个封装了错误信息的对象。

此时，这个方法会立刻退出同时不返回任何值。

另外，调用这个方法的其他代码也无法继续执行，异常处理机制会将代码执行交给异常处理器。

## 异常分类
- Throwable 
  - Error（程序无法处理的错误）
  - Exception（可以处理的异常）
    - RuntimeException
    - 非 RuntimeException

Throwable 是 Java 语言中所有错误或异常的超类。下一层分为 **Error 和 Exception**。

**从概念角度解析 Java 的异常处理机制**：
- Error：程序无法处理的系统错误，编译器不做检查
Error 类是指 Java 运行时系统的内部错误和资源耗尽错误。应用程序不会抛出该类对象。如果出现了这样的错误，除了告知用户，剩下的就是尽力使程序安全的终止。

- Exception（RuntimeException, CheckedException）：程序可以处理的异常，捕获后可能恢复
Exception 又有 2 个分支，一个是运行时异常（RuntimeException），一个是 CheckedException。

### RuntimeException（不可预知的，程序应当自行避免） 
如：NullPointerException、ClassCastException；

RuntimeException 是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。

如果出现 RuntimeException，那么一定是程序员的错误。

### CheckedException（可预知的，从编译器校验的异常）
如：I/O 错误导致的 IOException、SQLException；

一般是外部错误，这种异常都发生在编译阶段，Java 编译器会强制程序去捕获这类异常，即会出现要求你把这段可能出现异常的程序进行 try catch，该类异常一般包括几个方面：
1. 试图在文件尾部读取数据
2. 试图打开一个错误格式的 URL
3. 试图根据给定的字符串查找 class 对象，而这个字符串表示的类并不存在

**从责任角度看**：
1. Error 属于 JVM 需要负担的责任
2. RuntimeException 是程序应该负担的责任
3. Checked Exception 可检查异常是 Java 编译器应该负担的责任

ErrorAndException.java
```java
public class ErrorAndException {
  private void throwError() {
    throw new StackOverflowError();
  }
  private void throwRuntimeException() {
    throw new RuntimeException();
  }
  private void throwCheckedException() {
    try {
      throw new FileNotFoundException();
    } catch {
      e.printStackTrace();
    }
  }
}
```

**常见的 Error 以及 Exception**：

RuntimeException：
1. NullPointerException - 空指针引用异常
2. ClassCastException - 类型强制转换异常
3. IllegalArgumentException - 传递非法参数异常
4. IndexOutOfBoundsException - 下标越界异常
5. NumberFormatException - 数字格式异常

CheckedException：
1. ClassNotFoundException - 找不到指定 class 的异常
2. IOException - IO 操作异常

Error：
1. NoClassDefFoundError - 找不到 class 定义的异常
    1. 类依赖的 class 或者 jar 不存在
    2. 类文件存在，但是存在不同的域中
    3. 大小写问题，javac 编译的时候是无视大小写的，很有可能编译出来的 class 文件就与想要的不一样
2. StackOverflowError - 深递归导致栈被耗尽而抛出的异常
3. OutOfMemoryError - 内存溢出异常

##  异常的处理机制
- 抛出异常：创建异常对象，交由运行时系统处理
- 捕获异常：寻找合适的异常处理器处理异常，否则终止运行

遇到问题不进行具体处理，而是继续抛给调用者（**throw, throws**）

**抛出异常有 3 种形式，一是 throw，一个 throws，还有一种系统自动抛异常。

### Throw 和 Throws 的区别：
位置不同：
- **throws 用在函数上**，后面跟的是异常类，可以跟多个；**而 throw 用在函数内**，后面跟的是异常对象。
功能不同：
- **throws 用来声明异常，让调用者只知道该功能可能出现的问题**，可以给出预先的处理方式；**throw 抛出具体的问题对象，执行到 throw，功能就已经结束了**，跳转到调用者，并将具体的问题对象抛给调用者。也就是说 throw 语句独立存在时，下面不要定义其他语句，因为执行不到。
- **throws 表示出现异常的一种可能性**，并不一定会发生这些异常；**throw 则是抛出了异常**，执行 throw 则一定抛出了某种异常对象。
- 两者都是消极处理异常的方式，只是抛出或者可能抛出异常，但是不会由函数去处理异常，真正的处理异常由函数的上层调用处理。

### try-catch 的性能
Java 异常处理消耗性能的地方：
- try-catch 块影响 JVM 的优化
- 异常对象实例需要保存栈快照等信息，开销较大

**Java 异常的处理原则**：
- 具体明确：抛出的异常应能通过异常类名和 message 准确说明异常的类型和产生异常的原因；
- 提早抛出：应尽可能早的发现并抛出异常，便于精确定位问题；
- 延迟捕获：异常的捕获和处理应尽可能延迟，让掌握更多信息的作用域来处理异常。

**高效主流的异常处理框架**：

在用户看来，应用系统发生的所有异常都是应用系统内部的异常：
- 设计一个通用的继承自 RuntimeException 的异常来统一处理
- 其余异常都同一转译为上述异常 AppException
- 在 catch 之后，抛出上述异常的子类，并提供足以定位的信息
- 由前端接收 AppException 做统一处理

org.springframework.core.NestedRuntimeException











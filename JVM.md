# JVM
## 基本概念
JVM 是可运行 Java 代码的假想计算机，包括：
- 一套字节码指令集
- 一组寄存器
- 一个栈
- 一个堆
- 一个垃圾回收
- 一个存储方法域

JVM 是运行在操作系统之上的，它与硬件没有直接的交互。

## 运行过程
Java 代码的执行：

**Java 源码首先被编译成字节码，再由不同平台的 JVM 进行解析，Java 语言在不同平台上运行时不需要进行重新编译，Java 虚拟机在执行字节码的时候，把字节码转换成具体平台上的机器指令**。

Java 源文件通过编译器，能够产生相应的 .class 文件，也就是字节码文件，而字节码文件又通过 Java 虚拟机中的解释器，编译成特定机器上的机器码。

① Java 源文件 -> 编译器 -> 字节码文件

② 字节码文件 -> JVM -> 机器码

每一种平台的解释器是不同的，但是实现的虚拟机是相同的，这也就是 Java 为什么能够跨平台的原因了，当一个程序从开始运行，这时虚拟机就开始虚拟化了，多个程序启动就会存在多个虚拟机实例。

程序退出或者关闭，则虚拟机实例消亡，多个虚拟机实例之间数据不同共享。

## JVM 如何加载 .class 文件
通过 ClassLoader 加载 class 文件到内存（将字节码转换为 JVM 中的 Class<> 对象），并通过 execution engine 对 class 文件中的字节码进行解析，并提交给操作系统去解析

## 类的加载方式
- 隐式加载：new
- 显式加载：loadClass, forName 等
  - 需要调用 newInstance() 方法
  - 不支持参数（需要通过反射）

## JVM 组成部分
- 运行时数据区（Runtime Data Area）
  - 方法区（Method Area）
  - 堆（Heap）
  - 虚拟机栈（VM Stack）
  - 本地方法栈（）
  - 程序计数器（PC Register）
- 执行引擎：对命令进行解析
  - 即时编译器（JITCompiler）
  - 垃圾收集（Garbage Collection）
- 本地库接口（Java Native Interface）：融合不同开发语言的原生库为 Java 所用
- 本地方法库（Native Method Libraries）
- 类加载器子系统（Class Loader Subsystem）：依据特定格式，加载 class 文件到内存

### Native Interface 
Class.forName(String, boolean, ClassLoader)

```java
public static Class<?> forName(String className) throws ClassNotFoundException {
  Class<?> caller = Reflection.getCallerClass();
  return forName0(className, true, ClassLoader.getClassLoader(caller), caller);   // initialize: true（初始化）
}

private static native Class<?> forName0(String name, boolean i, ClassLoader loader, Class<?> caller) throws ClassNotFoundException;       // native 接口，无法看到实际实现
```

#### 反射机制
Java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为 Java 语言的反射机制。

反射的例子：
```java
package com.interview.javabasic.reflect;

public class Robot {
  private String name;
  public void sayHi(String helloSentence) {
    System.out.println(helloSentence + " " + name);
  }
  private String throwHello(String tag) {
    return "Hello " + tag;
  }
}
```

```java
package com.interview.javabasic.reflect;

public class ReflectSample {
  public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException {
    Class rc = Class.forName("com.interview.javabasic.reflect.Robot");
    Robot r = (Robot) rc.newInstance();
    System.out.println("Class name is " rc.getName());
    
    Method getHello = rc.getDeclaredMethod("throwHello", String.class);   // 获取所有方法除了继承父类和接口的方法
    getHello.setAccessible(true);                     // 获取 private 方法
    Object str = getHello.invoke(r, "Bob");
    System.out.println("getHello result is " + str);    
    
    Method sayHi = rc.getMethod("sayHi", String.class);      // 只能获取 public 的方法，但是可以获取继承父类和继承接口的方法
    sayHi.invoke(r, "Welcome");
          
    Field name = rc.getDeclaredField("name");       // 获取 private 成员
    name.setAccessible(true);
    name.set(r, "Alice");
    sayHi.invoke(r, "Welcome");
  }
}
```

### ClassLoader
主要工作在 Class 装载的加载阶段，其主要作用是从系统外部获得 Class 二进制数据流。

它是 Java 的核心组件，所有的 Class 都是由 ClassLoader 进行加载的，ClassLoader 负责通过将 Class 文件里的二进制数据流装载进系统，然后交给 Java 虚拟机进行连接、初始化等操作。

ClassLoader 源代码：
```java
public abstract class ClassLoader {     // 抽象类  
  ...
  public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);      // 重载（overload）
  }
  ...
  protected Class<?> loadClass(string name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
      // First check is the class has already been loaded
      Class<?> c = findLoadedClass(name);
      if (c == null) {
        long t0 = System.nanoTime();
        try {
          if (parent != null) {     // private final ClassLoader parent; -> 表明了 ClassLoader 种类并不是单一的
            c = parent.loadClass(name, false);
          } else {
            c = findBootstrapClassOrNull(name);
          }
        } catch (ClassNotFoundException e) {
        }
        if (c == null) {
          // If still not found, then invoke findClass to find the class 
          long t1 = System.nanoTime();
          c = findClass(name);
          ...
        }
      }
    }
  }
}
```
#### ClassLoader 的种类
- BootstrapClassLoader：C++编写，加载核心库 java.*
- ExtClassLoader：Java 编写，加载扩展库 javax.*
- AppClassLoader：Java 编写，加载程序所在目录
- 自定义 ClassLoader：Java 编写，定制化加载

ExtClassLoader
```java
static class ExtClassLoader extends URLClassLoader {
  ...
  private static File[] getExtDirs() {
    String var0 = System.getProperty("java.ext.dirs");    // 用到某扩展类时去这个目录进行加载
    ...
  }
}
```

AppClassLoader
```java
static class AppClassLoader extends URLClassLoader {
  ...
  public static ClassLoader getAppClassLoader(final ClassLoader var0) {
    final String var1 = System.getProperty("java.class.path");    
    ...
  }
}
```

自定义 ClassLoader 的实现：

关键函数：
```java
protected Class<?> findClass(String name) throws ClassNotFoundException {   // 寻找 class 文件
  throw new ClassNotFoundException(name);
}
```
```java
protected final Class<?> defineClass(byte[] b, int off, int len) throws ClassFormatError {  // 定义 class，接收参数为字节流
  return defineClass(null, b, off, len, null);
}
```

MyClassLoader.java
```java
import java.io;

public class MyClassLoader extends ClassLoader {
  private String path;
  private String classLoaderName;
  
  public MyClassLoader(String path, String classLoaderName) {
    this.path = path;
    this.classLoaderName = classLoaderName;
  }
  
  // 用于寻找类文件
  @Override
  public Class findClass(String name) {
    byte[] b = loadClassData(name);
    return defineClass(name, b, 0, b.length);
  }
  
  // 用于加载类文件
  private byte[] loadClassData(String name) {
    name = path + name + ".class";
    InputStream in = null;
    ByteArrayOutputStream out = null;
    try {
      in = new FileInputStream(new File(name));
      out = new ByteArrayOutputStream();
      int i = 0;
      while ((i = in.read()) != -1) {
        out.write(i);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        out.close();
        in.close();
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
    return out.toByteArray();
  }
}
```

#### ClassLoader 的双亲委派机制
1. 自底向上检查类是否已经加载（如果是，直接返回；否则，委派给 parent 找该类是否加载过）
2. （如果所有类加载器都没有加载过）-> 自顶向下尝试加载类

loadClass()
```java
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);      // 重载（overload）
}
  ...
  protected Class<?> loadClass(string name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {  // 可能存在多个线程调用加载同一个类的情况，避免冲突加同步锁
      // First check is the class has already been loaded；如果有，直接返回
      Class<?> c = findLoadedClass(name);
      // 自底向上检查类是否已经加载
      if (c == null) {
        long t0 = System.nanoTime();
        try {
          // 自定义ClassLoader -> AppClassLoader -> ExtClassLoader 
          // ExClassLoaoder -> null；因为 BootstrapClassLoader 是 C++编写的
          if (parent != null) {     // private final ClassLoader parent; -> 表明了 ClassLoader 种类并不是单一的
            c = parent.loadClass(name, false);
          } else {
            // 到达最顶端的类加载器：BootstrapClassLoader
            c = findBootstrapClassOrNull(name);
          }
        } catch (ClassNotFoundException e) {
        }
        // 自顶向下尝试加载类
        if (c == null) {
          // If still not found, then invoke findClass in order to find the class 
          long t1 = System.nanoTime();
          c = findClass(name);
          ...
        }
      }
      if (resolve) {
        resolveClass(c);
      }
      return c;
    }
```

为什么要使用双亲委派机制去加载类？
- 避免多份同样字节码的加载：不同类调用System.out.println()，只加载一份 system 字节码

### loadClass 和 forName 的区别
类的装载过程：
- 加载：通过 ClassLoader 加载 class 文件字节码，生成 Class 对象
- 链接
  - 校验：检查加载的 class 的正确性和安全性
  - 准备：为类变量分配存储空间并设置类变量初始值
  - 解析：JVM 将常量池内的符号引用转换为直接引用
- 初始化：执行类变量赋值和静态代码块

ClassLoader.java
```java
/**
 *  @param resolve: If true, then resolve the class
 */
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);      
}
  ...
  if (resolve) {
        resolveClass(c);
  }
  
/** Links the specified class.
 *  @param resolve: If true, then resolve the class
 */
public final void resolveClass(Class<?> c) {
    resolveClass0(c);      
}  
```

```java
public static Class<?> forName(String className) throws ClassNotFoundException {
  Class<?> caller = Reflection.getCallerClass();
  return forName0(className, true, ClassLoader.getClassLoader(caller), caller);   // initialize: true（初始化）
}
```
结论：
- Class.forName 得到的 class 是已经初始化完成的（已经完成第三步）
- classLoader.loadClass 得到的 class 是还没有链接的（只完成了第一步）

```java
package com.interview.javabasic.reflect;

public class Robot {
  static {                            // 静态代码段在类初始化的时候执行
    System.out.println("Hello Robot")
  }
}
```

LoadDifference.java
```java
public class LoadDifference {
  public static void main(String[] args) throws ClassNotFoundException {
    ClassLoader cl = Robot.class.getClassLoader();                    // 没有打印
    Class r = Class.forName("com.interview.javabasic.reflect.Robot"); // 打印出了 Hello Robot
    
    // 连接 MySQL 加载其 driver 的时候必须用 Class.forName
    // 因为 Driver 里面有一段静态代码段
    
    // Spring IOC lazy loading 用的是 ClassLoader，加快加载速度
  }
}
```

## JVM 内存模型——JDK8
JVM 内存区域主要分为：
- 线程私有区域
  - 程序计数器（字节码指令 no OOM）
  - 虚拟机栈（Java 方法 SOF & OOM）自动释放不需要 GC
  - 本地方法区（Native 方法 SOF & OOM）
- 线程共享区
  - 元空间（类加载信息 OOM）
  - Java 堆（数组和类对象 OOM）
    - 常量池（字面量和符号引用量 OOM）
- 直接内存

### 线程私有区域
**线程私有数据区域生命周期与线程相同，依赖用户线程的启动/结束，而创建/销毁（在 Hotspot VM 内**，每个线程都与操作系统的本地线程直接映射，因此这部分内存区域的存/否跟随本地线程的生/死对应）。

#### 程序计数器（线程私有）
- 当前线程所执行的字节码行号指示器（逻辑）
- 改变计数器的值来选取下一条需要执行的字节码指令
- 和线程是一对一的关系即“线程私有”
- 对 Java 方法计数，如果是 Native 方法则计数器值为 Undefined
- 不会发生内存泄漏

一块较小的内存空间，**是当前线程所执行的字节码的行号指示器**，每条线程都要有一个独立的程序计数器，这类内存也被称为“线程私有”的内存。

正在执行 Java 方法的话，计数器记录的时虚拟机字节码指令的地址（当前指令的地址）。如果还是 Native 方法，则为空。

这个内存区域是唯一一个在虚拟中没有规定任何 OutOfMemoryError 情况的区域。

#### 虚拟机栈（线程私有）—— 深度有限 
- Java 方法执行的内存模型
- 包含多个栈帧（java.lang.StackOverflowError 异常）
  - 局部变量表（Local Variable Table）：包含方法执行过程中的所有变量
  - 操作栈（Operand Stack）：入栈、出栈、复制、交换、产生消费变量
  - 动态连接（Dynamic Linking）
  - 返回地址（Return Address）

**是描述 Java 方法执行的内存模型，每个方法在执行的同事都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息**。

**每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程**。

**线程共享区域随虚拟机的启动/关闭而创建/销毁**。

**直接内存并不是 JVM 运行时数据区的一部分**，但也会被频繁的使用：在 JDK 1.4 引入的 **NIO 提供了基于 Channel 与 Buffer 的 IO 方式，它可以使用 Native 函数库直接分配堆外内存，然后使用 DirectByteBuffer 对象作为这块内存的引用进行操作，这样就避免了在 Java 堆和 Native 堆中来回复制数据，因此在一些场景中可以显著提高性能**。

虚拟栈过多会引发 java.lang.OutOfMemoryError 异常

#### 本地方法栈（线程私有） 
- 与虚拟机栈相似，主要作用于标注了 native 的方法

#### 元空间（MetaSpace）（线程共享） 
与永久代（PermGen）的区别：

**元空间使用本地内存，而永久代使用的是 JVM 的内存**：java.lang.OutOfMemoryError: PermGen space 不复存在

MetaSpace 相比 PermGen 的优势：
- 字符串常量池存在永久代中，容易出现性能问题和内存溢出
- 类和方法的信息大小难以确定，给永久代的大小指定带来困难
- 永久代会为 GC 带来不必要的复杂性
- 方便 HotSpot 与其他 JVM 如 Jrockit 的集成

#### Java 堆（线程共享） 
- 对象实例的分配区域
- GC 管理的主要区域

## Java 内存模型中堆和栈的区别——内存分配策略
- 静态存储：编译时确定每个数据目标在运行时的存储空间需求
- 栈式存储：数据区需求在编译时未知，运行时模块入口前确定
- 堆式存储：编译时或运行时模块入口都无法确定，动态分配

### Java 内存模型中堆和栈的区别
- 联系：引用对象、数组时，栈里定义变量保存堆中目标的首地址

区别：
- 管理方式：栈自动释放，堆需要 GC
- 空间大小：栈比堆小
- 碎片相关：栈产生的碎片远小于堆
- 分配方式：栈支持静态和动态分配，而堆仅支持动态分配
- 效率：栈的效率比堆高

## 元空间、堆、线程独占部分间的联系——内存角度
```java
public class HelloWorld {
  private String name;
  public void sayHello() {
    System.out.println("Hello " + name);
  }
  
  public void setName(String name) {
    this.name = name;
  }
  
  public static void main(String[] args) {
    int a = 1;
    HelloWorld hw = new HelloWorld();
    hw.setName("test");
    hw.sayHello();
  }
}
```

- 元空间：
  - Class: HelloWorld - Method: sayHello\setName\main - Field: name
  - Class: System
- Java 堆：
  - Object: String("test")
  - Object: HelloWorld
- 线程独占：
  - Parameter reference: "test" to String object
  - Variable reference: "hw" to HelloWorld object
  - Local Variables: a with 1, lineNo

## 不同 JDK 版本之间的 intern() 方法的区别——JDK6 VS. JDK6+
```java
String s = new String("a");
s.intern();
```

- JDK6：当调用 intern 方法时，如果字符串常量池先前已创建出该字符串对象，则返回池中的该字符串的引用。否则，将此字符串对象添加到字符串常量池中，并且返回该字符串对象的引用。
- JDK6+：当调用 intern 方法时，如果字符串常量池先前已创建出该字符串对象，则返回池中的该字符串的引用。否则，如果该字符串对象已经存在于 Java 堆中，则将堆中该对象的引用添加到字符串常量池中，并且返回该引用；如果堆中不存在，则在池中创建该字符串并返回其引用。

InternDiffernece.java
```java
public class InternDifference {
  public static void main (String[] args) {
    String s = new String("a");
    s.intern();
    String s2 = "a";
    System.out.println(s == s2);
    
    String s3 = new String("a") + new String("a");
    s3.intern();
    String s4 = "aa";
    System.out.println(s3 == s4);
  }
}
```
- JDK6: false false 
- JDK6+: false true

## JVM 类加载机制
JVM 类加载机制分为 5 个部分：加载，验证，准备，解析，初始化。

### 加载
加载是类加载过程的一个阶段，**这个阶段会在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的入口**。

注意这里不一定非得要从一个 Class 文件获取，这里既可以从 ZIP 包中获取（比如从 jar 包和 war 包中读取），也可以在运行时计算生成（动态代理），也可以由其它文件生成（比如 JSP 文件转换成对应的 Class 类）。

### 验证
这一阶段的主要目的是为了**确保 Class 文件的字节流中包含的信息是否符合当前虚拟机的要求**，并且不会危害虚拟机自身的安全。

### 准备
准备阶段是正式为类变量分配内存并设置类变量的初始值阶段，集**在方法区中分配这些变量所使用的内存空间**。

注意这里所说的初始值概念，比如一个类变量定义为：
```java
public static int v = 8080;
```
**实际上变量v在准备阶段过后的初始值为 0 而不是8080**，将v赋值为 8080 的 put static 指令是程序被编译后，**存放于类构造器&lt;client&gt;方法之中。
  
但是注意如果声明为：
```java
public static int v = 8080;
```
在编译阶段会为v生成 ConstantValue 属性，在**准备阶段虚拟机会根据 ConstantValue 属性将v赋值为 8080**。

### 解析
解析阶段是指**虚拟机将常量池中的符号引用替换为直接引用的过程**。

符号引用就是 class 文件中的：
1. CONSTANT_Class_info
2. CONSTANT_Field_info
3. CONSTANT_Method_info
等类型的常量。

#### 符号引用
符号引用与虚拟机实现的布局无关，**引用的目标并不一定要已经加载到内存中**。

**各种虚拟机实现的内存布局可以各不相同**，但是它们能接受的符号引用必须是一致的，因为符号引用的字面量形式明确定义在 Java 虚拟机规范的 Class 文件格式中。

#### 直接引用
直接引用可以是**指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄**。

如果有了直接引用，那**引用的目标必定已经在内存中存在**。

### 初始化
初始化阶段是类加载的最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载器以外，其它操作都由 JVM 主导。

到了初始阶段，才开始真正执行类中定义的 Java 程序代码。

#### 类构造器 &lt;client&gt;
初始化阶段是**执行类构造器&lt;client&gt;方法的过程**。
  
&lt;client&gt; 方法是由编译器自动收集类中的类变量的赋值操作和静态语句块中的语句合并而成。
  
虚拟机会保证子&lt;client&gt;方法执行之前，父类的&lt;client&gt;方法已经执行完毕，**如果一个类中没有对静态变量赋值也没有静态语句块，那么编译器可以不为这个类生成&lt;client&gt;()方法**。
  
注意以下几种情况不会执行类初始化：
1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。
2. 定义对象数组，不会触发该类的初始化。
3. 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不会触发定义常量所在的类。
4. 通过类名获取 Class 对象，不会触发类的初始化。
5. 通过 Class.forName 加载指定类时，如果指定参数 initialize 为 false 时，也不会触发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。
6. 通过 ClassLoader 默认的 loadClass 方法，也不会触发初始化动作。

## 类加载器
虚拟机设计团队把加载动作放到 JVM 外部实现，以便让应用程序决定如何获取所需的类，JVM 提供了 3 中类加载器：

### 启动类加载器（Bootstrap ClassLoader）
负责加载 **JAVA_HOME\lib** 目录中的，或通过 -Xbootclasspath 参数指定路径中的，且被虚拟机认可（按文件名识别，如rt.jar）的类。

### 扩展类加载器（Extension ClassLoader）
负责加载 **JAVA_HOME\lib\ext** 目录中的，或通过 java.ext.dirs 系统变量指定路径中的类库。

### 应用程序类加载器（Application ClassLoader）
负责加载**用户路径（classpath）上的类库**。

JVM 通过双亲委派模型进行类的加载，当然我们也可以通过继承 java.lang.ClassLoader 实现自定义的类加载器。

## 双亲委派
**当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成**，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载器中，只有当**父类加载器反馈自己无法完成这个请求的时候**（在它的加载路径下没有找到所需加载的 Class），**子类加载器才会尝试自己去加载**。

采用双亲委派的一个好处是比如加载位于 rt.jar 包中的类 java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了**使用不同的类加载器最终得到的都是同样一个 Object 对象**。

## JVM 三大性能调优参数 -Xms -Xmx -Xss 的含义
- Xss：规定了每个线程虚拟机栈（堆栈）的大小（一般情况下，256k）
- Xms：堆的初始值
- Xmx：堆能达到的最大值（一般和 Xms 设为一样，防止堆扩容造成内存抖动）

# GC

## JVM 运行时内存
Java 堆从 GC 的角度还可以细分为：
- 新生代
  - Eden 区
  - From Survivor 区
  - To Survivor 区
- 老年代

## 新生代 (1/3)
是用来存放新生的对象。一般占据堆的 1/3 空间。

由于频繁创建对象，所以新生代会频繁触发 MinorGC 进行垃圾回收。

尽可能快速地收集掉那些生命周期短的对象

### Eden 区 (8/10)
**Java 新对象的出生地**（如果新创建的对象占用内存很大，则直接分配到老年代）。

当 Eden 区内存不够的时候就会触发 MinorGC，对新生代进行一次垃圾回收。

### SurvivorFrom (1/10)
上一次 GC 的幸存者，作为这一次 GC 的被扫描者。

### SurvivorTo (1/10)
保留了一次 MinorGC 过程中的幸存者

### MinorGC 的过程（复制 -> 清空 -> 互换）
MinorGC 采用**复制算法**。

1. Eden, SurvivorFrom 复制到 SurvivorTo，年龄+1

首先，把 Eden 区和 SurvivorFrom 区域中存活的对象复制到 SurvivorTo 区域（如果有对象的年龄已经达到了老年的标准，则复制到老年代区：-XX:MaxTenuringThreshold），同时把这些对象的年龄+1（如果 SurvivorTo 不够位置了就放到老年区）

2. 清空 Eden，SurvivorFrom

然后，情况 Eden 和 SurvivorFrom 中的对象

3. SurvivorTo 和 SurvivorFrom 互换

最后，SurvivorTo 和 SurvivorFrom 互换，原 SurvivorTo 成为下一次 GC 的 SurvivorFrom 区。

**对象如何晋升到老年代**
- 经历一定 Minor 次数依然存活的对象
- Survivor 区中存放不下的对象
- 新生成的大对象（-XX:+PretenuerSizeThreshold）

**常用的调优参数**
- XX:SurvivorRatio：Eden 和 Survivor 的比值，默认 8:1
- XX:NewRatio：老年代和年轻代内存大小的比例
- XX:MaxTenuringThreshold：对象从新生代晋升到老年代经过 GC 次数的最大阈值

## 老年代 (2/3)
主要存放应用程序中生命周期长的内存对象。

老年代的对象比较稳定，所以 MajorGC 不会频繁执行。

在进行 MajorGC 前一般都先进行了一次 MinorGC，使得有新生代的对象晋升入老年代，导致空间不够用时才触发。

当无法找到足够大的连续空间分配给新创建的较大对象时也会提前触发一次 MajorGC 进行垃圾回收腾出空间。

MajorGC 采用**标记清除算法**：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。

MajorGC 的耗时比较长，因为要扫描再回收。

MajorGC 会产生内存碎片，为了减少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。

当老年代也满了装不下的时候，就会抛出 OOM（Out of Memory）异常。

触发 Full GC（Major GC）的条件：
- 老年代空间不足
- 永久代空间不足（Java8 之前）
- CMS GC 时出现 promotion failed,concurrent mode failure
- Minor GC 晋升到老年代的平均大小大于老年代的剩余空间
- 调用 System.gc()：程序员只能提醒 JVM，回不回收还是由 JVM 决定
- 使用 RMI 来进行 RPC 或管理的 JDK 应用，每小时执行 1 次 Full GC

## 永久代（JDK8 之前）
指内存的永久保存区域，主要存放 Class 和 Meta（元数据）的信息，Class 在被加载的时候被放入永久区域，它和存放实例的区域不同，GC 不会在主程序运行期对永久区域进行清理。

所以这也导致了永久代的区域会随着加载的 Class 的增多而胀满，最终抛出 OOM 异常.

## Java8 与元数据
在 Java8 中,**永久代已经被移除,被一个称为“元数据区”（元空间）的区域所取代**。

元空间的本质和永久代类似，元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**。

因此，默认情况下，元空间的大小仅受本地内存限制。

**类的元数据放入 native memory，字符串池和类的静态变量放入 Java 堆中**，这样可以加载多少类的元数据就不再由 MaxPermSize 控制，而由系统的实际可用空间来控制。

## 垃圾回收算法

如何确定垃圾？

对象被判定为垃圾的标准：
- 没有其他对象引用

### 引用计数算法
- 通过判断对象的引用数量来决定对象是否可以被回收
- 每个对象实例都有一个引用计数器，被引用则+1，完成引用则-1
- 任何引用计数为 0 的对象实例可以被当做垃圾收集

在 Java 中，引用和对象是有关联的。如果要操作对象必须用引用进行。

因此，很显然一个简单的方法是通过引用计数来判断一个对象是否可以回收。

简单说，即一个**对象如果没有任何与之关联的引用，即他们的引用计数都为 0，则说明对象不太可能再被用到，那么这个对象就是可回收对象**。

- 优点：执行效率高，程序执行受影响较小
- 缺点：无法检测出循环引用的情况，导致内存泄漏

### 可达性分析算法
为了解决引用计数法的循环引用问题，Java 使用了可达性分析的方法。

**通过判断对象的引用链是否可达来决定对象是否可以被回收**。

通过一系列的“GC roots”对象作为起点搜索。

如果**在“GC roots”和一个对象之间没有可达路径，则称该对象是不可达的**。

**可以作为 GC Root 的对象**：
- 虚拟机栈中引用的对象（栈帧中的本地变量表）
- 方法区中的常量引用的对象
- 方法区中的类静态属性引用的对象
- 本地方法栈中 JNI（Native 方法）的引用对象
- 活跃线程的引用对象   

要注意的是，不可达对象不等价于可回收对象，**不可达对象变为可回收对象至少需要经过两次标记过程**。

两次标记后仍然是可回收对象，则将面临回收。

### 标记清除算法（Mark and Sweep）
最基础的垃圾回收算法，分为**两个阶段，标记和清除**。

- 标记：从根集合进行扫描，对存活的对象进行标记
- 清除：对堆内存从头到尾进行线性扫描，回收不可达对象内存

标记阶段标记出所有需要回收的对象，清楚阶段回收被标记的对象所占用的空间。

该算法**最大的问题是内存碎片化严重**，后序可能发生大对象不能找到可利用空间的问题。

### 复制算法（copying）
为了解决 Mark-Sweep 算法内存碎片化的缺陷而被提出的算法。

- 分为对象面和空闲面
- 对象在对象面上创建
- 存活的对象被从对象面复制到空闲面
- 将对象面所有对象内存清除

按内存容量将内存划分为等大小的两块。

每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上去，把已使用的内存清掉。

这种算法虽然实现简单，内存效率高，不易产生碎片，但是最大的问题是可用内存被压缩到了原本的一半。

且存活对象增多的话，copying 算法的效率会大大降低。

- 解决碎片化问题
- 顺序分配内存，简单高效
- 适用于对象存活率低的场景

### 标记整理算法（Mark-Compact）
结合了以上两个算法，为了避免缺陷而提出。

标记阶段和 Mark-Sweep 算法相同，**标记后不是清理对象，而是将存活对象移向内存的一端。然后清除端边界外的对象**。

- 标记：从根集合进行扫描，对存活的对象进行标记
- 清除：移动所有存活的对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收

- 避免内存的不连续行
- 不用设置两块内存互换
- 适用于存活率高的场景

### 分代收集算法（Generational Collector）
分代收集算法是目前大部分 JVM 所采用的方法，其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将 GC 堆划分为老年代（Tenured/Old Generation）和新生代（Yong Generation）。

- 垃圾回收算法的组合拳
- 按照对象生命周期的不同划分区域以采用不同的垃圾回收算法
- 目的：提高 JVM 的回收效率

老年代的特点是每次垃圾回收时只有少量对象需要被回收，新生代的特点是每次垃圾回收时都有大量垃圾需要被回收，因此可以根据不同区域选择不同的算法。

#### GC 的分类
- Minor GC （新生代）
- Full GC（老年代）

#### 新生代与复制算法
目前大部分 JVM 的 GC 对于新生代都采用 Copying 算法，因为新生代中每次垃圾回收都要回收大部分对象，即要复制的操作比较少，但通常并不是按照 1:1 来划分新生代。

一般将新生代划分为一块较大的 Eden 空间和两个较小的 Survivor 空间（From Space, To Space），每次使用 Eden 空间和其中一块 Survivor 空间，当进行回收时，将该两块空间中还存活的对象复制到另一块 Survivor 空间中。

#### 老年代与标记整理算法
而老年代因为每次只回收少量对象，因而采用 Mark-Compact 算法。

1. Java 虚拟机提到过的处于**方法区的永生代（Permenant Generation），它用来存储 class 类，常量，方法描述等**。对永生代的回收主要包括废弃常量和无用的类。
2. 对象的内存分配主要在新生代的 Eden Space 和 Survivor Space 的 From Space（Survivor 目前存放对象的那一块），少数情况会直接分配到老年代。
3. 当新生代的 Eden Space 和 From Space 空间不足时就会发生一次 GC,进行 GC 后，Eden Space 和 From Space 区的存活对象会被挪到 To Space，然后将 Eden Space 和 From Space 进行清理。
4. 如果 To Space 无法足够存储某个对象，则将这个对象存储到老年代。
5. 在进行 GC 后，使用的便是 Eden Space 和 To Space 了，如此反复循环。
6. 当对象在 Survivor 区躲过一次 GC 后，其年龄就会+1。**默认情况下年龄达到 15 的对象会被移到老年代中**。

## Stop-the-World
- JVM 由于要执行 GC 而停止了应用程序的执行
- 任何一种 GC 算法中都会发生
- 多数 GC 优化通过减少 Stop-the-World 发生的时间来提高程序性能

## Safepoint
- 分析过程中对象引用关系不会发生关系的点
- 产生 Safepoint 的地方：方法调用；循环跳转；异常跳转等；
- 安全点数量得适中

## GC 垃圾收集器
Java 堆内存被划分为新生代和老年代两部分，新生代主要使用复制和标记-清除垃圾回收算法；老年代主要使用标记-整理垃圾回收算法，因此 Java 虚拟机针对新生代和老年代分别提供了多种不同的垃圾收集器，JDK1.6 中 Sun HotSpot 虚拟机的垃圾收集器如下：
[!GC Collector](images/GC%20collector.png)

JVM 的运行模式：
- Server
- Client

java -version -> Server VM   

### 新生代常见的垃圾收集器：

#### Serial 收集器（-XX:+UseSerialGC，复制算法）
- 单线程收集，进行垃圾收集时，必须暂停所有工作线程
- 简单高效，Client 模式下默认的新生代收集器

#### Parnew 收集器（-XX:+UseParNewGC，复制算法）
- 多线程收集，其余的行为、特点和 Serial 收集器一样
- 单核执行效率不如 Serial，在多核下执行才有优势

#### Parallel Scavenge 收集器（-XX:+UseParallelGC，复制算法）
- 比起关注用户线程停顿时间，更关注系统的吞吐量（吞吐量 = 运行用户代码时间 /（运行用户代码时间+垃圾收集时间））
- 在多核下执行才有优势，Server 模式下默认的新生代收集器

### 老生代常见的垃圾收集器：

#### Serial Old 收集器（-XX:+UseSerialOldGC，标记-整理算法）
- 单线程收集，进行垃圾收集时，必须暂停所有工作线程
- 简单高效，Client 模式下默认的老年代收集器

#### Parallel Old 收集器（-XX:+UseParallelOldGC，标记-整理算法）
- 多线程，吞吐量优先

#### CMS 收集器（-XX:+UseConcMarkSweepGC, 标记-清除算法）
Concurrent Mark Sweep（CMS）收集器是一种老年代垃圾收集器，其**最主要目标是获取最短垃圾回收停顿时间**，和其他老年代使用标记-整理算法不同，它使用**多线程的标记-清除算法**。

最短的垃圾收集停顿时间可以为交互比较高的程序提高用户体验。

CMS 工作机制相比其他的垃圾收集器来说更复杂，整个过程分为以下 6 个阶段：

##### 初始标记（stop-the-world）
只是标记以下 GC Roots 能直接关联的对象，速度很快，仍然需要暂停所有的工作线程。

##### 并发标记（一遍产生垃圾，一遍回收垃圾）
进行 GC Roots 并发追溯的过程，和用户线程一起工作，不需要暂停工作线程。

##### 并发预清理
查找执行并发标记阶段从新生代晋升到老年代的对象

##### 重新标记（stop-the-world）
**暂停虚拟机，扫描 CMS 堆中的剩余对象**

为了修正正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程。

##### 并发清除
清除 GC Roots 不可达对象，和用户线程一起工作，不需要暂停工作线程。

由于耗时最长的并发标记和并发清除过程中，垃圾收集线程可以和用户线程在一起并发工作，**所以总体上来看 CMS 收集器的内存回收和用户线程是一起并发地执行**。

##### 并发重置
重置 CMS 收集器的数据结构

#### G1 收集器（-XX:+UseG1GC，复制+标记-整理算法）
- 并行和并发
- 分代收集
- 空间整合
- 可预测的停顿

Garbage First 垃圾收集器是目前垃圾收集器理论发展的最前沿结果，相比于 CMS 收集器，G1 收集器两个最突出的改进是：
1. 基于标记-整理算法，不产生内存碎片。
2. 可以非常精准控制停顿时间，在不牺牲吞吐量的前提下，实现低停顿垃圾回收。

- 将整个 Java 堆内存划分成多个大小相等的 Region
- 新生代和老年代不再物理隔离

**G1 收集器避免全区域垃圾收集，它把堆内存划分为大小固定的几个独立区域**，并且跟踪这些区域的垃圾收集进度，同时在后台维护一个优先级列表，每次根据所允许的收集时间，**优先回收垃圾最多的区域**。

区域划分和优先级区域回收机制，确保 G1 收集器可以在有限时间获得最高的垃圾收集效率。

## JDK11: Epsilon GC & ZGC

## GC 相关的面试题

### Object 的 finalize() 方法的作用是否与 C++ 的析构函数作用相同？
- 与 C++ 的析构函数不同，析构函数调用确定，而它的是不确定的
- 将未被引用的对象放置于 F-Queue 队列
- 方法执行随时可能会被终止
- 给予对象最后一次重生的机会

Finalization.java
```java
public class Finalization {
  public static Finalization finalization;
  @Override 
  protected void finalize() {
    System.out.println("Finalized");
    finalization = this;
  }
  
  public static void main(String[] args) {
    Finalization f = new Finalization();
    System.out.println("First print: " + f);    
    f = null;
    System.gc();
    try {       // 休息一段时间，让上面的垃圾回收线程执行完成
      Thread.currentThread().sleep(1000);
    } catch (InterruptException e) {
      e.printStackTrace();
    }
    System.out.println("Second print: " + f);       
    System.out.println(f.finalization);
  }
}
```
Fist print: 对象 1
Finalized 
Second print: null
对象 1

### Java 中的强引用，软引用，弱引用，虚引用有什么用？
强引用 > 软引用 > 弱引用 > 虚引用

| 引用类型 | 被垃圾回收时间 | 用途 | 生存时间 | 
| --- | --- | --- | --- | 
| 强引用 | 从来不会 | 对象的一般状态 | JVM 停止运行时终止 | 
| 软引用 | 在内存不足时 | 对象缓存 | 内存不足时终止 |
| 弱引用 | 在垃圾回收时 | 对象缓存 | GC 运行后终止 |
| 虚引用 | unknown | 标记、哨兵 | unknown | 

类层次结构：
- java.lang.Object 
  - java.lang.ref.Reference
    - java.lang.ref.SoftReference 
    - java.lang.ref.WeakReference
    - java.lang.ref.PhantomReference
  - java.lang.ref.ReferenceQueue

#### 强引用（Strong Reference）
- 最普遍的引用：Object = new Object()
- 抛出 OutOfMemoryError 终止程序也不会回收具有强引用的对象
- 通过将对象设置为 null 来弱化引用，使其被回收

**把一个对象赋给一个引用变量，这个引用变量就是一个强引用**。

当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即该对象以后永远都不会被用到，JVM 也不会回收。

强引用是造成 Java 内存泄漏的主要原因之一。

#### 软引用（Soft Reference）
- 对象处在有用但非必须的状态
- 只有当内存空间不足时，GC 会回收该引用的对象的内存
- 可以实现高速缓存

**软引用需要用 SoftReference 类实现**，对于只有软引用的对象来说，当系统内存组够时它不会被回收，当系统内存空间不足时它会被回收。

软引用通常用在对内存敏感的程序中。

```java
string str = new String("abc");   // 强引用
SoftReference<String> softRef = new SoftReference<String>(str);   // 软引用
```

#### 弱引用（Weak Reference）
- 非必须的对象，比软引用更弱一些
- GC 时会被回收
- 被回收的概率也不大，因为 GC 线程优先级比较低
- 适用于引用偶尔被使用且不影响垃圾收集的对象

弱引用需要用 WeakReference 类来实现，它比软引用的生存期更短，对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管 JVM 的内存空间是否足够，总会回收该对象占用的内存。

```java
string str = new String("abc");   // 强引用
WeakReference<String> abcWeakRef = new WeakReference<String>(str);   // 弱引用
```

#### 虚引用（Phantom Reference）
- 不会决定对象的声明周期
- 任何时候都可能被垃圾收集器回收
- 跟踪对象被垃圾收集器回收的活动，起哨兵作用
- 必须和引用队列 ReferenceQueue 联合使用

虚引用需要 PhantomReference 类来实现，它不能单独使用，必须和引用队列联合使用。

**虚引用的主要作用是跟踪对象被垃圾回收的状态**。

```java
string str = new String("abc");   // 强引用
ReferenceQueue queue = new ReferenceQueue();
PhantomReference ref = new PhantomReference(str, queue);   // 虚引用
```

#### 引用队列（ReferenceQueue）
- 无实际存储结构，存储逻辑依赖于内部节点之间的关系来表达
- 存储关联的且被 GC 的软引用、弱引用以及虚引用

ReferenceQueue.java
```java
private volatile Reference<? extends T> head = null;

boolean enqueue(Reference<? extends T> r) {
  synchronized (lock) {
    ReferenceQueue<?> queue = r.queue;
    if ((queue == null) || (queue == ENQUEUED)) {
      return false;
    }
    assert queue == this;
    r.queue = ENQUEUED;
    r.next = (head == null) ? r : head;
    head = r;
    queueLength++;
    ...
  }
}

public Reference<? extends T> poll() {
  if (head == null) {
    return null;
  }
  synchronized (lock) {
    return reallyPoll();
  }
}
```
Reference.java
```java
private T referent;
volatile ReferenceQueue<? super T> queue;
Reference next;
```














































































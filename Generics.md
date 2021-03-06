# Java 泛型
泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。

泛型的本质是**参数化类型**，也就是说所操作的数据类型被指定为一个参数，解决不确定对象具体类型的问题。

比如我们要写一个排序方法，能够对整型数组、字符串数组甚至其他任何类型的数组进行排序，我们就可以使用 Java 泛型。

## 泛型好处
1. 类型安全，放置什么类型，取出来就是什么类型，不存在 ClassCastException 类型转换异常。
2. 提升可读性，编码阶段就显式知道泛型集合、泛型方法等处理的对象类型。
3. 代码重用，合并了同类型的处理代码。

## 泛型方法&lt;E&gt;
你可以写一个泛型方法，该方法在调用时可以接受不同类型的参数。

根据传递给泛型方法的参数类型，编译器适当地处理每一个方法的调用。
```java
// 泛型方法 printArray
public static <E> void printArray(E[] inputArray) {
  for (E element : inputArray) {
    System.out.println("%s", element);
  }
}
```
1. &lt;? extends T&gt;表示该通配符所代表的的类型是 T 类型的子类。
2. &lt;? super T&gt;表示该通配符所代表的的类型是 T 类型的父类。

## 泛型类&lt;T&gt;
泛型类的声明和非泛型类的声明类似，除了在类名后面添加了类型参数声明部分。

和泛型方法一样，泛型类的类型参数声明部分也包含一个或多个类型参数，参数间用逗号隔开。

一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。

因为他们要接收一个或多个参数，这些类被称为参数化的类或参数化的类型。
```java
public class Box<T> {
  private T t;
  public void add(T t) {
    this.t = t;
  }
  public T get() {
    return t;
  }
}
```

### 类型通配符？
类型通配符一般是使用？代替具体的参数类型。

例如 List<?> 在逻辑上是 List&lt;String&gt;, List&lt;Integer&gt; 等所有 List&lt;具体类型实参&gt;的父类。

### 泛型擦除
**Java 中的泛型基本上都是在编译器这个层次来实现的。在生成的 Java 字节码中是不包含泛型中的类型信息的**。

**泛型用于编译阶段**

使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉，编译后的字节码文件不包含泛型类型信息，因为虚拟机没有泛型类型对象，所有对象都属于普通类。

这个过程就称为类型擦除。

如在代码中定义的 List&lt;String&gt;, List&lt;Object&gt; 等类型，在编译之后都会变成 List。

JVM 看到的只是 List，而由泛型附加的类型信息对 JVM 来说是不可见的。

类型擦除的基本过程也比较简单，首先是找到用来替换类型参数的具体类。这个具体类一般是 Object。

如果指定了类型参数的上界的话，则使用这个上界。把代码中的类型参数都替换成具体的类。

定义一个泛型类型，会自动提供一个对应原始类型，类型变量会被擦除。

如果没有限定类型就会替换为 Object，如果有限定类型就会替换为第一个限定类型，例如 <T extends A & B> 会使用 A 类型替换 T。
















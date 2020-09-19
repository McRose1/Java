# Java 内部类
Java 类中不仅可以定义变量和方法，还可以定义类，这样定义在类内部的类就被称为内部类。

特点：
- 可以使用private、protected修饰，也可以使用abstract、final等修饰
- 内部类可以直接或引用访问外部类的属性和方法，包括私有属性和方法（但静态内部类不能访问外部类的非静态成员变量和方法）。内部类所访问的外部属性的值由构造时的外部类对象决定
- 外部类要访问内部类的成员，则只能通过引用的方式进行，可访问内部类的所有成员
- 内部类可以继承同级的内部类，也可继承其他类（除内部类和外部类）
- 内部类可以定义为接口，并且可以定义另外一个类来实现它
- 内部类可以定义为抽象类，并且可以定义另外一个内部类来继承它
- 方法内的内部类不能加范围限定（protected、private、public），方法内的内部类不能加static修饰符，方法内的内部类只能在方法内构建，方法内的内部类如果访问方法局部变量，则此局部变量必须使用final修饰

根据定义的方式不同，内部类分为静态内部类，成员内部类，局部内部类，匿名内部类 4 种。

## 静态内部类
定义在类内部的静态类，就是静态内部类。
```java
public class Out {
  private static int a;
  private int b;
  public static class Inner {
    public void print() {
      System.out.println(a);
    }
  }
}
```
1. 静态内部类可以访问外部类所有的静态变量和方法，即使是 private 的也一样。
2. 静态内部类和一般类一致，可以定义静态变量、方法、构造方法等。
3. 其它类使用静态内部类需要使用“外部类.静态内部类”方式，如下所示：
```java
Out.Inner inner = new Out.Inner();
inner.print();
```
4. **Java 集合类 HashMap 内部就有一个静态内部类 Entry**。Entry 是 HashMap 存放元素的抽象，HashMap 内部维护 Entry 数组用了存放元素，但是 Entry 对于使用者是透明的。像这种和外部类关系密切的，且不依赖外部类实例的，都可以使用静态内部类。

## 成员内部类
定义在类内部的非静态类，就是成员内部类。

成员内部类不能定义静态方法和变量（final 修饰的除外）。

这是因为成员内部类是非静态的，**类初始化的时候先初始化静态成员，如果允许成员内部类定义静态变量，那么成员内部类的静态变量初始化顺序是有歧义的**。
```java
public class Out {
  private static int a;
  private int b;
  public class Inner {
    public void print() {
      System.out.println(a);
      System.out.println(b);
    }
  }
}
```

## 局部内部类（定义在方法中的类）
定义在方法中的类，就是局部类。如果一个类只在某个方法中使用，则可以考虑使用局部类。
```java
public class Out {
  private static int a;
  private int b;
  public void test(final int c) {
    final int d = 1;
    class Inner {
      public void print() {
        System.out.println(c);
      }
    }
  }
}
```

## 匿名内部类
匿名内部类我们必须要继承一个父类或者实现一个接口，当然也仅能继承一个父类或者实现一个接口。

同时它也是没有 class 关键字，这是因为匿名内部类是直接使用 new 来生成一个对象的引用。
```java
public abstract class Bird {
  private String name;
  public String getName() {
    return name;
  }
  publicvoid setName(String name) {
    this.name = name;
  }
  public abstract int fly();
}
public class Test {
  public void test(Bird bird) {
    System.out.println(bird.getName() + "能够飞" + bird.fly() + "米");
  }
  public static void main(String[] args) {
    Test test = new Test();
    test.test(new Bird() {
      public int fly() {
        return 10000;
      }
      public String getName() {
        return "大雁";
      }
    });
  }
}
```





















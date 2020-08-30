# Class

## 局部变量 && 成员变量
```java
public class TestA {
  private String a = "成员变量（类变量）中的 a";
  
  public void useA() {
    String a= "局部变量（方法）中的 a";
    System.out.println(this.a);
    System.out.println(a);
  }
  
  public static void main (String[] args) {
    TestA t = new TestA();
    t.useA();
  }
}
```

局部变量可以和成员变量重名，不加“this”修饰时，优先使用最近的变量。

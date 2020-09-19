区别|this|super
----|-----|-----|
操作属性|this.属性：表示调用本类中的属性，如果本类中的属性不存在，则从父类查找|super.属性：表示调用父类中的属性|
操作方法|this.方法()：表示调用本类中的方法，如果本类中的方法不存在，则从父类查找|super.方法()：表示调用父类中的方法|
调用构造|this()调用本类中的其他构造方法|super()：由子类调用父类中的构造方法|
查找范围|先从子类中查找，如果没有从父类中查找|不查子类直接查找父类|
特殊|当前对象|-|


# super 
让子类可以使用父类的**变量或方法**

```java
class Vehicle {
  int maxSpeed = 120;
  
}

class Car extends Vehicle {
  int maxSpeed = 100;
  
  public void display() {
    System.out.println(super.maxSpeed);
  }
}

class Main {
  public static void main(String[] args) {
    Car c = new Car();
    System.out.println(c.maxSpeed);    // -> 100
    c.display();                       // -> 120
  }
}
```

# This

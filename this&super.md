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

# Abstract Class 抽象类

** Abstract class 不能有方法体，只能申明**
```java
abstract class xy {
  abstract sum (int x, int y);
}
```

抽象类中的抽象方法（其前有 abstract 修饰）不能用
- private：抽象方法没有方法体，是用来被继承的
- static：可以通过类名来访问该方法（即该方法的方法体），抽象方法用 static 修饰没有意义
- synchronized：使用 synchronized 关键字是为该方法加一个锁。而如果该关键字修饰的时 static 方法，则使用的锁就是 class 变量的锁；如果是修饰类方法，则用 this 变量锁。但是抽象类不能实例化对象，因为该方法不是在抽象类中实现的，是在其子类实现的，所以，锁应该归其子类所有。
- native：本身就和 abstract 冲突，它们都是方法的声明，只是一个把方法实现移交给子类，另一个是移交给本地操作系统。如果同时出现，就相当于把实现既交给子类，又交给本地操作系统

访问修饰符修饰。

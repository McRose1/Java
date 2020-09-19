# Java对象
Java对象的声明周期包括**创建、使用和清除**。

## 创建
### 显式创建对象
#### 使用new关键字创建对象
类名 对象名 = new 类名();

#### 调用 java.lang.Class 或者 java.lang.reflect.Constructor 类的 newInstance() 实例方法
- java.lang.Class 类对象名称 = java.lang.Class.forName(要实例化的类全称);
  - 调用 java.lang.Class 类中的 forName() 方法时，需要把要实例化的类的全程（比如 com.mxl.package.Student）作为参数传递进去，然后再调用 java.lang.Class 类对象的 newInstance() 方法创建对象
- 类名 对象名 = （类名）Class类对象名.newInstance();

#### 调用对象的 clone() 方法
该方法不常用，**使用该方法创建对象时，要实例化的类必须继承 java.lang.Cloneable 接口**。

类名对象名 = （类名）已创建好的类对象名.clone();

#### 调用 java.io.ObjectInputStream 对象的 readObject() 方法

### 隐式创建对象
- String strName = "strValue";
- String str1 = "Hello"; String str2 = "Java"; String str3 = str1 + str2;
- 当Java虚拟机加载一个类时，会隐式地创建描述这个类的Class实例

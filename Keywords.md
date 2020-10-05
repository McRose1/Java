# Java 关键字
Java的关键字对Java的编译器有特殊的意义，它们用来表示一种数据类型，或者表示数据的结构等，关键字不能作为变量名、方法名、类名、包名和参数。
## 数据类型关键字
- byte
- short
- int
- long
- double 
- long
- char
- boolean
- new 
- void
- instanceof

## 语句关键字
- if
- else
- return 
- for
- while
- break
- continue
- do
- try
- catch
- throw 
- finally
- switch
- case
- default
- this
- super

## 修饰关键字
- public 
- private 
- protected
- abstract 
- final 
- static 
- native
- volatile 
- synchronized 
- transient 

---

### Static 
static 关键字主要有两个作用：
1. 为某特定数据类型或对象分配单一的存储空间，而与创建对象的个数无关
2. 让某个属性或方法**与类关联在一起，而不是与对象关联在一起**，可以在不创建对象的情况下通过类名来访问

可以修饰变量、方法和代码块

#### static variable
static 修饰的变量称为静态变量，也叫类变量，可以直接通过类名访问，静态变量存储在JVM的方法区（Method Area）中。

#### static method
static 修饰的方法称为静态方法，也叫类方法，可以直接通过类名访问，静态方法**只能访问静态变量或静态方法，不能访问实例成员变量和实例方法，也不能使用this和super关键字**，通常用于定义工具类方法。

#### static block
static 修饰的代码块称为静态代码块，**只能定义在类下**，**在类加载时只会执行一次**，通常用于初始化属性和环境配置等。

#### static class
static 修饰的类称为静态内部类，**可以访问外部类的静态变量和方法**

static 也可以用来导入包下的静态变量。

#### 类初始化的顺序
1. 父类静态代码块和静态变量
2. 子类静态代码块和静态变量
3. 父类普通代码块和普通变量
4. 父类构造方法
5. 子类普通代码块和普通变量
6. 子类构造方法

其中代码块和变量初始化顺序按照类中声明的顺序执行

---


## 用于方法、类、接口、包和异常
- class
- extends 
- implements 
- interface
- package 
- import 
- throws 

## 保留字（保留的没有意义的关键字）
- cat
- future
- generic
- inner 
- operator 
- outer 
- rest
- var
- ...

不是关键字，而是文字，不可以作为标识符使用
- true
- false
- null

# Modifier 修饰符
Access level modifiers determine whether other classes can use a particular field or invoke a particular method.

There are 2 levels of access control:
- At the top level
  - public 
  - default
- At the member level
  - public 
  - private 
  - protected 
  - default 

---

Access Levels 
|Modifier|Class|Package|Subclass|World|
|---|---|---|---|---|
|public|√|√|√|√|
|protected|√|√|√|×|
|default|√|√|×|×|
|private|√|×|×|×|

## class modifier（类修饰符）
- public: 可以从其他类中访问
- abstract: 本类不能被实例化
- final: 不能再声明子类

## constructor modifier（构造函数修饰符）
- public: 可以从所有的类中访问
- protected: 只能从自己的类和它的子类中访问
- private: 只能在本类中访问

## field modifier（域/成员变量修饰符）
- public: 可以从所有的类中访问
- protected: 只能从自己的类和它的子类中访问
- private: 只能在本类中访问
- static: 对该类的所有实例只能有一个域值存在
- transient: 不是一个对象持久状态的一部分
- volatile: 可以被异步的线程所修改
- final: 必须对它赋予初值并且不能修改它

## local variable modifier（局部变量修饰符）
- final：必须对它赋予初值并且不能修改它

## method modifier（方法修饰符）
- public: 可以从所有的类中访问
- protected: 只能从自己的类和它的子类中访问
- private: 只能在本类中访问
- abstract: 没有方法体，属于一个抽象类
- final: 子类不能覆盖它
- static: 被绑定于类本身而不是类的实例
- native: 该方法由其他编程语言实现
- asynchronized: 在一个线程调用它之前必须先给它加锁 


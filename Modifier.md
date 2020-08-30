# Modifier 修饰符

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

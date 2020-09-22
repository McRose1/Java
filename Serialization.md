# Java 序列化（创建可复用的 Java 对象）
Serialzation is the process of translating data stuctures or object state into a format that can be stored (for example, in a file or memory buffer) or transmitted (for example, accorss the network connection link) and reconstructed later (possibly in a different computer environment).

对象序列化机制允许把内存的 Java 对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。（序列化）

当其他程序获取了这种二进制流，就可以恢复成原来的 Java 对象（反序列化）

Why do we need serialization?

For a complex object, "data" are everywhere in memory.

How can we store this object to a file?

How can we pass this object to another process for Inter-process communicatiion (IPC) or even machine for remote procedure call (RPC) ?

We can't just store the memory addresses since there is no way to restore it.

We need serialization!

We serialize an object into text (a human readable string/bytes array) or bytes (bianry format)

- 保存（持久化）对象及其状态到内存或者磁盘
Java 平台允许我们在内存中创建可复用的 Java 对象，但一般情况下，只有当 JVM 处于运行时，这些对象才可能存在，即，这些对象的声明周期不会比 JVM 的生命周期更长。

**但在现实应用中，就可能要求在 JVM 停止运行之后能够保存（持久化）指定的对象，并在讲来重新读取被保存的对象。

Java 对象序列化就能够帮助我们实现该功能。

## 序列化对象以字节数组保持-静态成员不保存
使用 Java 对象序列化，**在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象**

必须注意的是，**对象序列化保存的是对象的“状态”，即它的成员变量**。

由此可知，对象序列化不会关注类中的静态变量。

## 序列化用户远程对象传输
**除了在持久化对象时会用到对象序列化之外，当使用 RMI（远程方法调用），或在网络中传递对象时**，都会用到对象序列化。

Java 序列化 API 为处理对象序列化提供了一个标准机制，该 API 简单易用。

如果需要让某个对象支持序列化机制，则必须让对象所属的类及其属性是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。

否则，会抛出 NotSerializableException 异常

To serialize an object, the parent class should implement either of the following interfaces:
- java.io.Serializable 
- java.io.Externalizable 

## Serializable 实现序列化
在 Java 中，**只要一个类实现了 java.io.Serializable 接口，那么它就可以被序列化**。



















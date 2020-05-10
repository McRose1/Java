# Java 集合

## Interface 继承关系和实现
集合类存放于 java.util 包中，主要有 3 种：set（集合）、list（列表包含 Queue）和 map（映射）。

1. Collection：Collection 是集合 List、Set、Queue 的最基本的接口。
2. Iterator：迭代器，可以通过迭代器遍历集合中的数据
3. Map：是映射表的基础接口

## List
Java 的 List 是非常常用的数据类型。List 是有序的 Collection。Java List 一共 3 个实现类：ArrayList, LinkedList, Vector

### ArrayList（数组）
ArrayList 是最常用的 List 实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。

数组的缺点是每个元素之间不能有间隔，**当数组大小不满足时需要增加存储能力，就要将已经有数组的数据复制到新的存储空间中**。

**当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进行复制、移动，代价比较高。因此，它适合随机查找和遍历，不适合插入和删除**。

### LinkedList（链表）
**LinkedList 是用链表结构存储数据的，很适合数据的动态插入和删除**，随机访问和遍历速度比较慢。

另外，它还提供了 List 接口中没有定义的方法，专门用于操作表头和表尾元素，可以当做堆栈、队列和双向队列使用。

### Vector（数组实现线程同步）
Vector 和 ArrayList 一样，也是通过数组实现的，不同的是它**支持线程的同步，即某一时刻只有一个线程能够写 Vector**，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问 ArrayList 慢。

## Set
Set 注重独一无二的性质，该体系集合用于存储无序（存入和取出的顺序不一定相同）元素，**值不能重复**。

对象的相等性本质是对象的 hashCode 值（Java 是依据对象的内存地址计算出的此序号）判断的，**如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的 hashCode 方法和 equals 方法**。

### HashSet（Hash 表）
哈希表存放的是哈希值。HashSet 存储元素的顺序并不是按照存入时的顺序（和 List 显然不同），而是按照哈希值来存的所以取数据也是按照哈希值取得。

元素的哈希值是通过元素的 hashcode 方法来获取的，**HashSet 首先判断两个元素的哈希值，如果哈希值一样，接着会比较 equals 方法，如果 equals 结果为 true，HashSet 就视为同一个元素。如果为 false 就不是同一个元素**。

哈希值相同 equals 为 false 的元素是怎么存储的呢？就是在同样的哈希值下顺延（可以认为哈希值相同的元素放在一个哈希桶中），也就是哈希值一样的存一列。

HashSet 通过 hashCode 值来确定元素在内存中的位置，**一个 hashCode 位置上可以存放多个元素**。

## Map

### HashMap（数组+链表+红黑树）
HashMap 根据键的 hashCode 值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。

HashMap 最多只允许一条记录的键为 null，允许多条记录的值为 null。

HashMap 非线程安全，即任一时刻可以有多个线程同时写 HashMap，可能会导致数据的不一致。

如果需要满足线程安全，可以用 Collections 的 synchronizedMap 方法时 HashMap 具有线程安全的能力，或者使用 ConcurrentHashMap。

#### Java7 实现
大方向上，HashMap 里面是一个数组，然后数组中每个元素是一个单向链表。

每个元素的实体是嵌套类 Entry 的实例，Entry 包含 4 个属性：**key, value, hash值, 和用于单向链表的 next**。

1. capacity：当前数组容量，始终保持 2^n，可以扩容，扩容后数组大小为当前的 2 倍。
2. loadFactor：负载因子，默认为 0.75.
3. threshold：扩容的阈值，等于 capacity * loadFactor

#### Java8 实现
Java8 对 HashMap 进行了一些修改，**最大的不同就是利用了红黑树，所以其由 数组+链表+红黑树 组成。

根据 Java7 HashMap，我们知道查找的时候，根据 hash 值我们能够快速定位到数组的具体下标，但是之后的话，**需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，即 O(n)**。

为了降低这部分的开销，在 Java8 中，**当链表中的元素超过了 8 个以后，会将链表转换为红黑树**， 在这些位置进行查找的时候可以降低时间复杂度为 O(logN)











































# Java 集合框架

## Interface 继承关系和实现
集合类存放于 java.util 包中，主要有 3 种：set（集合）、list（列表包含 Queue）和 map（映射）。

1. Collection：Collection 是集合 List、Set、Queue 的最基本的接口。
    - List
    - Set
    - Queue
2. Iterator：迭代器，可以通过迭代器遍历集合中的数据
3. Map：是映射表的基础接口

## List
- 特点
    - 有序（存储和取出的顺序）
    - 可重复
    - 可通过索引值操作元素
- 分类
    - 底层是数组，查询快，增删慢
        - ArrayList —— 线程不安全，效率高
        - Vector —— 线程安全，效率低
    - 底层是链表，查询慢，增删快
        - LinkedList —— 线程不安全，效率高

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
- 特点
    - 无序（存储和取出的顺序）
    - 元素唯一
- 分类
    - 底层是 HashMap
        - HashSet：保证元素唯一性
            - hashCode()
            - equals()
    - 底层是二叉树
        - TreeSet：保证元素排序
            - 自然顺序，让对象所属的类去实现 comparable 接口，无参构造
            - 比较器接口 comparator，带参构造

Set 注重独一无二的性质，该体系集合用于存储无序（存入和取出的顺序不一定相同）元素，**值不能重复**。

对象的相等性本质是对象的 hashCode 值（Java 是依据对象的内存地址计算出的此序号）判断的，**如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的 hashCode 方法和 equals 方法**。

### HashSet（Hash 表）
哈希表存放的是哈希值。HashSet 存储元素的顺序并不是按照存入时的顺序（和 List 显然不同），而是按照哈希值来存的所以取数据也是按照哈希值取得。

元素的哈希值是通过元素的 hashcode 方法来获取的，**HashSet 首先判断两个元素的哈希值，如果哈希值一样，接着会比较 equals 方法，如果 equals 结果为 true，HashSet 就视为同一个元素。如果为 false 就不是同一个元素**。

哈希值相同 equals 为 false 的元素是怎么存储的呢？就是在同样的哈希值下顺延（可以认为哈希值相同的元素放在一个哈希桶中），也就是哈希值一样的存一列。

HashSet 通过 hashCode 值来确定元素在内存中的位置，**一个 hashCode 位置上可以存放多个元素**。

## Map
<key, values> pair
- key: Set<K> keySet();   -> 唯一
- values: Collection<V> values(); -> 允许重复

### HashMap（数组+链表+红黑树）
HashMap 根据键的 hashCode 值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。

HashMap 最多只允许一条记录的键为 null，允许多条记录的值为 null。

HashMap 非线程安全，即任一时刻可以有多个线程同时写 HashMap，可能会导致数据的不一致。

如果需要满足线程安全，可以用 Collections 的 synchronizedMap 方法时 HashMap 具有线程安全的能力，或者使用 ConcurrentHashMap。

SafeHashMapDemo.java
```java
public class SafeHashMapDemo {
    public static void main(String[] args) {
        Map hashMap = new HashMap<>();
        Map safeHashMap = Collections.synchronizedMap(hashmap);
        safeHashMap.put("aa", "1");
        safeHashMap.put("bb", "2");
        System.out.println(safeHashMap.get("bb"));
    }
}
```
synchronizedMap 和 HashTable 的实现几乎一样，多线程由于都是串行执行所以效率很低


HashMap: put 方法逻辑：
1. 如果 HashMap 未被初始化过，则初始化
2. 对 Key 求 Hash 值，然后再计算下标
3. 如果没有哈希碰撞，直接放入桶中
4. 如果碰撞了，以链表的方式链接到后面
5. 如果链表长度超过阈值，就把链表转成红黑树
6. 如果链表长度低于 6，就把红黑树转回链表
7. 如果节点已经存在就替换旧值
8. 如果桶满了（容量 16 * 加载因子 0.75），就需要 resize（扩容 2 倍后重排）

HashMap：如何有效减少碰撞？
- 扰动函数：促使元素位置分部均匀，减少碰撞几率（目的是让不同的对象返回不同的 hashCode）
- 使用 final 对象（不可变性使得能够缓存不同键的 hashCode，可以防止键值改变，将会提高获取对象的速度），并采用合适的 equals() 和 hashCode() 方法（使用String, Integer 这样的 wrap 类作为键有好处因为 String 是 final 的并且已经重写了 equals() 和 hashCode() 方法）

HashMap：扩容所带来的问题：
- 多线程环境下，调整大小会存在条件竞争，容易造成死锁
- rehashing 是一个比较耗时的过程

HashMap 知识点回顾：
- 成员变量：数据结构，树化阈值
- 构造函数：延迟创建
- put 和 get 的流程
- 哈希算法，扩容，性能

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

### HashMap、HashTable、ConcurrentHashMap 之间的区别
- HashMap 线程不安全，数组 + 链表 + 红黑树
- HashTable 线程安全，锁住整个对象，数组 + 链表
- ConcurrentHashMap 线程安全，CAS + 同步锁，数组 + 链表 + 红黑树
- HashMap 的 key、value 均可为 null，而其他的两个类不支持

HashMap（Java8 之前）：数组（长度默认 16）+链表，节点叫 Entry

hash(key.hashCode()) % len(16)  -> 实际上是通过位运算进行的，比取模运算效率更高

每个数组元素存储的是链表的头结点

最极端情况都分配到同一个 bucket，相当于变成链表查询，查找性能从 O(1) 恶化到 O(n)

为了解决这个问题，Java8 以后引入了红黑树（TREEIFY_THRESHOLD（默认是 8 ） 来控制是否将链表转换成红黑树），使得性能从 O(n) 提高到 O(logn)，节点叫 Node

UNTREEIFY_THRESHOLD = 6 -> 又重新从红黑树变回链表以保证性能

Node
```java
static class Node<K, V> implements Map.Entry<K, V> {
  final int hash;
  final K key;
  V value;
  Node<K, V> next;
}
```

HashMap.java
```java
public V put(K key, V value) {
  return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
  Node<K, V>[] tab; Node<K, V> p; int n, i;
  if ((tab == table) == null || (n == tab.length) == 0) {
    // resize 方法既具备初始化，还具备扩容的功能
    n = (tab = resize()).length;
  }
  // 哈希运算
  if ((p = tab[i - (n - 1) & hash]) == null) {
    tab[i] = newNode(hash, key, value, null);
  } else {
    Node<K, V> e; K k;
    // 已经存在该键值对
    if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
      // 直接替换数组里的元素
      e = p;
    }
    // 判断当前节点是否已经是树化后的节点
    else if (p instanceof TreeNode) {
      // 按照树的方式尝试存储键值对
      e = ((TreeNode<K, V>)p).putTreeVal(this, tab, hash, key, value);
    } else {
      // 按链表插入方式往链表中添加元素
      for (inr binCount = 0; ; ++binCount) {
        if ((e = p.next) == null) {
          p.next = newNode(hash, key, value, null);
          if (binCount >= TREEIFY_THRESHOLD - 1) {  // -1 for 1st
            treeifyBin(tab, hash);
            break;
          }
        }
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
          break;
        }
        p = e;
      }
    }
    if (e != null) {        // existing mapping for key
      V oldValue = e.value;
      if (!onlyIfAbsent || oldValue == null) {
        e.value = value;
      }
      afterNodeAccess(e);
      return oldValue;
    }
  }
}

public V get(Object key) {
  Node<K, V> e;
  return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K, V> getNode(int hash, Object key) {
  Node<K,V>[] tab; Node<K, V> first, e; int n; K k;
  if ((tab = table) != null && (n = table.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
    if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) { // always check first node
      return first;
    }
    if ((e = first.next) != null) {
      if (first instanceof TreeNode) {
        return ((TreeNode<K, V>)first).getTreeNode(hash, key, value);
      }
      do {
        if (e.hash == hash && (k = e.key) == key || (key != null && key.equals(k)))) {
          return e;
        }
      } while ((e = e.next) != null);
    }
  }
  return null;
}

static final int hash(Object key) {
  int h;
  return (k == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final Node<K, V>[] resize() {
  Node<K, V>[] oldTab = table;
  int oldCap = (oldTab == null) ? 0 : oldTab.length;
  int oldThr = threshold;
  int newCap, newThr = 0;
  if (oldCap > 0) {
    if (oldCap >= MAXIMUM_CAPACITY) {
      threshold = Integer.MAX_VALUE;
      return oldTab;
    }
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY) {
      // 用新的更大的数组存
      newThr = oldThr << 1;   // double threshold
    }
  } else if (oldThr > 0) {    // initial capacity was placed in  threshold
    newCap = oldThr;          
  } else {                    // zero initial threshlod signifies using 
    
  }
}
```

**如何优化 Hashtable**？
- 通过锁细粒度化，将整锁拆解成多个锁进行优化
    - 早期的 ConcurrentHashMap：通过分段锁 Segment 来实现（数组+链表）
    - 当前的 ConcurrentHashMap：CAS + synchronized 使锁更细化（数组+链表+红黑树）

### ConcurrentHashMap
ConcurrentHashMap.java
```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K, V>[] tab = table;;) {
        Node<K, V> f; int n, i, fh; K fk; V fv;
        if (tab == null || (n = tab.length) == 0) {
            tab = initTable();
        }
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K, V>(hash, key, value))) {
                break;                  // no lock when adding to empty bin
            }
        }
        else if ((fh = f.hash) == MOVED) {
            tab = helpTransfer(tab, f);
        }
        else if (onlyIfAbsent // check first node without acquring lock
                && fh == hash 
                && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                && (fb = f.val) != null) {
            return fv;
        }
        // 发生哈希碰撞
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 链表头结点
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K, V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash && 
                                ((ek = e.key) == key ||
                                (ek != null && key.equals(ek))) {
                                if (!onlyIfAbsent) {
                                    e.val = value;
                                    break;
                                }
                            }
                            Node<K, V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K, V>(hash, key, value);
                                break;
                            }
                        }
                    }
                    // 红黑树的节点
                    else if (f instanceof TreeBin) {
                        Node<K, V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K, V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent) {
                                p.val = value;
                            }
                        }
                    }
                    // 可能存在缓存
                    else if (f instanceof ReservationNode) {
                        throw new IllegalStateException("Recursive update");
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD) {
                    treeifyBin(tab, i);
                }
                if (oldVal != null) {
                    return oldVal;
                }
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

**ConcurrentHashMap：put 方法的逻辑**：
1. 判断 Node[] 数组是否初始化，没有则进行初始化操作
2. 通过 hash 定位数组的索引坐标，是否有 Node 节点，如果没有则使用 CAS 进行添加（链表的头节点），添加失败则进入下次循环
3. 检查到内部正在扩容，就帮助它一块扩容
4. 如果 f != null，则使用 synchronized 锁住 f 元素（链表/红黑二叉树的头元素）
    1. 如果是 Node（链表结构）则执行链表的添加操作
    2. 如果是 TreeNode（树形结构）则执行树添加操作
5. 判断链表长度已经达到临界值 8，当然这个 8 是默认值，可以自行调整，当节点数超过这个值就需要把链表转换为树结构。

**ConcurrentHashMap 总结**：比起 Segment，锁拆得更细
- 首先使用无锁操作 CAS 插入头节点，失败则循环重试
- 若头节点已存在，则尝试获取头节点的同步锁，再进行操作

别的需要注意的点：
- size()方法和 mappingCount()方法的异同，两者计算是否准确？
- 多线程环境下如何进行扩容？



































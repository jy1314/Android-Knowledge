# HashMap

## HashMap是什么？

hashmap是一种基于Key-Value的数据结构，相信大家应该都明白其原理，就不赘述了。那么JDK中的HashMap是怎么实现的呢？

## HashMap的实现
JDK中对HashMap的描述原文如下：

 * Hash table based **implementation of the {@code Map} interface**.  This
 * implementation provides all of the optional map operations, and permits
 * **{@code null} values** and the **{@code null} key**.  (The {@code HashMap}
 * class is roughly equivalent to {@code Hashtable}, except that it is
 * **unsynchronized** and permits nulls.)  This class makes **no guarantees as to**
 * **the order of the map**; in particular, it **does not guarantee that the order**
 * **will remain constant over time**.
 
 关键点如下：
 + 基于Map接口；
 + 允许Key和Value都允许为null；
 + 非同步；
 + 不保证顺序和插入时相同；
 + 不保证顺序时不变。
 
 HashMap是非同步的，想要线程同步的HashMap可以用HashTable或ConcurrentHashMap，具体请看[这里](https://github.com/jy1314/Android-Knowledge/blob/master/Java%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88%E7%B1%BB/ConcurrentHashMap.md)。
 
 ### HashMap和HashTable以及ConcurrentHashMap的区别：

**Hashtable**

数组+链表实现，key，value都不能为null，线程安全，实现方式是修改数据时锁住整个HashTable，效率低。
初始大小11，扩容*2+1。

**HashMap**

数组+链表实现，key，value都可以为null，线程不安全。
初始大小16，扩容*2。

**ConcurrentHashMap**

分段数组+链表实现，线程安全。用分离锁实现，允许多个修改操作并发。跨段修改时按顺序锁定所有段，再按顺序释放。







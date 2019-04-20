# HashMap

## HashMap是什么？

hashmap是一种基于Key-Value的数据结构，相信大家应该都明白其原理，就不赘述了。

## JDK中的HashMap
JDK中对HashMap的描述原文如下：

Hash table based **implementation of the {@code Map} interface**.This implementation provides all of the optional map operations, and permits **{@code null} values** and the **{@code null} key**.  (The {@code HashMap} class is roughly equivalent to {@code Hashtable}, except that it is **unsynchronized** and permits nulls.)  This class makes **no guarantees as to** **the order of the map**; in particular, it **does not guarantee that the order** **will remain constant over time**.
 
 关键点如下：
 + 基于Map接口；
 + 允许Key和Value都允许为null；
 + 非同步；
 + 不保证顺序和插入时相同；
 + 不保证顺序时不变。
 
 HashMap是非同步的，想要线程同步的HashMap可以用HashTable或ConcurrentHashMap，具体请看[这里](https://github.com/jy1314/Android-Knowledge/blob/master/Java%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88%E7%B1%BB/ConcurrentHashMap.md)。

### 实现
HashMap在底层是用数组+链表实现的。其中数组是HashMap的主体，而链表主要是为了解决哈希冲突而存在的。查找时，如果定位到的数组位置不含链表，时间复杂度为O(1)；如果定位到链表，那么需要遍历链表，存在即覆盖，不存在就添加。链表出现的越少，HashMap性能越高。

重要参数：

+ Size：储存的键值对对数；
+ loadFactor：负载因子，代表table最大的填充度，默认0.75，超过后会自动扩容。
+ capacity：容量，哈希表中桶的数量，默认初始为16，扩容后一定为2的n次幂。

关于扩容因子为什么要设为0.75，注释中的解释如下：

```
* Because TreeNodes are about twice the size of regular nodes, we
* use them only when bins contain enough nodes to warrant use
* (see TREEIFY_THRESHOLD). And when they become too small (due to
* removal or resizing) they are converted back to plain bins.  In
* usages with well-distributed user hashCodes, tree bins are
* rarely used.  Ideally, under random hashCodes, the frequency of
* nodes in bins follows a Poisson distribution
* (http://en.wikipedia.org/wiki/Poisson_distribution) with a
* parameter of about 0.5 on average for the default resizing
* threshold of 0.75, although with a large variance because of
* resizing granularity. Ignoring variance, the expected
* occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
* factorial(k)). The first values are:
*
* 0:    0.60653066
* 1:    0.30326533
* 2:    0.07581633
* 3:    0.01263606
* 4:    0.00157952
* 5:    0.00015795
* 6:    0.00001316
* 7:    0.00000094
* 8:    0.00000006
* more: less than 1 in ten million
```
研究表明，理想情况下，使用随机哈希码，节点出现的频率在hash桶中服从泊松分布，当桶中元素达到8个时，几乎已经不可能碰撞。即0.75作为扩容因子时，链表长度几乎不可能超过8个。

**为什么数组长度一定要2的n次幂呢？**

这就要从源码将起了

首先是put方法：

```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
可以看到Key传入方法后，用hash(key)这个方法取得hash值，那么我们来看看这个方法。

```
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
可以看到，先取key的hashCode，然后高16位不变，低16位和高16位异或，得到的值为最终的hash值。那么我们接着来看看putVal()这个方法。

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        ......省略其他代码
}
      
```
可以看到，参数传入后，首先判断是否为空，之后重点来了，n = tab.length，即n为数组长度。而i就是key在数组中的位置。

**i = (n - 1) & hash**。这个设计就非常巧妙了，用数组长度-1再和前面算出的hash值进行相与得到位置。由于n为2的k次幂，因此n-1的二进制肯定是全1。

举个例子：

N = 2^4 = 16 转换为二进制为10000

n - 1 = 15 转换为二进制为01111

因此，无论hash是多少，(n - 1) & hash都在0000～1111之间。这就保证了数组下标不会越界。

那么如果n-1不是全1会发生什么呢？

再举个例子：

N = 10 转换为二进制为01010

n - 1 = 9 转换为二进制为01001

hash = 1001 -> (n - 1) & hash = 1001;

hash = 1101 -> (n - 1) & hash = 1001;

显然有些结果出现的概率将会变大，而有些结果则不可能再出现（例如：1110），这显然是不符合hash算法的均匀分布原则的。
因此，数组长度一定要2的n次幂。这种情况下，只要输入的hashcode是均匀的，hash算法得到的结果就是均匀的。

### 为什么要重写hashCode()和equals()方法？

大家都知道，hashMap中，如果key为自定义类，那么就必须要重写hashCode()和equals()方法。那么这是为什么呢？

首先为什么要重写equals()方法呢？

还得看源码：


```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
		Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
 			Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
		......省略其他代码
}
```
可以看到put方法中，判断key是否相等，用的是equals方法。其实get方法中也是一样。那么默认的equals方法就不一定能满足我们的要求了。因此要重写equals方法。

那么又为什么要重写hashCode方法呢？

这就涉及到Object.hashCode()的通用约定了：

1. 每次应用执行期间，在对象做equals比较所用到的信息没有修改之前，hashcode方法的返回值应该是相同的。
2. 两个对象equals方法相同，那么hashCode值必须相同；
3. 两个对象equals方法不同，那么hashCode值不一定不同。

如果只重写equals方法，不重写hashCode方法，那么显然会违反第二条约定。

具体到HashMap中，有如下源码：


```
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
其中if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
这句明显可以看出，get的时候会先进行hash的比较，之后才会进行equals判断。上面源码分析过，hash的计算是基于hashCode()方法的。不重写的话，一般情况下hashCode()方法的返回值不同，那么就无法get到正确的key所对应的value。

如果只重写equals，不重写hashCode，那么存入key内容相同的对象时，会存入两个key内容相同的对象，而不是覆盖掉原有的对象。
取出的时候也取不出正确的对象。

因此必须要重写hashCode()和equals()方法。



---
 
### HashMap和HashTable以及ConcurrentHashMap的区别：

**Hashtable**

数组+链表实现，key，value都不能为null，线程安全，实现方式是修改数据时锁住整个HashTable，效率低。
初始大小11，扩容*2+1。

**HashMap**

数组+链表实现，key，value都可以为null，线程不安全。
初始大小16，扩容*2。

**ConcurrentHashMap**

分段数组+链表实现，线程安全。用分离锁实现，允许多个修改操作并发。跨段修改时按顺序锁定所有段，再按顺序释放。







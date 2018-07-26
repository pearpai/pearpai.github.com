---
layout:     post
title:      "Java 基础库源码学习Map（4）---- HashMap"
date:       UTC2017-11-15 13:37:00
author:     "Pearpai"
header-img: "img/head/spring-cloud-head.jpeg"
catalog: true
tags:
    - java Map 源码
---
## 概要
- HashMap 是基于哈希表的Map接口的非同步实现。此实现提供所有可选映射操作，并允许使用null值和null建。
- 此类映射不保证顺序，且其中的数据结构也是在变化着的

## 源码分析
### 初始化构造方法

- 初始化HashMap 目前有4中方式，总体分为两类：一种为初始化空的HashMap，一种为将原有的HashMap数据导入到新初始的HashMap。

- HashMap 有初始容量为 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4，即为16，最大容量为 static final int MAXIMUM_CAPACITY = 1 << 30，如果在初始化的过程中初始容量大于MAXIMUM_CAPACITY 这重新设置为MAXIMUM_CAPACITY 容量，Capacity设置的大小在内部会进行换算，将容量的大小设置为**2幂的大小**，即为当 2^n<initialCapacity<=2^n+1 时，将进行Capacity设置为2^n+1，此处使用的方法为tableSizeFor。

- 加载因素，与容量CAPACITY 结合使用，当存入的数据达到这 Capacity * loadFactor时，HashMap的结构将进行重构。

- 1.8的hashMap 存储数据结构较之前版本有了很多的变化，例如对hashCode的算法发生了变化，使得撞桶的概率降低，同时引入了红黑树的结构，当撞桶中的数据连接数量达到了8个，桶中的链表数据结构将进行一次重构，变成红黑树结构，使得在查询时间复杂度变为了log(n),当数据量大的时候，查询变的更快。

```java

/**
 * 初始化容量，以及加载因子来构造一个空的 HashMap
 * @param  initialCapacity 容量
 * @param  loadFactor      加载因子
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量 小于零 直接报错
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 初始化容量的值大于 MAXIMUM_CAPACITY时将以MAXIMUM_CAPACITY为初始值                                      
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 当加载因素小于0 或者 对loadFactor 判断是否为数值如果符合条件则报错
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // 设置加载因子                                       
    this.loadFactor = loadFactor;

    this.threshold = tableSizeFor(initialCapacity);
}

/**
 * 初始化容量，构造空的 HashMap 此时默认加载因子为0.75
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

/**
 * 构造一个默认的 HashMap
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

/**
 * 构造一个新的HashMap，同时将原有的指定的映射关系存入其中，加载因子使用默认值0.75
 * @param   m 原有的map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
/**
 * 返回2的幂的大小给定目标的容量。2^n<initialCapacity<=2^n+1 时，
 * 将进行Capacity设置 为2^n+1
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    // 无符号移位，并进行或运算
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
### 节点实例

- 节点实例 继承了Map.Entry<K,V> 。


```Java

// key 值hash 算法
static final int hash(Object key) {
    int h;
    // null hash 为0
    // key 其他 进行 hashCode 异或运算 进行无符号移位16位
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

static class Node<K,V> implements Map.Entry<K,V> {
    // 实例对象的hash值  
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    // 构造函数 hash值， k-v 下一个节点node 实例
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    // 返回 k-v hashcode  通过异或运算
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    // value 值进行更新
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    // equals 覆写
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 数据存入 put(K key, V value)

```Java

public V put(K key, V value) {
    // 存入前 先对key进行hash 运算 得到hash值
    return putVal(hash(key), key, value, false, true);
}
/**
 * 实现 Map.put 相关方法
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent 如果 true则不改变已经存在的值
 * @param evict 如果 false则表处于创建模式.
 * @return 返回之前的值, 如果不存在则返回null
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // node 表
    Node<K,V>[] tab;
    Node<K,V> p;
    // n 为 table 的容量长度
    int n, i;
    // 如果 table == null 或者 length=0 即 初始话 或者重构 table
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 此时进行hash与运算 如果计算出的 tab[i] == null即此处位置没有存入过数据
    // 此时直接存入即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // 即此时tab[i] != null
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // 这里的功能其实就是替换 value
            // 将e 指向 p
            e = p;
        else if (p instanceof TreeNode)
            // 红黑树处理 putTreeVal
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 这里是链表处理方式
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 当链表的长度 大于等于8 将链表进行红黑树转换
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // todo 这里说明是 链表中有相同的key 放在后面 是因为 if 第一个条件判断 可以进行处理，这里处理的是 中间的
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 替换原先的值
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 如果onlyIfAbsent=true 将不替换原有的值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // linkhashmap 调用
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    // linkhashmap 调用
    afterNodeInsertion(evict);
    return null;
}
```

### resize 重置table表

```Java

/**
* 初始化 或者 对 table容量翻倍。如果为table为null则分配一个初始容量。
* 另外，因为我们用2幂法进行扩容，所有的元素必须呆在相同的索引中，或移动到新的表中
* @return the table
*/
final Node<K,V>[] resize() {
   // 将oldTab指向当前的table表
   Node<K,V>[] oldTab = table;
   // 设置oldCap 即resize前原有的容量 空则为0， 如果不为空则获取表的长度
   int oldCap = (oldTab == null) ? 0 : oldTab.length;
   // 即原有的 容量 此处的容量通过tableSizeFor处理为2的n次方 。
   int oldThr = threshold;
   // 初始化 新容量为0 下一次扩容的大小 为0
   int newCap, newThr = 0;
   // oldCap > 0 表示原有容量 大于0
   if (oldCap > 0) {
       if (oldCap >= MAXIMUM_CAPACITY) {
           // 当 oldCap 容量 大于 可设置的最大容量，将修改容量为Integer.MAX_VALUE
           // 同时返回原有的table，不再进行重构
           threshold = Integer.MAX_VALUE;
           return oldTab;
       }
       // 向左移位操作即乘以2 同时可用容量也乘以2
       else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
           newThr = oldThr << 1; // double threshold
   }
   else if (oldThr > 0) // initial capacity was placed in threshold
       //设置新容量 为 原有的 下一次将要设置的容量
       // 此处为HashMap(int initialCapacity)/HashMap(int initialCapacity, float loadFactor) 初始构造方式 进行的容量设置
       newCap = oldThr;
   else {               // zero initial threshold signifies using defaults
        //  HashMap() 初始化
       // 设置 初始容量 为 默认值
       newCap = DEFAULT_INITIAL_CAPACITY;
       // 设置 可存入容量 为 容量 * 加载因素 默认是 16 * 0.75
       newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
   }
   if (newThr == 0) {
      // newThr 如果为0 即为 上面oldThr > 0 处 未设置
       float ft = (float)newCap * loadFactor;
       // 进行 可加载容量 的运算
       newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                 (int)ft : Integer.MAX_VALUE);
   }
   // 重新设置可加载容量
   threshold = newThr;
   // 新生成节点表
   @SuppressWarnings({"rawtypes","unchecked"})
   Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
   // 将map中的table 指向 newTab
   table = newTab;
   // 如果为 null 则直接返回 空表，如果不为null 将进行重构
   if (oldTab != null) {
        // 原有map 已经存入了 一定数量的值
        // 遍历 原有的表，原有的表存入的的 k-v 值为oldCap
       for (int j = 0; j < oldCap; ++j) {
           Node<K,V> e;
           if ((e = oldTab[j]) != null) {
               oldTab[j] = null;
               if (e.next == null)
                   // 对key的hashcode 与 （容量 -1）  进行与运算 或者值即为需要将k-v 存入的位置 这个算法 过程 非常重要
                   newTab[e.hash & (newCap - 1)] = e;
               else if (e instanceof TreeNode)
                  // 这边是treenode 存储方式 即为红黑树处理 提升了查询性能
                   ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
               else { // preserve order
                  // 这里是链表的操作 当链表的长度达到了8，将进行红黑树重构
                   Node<K,V> loHead = null, loTail = null;
                   Node<K,V> hiHead = null, hiTail = null;
                   Node<K,V> next;
                   do {
                       next = e.next;
                       // (e.hash & oldCap) == 0 表示在二进制与运算中 存入低位
                       // 其他存入高位
                       // 如 17 & 16 、17 & 15 如果 hash 为17 在原有的cap=16 时存入的位置未 17 & 15 = 1，而重置 17 & 31 = 17 即 17 & 16 = 1，而后 1+ 16 = 17
                       // 这种计算 即为原有hash 二进制运算中，在resize 时候介入运算的增加了一位
                       if ((e.hash & oldCap) == 0) {
                          // 即当 newTab[0] == null，添加第一个hash值为0的对象
                           if (loTail == null)
                              // 此时添加头对象为e
                               loHead = e;
                           else
                              // 如果loTail != null 此时将loTail.next指向e
                               loTail.next = e;
                           loTail = e;
                       }
                       else {
                          // 此处hash 不为0的 存入hi 即高位
                           if (hiTail == null)
                               hiHead = e;
                           else
                               hiTail.next = e;
                           hiTail = e;
                       }
                   } while ((e = next) != null);
                   // 因为while 是遍历的 链表 即 (e.hash & oldCap) == 0)每次运行的值都是相同的
                   // hash为 0 时(e.hash & oldCap) == 0 即低半区即 n+1 hash位 为0 不为1
                   if (loTail != null) {
                       loTail.next = null;
                       newTab[j] = loHead;
                   }
                   // 存入高位 n+1 hash位 为1 二进制
                   if (hiTail != null) {
                       hiTail.next = null;
                       newTab[j + oldCap] = hiHead;
                   }
               }
           }
       }
   }
   return newTab;
}
```

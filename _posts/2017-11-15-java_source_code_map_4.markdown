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
```Java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

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

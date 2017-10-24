---
layout:     post
title:      "Java 基础库源码学习List（5）----  AbstractCollection"
date:       UTC2017-10-19 10:15:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - java List 源码
---
```java
package java.util;
/**
 * 这个类通过了集合实现的骨架，以减少接口实现所需要的实现
 * 实现一个无法改变的集合,程序员只需要扩展这个类,并提供实现迭代器和大小的方法。
 * (迭代器   返回的迭代器方法必须实现hasNext。)实现一个可修改的集合,程序员必须另外覆盖这个类的添加方法(否则抛
 * 出UnsupportedOperationException)方式,和迭代器返回的迭代器方法必须另外实现其去除方法。
 * 程序员通常提供一个空白(无参数)和构造函数集合,按照推荐的接口规范。每个非抽象的方法在这类的文档详细描述了它的实现。
 * 这些方法可能会覆盖如果集合实现承认一个更高效的实现。（直接有道翻译了啊）
 */

public abstract class AbstractCollection<E> implements Collection<E> {
    /**
     * 唯一的构造函数。(用于调用子类的构造函数,通常是隐性的。)
     */
    protected AbstractCollection() {
    }
}
```
### iterator()
```java
    // Query Operations

    /**
     * 返回一个迭代器
     *
     */
    public abstract Iterator<E> iterator();
```
### size()
```java
    /**
    * 返回集合的中元素的数量
    */
    public abstract int size();
```
### isEmpty()
```java
    /**
     * 集合是否为空
     */
    public boolean isEmpty() {
        return size() == 0;
    }
```
### contains(Object o)
```java
    /**
     * 集合是否包含 o元素
     * 获取集合迭代器 对迭代器进行迭代 进行迭代 对元素进行比较
     */
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }
```
### toArray()
```java
    /**
     * 返回集合的数组
     *
     */
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        // 获取集合中元素的数量，new一个大小为size 的Object 数组
        Object[] r = new Object[size()];
        // 获取集合迭代器
        Iterator<E> it = iterator();
        // 数组循环 同时循环迭代器
        for (int i = 0; i < r.length; i++) {
            // 当迭代器中没有数据时，将更新 数组的长度为i 同时将r 复制出新的数组
            if (! it.hasNext()) // fewer elements than expected
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        // finishToArray 重新分配迭代器 使得数组变变大
        // 在多线程中可能存在这一需求
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```
### toArray(T[] a)
```java
    /**
     * 返回集合指定的数组 如果没数组大小有出入则 重新生成新的数组
     * @throws ArrayStoreException  {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        // 估算集合的长度
        int size = size();
        // 判断使用声明什么数组 如果指定数组的长度小于集合元素的格式 将新生成一个数组
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        // 获取集合迭代器
        Iterator<E> it = iterator();
        // 复制迭代器元素到数组
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    // 迭代过程中集合数据增加
                    // 复制数据到新的数组中 底层调用的 System.arraycopy native方法
                    return Arrays.copyOf(r, i);
                } else {
                    // 重新复制数据到
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```
### finishToArray(T[] r, Iterator<?> it)
```java
    /**
     * 数组的最大元素数量，有些虚拟机数组前有个头，所以减去了8，集合最大元素 Integer.MAX_VALUE - 8
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 重新分配数组，返回比预期更多的元素
     */
    @SuppressWarnings("unchecked")
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        // 获取原先存储的数组的长度
        int i = r.length;
        // 迭代器迭代
        while (it.hasNext()) {
            // 每次获取新数组的长度
            int cap = r.length;
            // 当数组i == 新数组的长度
            if (i == cap) {
                // 增加数组的大小 增加为原有数组的一半 + 1
                int newCap = cap + (cap >> 1) + 1;
                // overflow-conscious code 如果数值过大 抛错
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                // 复制数组
                r = Arrays.copyOf(r, newCap);
            }
            // 数组赋值
            r[i++] = (T)it.next();
        }
        // trim if overallocated
        // 如果 长度不变 返回原有的，如果发生变化 仅返回前i个元素
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }


    private static int hugeCapacity(int minCapacity) {
        // 数值越界 变为赋值
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError
                ("Required array size too large");
        // 如果 minCapacity > MAX_ARRAY_SIZE 且为正数 这 再扩展到  Integer.MAX_VALUE
        // 最多 + 8
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
###  add(E e)
```java
    // Modification Operations

    /**
     * 添加 元素
     */
    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }
```
### remove(Object o)
```java
    /**
     * 删除 集合中的元素 只删除其中的第一个
     */
    public boolean remove(Object o) {
        // 获取迭代器
        Iterator<E> it = iterator();
        // 判断 元素 基本类型 走不同的判断删除方式 == 是指针相同
        // equals 值相同
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }

```
### containsAll(Collection<?> c)
```java
    // Bulk Operations

    /**
     * c 集合中的元素 是否被集合全部包含
     */
    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    /**
     * 将集合c中的元素添加到集合中
     */
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```
### removeAll(Collection<?> c)
```java
    /**
     * 删除集合中包含c集合元素
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }
```
### retainAll(Collection<?> c)
```java
    /**
     * 集合中保留在c中存在的元素
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }
```
### clear()
```java
    /**
     * 清空集合
     */
    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }

```
### toString()
```java
    //  String conversion

    /**
     * 不解释
     */
    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        // String StringBuilder（非线程保护/快） StringBuffer（线程保护/慢） 区别
        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

}
```

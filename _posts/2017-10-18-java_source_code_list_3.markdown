---
layout:     post
title:      "Java 基础库源码学习List（3）---- List interface"
date:       UTC2017-10-18 16:56:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - java List 源码
---
## 接口方法功能简要说明
具体的实现将在具体的类中进行相关分析

```java
package java.util;
import java.util.function.UnaryOperator;

/**
 * 一个有序集合 精确的控制列表中没个元素的插入。用户可以通过数值索引访问查询列表中的元素，不同于set，list通常运行重复元素，即使e1.equals(e2),同时可以存储null
列表接口提供了两个方法来搜索指定的对象。从性能的角度来看,这些方法应该小心使用。在很多实现中他们将执行昂贵的线性搜索。
列表接口提供了两种方法来有效地插入和删除多个元素在列表中的任意位置。
注意:虽然允许列表包含自己为元素,极其谨慎的建议:equals和hashCode方法不再是定义良好的在这样的一个列表。
*/
public interface List<E> extends Collection<E> {
}
```
### size()
```java
    // Query Operations

    /**
	 * 返回列表的元素数量，如果数量大于Integer.MAX_VALUE 将返回 Integer.MAX_VALUE
     * Returns the number of elements in this list.  If this list contains
     */
    int size();
```
### isEmpty()
```java
    /**
     * 返回列表是否含有元素
     */
    boolean isEmpty();
```
### contains(Object o)
```java
    /**
     * 列表中是否包含此元素
     * (o==null ? e==null : o.equals(e))
     */
    boolean contains(Object o);
```
### iterator()
```java
    /**
     * 返回一个跌打器
     */
    Iterator<E> iterator();
```
### toArray()
```java
    /**
     * 返回所有元素的 数组
     */
    Object[] toArray();
```
### toArray(T[] a)
```java
    /**
     * 返回所有元素到指定数组
     * 如果类型 不匹配 将报错 ArrayStoreException
     * array a 如果为空 报错 NullPointerException
     */
    <T> T[] toArray(T[] a);
```
### add(E e)
```java
    // Modification Operations

    /**
     * 添加指定的元素 到列表中
     */
    boolean add(E e);
```
### remove(Object o)
```java
    /**
     * 删除列表中对应的 第一个元素
     */
    boolean remove(Object o);

```
### containsAll(Collection<?> c)
```java
    // Bulk Modification Operations

    /**
     * 返回是否包含所有c 中元素
     */
    boolean containsAll(Collection<?> c);
```
### addAll(Collection<? extends E> c)
```java
    /**
     * 在列表结尾添加所有c中元素
     */
    boolean addAll(Collection<? extends E> c);
```
### addAll(int index, Collection<? extends E>
```java
    /**
     * 在指定的索引 下 添加所有c中元素
     */
    boolean addAll(int index, Collection<? extends E> c);
```
### removeAll(Collection<?> c)
```java
    /**
     * 删除列表中所包含的c元素
     */
    boolean removeAll(Collection<?> c);
```
### retainAll(Collection<?> c)
```java
    /**
     * 保留列表中指定的c中的元素
     */
    boolean retainAll(Collection<?> c);
```
### replaceAll(UnaryOperator<E> operator)
```java
    /**
     * 对指定的参数进行替换
     */
    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }
```
### sort(Comparator<? super E> c)
```java
    /**
     * 对集合中 指定的 元素进行计算
     */
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
```
### clear()
```java
    /**
     * 清除list 中的 信息
     */
    void clear();

```

### equals(Object o)
```java
    // Comparison and hashing
    /**
     * 判断集合是相等
     */
    boolean equals(Object o);
```
### hashCode()
```java
    /**
     * 返回集合的hash值，hash code 是对集合的的计算

     int hashCode = 1;
     for (E e : list)
         hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
     */
    int hashCode();
```
### get(int index)
```java

    // Positional Access Operations

    /**
     * 返回集合中指定位置的元素
     */
    E get(int index);
```
### set(int index, E element)
```java
    /**
     * 替换集合中指定位置的的元素
     */
    E set(int index, E element);
```
### add(int index, E element)
```java
    /**
     * 在集合中指定文章添加元素
     */
    void add(int index, E element);
```
### remove(int index)
```java
    /**
     * 删除指定位置的元素
     */
    E remove(int index);

```
### indexOf(Object o)
```java
    // Search Operations

    /**
	 * 返回集合中包含o元素的第一个位置，如果没有返回-1
     */
    int indexOf(Object o);
```
### lastIndexOf(Object o)
```java
    /**
     * 返回集合中包含o元素的 最后一个的位置 如果没有返回-1
     * (o==null ? get(i)==null : o.equals(get(i)))
     */
    int lastIndexOf(Object o);
```
### listIterator()
```java

    // List Iterators

    /**
     * 返回集合的迭代器
     */
    ListIterator<E> listIterator();
```
### listIterator(int index)
```java
    /**
     * 返回集合一个迭代器，以index 位置为初始位置
     */
    ListIterator<E> listIterator(int index);
```
### subList(int fromIndex, int toIndex)
```java
    // View

    /**
     * 返回一个集合以fromIndex（包含）为初始位置，终止位置为toIndex（不包含）
     */
    List<E> subList(int fromIndex, int toIndex);
```
### spliterator()
```java
    /**
     * 分割处理，用于流并行处理等
     */
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }
}

```

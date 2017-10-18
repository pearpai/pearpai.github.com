---
layout:     post
title:      "Java 基础库源码学习List（2）---- Iterable"
date:       UTC2017-10-18 16:12:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - List
    - 源码
    - java
---
## 源码阅读
```java
package java.lang;

import java.util.Iterator;
import java.util.Objects;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.function.Consumer;

/**
  * 实现这个接口是可以实现for-each 循环的声明
  * 具体的看foreach（后续完善）
 */
public interface Iterable<T> {
    /**
     * 返回一个迭代器里面的元素类型为 T
     */
    Iterator<T> iterator();

    /**
     * 默认方法
     * 1.8 forEach 流处理实现
     * Objects.requireNonNull(action); 判断是否为null 如果为null
     * 向调用者抛错
     */
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
     /**
      * 分割迭代器
      * java 1.8 处理中的并行分割执行
     */
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}


```
## demo
spliterator 功能强大 后续单独说明。
```java
public static void main(String[] args) {

        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        list.add("3");

        Iterator<String> iterator = list.iterator();

        // iterator 原始处理
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }

        // 语法糖
        for (String aList : list) {
            System.out.println(aList);
        }

        // 流处理
        list.forEach(System.out::println);

    }

```

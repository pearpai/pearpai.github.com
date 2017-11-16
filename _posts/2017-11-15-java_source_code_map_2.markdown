---
layout:     post
title:      "Java 基础库源码学习Map（2）---- Interface Map"
date:       UTC2017-11-15 10:09:00
author:     "Pearpai"
header-img: "img/head/spring-cloud-head.jpeg"
catalog: true
tags:
    - java Map 源码
---

- Map 提供了一个更通用的元素存储方法。Map 集合类用于存储元素对（称作“键”和“值”），其中每个键映射到一个值。从概念上而言，您可以将 List 看作是具有数值键的 Map。而实际上，除了 List 和 Map 都在定义 java.util 中外，两者并没有直接的联系。

## 接口方法简要说明

| 序号      |    方法描述 |
| :-------- | :--------|
| 1  | **int size();** <br> 返回map中key-value映射的数量 最大数量为<tt>Integer.MAX_VALUE</tt>|  
| 2 | **boolean isEmpty();** <br>如果map中不存在key-value映射则返回<tt>true</tt>
| 3 | **boolean containsKey(Object key);**<br> 如果map中包含一个与key相等的，将返回true
| 4  |**boolean containsValue(Object value);**<br> 如果map中的key 所映射的value中包含指定的值，则返回true |
| 5  |**V get(Object key);**<br>返回key所指定的映射的值如果没有则返回null|
| 6  |**V put(K key, V value);**<br>将指定的值与此映射中的指定键关联|
| 7  |**V remove(Object key);**<br>如果存在一个键的映射关系，则将其从此映射中移除|
| 8  |**void putAll(Map<? extends K, ? extends V> m);**<br>从指定映射中|
| 9  |**void clear();**<br>从映射中移除所有映射关系|
| 10 |**Set&lt;K> keySet();**<br>返回此映射中的包含的键的Set视图|
| 11 |**Collection&lt;V> values();**<br>返回映射中包含的值的Collection视图|
| 12  |**Set<Map.Entry<K, V> entrySet();**<br>返回一个Set视图包含所有的映射|
| 13  |**boolean equals(Object o);**<br>Object类方法|
| 15  |**int hashCode();**<br>Object类方法|
| 16  |**default V getOrDefault(Object key, V defaultValue)**<br>返回key的映射值，如果没有返回defaultValue|
| 17  |**default void forEach(BiConsumer<? super K, ? super V> action)**<br>1.8版本 进行流处理|
| 18  |**default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function)**<br> 对所有映射关系中的映射值进行替换|
| 19  |**default V putIfAbsent(K key, V value)**<br>如果不存在关于key的映射 将存入key-value，如果存在key映射将不存入|
| 20  |**default boolean remove(Object key, Object value)**<br>映射表中如果存在key映射且映射值与value相同则进行删除操作并返回true 否则返回false|
| 21  |**default boolean replace(K key, V oldValue, V newValue)**<br>映射表中存在key的映射且映射值与oldVale相同则对映射值进行替换|
| 22  |**default V replace(K key, V value)**<br>如果存在key映射，将映射替换值value|
| 23  |**default V computeIfAbsent(K key, Function&lt;? super K, ? extends V> mappingFunction)**<br>实现本地缓存功能，如果有key映射读取映射值，如果没有则进行Function计算得出结果存入map并返回计算值|
| 24  |**default V computeIfPresent(K key,BiFunction<? super K, ? super V, ? extends V> remappingFunction)**<br>对本地缓存进行更新，当映射中存在值且不为null，执行remappingFunction 方法，保存新值且返回新值|
| 25  |**default V compute(K key,BiFunction<? super K, ? super V, ? extends V> remappingFunction)**<br>执行remappingFunction方法如果值不等于null则存入新值，如果为null则从映射关系中删除指定映射|
| 26  |**default V merge(K key, V value,BiFunction<? super V, ? super V, ? extends V> remappingFunction)**<br>进行合并处理，如果key原有映射为null则默认用新value，如果oldvalue不为null这调用remappingFunction执行并返回新值，如果新值为null则将此映射移除，如果不为null则进行重新存入并返回新值|

## 部分default 方法分析。
- Map接口中的大部分的default方法会被覆写，因为继承类中基本是对适用场景的具体描述，因此会对具体的场景的运行进行速度、算法上的优化，如HashMap、ConcurrentHashMap实现方式都是有区别。这等到后续的源码中进行分析。

### getOrDefault
```java
default V getOrDefault(Object key, V defaultValue) {
      V v;
      // 如果存在key的相关映射且不为null 或者 存在key映射此时不管映射值为多少
      // 即 存在映射则用映射值
      return (((v = get(key)) != null) || containsKey(key))
          ? v
          : defaultValue;
  }
```
### forEach
```java
default void forEach(BiConsumer<? super K, ? super V> action) {
     // 验证非null
     Objects.requireNonNull(action);
     // 获取到映射表的set视图，并对视图进行遍历
     for (Map.Entry<K, V> entry : entrySet()) {
         K k;
         V v;
         try {
            //获取映射关系的k-v
             k = entry.getKey();
             v = entry.getValue();
         } catch(IllegalStateException ise) {
             // this usually means the entry is no longer in the map.
             throw new ConcurrentModificationException(ise);
         }
         // 执行action 方法
         action.accept(k, v);
     }
 }
```
### replaceAll
```java
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    // 验证非null
    Objects.requireNonNull(function);
    // 获取到映射表的set视图，并对视图进行遍历
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }

        // ise thrown from function is not a cme.
        v = function.apply(k, v);

        try {
            entry.setValue(v);
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
    }
}
```
### 写感
- 到这里 map的源码中的default基本比较简单容易理解就不全部进行注释了。
- 1.8 中提供了default 功能确实给后面的接口继承中提供了 方法的构建思路，在后续自己的项目开发中不妨使用这一的方式。
- 1.8 提供了map的本地缓存处理方式，compute操作是一个很不错的功能，有兴趣的同学可以尝试一下。

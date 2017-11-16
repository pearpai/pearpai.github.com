---
layout:     post
title:      "Java 基础库源码学习Map（3）---- AbstractMap"
date:       UTC2017-11-15 10:50:00
author:     "Pearpai"
header-img: "img/head/spring-cloud-head.jpeg"
catalog: true
tags:
    - java Map 源码
---
## 概要
- AbstractMap 和 AbstractCollection 接口，AbstractList 接口 作用相似， AbstractMap 是一个基础实现类，实现了 Map 的主要方法，默认不支持修改。

- 此处我们其一些基础方法进行注释说明其中的基础功能。太简单的功能将不进行说明。

## 基础方法

### containsValue
- 判断map 映射表中是否存在值与value相等，如果相等则返回true 反之则返回false。

```java
public boolean containsValue(Object value) {
    // 获取映射表的Set视图，并转换为迭代器
    Iterator<Entry<K,V>> i = entrySet().iterator();
    // 如果值为value==null 判断使用 == 即地址相同
    // 如果是其他则为equals 为值相同 不问地址是否相同
    // 如果有相同的 则返回true
    if (value==null) {
        // 迭代器迭代
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getValue()==null)
                return true;
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (value.equals(e.getValue()))
                return true;
        }
    }
    return false;
}
```
### containsKey
- 与containsValue 处理方式一样 与containsValue 之于 value，containsKey 之于key
```java
public boolean containsKey(Object key) {
    // 获取映射表的Set视图，并转换为迭代器
    Iterator<Map.Entry<K,V>> i = entrySet().iterator();
    if (key==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                return true;
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                return true;
        }
    }
    return false;
}
```
### get
- 通过key 获取 key 的映射值，如果没有值 则返回null
- 查询使用的是迭代查询行数，在后续的继承类中有的会进行相关的优化如果hashmap 理由了哈希算法后进行了 连表 或者红黑树的 数据结构，查询速度将大大提升。
- 同样此处对key null 与not null 进行分类处理
```java
public V get(Object key) {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (key==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                return e.getValue();
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                return e.getValue();
        }
    }
    return null;
}
```

### remove
- 在映射表中删除key对应的相关映射，同时返回key的映射值，如果没有指定映射则返回null

```java
public V remove(Object key) {
    // 获取映射表的Set视图，并转换为迭代器
    Iterator<Entry<K,V>> i = entrySet().iterator();
    // 设置 当前映射实例为null
    Entry<K,V> correctEntry = null;
    // 对key 的null 进行特殊处理
    // 对key的指定映射进行定位
    if (key==null) {
        // 当在迭代的过程中 correctEntry 被赋值
        // 当correctEntry 不为 null 或者 i.hasNext== false 则跳出循环
        while (correctEntry==null && i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                // key 值相等则向 correctEntry 赋值
                correctEntry = e;
        }
    } else {
        // 功能如上
        while (correctEntry==null && i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                correctEntry = e;
        }
    }

    V oldValue = null;
    // 如果存在映射关系
    if (correctEntry !=null) {
        // 获取原有的值 并返回
        oldValue = correctEntry.getValue();
        // 在迭代器中将i出的 原有映射移除
        i.remove();
    }
    return oldValue;
}
```

### keySet
- 返回key 的Set视图
- 此处的Set 是通过继承抽象类AbstractSet 的实现，并实现了迭代器的动能。
```java
public Set<K> keySet() {
    if (keySet == null) {
        keySet = new AbstractSet<K>() {
            public Iterator<K> iterator() {
                return new Iterator<K>() {
                    private Iterator<Entry<K,V>> i = entrySet().iterator();

                    public boolean hasNext() {
                        return i.hasNext();
                    }

                    public K next() {
                        return i.next().getKey();
                    }

                    public void remove() {
                        i.remove();
                    }
                };
            }

            public int size() {
                return AbstractMap.this.size();
            }

            public boolean isEmpty() {
                return AbstractMap.this.isEmpty();
            }

            public void clear() {
                AbstractMap.this.clear();
            }

            public boolean contains(Object k) {
                return AbstractMap.this.containsKey(k);
            }
        };
    }
    return keySet;
}
```

### values
- 返回key 的Collection视图
- 此处的Collection 是通过继承抽象类AbstractCollection 的实现，并实现了迭代器的动能。
```java
public Collection<V> values() {
    if (values == null) {
        values = new AbstractCollection<V>() {
            public Iterator<V> iterator() {
                return new Iterator<V>() {
                    private Iterator<Entry<K,V>> i = entrySet().iterator();

                    public boolean hasNext() {
                        return i.hasNext();
                    }

                    public V next() {
                        return i.next().getValue();
                    }

                    public void remove() {
                        i.remove();
                    }
                };
            }

            public int size() {
                return AbstractMap.this.size();
            }

            public boolean isEmpty() {
                return AbstractMap.this.isEmpty();
            }

            public void clear() {
                AbstractMap.this.clear();
            }

            public boolean contains(Object v) {
                return AbstractMap.this.containsValue(v);
            }
        };
    }
    return values;
}
```

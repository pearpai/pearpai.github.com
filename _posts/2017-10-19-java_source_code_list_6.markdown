---
layout:     post
title:      "Java 基础库源码学习List（6）----  AbstractList"
date:       UTC2017-10-20 11:20:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - List
    - 源码
    - java
---
```java

package java.util;


public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    /**
     * 构造方法
     */
    protected AbstractList() {
    }

    /**
     * 向集合添加元素
     */
    public boolean add(E e) {
        add(size(), e);
        return true;
    }

    /**
     * 获取集合元素
     */
    abstract public E get(int index);

    /**
     * 在集合的索引位置添加元素
     */
    public E set(int index, E element) {
        throw new UnsupportedOperationException();
    }

    /**
     * 集合添加元素
     */
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }

    /**
     * 删除集合索引位置的元素
     */
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }


    // Search Operations

    /**
     * 获取元素在集合中的位置
     */
    public int indexOf(Object o) {
        // 获取迭代器
        ListIterator<E> it = listIterator();
        // 如果查询的位置 元素为null 与 对象 判断条件不同
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    // 返回迭代器中的索引位置
                    return it.previousIndex();
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return it.previousIndex();
        }
        // 如果没有返回-1
        return -1;
    }

    /**
     * 返回o元素在集合中最后一次出现的位置
     */
    public int lastIndexOf(Object o) {
        // 返回迭代器 最后的位置
        ListIterator<E> it = listIterator(size());
        // 倒序迭代器
        if (o==null) {
            while (it.hasPrevious())
                if (it.previous()==null)
                    return it.nextIndex();
        } else {
            while (it.hasPrevious())
                if (o.equals(it.previous()))
                    return it.nextIndex();
        }
        return -1;
    }


    // Bulk Operations

    /**
     * 清空集合
     */
    public void clear() {
        removeRange(0, size());
    }

    /**
     * 在索引位置 将 集合c中的元素添加到 集合中
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);
            modified = true;
        }
        return modified;
    }


    // Iterators

    /**
     * 返回一个迭代器
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * 返回一个起始位置未0的迭代器
     */
    public ListIterator<E> listIterator() {
        return listIterator(0);
    }

    /**
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public ListIterator<E> listIterator(final int index) {
        rangeCheckForAdd(index);

        return new ListItr(index);
    }

    private class Itr implements Iterator<E> {
        /**
         * 元素索引位置 在 next 方法中添加 即当前可以获取元素的位置
         */
        int cursor = 0;

        /**
         * 上一次获取元素的位置 通过 remove 方法将lastRet重置为-1
         */
        int lastRet = -1;

        /**
         * 迭代器需要确信 modCount值 需要List含有的
         */
        int expectedModCount = modCount;

        /**
         * 是否还有值
        */
        public boolean hasNext() {
            return cursor != size();
        }

        /**
         * 返回迭代器当前指定索引 cur
           sor 处 的值
        */
        public E next() {
            // 判断在迭代的过程中 集合数据是否发生了变化
            // 如果变化将会抛错
            checkForComodification();
            try {
                int i = cursor;
                // 获取索引位置元素
                E next = get(i);
                // 更新 lastRet 即 最后一次获取元素的索引位置
                lastRet = i;
                // 更新 cursor 即 下一次返回元素的位置
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        // 迭代器中删除列表集合元素
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                // 通过迭代器 调用 list 方法 删除 lastRet 位置元素
                AbstractList.this.remove(lastRet);
                // 如果位置 小于 cursor（下一读取位置）则cursor-- 集合读取位置向前进一位
                if (lastRet < cursor)
                    cursor--;
                // 重置上一次 读取的位置 因为删除了 获取不到了 所有重置为-1
                lastRet = -1;
                // 同时 修改修改期望的count  
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * list 迭代器
    */
    private class ListItr extends Itr implements ListIterator<E> {
        // 构造方法 设置迭代器 迭代初始位置
        ListItr(int index) {
            cursor = index;
        }
        // 是否含有前一个元素 即倒序过程 是否含有元素判断
        // 当不等于 cursor ！= 0 即有前一个元素 cursor >= 0
        public boolean hasPrevious() {
            return cursor != 0;
        }

        // 获取 集合中前一个元素
        public E previous() {
            checkForComodification();
            try {
                int i = cursor - 1;
                E previous = get(i);
                lastRet = cursor = i;
                return previous;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        // 返回 下一个 元素的 光标位置
        public int nextIndex() {
            return cursor;
        }

        // 返回前一个值的光标位置
        public int previousIndex() {
            return cursor-1;
        }

        // set 在 光标lastRet位置的 设值为 e
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                // 在最后的位置设置值 即 调用previous()获取值的地方 替换原有的值
                AbstractList.this.set(lastRet, e);
                // 同时更新 expectedModCount
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        // 迭代器添加 元素 在 集合的尾部
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                AbstractList.this.add(i, e);
                // 因为元添加了 所有 上一次的位置 即为当前add位置 不是之前的最后的位置
                // 所以需要重置
                lastRet = -1;
                // 在最后添加 需要将光标往后移动一位
                cursor = i + 1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * 截取 集合
     */
    public List<E> subList(int fromIndex, int toIndex) {
        // RandomAccessSubList 则截取出 只是 修改了 如下
        // l = list;
        // offset = fromIndex;
        // size = toIndex - fromIndex;
        // sublist 截取的 还是原来的集合 如果集合中的元素发生了改变对应的 截取的 集合也会发生相应变化
        this.modCount = l.modCount;
        return (this instanceof RandomAccess ?
                new RandomAccessSubList<>(this, fromIndex, toIndex) :
                new SubList<>(this, fromIndex, toIndex));
    }

    // Comparison and hashing

    /**
     * 集合o 与 集合之间判断是否相等
     * 这里可能是同一个集合 也可能是不是同一个集合 但是其中的元素相同
     */
    public boolean equals(Object o) {
        // o 就是 当前集合 无须再进行判断，可以直接返回true
        if (o == this)
            return true;
        // 如果 对象o 不是 List 直接返回false
        if (!(o instanceof List))
            return false;
        // 返回两个迭代器 进行迭代
        ListIterator<E> e1 = listIterator();
        ListIterator<?> e2 = ((List<?>) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            // 判断 是否 相等
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        // 如果 两者有迭代完之后 next 返回true 则返回false
        return !(e1.hasNext() || e2.hasNext());
    }

    /**
     * 返回集合的哈希值 用于集合的 对比 去重等
     */
    public int hashCode() {
        int hashCode = 1;
        for (E e : this)
            hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
        return hashCode;
    }

    /**
     * 删除迭代删除 fromIndex 到 toIndex 光标下的值
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }

    /**
     * 这个列表的次数已经结构修改。结构修改是那些改变的大小列表,或者扰乱它在这样一个时尚的迭代进展可能产生不正确的结果。
     * transient 阻止实例中那些用此关键字声明的变量持久化、对象序列化时不被持久化和恢复
     */
    protected transient int modCount = 0;

    // 判断 是否 越界
    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size();
    }
}

// 截取集合类
class SubList<E> extends AbstractList<E> {
    private final AbstractList<E> l;
    private final int offset;
    private int size;

    // 截取集合类构造方法
    SubList(AbstractList<E> list, int fromIndex, int toIndex) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > list.size())
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
        l = list;
        offset = fromIndex;
        size = toIndex - fromIndex;
        this.modCount = l.modCount;
    }

    public E set(int index, E element) {
        rangeCheck(index);
        checkForComodification();
        return l.set(index+offset, element);
    }

    public E get(int index) {
        rangeCheck(index);
        checkForComodification();
        return l.get(index+offset);
    }

    public int size() {
        checkForComodification();
        return size;
    }

    public void add(int index, E element) {
        rangeCheckForAdd(index);
        checkForComodification();
        l.add(index+offset, element);
        this.modCount = l.modCount;
        size++;
    }

    public E remove(int index) {
        rangeCheck(index);
        checkForComodification();
        E result = l.remove(index+offset);
        this.modCount = l.modCount;
        size--;
        return result;
    }

    protected void removeRange(int fromIndex, int toIndex) {
        checkForComodification();
        l.removeRange(fromIndex+offset, toIndex+offset);
        this.modCount = l.modCount;
        size -= (toIndex-fromIndex);
    }

    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        int cSize = c.size();
        if (cSize==0)
            return false;

        checkForComodification();
        l.addAll(offset+index, c);
        this.modCount = l.modCount;
        size += cSize;
        return true;
    }

    public Iterator<E> iterator() {
        return listIterator();
    }

    public ListIterator<E> listIterator(final int index) {
        checkForComodification();
        rangeCheckForAdd(index);

        return new ListIterator<E>() {
            private final ListIterator<E> i = l.listIterator(index+offset);

            public boolean hasNext() {
                return nextIndex() < size;
            }

            public E next() {
                if (hasNext())
                    return i.next();
                else
                    throw new NoSuchElementException();
            }

            public boolean hasPrevious() {
                return previousIndex() >= 0;
            }

            public E previous() {
                if (hasPrevious())
                    return i.previous();
                else
                    throw new NoSuchElementException();
            }

            public int nextIndex() {
                return i.nextIndex() - offset;
            }

            public int previousIndex() {
                return i.previousIndex() - offset;
            }

            public void remove() {
                i.remove();
                SubList.this.modCount = l.modCount;
                size--;
            }

            public void set(E e) {
                i.set(e);
            }

            public void add(E e) {
                i.add(e);
                SubList.this.modCount = l.modCount;
                size++;
            }
        };
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new SubList<>(this, fromIndex, toIndex);
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkForComodification() {
        if (this.modCount != l.modCount)
            throw new ConcurrentModificationException();
    }
}

class RandomAccessSubList<E> extends SubList<E> implements RandomAccess {
    RandomAccessSubList(AbstractList<E> list, int fromIndex, int toIndex) {
        super(list, fromIndex, toIndex);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new RandomAccessSubList<>(this, fromIndex, toIndex);
    }
}

```

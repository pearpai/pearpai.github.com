---
layout:     post
title:      "Java 基础库源码学习List（7）----  ArrayList"
date:       UTC2017-10-20 18:43:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - List
    - 源码
    - java
---
### 感慨
将ArrayList 注释一遍真的好累，需要耐心，坚持，首次这样写博客应该有很多问题，如果各位有说明建议的话 欢迎指教啊。

### 源码
```java

package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 共享空数组用于空实例
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 共享一个空数组实例，与EMPTY_ELEMENTDATA 区分开来，当第一个元素添加进来将膨胀。
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 这是存储元素的数组，capacity 就是这个数组的长度，任何情况当
     * elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * 将被初始为DEFAULT_CAPACITY长度的数组，知道第一个元素添加
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList 集合中元素的数量
     */
    private int size;

    /**
     * Constructs an empty list with the specified initial capacity.
     * 构造一个空的集合，并初始化容量的大小为initialCapacity
     */
    public ArrayList(int initialCapacity) {
        // 当初始化大小为initialCapacity 数组
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // 如果为0 这使用默认的空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // 其他情况 报错
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 初始化一个空的列表同时容量为10
     * 此时使用的是空数组，当进行add等操作的时候进行判断，如果是DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * 则重新进行初始化elementData
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个机会包含c中的所有元素，返回一个有序的机会迭代器
     */
    public ArrayList(Collection<? extends E> c) {
        // 返回一个集合c的 数组
        elementData = c.toArray();
        // 判断 elementData 的长度，如果不为0，则
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            // c.toArray() 返回的类型可能不是Object[]，此时需要重新对elementData 初始化赋值
            if (elementData.getClass() != Object[].class)
                // 重新对数组copy
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 长度为0 返回空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 裁剪集合的数组，如果size 等于0 则直接返回空的数组，如果不为0 这将elementData 重新copy数组，
     * 其长度即为size的大小
     */
    public void trimToSize() {
        // 对 数组的增删改等操作都会使得modCount的增加，用于对迭代器操作时候回的判断，
        // 确保迭代过程中可以能够进行正确的判断
        modCount++;
        // 长度判断，并进行剪裁数组
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

    /**
     * 添加容量的大小，同时确保 容量的正确
     * 最大为MAX_ARRAY_SIZE 或者 Integer.MAX_VALUE 取决于
     * 下面的为极端情况判断 见方法grow(int minCapacity)的处理
     * (minCapacity > MAX_ARRAY_SIZE) ?
     *    Integer.MAX_VALUE :
     *    MAX_ARRAY_SIZE;
     */
    public void ensureCapacity(int minCapacity) {
        //
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            // 任何大小如果不是默认的元素表
            ? 0
            // 如果不为空，我们这边也默认为DEFAULT_CAPACITY 即为10
            : DEFAULT_CAPACITY;
        // 当minCapacity > minExpand
        if (minCapacity > minExpand) {
            //重新设置容量的大小
            ensureExplicitCapacity(minCapacity);
        }
    }

    // 内部容量是否添加
    private void ensureCapacityInternal(int minCapacity) {
        // 如果 elementData = {}
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // minCapacity 与 DEFAULT_CAPACITY 取最大值 为容量
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        // 重新生成 elementData 操作
        ensureExplicitCapacity(minCapacity);
    }

    // queren 操作数+1
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code  只有minCapacity大于elementData的长度才进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 最大的 数组的长度，因为在一些虚拟机中 数组会存在头信息 如果直接Integer.MAX_VALUE
     * 可能是数据越界
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     * 添加容量，至少确保能够保存最少容量的元素
     * 当minCapacity<0 即为越界 将报错
     */
    private void grow(int minCapacity) {
        // 原有的容量
        int oldCapacity = elementData.length;
        // 新有的容量 为老的容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 当心的容量小于minCapacity 则新的容量为minCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 当newCapacity大于MAX_ARRAY_SIZE即Integer.MAX_VALUE - 8
        // 则进行设置容量为Integer.MAX_VALUE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    // 容量设置
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     * 返回集合中元素的值
     */
    public int size() {
        return size;
    }

    /**
     * 集合 是否为空
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 集合是否 包含 o 元素
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    /**
     * 集合元素的判断 null 与 对象的判断方式 进行区别
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 查询o元素 在最后一次出现的位置
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回一个浅复制
     * 集合中的元素没有进行复制
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }

    /**
     * 返回 集合的 数组
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     *  返回一个数组包含集合的所有元素的数组  a
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 如果a.length小于元素的数量，则重新生成一个数组
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        // todo 此处不清楚为什么 需要对a[size] 设置为null
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // 返回索引位置的元素
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回指定位置的元素
     */
    public E get(int index) {
        // 判断 索引位置 是否大于了 size
        rangeCheck(index);
        return elementData(index);
    }

    /**
     * 替换索引位置的元素
     * 如果替换成功 返回原有的元素
     */
    public E set(int index, E element) {
        // 判断 索引位置 是否大于了 size
        rangeCheck(index);
        // 获取原有元素
        E oldValue = elementData(index);
        // 更新原有元素
        elementData[index] = element;
        // 返回原有元素
        return oldValue;
    }

    /**
     * 添加元素到
     */
    public boolean add(E e) {
        // 添加操作数量 同时判断 容量是否需要添加
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 在size位置添加e元素
        elementData[size++] = e;
        return true;
    }

    /**
     * 在指定索引位置添加元素
     */
    public void add(int index, E element) {
        // 判断 索引位置是否越界
        rangeCheckForAdd(index);
        // 添加操作数量 同时判断 容量是否需要添加
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 索引位置后的元素向后移动一个位置 此方法值得学习
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 删除索引位置的元素
     */
    public E remove(int index) {
        // 判断索引位置是否越界
        rangeCheck(index);
        // 添加操作数
        modCount++;
        // 索引位置的元素
        E oldValue = elementData(index);
        // 将要移动元素的的数量
        int numMoved = size - index - 1;
        // numMoved 如果 数量大于0 则进行复制操作 将index后的元素 向前移动一位
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        // 同时将原有size的位置的元素 设置为null去除引用
        // 因为是索引是0开始 所以先要减去1
        elementData[--size] = null; // clear to let GC do its work
        // 返回原有的元素
        return oldValue;
    }

    /**
     * 删除集合数组中的 第一个与o 相等的元素
     */
    public boolean remove(Object o) {
        // if 针对元素的不同执行的两种比较
        // for 遍历 size 内  elementData 中的元素并进行比较
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    // 快速移除
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    // // 快速移除
                    fastRemove(index);
                    return true;
                }
        }
        // 执行到此 说明比较已经过程中没有出现相同的元素
        return false;
    }

    /*
     * 私有方法 已经 跳过了 判断条件 直接进行了 数组的复制 已经 对原有size-1 位置处的设置null
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
         // 同时将原有size的位置的元素 设置为null去除引用
         // 因为是索引是0开始 所以先要减去1
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 清空集合
     */
    public void clear() {
        // 操作数添加
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 将集合c中的元素 添加到集合中
     */
    public boolean addAll(Collection<? extends E> c) {
        // 将集合c 转换为数组
        Object[] a = c.toArray();
        // 获取数组的大小
        int numNew = a.length;
        // 更新容量 以及  添加操作数
        ensureCapacityInternal(size + numNew);  // Increments modCount
        // 复制数组
        System.arraycopy(a, 0, elementData, size, numNew);
        // 更新 集合元素的数量
        size += numNew;
        // 返回 添加元素的数量 与0 的对比
        return numNew != 0;
    }

    /**
     * 在索引位置 添加 c集合中的元素
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        // 判断元素
        rangeCheckForAdd(index);
        // 集合转换为数组
        Object[] a = c.toArray();
        // 数组长度
        int numNew = a.length;

        // 添加操作数  同时更新 容量
        ensureCapacityInternal(size + numNew);  // Increments modCount
        // 原有集合中的元素需要移动的数量
        int numMoved = size - index;
        // 判断移动数量 是否为0  如果为0 则原有数组elementData元素不需要移动
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
        // 将 集合c 的数组 a 复制到 elementData 数组中的index的起始位置
        System.arraycopy(a, 0, elementData, index, numNew);
        // 更新 集合元素的数量
        size += numNew;
        // 返回 添加元素的数量 与0 的对比
        return numNew != 0;
    }

    /**
     * 移除list 中的fromIndex 到 toIndex 中的 元素
     */
    protected void removeRange(int fromIndex, int toIndex) {
        // 添加操作数
        modCount++;
        // 获取 需要移动的
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        // 清楚引用 使其可以进行gc操作
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 对索引值的判断
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 对索引值的判断
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 构建报错信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 移除集合中所有在c集合中存在的元素
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }

    /**
     * 保留所有在c中存在的元素
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }

    /**
     * 私有方法用于删除或保留元素
     */
    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            // 这是个保护措施c.contains() 可能会 throws 抛错
            // 如果发生此情况 将后续的元素 将后续的元素 继续复制到 elementData
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                // 此时更新 复制元素的索引位置
                w += size - r;
            }
            // w != size 说明有部分元素 被移除
            if (w != size) {
                // 移除引用，使得可以进行后续的系统GC操作
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                // 更新操作数
                modCount += size - w;
                size = w;
                // 返回修改状态
                modified = true;
            }
        }
        return modified;
    }

    /**
     * 定制序列化过程
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // 设置当前的操作数为期望的操作数
        int expectedModCount = modCount;
        // 将当前的类写入一个非静态同时非暂时性的域到这个流中
        s.defaultWriteObject();

        // 写入容量 与 clone() 兼容
        s.writeInt(size);

        // 将elementData 对象顺序的写入到流中
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
        // 如果操作数 与期望的操作数 说明有其他线程对集合进行了操作，此时抛错
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * 从流中重新进行实例化（也就是说反序列化）
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // 将当前的类读入一个非静态同时非暂时性的域到这个流中
        s.defaultReadObject();

        // 读取size
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            // 将要进行克隆，对容量进行设置
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // 以适当的顺序读取多有对象
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }

    /**
     * 返回一个以指定位置开始的迭代器
     */
    public ListIterator<E> listIterator(int index) {
        // 判断索引位置是否越界
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        // 返回实例迭代器
        return new ListItr(index);
    }

    /**
     * 返回集合的迭代器 以0 开始的
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * 返回一个 以 合适的顺序 集合所有元素的迭代
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * An optimized version of AbstractList.Itr
     * 对AbstractList.Itr 的最优化版本
     */
    private class Itr implements Iterator<E> {
        // 下一个迭代位置
        int cursor;       // index of next element to return
        // 上一次迭代位置，如果没有将返回-1
        int lastRet = -1; // index of last element returned; -1 if no such
        // 操作数，确保在操作的的过程中 集合没有被别的线程操作
        int expectedModCount = modCount;
        // cursor != size 表明还有未被迭代的，此时返回true
        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            // 判断操作数 是否发生了变化 如果变化则跑出错误
            checkForComodification();
            // i 为当前索引位置
            int i = cursor;
            // i >= size 则没有相关元素 抛错
            if (i >= size)
                throw new NoSuchElementException();
            // 集合的数组
            Object[] elementData = ArrayList.this.elementData;
            // i >= elementData.length 可能是多线程的操作导致数组长度发生了变化
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            // 此时设置的为想一次将要读取数组的位置
            cursor = i + 1;
            // 返回迭代的值 同时设置 本次读取的位置即上一次的读取索引位置
            return (E) elementData[lastRet = i];
        }

        // 移除当前迭代的元素
        public void remove() {
            // lastRet < 0 设置 此时属于非法的状态 两种可能 一种是直接的迭代器刚生成
            // 还一种可能是连续移除第二次
            if (lastRet < 0)
                throw new IllegalStateException();
            // 判断操作数 是否发生了变化 如果变化则跑出错误
            checkForComodification();
            // 执行移除方法
            try {
                ArrayList.this.remove(lastRet);
                // 同时设置下次将要读取的光标位置
                cursor = lastRet;
                // 一次移除后 将上一次读取的位置设置为-1 不能同时移除两次
                lastRet = -1;
                // 判断操作数 如果不对 将抛错
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                // ArrayList 非线程安全，在操作过程中需要验证是否收到了 其他线程的干扰
                throw new ConcurrentModificationException();
            }
        }

        // 在光标处 对 集合中数组执行consumer.accept 方法
        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
           // 确定 consumer 不为 null，如果是则报错
            Objects.requireNonNull(consumer);
            // 获取集合元素的 数量
            final int size = ArrayList.this.size;
            // 设置将要读取的位置 为当前光标所在的位置
            int i = cursor;
            // 判断 i 大于 元素数量 则没有需要执行的 直接返回了
            if (i >= size) {
                return;
            }
            // 获取 集合元素数组
            final Object[] elementData = ArrayList.this.elementData;
            // 长度判断
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            // 循环遍历 同时判断 操作数是否发生了变化
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            // 更新光标位置
            cursor = i;
            // 更新上一次光标的位置
            lastRet = i - 1;
            checkForComodification();
        }
        // 操作数 判断 是否与期望的保持一致
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * 一个对AbstractList.ListItr的优化
     * 倒序
     */
    private class ListItr extends Itr implements ListIterator<E> {
        // 构造方法
        ListItr(int index) {
            super();
            // 设置光标的位置
            cursor = index;
        }
        // 如果不为0 这说明 光标可以继续前移
        public boolean hasPrevious() {
            return cursor != 0;
        }
        // 数组下一个索引位置
        public int nextIndex() {
            return cursor;
        }

        // 集合数组前一个索引位置
        public int previousIndex() {
            return cursor - 1;
        }

        // 获取集合数组前一个元素
        @SuppressWarnings("unchecked")
        public E previous() {
            // 操作数的判断
            checkForComodification();
            // 将要读取的数组的索引位置
            int i = cursor - 1;
            // 判断 读取的位置 是否越界
            if (i < 0)
                throw new NoSuchElementException();
            // 获取 集合数组
            Object[] elementData = ArrayList.this.elementData;
            // 判断 i 是否越界
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            // 设置下次将要读取的光标位置
            cursor = i;
            // 返回 前一个元素 同时设置 上一次读取位置 lastRet
            return (E) elementData[lastRet = i];
        }

        // 替换 上一次读取位置的 元素为e
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            // 操作数判断
            checkForComodification();
            // 设置集合数组的lastRet位置的值
            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                // 多线程操作报错
                throw new ConcurrentModificationException();
            }
        }

        // 迭代器添加数据e
        public void add(E e) {
            // 操作数判断
            checkForComodification();

            try {
                // 设置当前的光标位置
                int i = cursor;
                ArrayList.this.add(i, e);
                // 添加完毕后 光标移动到下一位
                cursor = i + 1;
                // 此时 上一个读取元素位置 值为-1 因为上一个位置为新添加的位置
                lastRet = -1;
                // 修改操作数
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * 返回一个 SubList 实例，截取原有集合的 fromIndex 到 toIndex
     */
    public List<E> subList(int fromIndex, int toIndex) {
        // 对截取的索引位置进行判断是否正常
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }

    // 进行截取集合限制条件判断
    static void subListRangeCheck(int fromIndex, int toIndex, int size) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > size)
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
    }

    // 截取类
    private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        // 即原有 集合 数组 0 到 fromIndex 的位置 即为 截取数组的初始位置
        private final int offset;
        int size;
        // 构造函数
        SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
        }
        // 修改 索引位置值 为 e
        public E set(int index, E e) {
            // 索引值进行判断 是否越界
            rangeCheck(index);
            // 判断操作数 是否发生了变化
            checkForComodification();
            // 获取需要替换的 集合数组中原有的值
            E oldValue = ArrayList.this.elementData(offset + index);
            // 设置 offset + index 索引位置的值为e
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }

        public E get(int index) {
            // 索引值进行判断 是否越界
            rangeCheck(index);
            // 判断操作数 是否发生了变化
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }

        public int size() {
            // 判断操作数 是否发生了变化
            checkForComodification();
            // 返回截取段 元素的数量
            return this.size;
        }

        public void add(int index, E e) {
            // 判断 添加元素的索引是否越界
            rangeCheckForAdd(index);
            // 判断操作数 是否发生了变化
            checkForComodification();
            // 在原有集合 parentOffset + index 索引位置处添加 e 元素
            parent.add(parentOffset + index, e);
            // 更新操作数
            this.modCount = parent.modCount;
            // 此处的size 值变+1
            this.size++;
        }

        public E remove(int index) {
            // 判断 移除元素的索引是否越界
            rangeCheck(index);
            // 判断操作数 是否发生了变化
            checkForComodification();
            // 调用父集合的实例的remove 方法 删除索引位置
            E result = parent.remove(parentOffset + index);
            // 更新操作数
            this.modCount = parent.modCount;
            // 截取集合实例的元素 -1
            this.size--;
            // 返回被移除的元素
            return result;
        }

        // 移除fromIndex 到 toIndex 范围内的的元素
        protected void removeRange(int fromIndex, int toIndex) {
            // 判断操作数是否正确
            checkForComodification();
            // 调用父集合 remove 方法 移除相关元素
            parent.removeRange(parentOffset + fromIndex,
                               parentOffset + toIndex);
            // 更新操作数
            this.modCount = parent.modCount;
            // 更新 截取截取集合的 元素数量
            this.size -= toIndex - fromIndex;
        }

        // 将集合c中的元素添加到 截取的集合中
        public boolean addAll(Collection<? extends E> c) {
            // 在截取集合的最后位置添加c集合中的元素
            return addAll(this.size, c);
        }

        // 向截取集合添加 c集合元素的 具体方法
        public boolean addAll(int index, Collection<? extends E> c) {
            // 对索引进行 越界判断
            rangeCheckForAdd(index);
            // 获取c 集合元素的数量
            int cSize = c.size();
            // 判断c集合的元素 如果0 这支付返回添加失败
            if (cSize==0)
                return false;

            // 对操作数进行验证
            checkForComodification();
            // 调用父集合 的添加方法
            parent.addAll(parentOffset + index, c);
            // 更新操作数
            this.modCount = parent.modCount;
            // 更新 截取集合的 元素数量
            this.size += cSize;
            return true;
        }

        // 迭代器
        public Iterator<E> iterator() {
            return listIterator();
        }

        // 截取集合 迭代器 todo 参考 Itr ListItr 类
        public ListIterator<E> listIterator(final int index) {
            // 对操作数进行判断
            checkForComodification();
            // 判断索引位置是否越界
            rangeCheckForAdd(index);
            // offset 为 父集合被截取的起始位置
            final int offset = this.offset;

            // 返回一个迭代器实例
            return new ListIterator<E>() {
                // 光标位置
                int cursor = index;
                //
                int lastRet = -1;
                int expectedModCount = ArrayList.this.modCount;

                public boolean hasNext() {
                    return cursor != SubList.this.size;
                }

                @SuppressWarnings("unchecked")
                public E next() {
                    checkForComodification();
                    int i = cursor;
                    if (i >= SubList.this.size)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i + 1;
                    return (E) elementData[offset + (lastRet = i)];
                }

                public boolean hasPrevious() {
                    return cursor != 0;
                }

                @SuppressWarnings("unchecked")
                public E previous() {
                    checkForComodification();
                    int i = cursor - 1;
                    if (i < 0)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i;
                    return (E) elementData[offset + (lastRet = i)];
                }

                @SuppressWarnings("unchecked")
                public void forEachRemaining(Consumer<? super E> consumer) {
                    Objects.requireNonNull(consumer);
                    final int size = SubList.this.size;
                    int i = cursor;
                    if (i >= size) {
                        return;
                    }
                    final Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length) {
                        throw new ConcurrentModificationException();
                    }
                    while (i != size && modCount == expectedModCount) {
                        consumer.accept((E) elementData[offset + (i++)]);
                    }
                    // update once at end of iteration to reduce heap write traffic
                    lastRet = cursor = i;
                    checkForComodification();
                }

                public int nextIndex() {
                    return cursor;
                }

                public int previousIndex() {
                    return cursor - 1;
                }

                public void remove() {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        SubList.this.remove(lastRet);
                        cursor = lastRet;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void set(E e) {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        ArrayList.this.set(offset + lastRet, e);
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void add(E e) {
                    checkForComodification();

                    try {
                        int i = cursor;
                        SubList.this.add(i, e);
                        cursor = i + 1;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                final void checkForComodification() {
                    if (expectedModCount != ArrayList.this.modCount)
                        throw new ConcurrentModificationException();
                }
            };
        }

        // 返回一个 截取的集合  
        public List<E> subList(int fromIndex, int toIndex) {
            subListRangeCheck(fromIndex, toIndex, size);
            return new SubList(this, offset, fromIndex, toIndex);
        }

        // 对所有数据 进行越界判断
        private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        // 统一的是越界判断
        private void rangeCheckForAdd(int index) {
            if (index < 0 || index > this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        // 消息模板
        private String outOfBoundsMsg(int index) {
            return "Index: "+index+", Size: "+this.size;
        }

        // 对操作数进行判断 用于判断多线程操作时候的 变化验证
        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }

        // 分割 流处理的 中的并行处理
        public Spliterator<E> spliterator() {
            checkForComodification();
            return new ArrayListSpliterator<E>(ArrayList.this, offset,
                                               offset + this.size, this.modCount);
        }
    }

    // 流处理 类似于迭代
    // stream.forEach java8
    // 集合中的每个元素执行的是 action.accept(E)
    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * 分割并行流处理
     */
    @Override
    public Spliterator<E> spliterator() {
        return new ArrayListSpliterator<>(this, 0, -1, 0);
    }

    // 基于 索引，一分为二的 懒汉模式的 初始分割
    // 用于并行流的处理
    // 此处后续添加 详细demo 处理
    static final class ArrayListSpliterator<E> implements Spliterator<E> {

        private final ArrayList<E> list;
        private int index; // current index, modified on advance/split
        private int fence; // -1 until used; then one past last index
        private int expectedModCount; // initialized when fence set

        /** Create new spliterator covering the given  range */
        ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                             int expectedModCount) {
            this.list = list; // OK if null unless traversed
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
        }

        private int getFence() { // initialize fence to size on first use
            int hi; // (a specialized variant appears in method forEach)
            ArrayList<E> lst;
            if ((hi = fence) < 0) {
                if ((lst = list) == null)
                    hi = fence = 0;
                else {
                    expectedModCount = lst.modCount;
                    hi = fence = lst.size;
                }
            }
            return hi;
        }

        public ArrayListSpliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null : // divide range in half unless too small
                new ArrayListSpliterator<E>(list, lo, index = mid,
                                            expectedModCount);
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            if (action == null)
                throw new NullPointerException();
            int hi = getFence(), i = index;
            if (i < hi) {
                index = i + 1;
                @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            int i, hi, mc; // hoist accesses and checks from loop
            ArrayList<E> lst; Object[] a;
            if (action == null)
                throw new NullPointerException();
            if ((lst = list) != null && (a = lst.elementData) != null) {
                if ((hi = fence) < 0) {
                    mc = lst.modCount;
                    hi = lst.size;
                }
                else
                    mc = expectedModCount;
                if ((i = index) >= 0 && (index = hi) <= a.length) {
                    for (; i < hi; ++i) {
                        @SuppressWarnings("unchecked") E e = (E) a[i];
                        action.accept(e);
                    }
                    if (lst.modCount == mc)
                        return;
                }
            }
            throw new ConcurrentModificationException();
        }

        public long estimateSize() {
            return (long) (getFence() - index);
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }

    // 过滤器 filter.test(element) 判断需要过滤的 元素
    @Override
    public boolean removeIf(Predicate<? super E> filter) {
        // 判断 过滤器 不为null
        Objects.requireNonNull(filter);
        // 找出要删除哪些元素， 在个过滤的阶段，任何的抛错将使得厉害这个集合而不被修改
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        // 操作数 赋值到期盼中的操作数
        final int expectedModCount = modCount;
        // 赋值 需要 遍历的 数量
        final int size = this.size;
        // 返利赋值
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            // 如果过滤器 验证成功
            if (filter.test(element)) {
                // 存储 符合过滤条件的 序号
                removeSet.set(i);
                // 数量+1
                removeCount++;
            }
        }
        // 操作数判断
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // 留下的元素向左移动 判断 如果removeCount > 0 则说明有需要过滤的的元素
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            // 确定 过滤后集合 元素的 数量
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                // 从0 开始 找到第一个 符合条件的 元素集合数组的位置
                // 同时i 变为第一个符合过滤条件的元素位置
                i = removeSet.nextClearBit(i);
                // 将 符合条件的元素存入到elementData 中
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                // 解除引用 用于后续系统gc 操作
                elementData[k] = null;  // Let gc do its work
            }
            // 更新 集合 元素数量
            this.size = newSize;
            // 判断是否操作数有变化 如果变化 则报错
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            // 更新操作数
            modCount++;
        }
        // 返回过滤状态
        return anyToRemove;
    }

    // 替换执行operator 替换原有的元素
    @Override
    @SuppressWarnings("unchecked")
    public void replaceAll(UnaryOperator<E> operator) {
        // 判断操作是否为空
        Objects.requireNonNull(operator);
        // 设定期望操作数
        final int expectedModCount = modCount;
        // 设置 集合 元素的数量 本地化
        final int size = this.size;
        // 遍历 elementData 数组 i < size， 操作数发生变化 将退出此遍历
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            // 执行 操作方法
            elementData[i] = operator.apply((E) elementData[i]);
        }
        // 如果操作数发生变化 表明有其他线程对此集合进行了操作将抛错
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        // 操作数 添加1
        modCount++;
    }

    // 集合排序
    @Override
    @SuppressWarnings("unchecked")
    public void sort(Comparator<? super E> c) {
        // 设定期望操作数
        final int expectedModCount = modCount;
        // 调用Arrays.sort 执行 排序操作
        Arrays.sort((E[]) elementData, 0, size, c);
        // 如果操作数发生变化 表明有其他线程对此集合进行了操作将抛错
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        // 操作数 添加1
        modCount++;
    }
}

```

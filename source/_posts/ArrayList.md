---
title: ArrayList
tags: collection
date: 2018-12-16 22:01:24
---

## ArrayList概述
ArrayList是List接口的可变数组的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。
每个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝，因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。 

注：建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

## ArrayList源码解析
1.类结构
```
//通过ArrayList实现的接口可知，其支持随机访问，能被克隆，支持序列化
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    //序列版本号
    private static final long serialVersionUID = 8683452581122892189L;

　　//默认初始容量
    private static final int DEFAULT_CAPACITY = 10;

    //被用于空实例的共享空数组实例
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //被用于默认大小的空实例的共享数组实例。其与EMPTY_ELEMENTDATA的区别是：当我们向数组中添加第一个元素时，知道数组该扩充多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * Object[]类型的数组，保存了添加到ArrayList中的元素。ArrayList的容量是该Object[]类型数组的长度
     * 当第一个元素被添加时，任何空ArrayList中的elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA将会被
     * 扩充到DEFAULT_CAPACITY（默认容量）。
     */
    transient Object[] elementData; //非private是为了方便嵌套类的访问

    // ArrayList的大小（指其所含的元素个数）
    private int size;

    ......  

}
```

ArrayList包含了两个重要的对象：elementData 和 size。
1.elementData 是"Object[] 类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建 ArrayList，则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具 体的增长方式，请参考源码分析中的ensureCapacity()函数。

2.size 则是动态数组的实际大小。

## 构造函数
ArrayList提供了三种方式的构造器，可以构造一个默认初始容量为10的空列表、构造一个指定初始容量的空列表以及构造一个包含指定collection的元素的列表，这些元素按照该collection的迭代器返回的顺序排列的。
```
/**
     * 构造一个指定初始容量的空列表
     * @param  initialCapacity  ArrayList的初始容量
     * @throws IllegalArgumentException 如果给定的初始容量为负值
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    // 构造一个默认初始容量为10的空列表
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定collection的元素的列表，这些元素按照该collection的迭代器返回的顺序排列的
     * @param c 包含用于去构造ArrayList的元素的collection
     * @throws NullPointerException 如果指定的collection为空
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray()可能不会正确地返回一个 Object[]数组，那么使用Arrays.copyOf()方法
            if (elementData.getClass() != Object[].class)
                //Arrays.copyOf()返回一个 Object[].class类型的，大小为size，元素为elementData[0,...,size-1]
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

ArrayList构造一个默认初始容量为10的空列表：
1) 初始情况：elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}； size = 0;

2) 当向数组中添加第一个元素时，通过add(E e)方法中调用的ensureCapacityInternal(size + 1)方法，即ensureCapacityInternal(1)；

3) 在ensureCapacityInternal(int minCapacity)方法中，可得的minCapacity=DEFAULT_CAPACITY=10，然后再调用ensureExplicitCapacity(minCapacity)方法，即ensureExplicitCapacity(10)；

4) 在ensureExplicitCapacity(minCapacity)方法中调用grow(minCapacity)方法，即grow(10)，此处为真正具体的数组扩容的算法，在此方法中，通过elementData = Arrays.copyOf(elementData, 10)具体实现了elementData数组初始容量为10的构造。

## 调整数组的容量
从add()与addAll()方法中可以看出，每当向数组中添加元素时，都要去检查添加元素后的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容实质上是通过私有的方法ensureCapacityInternal(int minCapacity) -> ensureExplicitCapacity(int minCapacity) -> grow(int minCapacity)来实现的，但在jdk1.8中，向用户提供了一个public的方法ensureCapacity(int minCapacity)使用户可以手动的设置ArrayList实例的容量，以减少递增式再分配的数量。此处与jdk1.6中直接通过一个公开的方法ensureCapacity(int minCapacity)来实现数组容量的调整有区别。
```
/**
     * public方法，让用户能手动设置ArrayList的容量
     * @param   minCapacity 期望的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    private void ensureCapacityInternal(int minCapacity) {
        //当elementData为空时，ArrayList的初始容量最小为DEFAULT_CAPACITY（10）
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    //数组可被分配的最大容量；当需要的数组尺寸超过VM的限制时，可能导致OutOfMemoryError
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 增加数组的容量，确保它至少能容纳指定的最小容量的元素量
     * @param minCapacity 期望的最小容量
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //注意此处扩充capacity的方式是将其向右一位再加上原来的数，实际上是扩充了1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0) 
            newCapacity = hugeCapacity(minCapacity); //设置数组可被分配的最大容量
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### 为什么ArrayList自动容量扩充选择扩充1.5倍？
这种算法构造出来的新的数组长度的增量都会比上一次大( 而且是越来越大) ，即认为客户需要增加的数据很多，而避免频繁newInstance 的情况。

## 添加元素
ArrayList提供了add(E e)、add(int index, E element)、addAll(Collection<? extends E> c)、addAll(int index, Collection<? extends E> c)这些添加元素的方法。
```
//将指定的元素(E e)添加到此列表的尾部
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    //将指定的元素(E e)插入到列表的指定位置(index)
    public void add(int index, E element) {
        rangeCheckForAdd(index); //判断参数index是否IndexOutOfBoundsException

        ensureCapacityInternal(size + 1);  // Increments modCount!!  如果数组长度不足，将进行扩容
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index); //将源数组中从index位置开始后的size-index个元素统一后移一位
        elementData[index] = element;
        size++;
    }

    /**
     * 按照指定collection的迭代器所返回的元素顺序，将该collection中的所有元素添加到此列表的尾部
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //将数组a[0,...,numNew-1]复制到数组elementData[size,...,size+numNew-1]
        System.arraycopy(a, 0, elementData, size, numNew); 
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从指定的位置开始，将指定collection中的所有元素插入到此列表中，新元素的顺序为指定collection的迭代器所返回的元素顺序
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index); //判断参数index是否IndexOutOfBoundsException

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            //先将数组elementData[index,...,index+numMoved-1]复制到elementData[index+numMoved,...,index+2*numMoved-1]
            //即，将源数组中从index位置开始的后numMoved个元素统一后移numNew位
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
        //再将数组a[0,...,numNew-1]复制到数组elementData[index,...,index+numNew-1]
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
```

## 删除元素
ArrayList提供了remove(int index)、remove(Object o)、clear()、removeRange(int fromIndex, int toIndex)、removeAll(Collection<?> c)、retainAll(Collection<?> c)这些删除元素的方法。
```
/**
     * 移除此列表中指定位置上的元素
     * @param index 需被移除的元素的索引
     * @return the element 被移除的元素值
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        rangeCheck(index);  //判断index是否 <= size

        modCount++;
        E oldValue = elementData(index);
        //将数组elementData中index位置之后的所有元素向前移一位
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; //将原数组最后一个位置置为null，由GC清理

        return oldValue;
    }

    //移除ArrayList中首次出现的指定元素(如果存在)，ArrayList中允许存放重复的元素
    public boolean remove(Object o) {
        // 由于ArrayList中允许存放null，因此下面通过两种情况来分别处理。
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index); //私有的移除方法，跳过index参数的边界检查以及不返回任何值
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    //私有的删除指定位置元素的方法，跳过index参数的边界检查以及不返回任何值
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    //清空ArrayList，将全部的元素设为null
    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    //删除ArrayList中从fromIndex（包含）到toIndex（不包含）之间所有的元素，共移除了toIndex-fromIndex个元素
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;  //需向前移动的元素的个数
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    //删除ArrayList中包含在指定容器c中的所有元素
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);  //检查指定的对象c是否为空
        return batchRemove(c, false);
    }

    //移除ArrayList中不包含在指定容器c中的所有元素，与removeAll(Collection<?> c)正好相反
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c); 
        return batchRemove(c, true);
    }

    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;  //读写双指针
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement) //判断指定容器c中是否含有elementData[r]元素
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

## 修改元素
ArrayList提供了set(int index, E element)方法来修改指定索引上的值。
```
//将指定索引上的值替换为新值，并返回旧值
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

## 查找元素
ArrayList提供了get(int index)、contains(Object o)、indexOf(Object o)、lastIndexOf(Object o)、get(int index)这些查找元素的方法。
```
//判断ArrayList中是否包含Object(o)
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    //正向查找，返回ArrayList中元素Object o第一次出现的位置，如果元素不存在，则返回-1
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

    //逆向查找，返回ArrayList中元素Object o最后一次出现的位置，如果元素不存在，则返回-1
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

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    //返回指定索引处的值
    public E get(int index) {
        rangeCheck(index);

        return elementData(index); //实质上return (E) elementData[index]
    }
```

## 其他public方法
trimToSize()、size()、isEmpty()、clone()、toArray()、toArray(T[] a)

```
//将底层数组的容量调整为当前列表保存的实际元素的大小的功能
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

    //返回ArrayList的大小（元素个数）
    public int size() {
        return size;
    }

   //判断ArrayList是否为空
    public boolean isEmpty() {
        return size == 0;
    }

    //返回此 ArrayList实例的浅拷贝
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
    
    //返回一个包含ArrayList中所有元素的数组
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    //如果给定的参数数组长度足够，则将ArrayList中所有元素按序存放于参数数组中，并返回
    //如果给定的参数数组长度小于ArrayList的长度，则返回一个新分配的、长度等于ArrayList长度的、包含ArrayList中所有元素的新数组
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

支持序列化的写入函数writeObject(java.io.ObjectOutputStream s)和读取函数readObject(java.io.ObjectInputStream s)
```
//序列化：将ArrayList的“大小，所有的元素值”都写入到输出流中
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    //反序列化：先将ArrayList的“大小”读出，然后将“所有的元素值”读出
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

## ArrayList的四种遍历方式
1.通过迭代器Iterator遍历：
```
Iterator iter = list.iterator();
    while (iter.hasNext())
    {
        System.out.println(iter.next());
    }
```
2.通过迭代器ListIterator遍历：
```
ListIterator<String> lIter = list.listIterator();
　　//顺向遍历
　　while(lIter.hasNext()){
    　　System.out.println(lIter.next());
　　}
　　//逆向遍历
　　while(lIter.hasPrevious()){
    　　System.out.println(lIter.previous());
　　}
```

**Iterator与ListIterator主要的区别：**
①Iterator可以应用于所有的集合，Set、List和Map和这些集合的子类型。而ListIterator只能用于List及其子类型；

②Iterator只能实现顺序向后遍历，ListIterator可实现顺序向后遍历和逆向（顺序向前）遍历；

③Iterator只能实现remove操作，ListIterator可以实现remove操作，add操作，set操作。

3.随机访问，通过索引值去遍历，由于ArrayList实现了RandomAccess接口
```
int size = list.size();
    for (int i=0; i<size; i++) 
    {
        System.out.println(list.get(i));        
    }
```

4.foreach循环遍历
```
 for(String str:list)
    {
        System.out.println(str);
    }
```

foreach与迭代器：在Java SE5引入了新的被称为Iterable接口，该接口包含一个能够产生Iterator的Iterator()方法，并且Iterable接口被foreach用来在序列中移动。因此如果你创建了任何实现Iterable的类，都可以将它用于foreach语句中。

## Fail-Fast机制
ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。




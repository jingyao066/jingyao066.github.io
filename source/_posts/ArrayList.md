---
title: ArrayList
tags: collection
date: 2018-12-16 22:01:24
---

## ArrayList概述
ArrayList是一个数组队列，相当于动态数组。它继承于AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。
1.ArrayList 继承了AbstractList，实现了List。提供了相关的添加、删除、修改、遍历等功能。
2.ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。可以通过元素的序号快速获取元素对象，这就是快速随机访问。
3.ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。
4.ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输。

每个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝，
因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。 

注：建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

## API
```
// Collection中定义的API
boolean             add(E object)
boolean             addAll(Collection<? extends E> collection)
void                clear()
boolean             contains(Object object)
boolean             containsAll(Collection<?> collection)
boolean             equals(Object object)
int                 hashCode()
boolean             isEmpty()
Iterator<E>         iterator()
boolean             remove(Object object)
boolean             removeAll(Collection<?> collection)
boolean             retainAll(Collection<?> collection)
int                 size()
<T> T[]             toArray(T[] array)
Object[]            toArray()
// AbstractCollection中定义的API
void                add(int location, E object)
boolean             addAll(int location, Collection<? extends E> collection)
E                   get(int location)
int                 indexOf(Object object)
int                 lastIndexOf(Object object)
ListIterator<E>     listIterator(int location)
ListIterator<E>     listIterator()
E                   remove(int location)
E                   set(int location, E object)
List<E>             subList(int start, int end)
// ArrayList新增的API
Object               clone()
void                 ensureCapacity(int minimumCapacity)
void                 trimToSize()
void                 removeRange(int fromIndex, int toIndex)
```

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
1.elementData 是"Object[] 类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，
我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建 ArrayList，
则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具 体的增长方式，请参考源码分析中的ensureCapacity()函数。

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
从add()与addAll()方法中可以看出，每当向数组中添加元素时，都要去检查添加元素后的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。
数组扩容实质上是通过私有的方法ensureCapacityInternal(int minCapacity) -> ensureExplicitCapacity(int minCapacity) -> grow(int minCapacity)来实现的，但在jdk1.8中，
向用户提供了一个public的方法ensureCapacity(int minCapacity)使用户可以手动的设置ArrayList实例的容量，以减少递增式再分配的数量。
此处与jdk1.6中直接通过一个公开的方法ensureCapacity(int minCapacity)来实现数组容量的调整有区别。
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

### 效率比对
验证代码...略
1.迭代器iterator：8ms
2.for循环RandomAccess：3ms
3.foreach：5ms
**遍历ArrayList时，使用随机访问(即，通过索引序号访问)效率最高，而使用迭代器的效率最低！**

foreach与迭代器：
在Java SE5引入了新的被称为Iterable接口，该接口包含一个能够产生Iterator的Iterator()方法，并且Iterable接口被foreach用来在序列中移动。
因此如果你创建了任何实现Iterable的类，都可以将它用于foreach语句中。

## toArray()异常
当我们调用ArrayList中的 toArray()，可能遇到过抛出“java.lang.ClassCastException”异常的情况。
ArrayList提供了2个toArray()函数：
```
Object[] toArray()
<T> T[] toArray(T[] contents)
```
调用 toArray() 函数会抛出“java.lang.ClassCastException”异常，但是调用 toArray(T[] contents) 能正常返回 T[]。
toArray() 会抛出异常是因为 toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如：将Object[]转换为的Integer[])，则会抛出“java.lang.ClassCastException”异常，
**因为Java不支持向下转型**
解决办法：
调用<T> T[] toArray(T[] contents) ， 而不是 Object[] toArray()，下面列举<T> T[] toArray(T[] contents)的调用方式：
```
// toArray(T[] contents)调用方式一
public static Integer[] vectorToArray1(ArrayList<Integer> v) {
    Integer[] newText = new Integer[v.size()];
    v.toArray(newText);
    return newText;
}
// toArray(T[] contents)调用方式二。最常用！
public static Integer[] vectorToArray2(ArrayList<Integer> v) {
    Integer[] newText = (Integer[])v.toArray(new Integer[0]);
    return newText;
}
// toArray(T[] contents)调用方式三
public static Integer[] vectorToArray3(ArrayList<Integer> v) {
    Integer[] newText = new Integer[v.size()];
    Integer[] newStrings = (Integer[])v.toArray(newText);
    return newStrings;
}
```

## Arraylist 代码示例
```
import java.util.*;

/*
 * @desc ArrayList常用API的测试程序
 * @author skywang 
 * @email kuiwu-wang@163.com
 */
public class ArrayListTest {

    public static void main(String[] args) {
        
        // 创建ArrayList
        ArrayList list = new ArrayList();

        // 将“”
        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        // 将下面的元素添加到第1个位置
        list.add(0, "5");

        // 获取第1个元素
        System.out.println("the first element is: "+ list.get(0));
        // 删除“3”
        list.remove("3");
        // 获取ArrayList的大小
        System.out.println("Arraylist size=: "+ list.size());
        // 判断list中是否包含"3"
        System.out.println("ArrayList contains 3 is: "+ list.contains(3));
        // 设置第2个元素为10
        list.set(1, "10");

        // 通过Iterator遍历ArrayList
        for(Iterator iter = list.iterator(); iter.hasNext(); ) {
            System.out.println("next is: "+ iter.next());
        }

        // 将ArrayList转换为数组
        String[] arr = (String[])list.toArray(new String[0]);
        for (String str:arr)
            System.out.println("str: "+ str);

        // 清空ArrayList
        list.clear();
        // 判断ArrayList是否为空
        System.out.println("ArrayList is empty: "+ list.isEmpty());
    }
}
```

## Fail-Fast机制
fail-fast 机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。
例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### fail-fast示例
```
import java.util.*;
import java.util.concurrent.*;

/*
 * @desc java集合中Fast-Fail的测试程序。
 *
 *   fast-fail事件产生的条件：当多个线程对Collection进行操作时，若其中某一个线程通过iterator去遍历集合时，该集合的内容被其他线程所改变；则会抛出ConcurrentModificationException异常。
 *   fast-fail解决办法：通过util.concurrent集合包下的相应类去处理，则不会产生fast-fail事件。
 *
 *   本例中，分别测试ArrayList和CopyOnWriteArrayList这两种情况。ArrayList会产生fast-fail事件，而CopyOnWriteArrayList不会产生fast-fail事件。
 *   (01) 使用ArrayList时，会产生fast-fail事件，抛出ConcurrentModificationException异常；定义如下：
 *            private static List<String> list = new ArrayList<String>();
 *   (02) 使用时CopyOnWriteArrayList，不会产生fast-fail事件；定义如下：
 *            private static List<String> list = new CopyOnWriteArrayList<String>();
 */
public class FastFailTest {

    private static List<String> list = new ArrayList<String>();
    //private static List<String> list = new CopyOnWriteArrayList<String>();
    public static void main(String[] args) {
    
        // 同时启动两个线程对list进行操作！
        new ThreadOne().start();
        new ThreadTwo().start();
    }

    private static void printAll() {
        System.out.println("");

        String value = null;
        Iterator iter = list.iterator();
        while(iter.hasNext()) {
            value = (String)iter.next();
            System.out.print(value+", ");
        }
    }

    /**
     * 向list中依次添加0,1,2,3,4,5，每添加一个数之后，就通过printAll()遍历整个list
     */
    private static class ThreadOne extends Thread {
        public void run() {
            int i = 0;
            while (i<6) {
                list.add(String.valueOf(i));
                printAll();
                i++;
            }
        }
    }

    /**
     * 向list中依次添加10,11,12,13,14,15，每添加一个数之后，就通过printAll()遍历整个list
     */
    private static class ThreadTwo extends Thread {
        public void run() {
            int i = 10;
            while (i<16) {
                list.add(String.valueOf(i));
                printAll();
                i++;
            }
        }
    }

}
```
运行结果：
运行该代码，抛出异常java.util.ConcurrentModificationException！即，产生fail-fast事件！

结果说明：
(01) FastFailTest中通过 new ThreadOne().start() 和 new ThreadTwo().start() 同时启动两个线程去操作list。
    ThreadOne线程：向list中依次添加0,1,2,3,4,5。每添加一个数之后，就通过printAll()遍历整个list。
    ThreadTwo线程：向list中依次添加10,11,12,13,14,15。每添加一个数之后，就通过printAll()遍历整个list。
(02) 当某一个线程遍历list的过程中，list的内容被另外一个线程所改变了；就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### fail-fast解决办法
fail-fast机制，是一种错误检测机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用“java.util.concurrent包下的类”去取代“java.util包下的类”。
所以，本例中只需要将ArrayList替换成java.util.concurrent包下对应的类即可。
即，将代码
`List<String> list = new ArrayList<String>();`
替换为
`List<String> cList = new CopyOnWriteArrayList<>();`
即可解决

### fail-fast原理
产生fail-fast事件，是通过抛出ConcurrentModificationException异常来触发的。那么，ArrayList是如何抛出ConcurrentModificationException异常的呢?
我们知道，ConcurrentModificationException是在操作Iterator时抛出的异常。通过查看Iterator的源码(ArrayList的Iterator是在父类AbstractList.java中实现的)
```
package java.util;

public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

    ...

    // AbstractList中唯一的属性
    // 用来记录List修改的次数：每修改一次(添加/删除等操作)，将modCount+1
    protected transient int modCount = 0;

    // 返回List对应迭代器。实际上，是返回Itr对象。
    public Iterator<E> iterator() {
        return new Itr();
    }

    // Itr是Iterator(迭代器)的实现类
    private class Itr implements Iterator<E> {
        int cursor = 0;

        int lastRet = -1;

        // 修改数的记录值。
        // 每次新建Itr()对象时，都会保存新建该对象时对应的modCount；
        // 以后每次遍历List中的元素的时候，都会比较expectedModCount和modCount是否相等；
        // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            // 获取下一个元素之前，都会判断“新建Itr对象时保存的modCount”和“当前的modCount”是否相等；
            // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
            checkForComodification();
            try {
                E next = get(cursor);
                lastRet = cursor++;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
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

    ...
}
```
从中，我们可以发现在调用 next() 和 remove()时，都会执行 checkForComodification()。若 “modCount 不等于 expectedModCount”，则抛出ConcurrentModificationException异常，产生fail-fast事件。
要搞明白 fail-fast机制，我们就要需要理解什么时候“modCount 不等于 expectedModCount”！
从Itr类中，我们知道 expectedModCount 在创建Itr对象时，被赋值为 modCount。通过Itr，我们知道：expectedModCount不可能被修改为不等于 modCount。所以，需要考证的就是modCount何时会被修改。
查看ArrayList的源码，来看看modCount是如何被修改的：
```
package java.util;

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

    ...

    // list中容量变化时，对应的同步函数
    public void ensureCapacity(int minCapacity) {
        modCount++;
        int oldCapacity = elementData.length;
        if (minCapacity > oldCapacity) {
            Object oldData[] = elementData;
            int newCapacity = (oldCapacity * 3)/2 + 1;
            if (newCapacity < minCapacity)
                newCapacity = minCapacity;
            // minCapacity is usually close to size, so this is a win:
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }


    // 添加元素到队列最后
    public boolean add(E e) {
        // 修改modCount
        ensureCapacity(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }


    // 添加元素到指定的位置
    public void add(int index, E element) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(
            "Index: "+index+", Size: "+size);

        // 修改modCount
        ensureCapacity(size+1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
             size - index);
        elementData[index] = element;
        size++;
    }

    // 添加集合
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        // 修改modCount
        ensureCapacity(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
   

    // 删除指定位置的元素 
    public E remove(int index) {
        RangeCheck(index);

        // 修改modCount
        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // Let gc do its work

        return oldValue;
    }


    // 快速删除指定位置的元素 
    private void fastRemove(int index) {

        // 修改modCount
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // Let gc do its work
    }

    // 清空集合
    public void clear() {
        // 修改modCount
        modCount++;

        // Let gc do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    ...
}
```
从中，我们发现：无论是add()、remove()，还是clear()，只要涉及到修改集合中的元素个数时，都会改变modCount的值。

接下来，我们再系统的梳理一下fail-fast是怎么产生的。步骤如下：
(01) 新建了一个ArrayList，名称为arrayList。
(02) 向arrayList中添加内容。
(03) 新建一个“线程a”，并在“线程a”中通过Iterator反复的读取arrayList的值。
(04) 新建一个“线程b”，在“线程b”中删除arrayList中的一个“节点A”。
(05) 这时，就会产生有趣的事件了。
       在某一时刻，“线程a”创建了arrayList的Iterator。此时“节点A”仍然存在于arrayList中，创建arrayList时，expectedModCount = modCount(假设它们此时的值为N)。
       在“线程a”在遍历arrayList过程中的某一时刻，“线程b”执行了，并且“线程b”删除了arrayList中的“节点A”。“线程b”执行remove()进行删除操作时，在remove()中执行了“modCount++”，此时modCount变成了N+1！
“线程a”接着遍历，当它执行到next()函数时，调用checkForComodification()比较“expectedModCount”和“modCount”的大小；而“expectedModCount=N”，“modCount=N+1”,这样，便抛出ConcurrentModificationException异常，产生fail-fast事件。

至此，我们就完全了解了fail-fast是如何产生的！
即，当多个线程对同一个集合进行操作的时候，某线程访问集合的过程中，该集合的内容被其他线程所改变(即其它线程通过add、remove、clear等方法，改变了modCount的值)；这时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### 解决fail-fast的原理
上面，说明了“解决fail-fast机制的办法”，也知道了“fail-fast产生的根本原因”。接下来，我们再进一步谈谈java.util.concurrent包中是如何解决fail-fast事件的。
还是以和ArrayList对应的CopyOnWriteArrayList进行说明。我们先看看CopyOnWriteArrayList的源码：
```
package java.util.concurrent;
import java.util.*;
import java.util.concurrent.locks.*;
import sun.misc.Unsafe;

public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    ...

    // 返回集合对应的迭代器
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }

    ...
   
    private static class COWIterator<E> implements ListIterator<E> {
        private final Object[] snapshot;

        private int cursor;

        private COWIterator(Object[] elements, int initialCursor) {
            cursor = initialCursor;
            // 新建COWIterator时，将集合中的元素保存到一个新的拷贝数组中。
            // 这样，当原始集合的数据改变，拷贝数据中的值也不会变化。
            snapshot = elements;
        }

        public boolean hasNext() {
            return cursor < snapshot.length;
        }

        public boolean hasPrevious() {
            return cursor > 0;
        }

        public E next() {
            if (! hasNext())
                throw new NoSuchElementException();
            return (E) snapshot[cursor++];
        }

        public E previous() {
            if (! hasPrevious())
                throw new NoSuchElementException();
            return (E) snapshot[--cursor];
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor-1;
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

        public void set(E e) {
            throw new UnsupportedOperationException();
        }

        public void add(E e) {
            throw new UnsupportedOperationException();
        }
    }
  
    ...

}
```
从中，我们可以看出:
(01) 和ArrayList继承于AbstractList不同，CopyOnWriteArrayList没有继承于AbstractList，它仅仅只是实现了List接口。
(02) ArrayList的iterator()函数返回的Iterator是在AbstractList中实现的；而CopyOnWriteArrayList是自己实现Iterator。
(03) ArrayList的Iterator实现类中调用next()时，会“调用checkForComodification()比较‘expectedModCount’和‘modCount’的大小”；但是，CopyOnWriteArrayList的Iterator实现类中，没有所谓的checkForComodification()，更不会抛出ConcurrentModificationException异常！ 

至此，关于Arraylist的内容全部总结完毕。

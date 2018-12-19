---
title: List总结
tags: collection
date: 2018-12-19 16:01:54
---

## List概括
List的框架图
![](List总结/1.jpg)

1.List是一个接口，继承于Collection接口，代表有序队列。
2.AbstractList是一个抽象类，继承于AbstractCollection。AbstractList实现List接口中除size()、get(int location)之外的函数。
3.AbstractSequentialList 是一个抽象类，它继承于AbstractList。AbstractSequentialList 实现了“链表中，根据index索引值操作链表的全部函数”。
4.ArrayList, LinkedList, Vector, Stack是List的4个实现类。
ArrayList 是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，随机插入、随机删除效率低。
LinkedList 是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率高。
Vector 是矢量队列，和ArrayList一样，它也是一个动态数组，由数组实现。但是ArrayList是非线程安全的，而Vector是线程安全的。
Stack 是栈，它继承于Vector。它的特性是：先进后出(FILO, First In Last Out)。

## 使用场景
如果涉及到“栈”、“队列”、“链表”等操作，应该考虑用List，具体的选择哪个List，根据下面的标准来取舍。
(01) 对于需要快速插入，删除元素，应该使用LinkedList。
(02) 对于需要快速随机访问元素，应该使用ArrayList。
(03) 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类(如ArrayList)。
       对于“多线程环境，且List可能同时被多个线程操作”，此时，应该使用同步的类(如Vector)。
	   
通过下面的测试程序，验证上面(01)(02)的结论：
```
import java.util.*;
import java.lang.Class;

/*
 * @desc 对比ArrayList和LinkedList的插入、随机读取效率、删除的效率
 *
 * @author skywang
 */
public class ListCompareTest {

    private static final int COUNT = 100000;

    private static LinkedList linkedList = new LinkedList();
    private static ArrayList arrayList = new ArrayList();
    private static Vector vector = new Vector();
    private static Stack stack = new Stack();

    public static void main(String[] args) {
        // 换行符
        System.out.println();
        // 插入
        insertByPosition(stack) ;
        insertByPosition(vector) ;
        insertByPosition(linkedList) ;
        insertByPosition(arrayList) ;

        // 换行符
        System.out.println();
        // 随机读取
        readByPosition(stack);
        readByPosition(vector);
        readByPosition(linkedList);
        readByPosition(arrayList);

        // 换行符
        System.out.println();
        // 删除 
        deleteByPosition(stack);
        deleteByPosition(vector);
        deleteByPosition(linkedList);
        deleteByPosition(arrayList);
    }

    // 获取list的名称
    private static String getListName(List list) {
        if (list instanceof LinkedList) {
            return "LinkedList";
        } else if (list instanceof ArrayList) {
            return "ArrayList";
        } else if (list instanceof Stack) {
            return "Stack";
        } else if (list instanceof Vector) {
            return "Vector";
        } else {
            return "List";
        }
    }

    // 向list的指定位置插入COUNT个元素，并统计时间
    private static void insertByPosition(List list) {
        long startTime = System.currentTimeMillis();

        // 向list的位置0插入COUNT个数
        for (int i=0; i<COUNT; i++)
            list.add(0, i);

        long endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println(getListName(list) + " : insert "+COUNT+" elements into the 1st position use time：" + interval+" ms");
    }

    // 从list的指定位置删除COUNT个元素，并统计时间
    private static void deleteByPosition(List list) {
        long startTime = System.currentTimeMillis();

        // 删除list第一个位置元素
        for (int i=0; i<COUNT; i++)
            list.remove(0);

        long endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println(getListName(list) + " : delete "+COUNT+" elements from the 1st position use time：" + interval+" ms");
    }

    // 根据position，不断从list中读取元素，并统计时间
    private static void readByPosition(List list) {
        long startTime = System.currentTimeMillis();

        // 读取list元素
        for (int i=0; i<COUNT; i++)
            list.get(i);

        long endTime = System.currentTimeMillis();
        long interval = endTime - startTime;
        System.out.println(getListName(list) + " : read "+COUNT+" elements by position use time：" + interval+" ms");
    }
}
```

运行结果如下：
```
Stack : insert 100000 elements into the 1st position use time：1640 ms
Vector : insert 100000 elements into the 1st position use time：1607 ms
LinkedList : insert 100000 elements into the 1st position use time：29 ms
ArrayList : insert 100000 elements into the 1st position use time：1617 ms

Stack : read 100000 elements by position use time：9 ms
Vector : read 100000 elements by position use time：6 ms
LinkedList : read 100000 elements by position use time：10809 ms
ArrayList : read 100000 elements by position use time：5 ms

Stack : delete 100000 elements from the 1st position use time：1916 ms
Vector : delete 100000 elements from the 1st position use time：1910 ms
LinkedList : delete 100000 elements from the 1st position use time：15 ms
ArrayList : delete 100000 elements from the 1st position use time：1909 ms
```

从中，我们可以发现：
插入10万个元素，LinkedList所花时间最短：29ms。
删除10万个元素，LinkedList所花时间最短：15ms。
遍历10万个元素，LinkedList所花时间最长：10809 ms；而ArrayList、Stack和Vector则相差不多，都只用了几秒。

考虑到Vector是支持同步的，而Stack又是继承于Vector的；因此，得出结论：
(01) 对于需要快速插入，删除元素，应该使用LinkedList。
(02) 对于需要快速随机访问元素，应该使用ArrayList。
(03) 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类。

## LinkedList和ArrayList性能差异分析
通过查看源码，看看为什么LinkedList插入元素快，ArrayList插入慢。
### LinkedList.java中向指定位置插入元素的代码如下：
```
// 在index前添加节点，且节点的值为element
public void add(int index, E element) {
    addBefore(element, (index==size ? header : entry(index)));
}

// 获取双向链表中指定位置的节点
private Entry<E> entry(int index) {
    if (index < 0 || index >= size)
        throw new IndexOutOfBoundsException("Index: "+index+
                                            ", Size: "+size);
    Entry<E> e = header;
    // 获取index处的节点。
    // 若index < 双向链表长度的1/2,则从前向后查找;
    // 否则，从后向前查找。
    if (index < (size >> 1)) {
        for (int i = 0; i <= index; i++)
            e = e.next;
    } else {
        for (int i = size; i > index; i--)
            e = e.previous;
    }
    return e;
}

// 将节点(节点数据是e)添加到entry节点之前。
private Entry<E> addBefore(E e, Entry<E> entry) {
    // 新建节点newEntry，将newEntry插入到节点e之前；并且设置newEntry的数据是e
    Entry<E> newEntry = new Entry<E>(e, entry, entry.previous);
    // 插入newEntry到链表中
    newEntry.previous.next = newEntry;
    newEntry.next.previous = newEntry;
    size++;
    modCount++;
    return newEntry;
}
```

从中，我们可以看出：通过add(int index, E element)向LinkedList插入元素时。先是在双向链表中找到要插入节点的位置index；找到之后，再插入一个新节点。
双向链表查找index位置的节点时，有一个加速动作：若index < 双向链表长度的1/2，则从前向后查找; 否则，从后向前查找。

### ArrayList.java中向指定位置插入元素的代码。如下：
```
// 将e添加到ArrayList的指定位置
public void add(int index, E element) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(
        "Index: "+index+", Size: "+size);

    ensureCapacity(size+1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
         size - index);
    elementData[index] = element;
    size++;
}
```
ensureCapacity(size+1) 的作用是“确认ArrayList的容量，若容量不够，则增加容量。”
真正耗时的操作是 System.arraycopy(elementData, index, elementData, index + 1, size - index);

Sun JDK包的java/lang/System.java中的arraycopy()声明如下：
`public static native void arraycopy(Object src, int srcPos, Object dest, int destPos, int length);`


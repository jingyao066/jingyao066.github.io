---
title: Iterator和Enumeration比较
tags: 集合
date: 2019-04-03 14:16:36
---

# Iterator和Enumeration区别
在Java集合中，我们通常都通过 “Iterator(迭代器)” 或 “Enumeration(枚举类)” 去遍历集合。现在看一下它们之间到底有什么区别。

Enumeration是一个接口，它的源码如下：
```
package java.util;

public interface Enumeration<E> {

    boolean hasMoreElements();

    E nextElement();
}
```
Iterator也是一个接口，它的源码如下：
```
package java.util;

public interface Iterator<E> {
    boolean hasNext();

    E next();

    void remove();
}
```
- 函数接口不同
Enumeration只有2个函数接口。通过Enumeration，我们只能读取集合的数据，而不能对数据进行修改。
Iterator只有3个函数接口。Iterator除了能读取集合的数据之外，也能数据进行删除操作。

- Iterator支持fail-fast机制，而Enumeration不支持。
Enumeration 是JDK 1.0添加的接口。使用到它的函数包括Vector、Hashtable等类，这些类都是JDK 1.0中加入的，Enumeration存在的目的就是为它们提供遍历接口。Enumeration本身并没有支持同步，而在Vector、Hashtable实现Enumeration时，添加了同步。
而Iterator 是JDK 1.2才添加的接口，它也是为了HashMap、ArrayList等集合提供遍历接口。Iterator是支持fail-fast机制的：当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

# Iterator和Enumeration实例
下面，我们编写一个Hashtable，然后分别通过 Iterator 和 Enumeration 去遍历它，比较它们的效率。代码如下：
```
import java.util.Enumeration;
import java.util.Hashtable;
import java.util.Iterator;
import java.util.Map.Entry;
import java.util.Random;

/*
 * 测试分别通过 Iterator 和 Enumeration 去遍历Hashtable
 * @author skywang
 */
public class IteratorEnumeration {

    public static void main(String[] args) {
        int val;
        Random r = new Random();
        Hashtable table = new Hashtable();
        for (int i=0; i<100000; i++) {
            // 随机获取一个[0,100)之间的数字
            val = r.nextInt(100);
            table.put(String.valueOf(i), val);
        }

        // 通过Iterator遍历Hashtable
        iterateHashtable(table) ;

        // 通过Enumeration遍历Hashtable
        enumHashtable(table);
    }
    
    /*
     * 通过Iterator遍历Hashtable
     */
    private static void iterateHashtable(Hashtable table) {
        long startTime = System.currentTimeMillis();

        Iterator iter = table.entrySet().iterator();
        while(iter.hasNext()) {
            //System.out.println("iter:"+iter.next());
            iter.next();
        }

        long endTime = System.currentTimeMillis();
        countTime(startTime, endTime);
    }
    
    /*
     * 通过Enumeration遍历Hashtable
     */
    private static void enumHashtable(Hashtable table) {
        long startTime = System.currentTimeMillis();

        Enumeration enu = table.elements();
        while(enu.hasMoreElements()) {
            //System.out.println("enu:"+enu.nextElement());
            enu.nextElement();
        }

        long endTime = System.currentTimeMillis();
        countTime(startTime, endTime);
    }

    private static void countTime(long start, long end) {
        System.out.println("time: "+(end-start)+"ms");
    }
}
```
运行结果如下：

time: 9ms
time: 5ms

从中，我们可以看出。Enumeration 比 Iterator 的遍历速度更快。为什么呢？
这是因为，Hashtable中Iterator是通过Enumeration去实现的，而且Iterator添加了对fail-fast机制的支持；所以，执行的操作自然要多一些。

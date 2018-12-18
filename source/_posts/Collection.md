---
title: Collection
tags: 
- java
- collection
date: 2018-12-06 18:21:41
---

## 集合(Collection)
Collection是一个接口，它主要的两个分支是：List 和 Set。
List和Set都是接口，它们继承于Collection。List是有序的队列，List中可以有重复的元素；而Set是数学概念中的集合，Set中没有重复元素！
List和Set都有它们各自的实现类。
为了方便，我们抽象出了AbstractCollection抽象类，它实现了Collection中的绝大部分函数；这样，在Collection的实现类中，我们就可以通过继承AbstractCollection省去重复编码。AbstractList和AbstractSet都继承于AbstractCollection，具体的List实现类继承于AbstractList，而Set的实现类则继承于AbstractSet。
另外，Collection中有一个iterator()函数，它的作用是返回一个Iterator接口。通常，我们通过Iterator迭代器来遍历集合。ListIterator是List接口所特有的，在List接口中，通过ListIterator()返回一个ListIterator对象。

为了方便对多个对象进行操作，集合是存储对象的最常用的一种方式
集合的出现就是为了存储对象，可以存储任意类型的对象，而且长度可变。
集合和数组的区别?
.都是容器
.数组长度固定，集合长度可变
.数组可以存储基本数据类型，集合只能存对象
.数组存储的数据类型是单一的，集合可以存储任意类型的对象

## Collection简介
定义如下：
`public interface Collection<E> extends Iterable<E> {}`
它是一个接口，是高度抽象出来的集合，它包含了集合的基本操作：添加、删除、清空、遍历(读取)、是否为空、获取大小、是否保护某元素等等。
Collection接口的所有子类(直接子类和间接子类)都必须实现2种构造函数：不带参数的构造函数 和 参数为Collection的构造函数。带参数的构造函数，可以用来转换Collection的类型。

### API
--略，请参考jdk中的源码

## List简介
定义如下：
`public interface List<E> extends Collection<E> {}`
List是一个继承于Collection的接口，即List是集合中的一种。List是有序的队列，List中的每一个元素都有一个索引；第一个元素的索引值是0，往后的元素的索引值依次+1。和Set不同，List中允许有重复的元素。

### API
既然List是继承于Collection接口，它自然就包含了Collection中的全部函数接口；由于List是有序队列，它也额外的有自己的API接口。主要有“添加、删除、获取、修改指定位置的元素”、“获取List中的子队列”等。

## Set简介
定义如下：
`public interface Set<E> extends Collection<E> {}`
Set是一个继承于Collection的接口，即Set也是集合中的一种。Set是没有重复元素的集合。

### API
Set的API和Collection完全一样。

## AbstractCollection
定义如下：
`public abstract class AbstractCollection<E> implements Collection<E> {}`
AbstractCollection是一个抽象类，它实现了Collection中除iterator()和size()之外的函数。
主要作用：
它实现了Collection接口中的大部分函数。从而方便其它类实现Collection，比如ArrayList、LinkedList等，它们这些类想要实现Collection接口，通过继承AbstractCollection就已经实现了大部分的接口了。

## AbstractList
定义如下：
`public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {}`
AbstractList是一个继承于AbstractCollection，并且实现List接口的抽象类。它实现了List中除size()、get(int location)之外的函数。
AbstractList的主要作用：它实现了List接口中的大部分函数。从而方便其它类继承List。
另外，和AbstractCollection相比，AbstractList抽象类中，实现了iterator()接口。

## AbstractSet
定义如下：
`public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {}`
AbstractSet是一个继承于AbstractCollection，并且实现Set接口的抽象类。由于Set接口和Collection接口中的API完全一样，Set也就没有自己单独的API。和AbstractCollection一样，它实现了List中除iterator()和size()之外的函数。
AbstractSet的主要作用：它实现了Set接口中的大部分函数。从而方便其它类实现Set接口。

## Iterator
`public interface Iterator<E> {}`

Iterator是一个接口，它是集合的迭代器。集合可以通过Iterator去遍历集合中的元素。Iterator提供的API接口，包括：
1.是否存在下一个元素 `boolean hasNext();    //每次next之前，先调用此方法探测是否迭代到终点`
2.获取下一个元素   `E next();            //返回当前迭代元素 ，同时，迭代游标后移`
3.删除当前元素 void remove();
```
/*删除最近一次已近迭代出出去的那个元素。
     只有当next执行完后，才能调用remove函数。
     比如你要删除第一个元素，不能直接调用 remove()   而要先next一下( );
     在没有先调用next 就调用remove方法是会抛出异常的。
     这个和MySQL中的ResultSet很类似
    */
    void remove() 
    {
        throw new UnsupportedOperationException("remove");
    }
```
注意：Iterator遍历Collection时，是fail-fast机制的。即，当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。关于fail-fast的详细内容，我们会在后面专门进行说明。

实现了Iterable接口，就可以调用iterator()方法。
HashMap类就实现了Iterable接口，遍历map时，
```
Iterator iter = hashMap.iterator(); 
while(iter.hashNext()) { 
  String s = iter.next(); 
} 
```
### 手动迭代list，和foreach原理是一样的
```
 List<Integer> li = new ArrayList<>();
	li.add(1);
	li.add(2);
	li.add(3);
	//不使用foreach 而手动迭代
	Iterator<Integer> iter = li.iterator();    //获取ArrayList 的迭代器
	while(iter.hasNext())                      //①先探测能否继续迭代
	{
		System.out.println(iter.next());       //②后取出本次迭代出的元素
		//invoke  remove()                     //③最后如果需要，调用remove
	}
```

### AbstractList中实现的迭代器类解读：
```
private class Itr implements Iterator<E> {
    //AbstractList 中实现的迭代器，删除了一些细节。不影响理解,Itr为一个priavate成员内部类
    int cursor = 0;   //马上等待被迭代元素的index
    //最近一次，已经被迭代出的元素的index，如果这个元素迭代后，被删除了，则lastRet重置为-1
    int lastRet = -1;
    public boolean hasNext() {
        return cursor != size();      //当前游标值 等于  集合的size()  说明已经不能再迭代了。
    }
    public E next() {
        int i = cursor;
        E next = get(i);
        lastRet = i;     //lastRet 保存的是最近一次已经被迭代出去的元素索引
        cursor = i + 1;  //cursor为马上等待被迭代的元素的索引
        return next;

    }
    public void remove() {
        if (lastRet < 0)                             //调用remove之前没有调用next
            throw new IllegalStateException();       //则抛异常。这就是为什么在使用remove前，要next的原因
        OuterList.this.remove(lastRet);            //从集合中删除这个元素
        if (lastRet < cursor)                      //集合删除元素后，集合后面的元素的索引会都减小1，cursor也要同步后移
            cursor--;
        lastRet = -1;                                 //重置
    }
}
```
迭代出来的元素都是原来集合元素的拷贝
Java集合中保存的元素实质是对象的引用(可以理解为C中的指针)，而非对象本身。
迭代出的元素也就都是 引用的拷贝，结果还是引用。那么，如果集合中保存的元素是可变类型的，我们就可以通过迭代出的元素修改原集合中的对象。
而对于不可变类型，如String  基本元素的包装类型Integer 都是则不会反应到原集合中。

验证：
```
public class Main {
        public static void main(String[] args) {
            List<Person> li = new ArrayList<>();
            Person p = new Person("Tom");
            li.add(p);
            for (Person ap : li) {
                ap.setName("Jerry");
            }
            System.out.println(li.get(0).getName());     //Jerry  not Tom
        }
    }
    class Person {
        public Person(String name) {
            this.name = (name == null ? "" : name);

        }
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            if (name == null)
                name = "";
            this.name = name;
        }
    }
```
### 自己实现迭代器
```
public class Main {
        public static void main(String[] args) {
            MyString s = new MyString("1234567");
            for (char c : s) {
                System.out.println(c);
            }
        }
    }
    class MyString implements Iterable<Character> {
        private int length = 0;
        private String ineers = null;
        public MyString(String s) {
            this.ineers = s;
            this.length = s.length();
        }
        @Override
        public Iterator<Character> iterator() {
            class iter implements Iterator<Character>     //方法内部类
            {
                private int cur = 0;
                @Override
                public boolean hasNext() {
                    return cur != length;
                }
                @Override
                public Character next() {
                    Character c = ineers.charAt(cur);
                    cur++;
                    return c;
                }
                public void remove() {
                    // do nothing
                }
            }
            return new iter();     //安装Iterable接口的约定，返回迭代器
        }

    }
```


## Iterable
`public interface Iterable<T>`
Iterable是一个接口，Collection接口继承了Iterable(接口扩展)，Iterable中包含
1.iterator 方法的显示调用，
2.forEach方法，
3.spliterator方法

## Iterator和Iterable 区别
从英文意思去理解：
Iterable ：故名思议，实现了这个接口的集合对象支持迭代，是可迭代的。able结尾的表示 能...样，可以做...。
Iterator:   在英语中or 结尾是都是表示 ...样的人 or ... 者。如creator就是创作者的意思。这里也是一样：iterator就是迭代者，我们一般叫迭代器，它就是提供迭代机制的对象，具体如何迭代，都是Iterator接口规范的。

### Iterable
一个集合对象要表明自己支持迭代，能有使用foreach语句的特权，就必须实现Iterable接口，表明我是可迭代的，然而实现Iterable接口，就必需为foreach语句提供一个迭代器。
这个迭代器是用接口定义的 iterator方法提供的。也就是iterator方法需要返回一个Iterator对象。

### Iterator
1.每次在迭代前   ，先调用hasNext()探测是否迭代到终点（本次还能再迭代吗？）。
2.next方法不仅要返回当前元素，还要后移游标cursor
3.remove()方法用来删除最近一次已经迭代出的元素
4.迭代出的元素是原集合中元素的拷贝（重要）
5.配合foreach使用

为什么一定要实现Iterable接口，为什么不直接实现Iterator接口呢？
答：看一下JDK中的集合类，比如List一族或者Set一族，都是实现了Iterable接口，但并不直接实现Iterator接口。 仔细想一下这么做是有道理的。 
因为Iterator接口的核心方法next()或者hasNext() 是依赖于迭代器的当前迭代位置的。 如果Collection直接实现Iterator接口，势必导致集合对象中包含当前迭代位置的数据(指针)。 
当集合在不同方法间被传递时，由于当前迭代位置不可预置，那么next()方法的结果会变成不可预知。  除非再为Iterator接口添加一个reset()方法，用来重置当前迭代位置。 
但即时这样，Collection也只能同时存在一个当前迭代位置。 而Iterable则不然，每次调用都会返回一个从头开始计数的迭代器。 多个迭代器是互不干扰的。 

## ListIterator
`public interface ListIterator<E> extends Iterator<E> {}`
ListIterator是一个继承于Iterator的接口，它是队列迭代器。专门用于便利List，能提供向前/向后遍历。相比于Iterator，它新增了添加、是否存在上一个元素、获取上一个元素等等API接口。

### API
```
// ListIterator的API
// 继承于Iterator的接口
abstract boolean hasNext()
abstract E next()
abstract void remove()
// 新增API接口
abstract void add(E object)
abstract boolean hasPrevious()
abstract int nextIndex()
abstract E previous()
abstract int previousIndex()
abstract void set(E object)
```

## 集合的三个特点：
1.用于存储对象
2.长度可变
3.可以存储不同类型的对象
```
---|Collection:(包含list和set子接口)
	---|List: 有存储顺序, 可重复
		---|ArrayList：数组实现, 查找快, 增删慢
			由于是数组实现, 在增和删的时候会牵扯到数组
			增容, 以及拷贝元素. 所以慢。数组是可以直接
			按索引查找, 所以查找时较快
		---|LinkedList：链表实现, 增删快, 查找慢
			由于链表实现, 增加时只要让前一个元素记住自
			己就可以, 删除时让前一个元素记住后一个元
			素, 后一个元素记住前一个元素. 这样的增删效
			率较高但查询时需要一个一个的遍历, 所以效率
			较低
		---|Vector：和ArrayList原理相同, 但线程安全, 效率略低
				
	---|Set: 无存储顺序, 不可重复
		---|HashSet:线程不安全，存取速度快，底层以哈希表实现
		---|TreeSet:红黑二叉树结构，默认对元素进行自然排序，
		---|LinkedHashSet:会保存插入的顺序

---| Map:键值对(不允许重复的键)null可以作为键，这样的键只有一个
	注:map和collection没有关系
		---|HashMap:根据键的hashCode值存储数据，根据键可以直接获取它的值，查询快(存入的值在取的时候也是随机的)
		---|TreeMap:取出来后是排序后的键值对(如果需要按照自然排序或自定义排序,用这个)
		---|HashTable:和hashMap实现原理一样,区别在于线程安全
		---|LinkedHashMap:输出的顺序和输入的顺序是相同的
		---|ConcurrentHashMap：采用了分段锁的设计，并发map
```

## 问题
1.list和set的区别
list允许重复对象，set不允许重复对象
list是有序容器，保留了每个元素的插入顺序，输出顺序就是插入顺序。
set是无序容器，无法保证每个元素的存储顺序.
list可以插入多个null元素，set只允许一个null元素.


什么时候该使用什么样的集合
Collection	我们需要保存若干个对象的时候使用集合。
List
如果我们需要保留存储顺序, 并且保留重复元素, 使用List.
	如果查询较多, 那么使用ArrayList
	如果存取较多, 那么使用LinkedList
	如果需要线程安全, 那么使用Vector


Set
如果我们不需要保留存储顺序, 并且需要去掉重复元素, 使用Set.
	如果我们需要将元素排序, 那么使用TreeSet
	如果我们不需要排序, 使用HashSet, HashSet比TreeSet效率高.
	如果我们需要保留存储顺序, 又要过滤重复元素, 那么使用LinkedHashSet

看到array，就要想到角标。
看到link，就要想到first，last。
看到hash，就要想到hashCode,equals.
看到tree，就要想到两个接口。Comparable，Comparator。
		

集合方法
Collection接口的共性方法
1.add():将指定对象存储到容器中
  addAll():直接添加一整个集合 

2.remove():将指定对象从集合中删除
  removeAll():直接删除一个集合

3.clear():清空集合中的所有元素

4.contains():判断集合中是否包含指定对象

5.collection.size():返回集合的大小

6.toArray():集合转换成数组


List特有的方法：
1.增加：List.add(1,"指定元素");在指定位置添加元素
2.删除：List.remove(1,"指定元素");删除指定位置的元素
3.修改：List.set(1,"指定元素");修改指定位置的元素
4.查找：List.get(1,"指定元素");查找指定位置的元素
5.求子集合:List<E> subList(int fromIndex, int toIndex) // 不包含toIndex


LinkedList特有方法：
1.addFirst():在集合的第一个位置添加元素
  addLast(): 最后
2.getFirst():返回此集合的第一个元素
  getLast():最后	
3.removeFirst():移除集合的第一个元素
  removeLast():最后

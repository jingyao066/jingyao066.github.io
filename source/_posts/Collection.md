---
title: Collection
tags: 
- java
- collection
date: 2018-12-06 18:21:41
---

# 集合(Collection)
为了方便对多个对象进行操作，集合是存储对象的最常用的一种方式
集合的出现就是为了存储对象，可以存储任意类型的对象，而且长度可变。
集合和数组的区别?
.都是容器
.数组长度固定，集合长度可变
.数组可以存储基本数据类型，集合只能存对象
.数组存储的数据类型是单一的，集合可以存储任意类型的对象

#### 集合的三个特点：
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


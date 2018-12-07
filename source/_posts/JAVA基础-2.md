---
title: JAVA基础-2
tags: java
date: 2018-12-06 18:20:53
---

### 1.什么是Java虚拟机？为什么Java被称作是“平台无关的编程语言”？
Java虚拟机是一个可以执行Java字节码的虚拟机进程。Java源文件被编译成能被Java虚拟机执行的字节码文件。
Java被设计成允许应用程序可以运行在任意的平台，而不需要程序员为每一个平台单独重写或者是重新编译。
Java虚拟机让这个变为可能，因为它知道底层硬件平台的指令长度和其他特性。

### 2.JDK和JRE的区别是什么？
JRE:Java运行时环境(JRE)是将要执行Java程序的Java虚拟机。它同时也包含了执行applet需要的浏览器插件。
JDK:Java开发工具包(JDK)是完整的Java软件开发包，包含了JRE，编译器和其他的工具(比如：JavaDoc，Java调试器)，可以让开发者开发、编译、执行Java应用程序。

### 3.是否可以在static环境中访问非static变量？(不可以)
static变量在Java中是属于类的，它在所有的实例中的值是一样的。当类被Java虚拟机载入的时候，会对static变量进行初始化。
如果你的代码尝试不用实例来访问非static的变量，编译器会报错，因为这些变量还没有被创建出来，还没有跟任何实例关联上。

### 4.Java支持的数据类型有哪些？什么是自动拆装箱？
Java语言支持的8中基本数据类型是：
byte,short,int,long,float,double,boolean,char
自动装箱是Java编译器在基本数据类型和对应的对象包装类型之间做的一个转化。
Integer  a=1;//这就是一个自动装箱，如果没有自动装箱的话，需要这样Integer  a=new Integer(1)
int b=a;//这就是一个自动拆箱，如果没有自动拆箱的话，需要这样：int b=a.intValue()
自动装箱和自动拆箱是简化了基本数据类型和相对应对象的转化步骤

### 5.什么是值传递和引用传递？
对象被值传递，意味着传递了对象的一个副本。因此，就算是改变了对象副本，也不会影响源对象的值。
对象被引用传递，意味着传递的并不是实际的对象，而是对象的引用。因此，外部对引用对象所做的改变会反映到所有的对象上。

### 6.进程和线程的区别是什么？
进程是执行着的应用程序，而线程是进程内部的一个执行序列
一个进程可以有多个线程。线程又叫做轻量级进程。

### 7.什么是死锁(deadlock)？
两个进程都在等待对方执行完毕才能继续往下执行的时候就发生了死锁。结果就是两个进程都陷入了无限的等待中。

### 8.如何确保N个线程可以访问N个资源同时又不导致死锁？
使用多线程的时候，一种非常简单的避免死锁的方式就是：指定获取锁的顺序，并强制线程按照指定的顺序获取锁。因此，如果所有的线程都是以同样的顺序加锁和释放锁，就不会出现死锁了。

### 9.什么是迭代器(Iterator)？
Iterator接口提供了很多对集合元素进行迭代的方法。每一个集合类都包含了可以返回迭代器实例的迭代方法。迭代器可以在迭代的过程中删除底层集合的元素。

### 10.Iterator和ListIterator的区别是什么？
Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。
Iterator对集合只能是前向遍历，ListIterator既可以前向也可以后向。
ListIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。

### 11.Java中的HashMap的工作原理是什么？
Java中的HashMap是以键值对(key-value)的形式存储元素的。
HashMap需要一个hash函数，它使用hashCode()和equals()方法来向集合/从集合添加和检索元素。
当调用put()方法的时候，HashMap会计算key的hash值，然后把键值对存储在集合中合适的索引上。
如果key已经存在了，value会被更新成新值。
HashMap的一些重要的特性是它的容量(capacity)，负载因子(load factor)和扩容极限(threshold resizing)。

#### 12.Java中HashMap遍历的四种方式
1.entrySet().iterator()
效率高,以后一定要使用此种方式！
```
Map map = new HashMap();
　　Iterator iter = map.entrySet().iterator();
　　while (iter.hasNext()) {
　　Map.Entry entry = (Map.Entry) iter.next();
　　Object key = entry.getKey();
　　Object val = entry.getValue();
　　}
```
2.keySet().iterator()
效率低,以后尽量少使用！
```
Map map = new HashMap();
　　Iterator iter = map.keySet().iterator();
　　while (iter.hasNext()) {
　　Object key = iter.next();
　　Object val = map.get(key);
　　}
```
3.entrySet遍历key和value
```
for(Map.Entry<String, String> entry: map.entrySet()){
    Object key = entry.getKey();
　Object val = entry.getValue();
}
```

### 13.数组(Array)和列表(ArrayList)有什么区别？什么时候应该使用Array而不是ArrayList？
Array可以包含基本类型和对象类型，ArrayList只能包含对象类型。
Array大小是固定的，ArrayList的大小是动态变化的。

### 14.ArrayList和LinkedList有什么区别？
ArrayList和LinkedList都实现了List接口，他们有以下的不同点：
1.ArrayList数组实现，增删慢(增删元素时，需要重新计算大小或更新数组索引)，查找快
LinkedList链表实现，增删快，查找慢。
2.LinkedList比ArrayList更占内存，因为LinkedList为每一个节点存储了两个引用，一个指向前一个元素，一个指向下一个元素。

### 15.System.gc()和Runtime.gc()会做什么事情？
这两个方法用来提示JVM要进行垃圾回收。但是，立即开始还是延迟进行垃圾回收是取决于JVM的。

### 16.堆栈区别？
1.栈区:编译器自动分配,存放函数的参数值，局部变量等
2.2.堆区:手动分配释放

### 17.什么是线程安全和不安全？
1.线程安全:a,b线程同时操作一个变量,a操作的时候，b不能操作,相当于单线程
2.线程不安全:a,b线程同时操作一个变量,可同时操作

### 18.同步和异步？
同步:发送一个请求,等待返回结果，然后再发送下一个请求
特点:需要等待,避免出现死锁，脏读数据的发生
异步:发送一个请求,不等待返回,随时可以再发送下一个请求
特点:无需等待,提高效率，但是会出现死锁，脏读数据

### 19.九种基本数据类型的大小，以及他们的封装类(一个字节是8位)

基本类型|大小(字节)|默认值|封装类
--|--|--|--
byte | 1 | (byte)0 | Byte
short  |  2  |  (short)0  |  Short
int  |  4  |  0  |  Integer
long  |  8  |  0L  |  Long
float  |  4  |  0.0f  |  Float
double  |  8  |  0.0d  |  Double
boolean  |  -  |  false  |  Boolean
char  |  2  |  \u0000(null)  |  Character
void	  |  -	  |  -  |  Void

### 20.equals、==、hashcode  区别
1.基本数据类型
byte/int/double/boolean等，用==，比较的是他们的值
2.引用类型(对象)
用==比较的是内存地址，同一个new出来的对象，为true，反之为false
(对象是存放在堆中的，栈中存放的是对象的引用(地址),由此可见==比较的是栈中的值
如果要比较堆中的对象是否相同，要重写equals方法)

equals:比较对象的内容(就是最表面的值)

```
String a = "1";
String b = "1";
String c = new String("1");
System.out.println(a==b);//true,是一块内存地址
System.out.println(a==c);//false,不是一块内存地址
System.out.println(a.equals(b));//true,比较的是值
```

### 21.Java的四种引用，强弱软虚，用到的场景	
1.强引用:如果一个对象具有强引用，那垃圾回收器绝不会回收它
Object o=new Object();   //  强引用
2.软引用:内存够，不会被回收(垃圾回收器),反之会被回收.
引用可用来实现内存敏感的高速缓存。
3.弱引用:垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存
4.虚引用：任何时候都可能被垃圾回收器回收

### 22.Hashcode的作用
1.生成一个hash码,表示对象存储的物理地址
2.可以用来用来鉴定2个对象是否相等

### 23.String、StringBuffer与StringBuilder的区别。
string:字符串常量，不可改变的对象
StringBuffer:字符串变量,可变对象,线程安全，效率低
StringBuilder:字符串变量,可变对象,线程不安全，效率高

### 24.HashMap和HashTable的区别
HashMap:异步的，线程不安全
HashTable:同步的，线程安全

### 25.token
 一、我们先解释一下他的含义：
1.token的引入
token是客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，token便应运而生
2、Token的定义：Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。
3、使用Token的目的：Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮。
二、如何使用Token？
1、用设备号/设备mac地址作为Token（推荐）
客户端：客户端在登录的时候获取设备的设备号/mac地址，并将其作为参数传递到服务端。
 服务端：服务端接收到该参数后，便用一个变量来接收同时将其作为Token保存在数据库，并将该Token设置到session中，客户端每次请求的时候都要统一拦截，并将客户端传递的token和服务器端session中的token进行对比，如果相同则放行，不同则拒绝。
分析：此刻客户端和服务器端就统一了一个唯一的标识Token，而且保证了每一个设备拥有了一个唯一的会话。
该方法的缺点是客户端需要带设备号/mac地址作为参数传递，而且服务器端还需要保存；优点是客户端不需重新登录，只要登录一次以后一直可以使用，至于超时的问题是有服务器这边来处理，如何处理？若服务器的Token超时后，服务器只需将客户端传递的Token向数据库中查询，同时并赋值给变量Token，如此，Token的超时又重新计时。
2、用session值作为Token
客户端：客户端只需携带用户名和密码登陆即可。
客户端：客户端接收到用户名和密码后并判断，如果正确了就将本地获取sessionID作为Token返回给客户端，客户端以后只需带上请求数据即可。
分析：这种方式使用的好处是方便，不用存储数据，但是缺点就是当session过期后，客户端必须重新登录才能进行访问数据。
三、使用过程中出现的问题以及解决方案？
刚才我们轻松介绍了Token的两种使用方式，但是在使用过程中我们还出现各种问题，Token第一种方法中我们隐藏了一个在网络不好或者并发请求时会导致多次重复提交数据的问题。
    
该问题的解决方案：将session和Token套用，如此便可解决，如何套用呢？请看这段解释：
![](https://github.com/ayanamiq/images/blob/master/token.png?raw=true)
这就是解决重复提交的方案。

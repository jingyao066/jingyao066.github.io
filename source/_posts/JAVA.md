---
title: JAVA
tags: java
date: 2018-12-06 18:18:52
---

this：表示当前函数，所属对象的引用
super：用于调用父类方法

# 概念
## static
static可修饰属性、方法和代码块
static 修饰属性：是一个类的共享变量，属于整个类，在不实例化对象的情况下就能访问。
static 修饰方法：不需要实例化对象，可以使用类名调用，静态方法不能访问非静态成员，包括成员的方法和变量。因为此时用类调用，没有对象的概念，this是不可用的。
static 修饰代码块：一般静态代码块用来初始化静态成员。
因为静态优先于对象存在，静态方法不可以出现this。
非静态函数中可以访问静态成员变量

static关键字表明一个成员变量或者是成员方法可以在没有所属的类的实例变量的情况下被访问。
公有puclic：公有变量，外部也可以调用的变量
私有private：私有变量,只能在对象内部调用，外部调用需要设置get()，set()方法，get用于获取对象的属性，set用于设置对象的属性

## final
final修饰符可修饰类、属性、方法
final 修饰类：此类不可被继承，即final没有子类，这样可以保证用户调用时动作的一致性，防止子类覆盖情况的发生
final 修饰变量：此时的属性为常量，常量的地址不可变，但在地址中保存的值(即对象的属性)，是可以改变的
final 修饰方法：此方法不可覆盖

问题：final、finally、finalize的区别
- final，修饰符(关键字)，如果一个类被声明为final，意味着它不能再派生出新的子类，不能作为父类被继承。
- finally是在异常处理时提供finally块来执行任何清除操作，不管有没有异常被抛出、捕获，finally块都会被执行。
- finalize方法来自于java.lang.Object，用于回收资源。在实际应用中，不要依赖使用该方法回收任何短缺的资源，这是因为很难知道这个方法什么时候被调用。

## 函数的重载
重载是同一个类中，有两个及以上的方法，拥有相同的方法名，但是参数不同，方法体也不同。
系统会根据传入的参数类型和参数个数来判断要用到的方法。

## 函数的重写
重写指的是子类方法覆盖父类方法，要求方法名和参数都相同，但修饰符和返回值可能不同，构造方法无法覆盖

## 构造方法
构造方法主要作用是完成对象的初始化工作，能够把定义对象时的参数传递给对象的域
-  构造方法名称必须与类名相同
- 没有返回类型，也不能定义void，方法前不声明方法类型
- 一个类可以定义多个构造方法，如在定义类时没有定义构造方法，系统会自动插入一个无参数的默认构造器，这个构造器不执行任何代码
- 构造方法可以重载

## 封装
把数据和操作数据的方法绑定起来，对数据的访问只能通过已定义的接口。
封装就是隐藏一切可隐藏的东西，只向外界提供最简单的编程接口。

java中提供四个修饰符来进行对象属性和方法的访问权限控制：
private ：类内部可见
default ：包内部可见
protected ：包内部或子类可见
public ：所有可见

好处:
- 通过隐藏对象的属性来保护对象内部的状态。
- 提高了代码的可用性和可维护性，因为对象的行为可以被单独的改变或者是扩展。
- 禁止对象之间的不良交，互提高模块化。

## 继承
继承是从已有类得到继承信息创建新类的过程。
提供继承信息的类被称为父类，得到继承信息的类被称为子类。

- 父类非私有化的属性和方法可以继承到子类
- 父类的构造方法子类不可以继承，更不存在覆盖
- java中不允许多继承，一个类只有一个父类
- super()表示调用父类的构造方法，super()和this()一样，必须放在第一行。
- this()用于调用本类的构造方法

## 多态
多态性是指允许不同子类型的对象对同一消息作出不同的响应。
多态分为：编译时多态和运行时多态。

- 编译时多态：编译时动态加载(方法的重载)。
- 运行时多态：一个对象可以有多个类型(子类重写父类方法)，父类引用指向子类对象。

运行时的多态是面向对象最精髓的东西，要实现多态需要做两件事
- 方法重写（子类继承父类并重写父类中已有的或抽象的方法）
- 父类引用指向子类对象，这样同样的引用调用同样的方法就会根据子类对象的不同而表现出不同的行为

注意：
- 如果子类中重写了父类中的方法，那么在调用这个方法的时候，将会调用子类中的这个方法
- 变量不能被重写(覆盖)，重写只针对方法，如果在子类中重写了父类中的变量，那么编译会报错
- 如果继承的子类继承父类的一个方法后加以重载，则该父类引用不能使用重载后的那个方法

体现：父类引用指向子类对象
前提：要有继承，要有方法重写，父类引用指向子类对象
好处：把不同的子类对象都当做父类来看，可以屏蔽不同子类对象之间的差异，写出通用的代码。

例: 游戏中有不同的角色，它们都有一个父类，它们做相同动作时表现出来的效果就会不一样，比如跑，魔法师的跑跟战士的跑就不会一样，这就是俩者都覆盖了父类中的跑方法，各自有自己的现实，表现出来多态。如果有一天你想再加个角色，只用再写一个类继承该父类，覆盖其中的跑方法就行了，其他代码不用怎么改，所以可维护性也很好。

## 接口、抽象类区别:
- 方法实现：
抽象类可以有默认的方法实现，接口完全是抽象的。它根本不存在方法的实现

- 构造方法：
抽象类可以有构造器，接口不能有构造方法。

- 多继承：
子类只能继承一个抽象类(java单继承，子类->(继承extend)抽象类)，
子类可以实现多个接口(子类->(实现implements)接口)

- 声明: 
接口：`public inerface Person`
抽象类：`abstract class Person`

- 访问修饰符：
抽象方法可以有public/default等修饰符，接口方法默认修饰符是public，不能用其他修饰

- 实现：
抽象类：子类使用extends关键字来**继承**抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明方法的实现，反之可以按照需要实现方法。
接口：子类使用关键字implements来**实现**接口，它需要提供接口中所有声明的方法的实现。

2018/12/18 update：
今天在读到Collection的源码时，发现Collection接口继承(extends)了Iterable接口，按照我以前的理解，接口(interface)只能被实现(implements)，不能被继承(extends)。
而实际上，java有个说法叫：
接口对接口的扩展，严格上不叫继承，即`interface extends interface`。
但是普通的java类是绝对不可以继承(extends)一个接口的。

## 内部类
内部类是一个编译时的概念，一旦编译成功，就会成为完全不同的两类。
内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。
内部类的分类：
- 成员内部类：作为外部类的一个成员存在，与外部类的属性、方法并列
- 局部内部类：在方法中定义的内部类
- 静态内部类：静态内部类只能访问外部类的静态成员
- 匿名内部类：通过匿名接口实现，是一种特殊的局部内部类

注：所有使用内部类的地方都可以不用内部类，使用内部类可以使程序更加的简洁，便于命名规范和划分层次结构。

## 正向和反向代理
正向：代理服务器在客户端(client)那边就是正向代理，正向代理需要客户端配置，客户端是知道自己通过代理方式去访问服务器的，例如：vpn，ssr

反向：代理服务器在原始服务器(server)那边就是反向代理，反向代理，客户端根本就不知道，是服务端配置的。例如：nginx

## session和cookie的区别
- session保存在服务器，客户端不知道其中信息；cookie保存在客户端，服务器可以知道其中信息
- session保存的是对象，cookie保存的是字符串
- cookie不安全，考虑安全应当用session
- 访问量增多会占用服务器性能，考虑到这应当用cookie

## session的分布式处理
https://www.cnblogs.com/newP/p/6518918.html
1.粘性session：
负载均衡器设置了粘性Session，用户的每次请求都会转发到同一台服务器上，
相当于把用户和服务器黏在了一起
缺点：容错性低,如果当前服务器故障，用户被转移到另一台服务器上，session信息会失效
实现：nginx通过ip_hash属性即可实现粘性session

2.服务器session复制：
任何一个服务器上的session发生改变，该节点会把这个session的所有内容序列化，
然后广播给其他节点，不管其他服务器是否需要，以此来实现session同步
缺点：降低服务器性能，对网络造成压力
实现：tomcat中，server.xml开启tomcat集群功能

3.redis实现session共享
4.session入库

## jdbc基本流程
1. 加载数据库驱动
2. 获取数据库连接
3. 创建sql对象
4. 执行sql语句
5. 处理ResultSet
6. 释放资源

## 反射
首先我们了解一下JVM，什么是JVM，Java的虚拟机，java之所以能跨平台就是因为这个东西，
你可以理解成一个进程，程序，只不过他的作用是用来跑你的代码的。
假如你写了一段代码：Object o=new Object();  运行了起来！

1.首先JVM会启动，你的代码会编译成一个.class文件
2.然后被类加载器加载进jvm的内存中，你的类Object加载到方法区中，创建了Object类的class对象到堆中，注意这个不是new出来的对象，而是类的类型对象，每个类只有一个class对象，作为方法区类的数据结构的接口。

jvm创建对象前，会先检查类是否加载，寻找类对应的class对象，若加载好，则为你的对象分配内存，
初始化也就是代码:new Object()。
上面的流程就是你自己写好的代码扔给jvm去跑，跑完就over了，jvm关闭，你的程序也停止了。
为什么要讲这个呢？因为要理解反射必须知道它在什么场景下使用。

想想上面的程序对象是自己new的，程序相当于写死了给jvm去跑。
假如一个服务器上突然遇到某个请求要用到某个类，
但没加载进jvm，是不是要停下来自己写段代码，new一下，再启动一下服务器？错~~

反射是什么呢？当我们的程序在运行时，
需要动态的加载一些类这些类可能之前用不到所以不用加载到jvm。

举个例子我们的项目底层有时是用mysql，有时用oracle，
需要动态地根据实际情况加载驱动类，这个时候反射就有用了，
假设 com.java.dbtest.myqlConnection，com.java.dbtest.oracleConnection
这两个类我们要用，这时候我们的程序就写得比较动态化，
通过Class tc = Class.forName("com.java.dbtest.TestConnection");
通过类的全类名让jvm在服务器中找到并加载这个类，
而如果是oracle则传入的参数就变成另一个了。
这时候就可以看到反射的好处了，这个动态性就体现出java的特性了！
当然这里只是举了反射的一个应用，实际还有其他作用，只是这个例子能更好地理解！

一.概念
主要是指程序可以访问，检测和修改它本身状态或行为的一种能力，
并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。

程序中一般的对象的类型都是在编译期就确定下来的，而Java反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。
所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。
反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁

Java反射框架主要提供以下功能：

1.在运行时判断任意一个对象所属的类；
2.在运行时构造任意一个类的对象；
3.在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
4.在运行时调用任意一个对象的方法
重点：是运行时而不是编译时

当我们在使用IDE(如Eclipse，IDEA)时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。
很多框架（比如Spring）都是配置化的（比如通过XML文件配置JavaBean,Action之类的），
为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象.

二，反射机制的作用：
    1.反编译：.class-->.java
    2.通过反射机制访问java对象的属性，方法，构造方法等；


三，在这里先看一下sun为我们提供了那些反射机制中的类：
	java.lang.Class;
	java.lang.reflect.Constructor; 
	java.lang.reflect.Field;
	java.lang.reflect.Method;
	java.lang.reflect.Modifier;
	很多反射中的方法，属性等操作我们可以从这四个类中查询。
	
四.具体功能实现：
```java
1.反射机制获取类有三种方法，我们来获取Employee类型
//第一种方式：  (注意要包名+类名)
Class c1 = Class.forName("demo.Employee");

//第二种方式：
//java中每个类型都有class属性.
Class c2 = Employee.class;

//第三种方式：
//java语言中任何一个java对象都有getClass 方法
Employee e = new Employee();
Class c3 = e.getClass(); //c3是运行时类 (e的运行时类是Employee)
    
2.创建对象：获取类以后我们来创建它的对象，利用newInstance：
Class c =Class.forName("demo.Employee");

//创建此Class 对象所表示的类的一个新实例
Object o = c.newInstance(); //调用了Employee的无参数构造方法
    
3.获取属性：分为所有的属性和指定的属性：
b，获取特定的属性，对比着传统的方法来学习：
//Student类
public class Student {
    private String name ;
    int age;
}

//传统方法：
Student s = new Student();
s.age = 12; //set 
        System.out.println(s.age);//get 
//但是无法获取private的属性


//反射方法
Class<?> c1 = Class.forName("reflect.Student");//取得对象
Object o = c1.newInstance();//实例化对象 
Field nameF = c1.getDeclaredField("name");//获取name属性
        nameF.setAccessible(true); //使用反射机制可以打破封装性，导致了java对象的属性不安全。  
        nameF.set(o,"钢铁侠");//给属性赋值
        System.out.println(nameF.get(o));
            
4，获取方法，和构造方法，不再详细描述，只来看一下关键字：
方法和关键字:
getDeclaredMethods():获取所有的方法
getReturnType():获得方法的放回类型
getParameterTypes():获得方法的传入参数类型
getDeclaredMethod("方法名",参数类型.class,……):获得特定的方法
        构造方法关键字
getDeclaredConstructors():获取所有的构造方法
getDeclaredConstructor(参数类型.class,……):获取特定的构造方法

父类和父接口:
getSuperclass():获取某类的父类
getInterfaces():获取某类实现的接口
```
这样我们就可以获得类的各种内容，进行了反编译。
对于JAVA这种先编译再运行的语言来说，反射机制可以使代码更加灵活，更加容易实现面向对象。

最后:	
JAVA反射的再次学习，灵活的运用它，能够使我们的代码更加灵活，但是它也有它的缺点，
就是运用它会使我们的软件的性能降低，复杂度增加，所以还要我们慎重的使用它。
[参考地址](https://www.sczyh30.com/posts/Java/java-reflection-1/)

## this
this是自身的一个对象，代表对象本身，可以理解为：指向对象本身的一个指针。
this的用法在java中大体可以分为3种：
1.普通的直接引用
这种就不用讲了，this相当于是指向当前对象本身。

2.形参与成员名字重名，用this来区分：
```
class Person {
    private int age = 10;
    public Person(){
        System.out.println("初始化年龄："+age);
    }
    public int GetAge(int age){
        this.age = age;
        return this.age;
    }
}
 
public class test1 {
    public static void main(String[] args) {
        Person Harry = new Person();
        System.out.println("Harry's age is "+Harry.GetAge(12));
    }
}
```
运行结果：
```
初始化年龄：10
Harry's age is 12
```
可以看到，这里age是GetAge成员方法的形参，this.age是Person类的成员变量。

3.引用构造函数
这个和super放在一起讲，见下面。

## super
super可以理解为是指向自己超（父）类对象的一个指针，而这个超类指的是离自己最近的一个父类。
super也有三种用法：
1.普通的直接引用
与this类似，super相当于是指向当前对象的父类，这样就可以用super.xxx来引用父类的成员。

2.子类中的成员变量或方法与父类中的成员变量或方法同名
```
class Country {
    String name;
    void value() {
       name = "China";
    }
}
  
class City extends Country {
    String name;
    void value() {
    name = "Shanghai";
    super.value();      //调用父类的方法
    System.out.println(name);
    System.out.println(super.name);
    }
  
    public static void main(String[] args) {
       City c=new City();
       c.value();
       }
}
```
运行结果：

```
Shanghai
China
```
可以看到，这里既调用了父类的方法，也调用了父类的变量。若不调用父类方法value()，只调用父类变量name的话，则父类name值为默认值null。

3.引用构造函数
super（参数）：调用父类中的某一个构造函数（应该为构造函数中的第一条语句）。
this（参数）：调用本类中另一种形式的构造函数（应该为构造函数中的第一条语句）。
```
class Person { 
    public static void prt(String s) { 
       System.out.println(s); 
    } 
   
    Person() { 
       prt("父类·无参数构造方法： "+"A Person."); 
    }//构造方法(1) 
    
    Person(String name) { 
       prt("父类·含一个参数的构造方法： "+"A person's name is " + name); 
    }//构造方法(2) 
} 
    
public class Chinese extends Person { 
    Chinese() { 
       super(); // 调用父类构造方法（1） 
       prt("子类·调用父类”无参数构造方法“： "+"A chinese coder."); 
    } 
    
    Chinese(String name) { 
       super(name);// 调用父类具有相同形参的构造方法（2） 
       prt("子类·调用父类”含一个参数的构造方法“： "+"his name is " + name); 
    } 
    
    Chinese(String name, int age) { 
       this(name);// 调用具有相同形参的构造方法（3） 
       prt("子类：调用子类具有相同形参的构造方法：his age is " + age); 
    } 
    
    public static void main(String[] args) { 
       Chinese cn = new Chinese(); 
       cn = new Chinese("codersai"); 
       cn = new Chinese("codersai", 18); 
    } 
}
```
运行结果：
```
父类·无参数构造方法： A Person.
子类·调用父类”无参数构造方法“： A chinese coder.
父类·含一个参数的构造方法： A person's name is codersai
子类·调用父类”含一个参数的构造方法“： his name is codersai
父类·含一个参数的构造方法： A person's name is codersai
子类·调用父类”含一个参数的构造方法“： his name is codersai
子类：调用子类具有相同形参的构造方法：his age is 18
```
从本例可以看到，可以用super和this分别调用父类的构造方法和本类中其他形式的构造方法。
例子中Chinese类第三种构造方法调用的是本类中第二种构造方法，而第二种构造方法是调用父类的，因此也要先调用父类的构造方法，再调用本类中第二种，最后是重写第三种构造方法。

## super和this的异同
super（参数）：调用基类中的某一个构造函数（应该为构造函数中的第一条语句） 
this（参数）：调用本类中另一种形成的构造函数（应该为构造函数中的第一条语句）
super:　它引用当前对象的直接父类中的成员（用来访问直接父类中被隐藏的父类中成员数据或函数，基类与派生类中有相同成员定义时如：super.变量名    super.成员函数据名（实参）
this：它代表当前对象名（在程序中易产生二义性之处，应使用this来指明当前对象；如果函数的形参与类中的成员数据同名，这时需用this来指明成员变量名）
调用super()必须写在子类构造方法的第一行，否则编译不通过。每个子类构造方法的第一条语句，都是隐含地调用super()，如果父类没有这种形式的构造函数，那么在编译的时候就会报错。
super()和this()类似,区别是，super()从子类中调用父类的构造方法，this()在同一类内调用其它方法。
super()和this()均需放在构造方法内第一行。
尽管可以用this调用一个构造器，但却不能调用两个。
this和super不能同时出现在一个构造函数里面，因为this必然会调用其它的构造函数，其它的构造函数必然也会有super语句的存在，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。
this()和super()都指的是对象，所以，均不可以在static环境中使用。包括：static变量,static方法，static语句块。
从本质上讲，this是一个指向本对象的指针, 然而super是一个Java关键字。

# 问题
## 什么是Java虚拟机？为什么Java被称作是“平台无关的编程语言？
Java虚拟机是一个可以执行Java字节码的虚拟机进程。Java源文件被编译成能被Java虚拟机执行的字节码文件。
Java被设计成允许应用程序可以运行在任意的平台，而不需要程序员为每一个平台单独重写或者是重新编译。
Java虚拟机让这个变为可能，因为它知道底层硬件平台的指令长度和其他特性。

## JDK和JRE的区别是什么？
JRE:Java运行时环境(JRE)是将要执行Java程序的Java虚拟机。它同时也包含了执行applet需要的浏览器插件。
JDK:Java开发工具包(JDK)是完整的Java软件开发包，包含了JRE，编译器和其他的工具(比如：JavaDoc，Java调试器)，可以让开发者开发、编译、执行Java应用程序。

## 是否可以在static环境中访问非static变量？(不可以)
static变量在Java中是属于类的，它在所有的实例中的值是一样的。当类被Java虚拟机载入的时候，会对static变量进行初始化。
如果你的代码尝试不用实例来访问非static的变量，编译器会报错，因为这些变量还没有被创建出来，还没有跟任何实例关联上。

## Java支持的数据类型有哪些？什么是自动拆装箱？
Java语言支持的8中基本数据类型是：
byte,short,int,long,float,double,boolean,char
自动装箱是Java编译器在基本数据类型和对应的对象包装类型之间做的一个转化。
Integer  a=1;//这就是一个自动装箱，如果没有自动装箱的话，需要这样Integer  a=new Integer(1)
int b=a;//这就是一个自动拆箱，如果没有自动拆箱的话，需要这样：int b=a.intValue()
自动装箱和自动拆箱是简化了基本数据类型和相对应对象的转化步骤

## 什么是值传递和引用传递？
对象被值传递，意味着传递了对象的一个副本。因此，就算是改变了对象副本，也不会影响源对象的值。
对象被引用传递，意味着传递的并不是实际的对象，而是对象的引用。因此，外部对引用对象所做的改变会反映到所有的对象上。

## 进程和线程的区别是什么？
进程是执行着的应用程序，而线程是进程内部的一个执行序列
一个进程可以有多个线程。线程又叫做轻量级进程。

## 什么是死锁(deadlock)？
两个进程都在等待对方执行完毕才能继续往下执行的时候就发生了死锁。结果就是两个进程都陷入了无限的等待中。

## 如何确保N个线程可以访问N个资源同时又不导致死锁？
使用多线程的时候，一种非常简单的避免死锁的方式就是：指定获取锁的顺序，并强制线程按照指定的顺序获取锁。因此，如果所有的线程都是以同样的顺序加锁和释放锁，就不会出现死锁了。

## 什么是迭代器(Iterator)？
Iterator接口提供了很多对集合元素进行迭代的方法。每一个集合类都包含了可以返回迭代器实例的迭代方法。迭代器可以在迭代的过程中删除底层集合的元素。

## Iterator和ListIterator的区别是什么？
Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。
Iterator对集合只能是前向遍历，ListIterator既可以前向也可以后向。
ListIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。

## Java中的HashMap的工作原理是什么？
Java中的HashMap是以键值对(key-value)的形式存储元素的。
HashMap需要一个hash函数，它使用hashCode()和equals()方法来向集合/从集合添加和检索元素。
当调用put()方法的时候，HashMap会计算key的hash值，然后把键值对存储在集合中合适的索引上。
如果key已经存在了，value会被更新成新值。
HashMap的一些重要的特性是它的容量(capacity)，负载因子(load factor)和扩容极限(threshold resizing)。

## Java中HashMap遍历的四种方式
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

## 数组(Array)和列表(ArrayList)有什么区别？什么时候应该使用Array而不是ArrayList？
Array可以包含基本类型和对象类型，ArrayList只能包含对象类型。
Array大小是固定的，ArrayList的大小是动态变化的。

## ArrayList和LinkedList有什么区别？
ArrayList和LinkedList都实现了List接口，他们有以下的不同点：
1.ArrayList数组实现，增删慢(增删元素时，需要重新计算大小或更新数组索引)，查找快
LinkedList链表实现，增删快，查找慢。
2.LinkedList比ArrayList更占内存，因为LinkedList为每一个节点存储了两个引用，一个指向前一个元素，一个指向下一个元素。

## System.gc()和Runtime.gc()会做什么事情？
这两个方法用来提示JVM要进行垃圾回收。但是，立即开始还是延迟进行垃圾回收是取决于JVM的。

## 堆栈区别？
1.栈区:编译器自动分配,存放函数的参数值，局部变量等
2.2.堆区:手动分配释放

## 什么是线程安全和不安全？
1.线程安全:a,b线程同时操作一个变量,a操作的时候，b不能操作,相当于单线程
2.线程不安全:a,b线程同时操作一个变量,可同时操作

## 同步和异步？
同步:发送一个请求,等待返回结果，然后再发送下一个请求
特点:需要等待,避免出现死锁，脏读数据的发生
异步:发送一个请求,不等待返回,随时可以再发送下一个请求
特点:无需等待,提高效率，但是会出现死锁，脏读数据

## 九种基本数据类型的大小，以及他们的封装类(一个字节是8位)

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

## equals、==、hashcode  区别
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

## Java的四种引用，强弱软虚，用到的场景	
1.强引用:如果一个对象具有强引用，那垃圾回收器绝不会回收它
Object o=new Object();   //  强引用
2.软引用:内存够，不会被回收(垃圾回收器),反之会被回收.
引用可用来实现内存敏感的高速缓存。
3.弱引用:垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存
4.虚引用：任何时候都可能被垃圾回收器回收

## Hashcode的作用
1.生成一个hash码,表示对象存储的物理地址
2.可以用来用来鉴定2个对象是否相等

## String、StringBuffer与StringBuilder的区别。
string:字符串常量，不可改变的对象
StringBuffer:字符串变量,可变对象,线程安全，效率低
StringBuilder:字符串变量,可变对象,线程不安全，效率高

## HashMap和HashTable的区别
HashMap:异步的，线程不安全
HashTable:同步的，线程安全

## token
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

## java局部变量和成员变量名相同
java中，**在同一个作用域下**，不能定义一样的标识符(变量名)，因为需要保证在某个作用域下使用某个标识符的时候，JVM能够正确进行区分。
全局变量、局部变量和内存不存在绝对直接的关系。不管是全局的还是局部的变量，他的标识都是保存在栈中的。
成员变量的作用域是整个类，局部变量能够覆盖成员(全局)变量。在局部使用某个变量的时候JVM会优先找和当前使用位置"近"的变量的的定义。
如果在局部定义了和全局变量一样的名字，如果调用全局变量的话，可以使用this关键字。
```
public class Test {
    int i = 1;
    void f(){
        int i = 10;
        System.out.println(i);
        System.out.println(this.i);
    }
    public static void main(String[] args) {
        Test t = new Test();
        t.f();
    }
}
```

## 使用Iterator在遍历的时候删除List里的元素
```java
public void iteratorRemove() {
	List<Student> students = this.getStudents();
	System.out.println(students);
	Iterator<Student> stuIter = students.iterator();
	while (stuIter.hasNext()) {
		Student student = stuIter.next();
		if (student.getId() % 2 == 0)
			stuIter.remove();//这里要使用Iterator的remove方法移除当前对象，如果使用List的remove方法，则同样会出现ConcurrentModificationException
	}
	System.out.println(students);
}
```

## Java中的三元运算符
你是想写一个表达式语句，你认为：(x == 5) ? "yes" : "no" 是一个表达式,则加上;后就成为一个语句了? 

这个在C/C++中正确的,在JAVA中对表达式有特定的要求.即:  表达式E;  
要形成一个表达式语句,则表达式E必须只能是:
- 赋值表达式,
- 自增++表达式,
- 自减--表达式,
- 方法调用表达式,
- new 表达式

## ？？？
```java
Long a = 1l;
int b = 1;
System.out.println(a == b);

Map<String,Object> map = new HashMap<>();
map.put("a",a);
map.put("b",b);
System.out.println(map.get("a") == map.get("b"));
```

## 自动拆箱问题
今天在写接口时，有个方法的参数是int类型：
`public xxx(int id);`

我在传递参数时，可能为null，所以我这么写：
`token == null ? null : token.getId()`

这时编译器是不报错的，但是会给提示：
```
Unboxing of 'null' may produce 'NullPointerException'
Inspection info: This inspection analyzes method control and data flow to report possible conditions that are always true or false, expressions whose value is statically proven to be constant, and situations that can lead to nullability contract violations.
Variables, method parameters and return values marked as @Nullable or @NotNull are treated as nullable (or not-null, respectively) and used during the analysis to check nullability contracts, e.g. report NullPointerException (NPE) errors that might be produced.
More complex contracts can be defined using @Contract annotation, for example:
@Contract("_, null -> null") — method returns null if its second argument is null
@Contract("_, null -> null; _, !null -> !null") — method returns null if its second argument is null and not-null otherwise
@Contract("true -> fail") — a typical assertFalse method which throws an exception if true is passed to it

The inspection can be configured to use custom @Nullable@NotNull annotations (by default the ones from annotations.jar will be used)
```

运行时直接报错了。因为`null`为对象类型，参数为int，在调用方法时，jvm会尝试把null拆箱为int类型，然后就直接报空指针了。
直接把参数改成`Integer`就好了。

2019/7/26更新
`java.lang.NullPointerException: cannot unbox null value`
需求参数为int，如果不传入参数，或为null，会报上边的错误。
这时把需求参数改为Integer，就好了。

## 集合指定和不指定反省的区别
指定了参数类型编译器在编译期间就会帮助你检查存入容器的对象是不是参数类型，不是就会报错，保证了类型安全，性能上没什么影响，因为泛型在运行期间会擦除。
就是说用不用类型参数在运行期间编译后的运行代码是一样的。
```java
Map map = new HashMap();
Map<String,String> map1 =new  HashMap<String,String>();
System.out.println(map.getClass().equals(map1.getClass()));
```
返回结果会true；说明他们运行的是同一份字节码。

## 将代码打成jar包


## 遍历枚举类
```java
public enum Test {

    UPDATE(1,"更新"),
    LOGIN(2,"登陆"),
    STOCK_IN(3,"库存领用"),
    QUERY_ACCOUNT(4,"账户查询"),
    PERSTORE(5,"预存"),
    OPEN_CARD_APPLY(6,"开卡申请"),
    OPEN_CARD(7,"开卡"),
    YIJIETIAO_APPLY(8,"借条申请"),
    ADDED_SERVICE_MAG(9,"增值服务管理"),
    NEW_ACCOUNT(10,"新建账户");

    private Integer code;
    private String type;

    private Test(Integer code, String type) {
        this.code = code;
        this.type = type;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public static void main(String args[]) {
        System.out.println("第一种通过反射");
        Class<Test> clz = Test.class;
        for (Test obj : clz.getEnumConstants()) {
            System.out.println(obj.getType());
            System.out.println(obj.getCode());
        }

        System.out.println("第二种通过枚举静态方法values()");
        for (Test rate : Test.values()) {
            System.out.println(rate.getType());
        }
    }
}
```

## Java实现Office系列文件转PDF文件
[官网下载jar包](https://downloads.aspose.com/)
[获取在网盘下载破解包](https://pan.baidu.com/s/1vzYobuqDZYe6-VPRp06l3w)
官网下载的有水印，不过是最新版，破解的是比较旧的版本，不过都能用。
```java
package com.archive.util.AsposeUtil;

import java.io.*;

import com.aspose.cells.Workbook;
import com.aspose.slides.Presentation;
import com.aspose.words.Document;
import com.aspose.words.SaveFormat;
import com.aspose.words.License;

/**
 * @author: wjy
 * @Date: Create in 14:28 2019/10/31
 * @description : Office系列文件转PDF文件工具类
 */
public class AsposeUtil {

    //直接把凭证内容写进来，省的读取本地文件
    private static final String licenseStr =
            "<License>" +
            "  <Data>" +
            "    <Products>" +
            "      <Product>Aspose.Total for Java</Product>" +
            "    </Products>" +
            "    <EditionType>Enterprise</EditionType>" +
            "    <SubscriptionExpiry>20991231</SubscriptionExpiry>" +
            "    <LicenseExpiry>20991231</LicenseExpiry>" +
            "    <SerialNumber>8bfe198c-7f0c-4ef8-8ff0-acc3237bf0d7</SerialNumber>" +
            "  </Data>" +
            "  <Signature>sNLLKGMUdF0r8O1kKilWAGdgfs2BvJb/2Xp8p5iuDVfZXmhppo+d0Ran1P9TKdjV4ABwAgKXxJ3jcQTqE/2IRfqwnPf8itN8aFZlV3TJPYeD3yWE7IT55Gz6EijUpC7aKeoohTb4w2fpox58wWoF3SNp6sK6jDfiAUGEHYJ9pjU=</Signature>" +
            "</License>";

    /**
     * 获取word、excel、ppt的license
     */
    private static boolean getLicense(int type) {
        boolean result = false;
        try {
            InputStream license = new ByteArrayInputStream(licenseStr.getBytes("UTF-8"));
            //也可以读取本地文件license.xml
            //InputStream license = new FileInputStream(new File("E:\\IDEA2017\\something2pdf-demo\\src\\main\\resources\\license.xml"));
            switch(type){
                case 1://word
                    License wordLicense = new License();
                    wordLicense.setLicense(license);
                    break;
                case 2://excel
                    com.aspose.cells.License excelLicense = new com.aspose.cells.License();
                    excelLicense.setLicense(license);
                    break;
                case 3://ppt
                    com.aspose.slides.License pptLicense = new com.aspose.slides.License();
                    pptLicense.setLicense(license);
                    break;
                default:
                    break;
            }
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * word转pdf
     */
    public static void doc2pdf(String outputPath,String inputPath) {
        //验证License 若不验证则转化出的pdf文档会有水印产生
        if (!getLicense(1)) return;
        try {
            //pdf输出路径
            FileOutputStream os = new FileOutputStream(new File(outputPath));
            //word文件路径，即本地word文件路径
            Document doc = new Document(inputPath);

            //这里也可以获取上传的文件
            //Document doc = new Document(multipartFile.getInputStream());

            //保存转换的pdf文件
            doc.save(os, SaveFormat.PDF);
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * excel转pdf
     */
    public static void excel2pdf(String outputPath,String inputPath) {
        // 验证License 若不验证则转化出的pdf文档会有水印产生
        if (!getLicense(2)) return;
        try {
            //pdf输出路径
            File file = new File(outputPath);
            FileOutputStream os = new FileOutputStream(file);

            //excel输入路径，即本地excel文件路径
            Workbook wb = new Workbook(inputPath);
            wb.save(os, com.aspose.cells.SaveFormat.PDF);
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * ppt转pdf
     */
    public static void ppt2pdf(String outputPath,String inputPath) {
        // 验证License
        if (!getLicense(3)) return;
        try {
            //pdf输出路径
            File file = new File(outputPath);
            //ppt输入路径，即本地ppt文件路径
            Presentation pres = new Presentation(inputPath);

            FileOutputStream fileOS = new FileOutputStream(file);
            pres.save(fileOS, com.aspose.slides.SaveFormat.Pdf);
            fileOS.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        AsposeUtil.doc2pdf("D:\\doc2pdf.pdf","C:\\Users\\wjy\\Desktop\\测试word.docx");
        AsposeUtil.excel2pdf("D:\\excel2pdf.pdf","C:\\Users\\wjy\\Desktop\\测试excel.xlsx");
        AsposeUtil.ppt2pdf("D:\\ppt2pdf.pdf","C:\\Users\\wjy\\Desktop\\测试ppt.pptx");
    }

}
```
maven仓库里没有这三个包。需要自己下载，然后放到项目里，或放到自己的maven仓库里。
而且`new License()`和`SaveFormat`两个方法，三个包的名字是相同的，所以只能倒一个包，剩下两个写全路径。

三个包名：
`aspose-cells-18.9.jar`excel转pdf
`aspose-slides-19.6.jar`ppt转pdf
`aspose-words-16.8.0-jdk16.jar`word转pdf

[words转pdf参考](https://www.jianshu.com/p/86716c7122ef)
[参考地址](https://blog.csdn.net/qq_34190023/article/details/82999054)
[如何破解](https://blog.csdn.net/qq_42834405/article/details/98635337)

## 两个同名的jar包
```
import com.aspose.words.License;
import com.aspose.cells.License;
```
这样下边这个包会报错。
解决办法：
只倒一个包，另外的直接用全路径，不倒包。如：
`com.aspose.cells.License asposeLic = new com.aspose.cells.License();`
`java.io.File dest = new java.io.File(filePath + alias);`

## 读取pdf内容
文本内容提取:
pom引入
```
<dependency>
	<groupId>org.apache.pdfbox</groupId>
	<artifactId>pdfbox</artifactId>
	<version>2.0.15</version>
</dependency>

<dependency>
	<groupId>org.apache.pdfbox</groupId>
	<artifactId>fontbox</artifactId>
	<version>2.0.15</version>
</dependency>

<dependency>
	<groupId>org.apache.pdfbox</groupId>
	<artifactId>jempbox</artifactId>
	<version>1.8.16</version>
</dependency>
```

```java
/**
 * 读PDF文件
 * @param fileName
 */
public static void readPDF(String fileName) {
	File file = new File(fileName);
	FileInputStream in = null;
	try {
		PDDocument document=PDDocument.load(file);
		// 获取页码
		int pages = document.getNumberOfPages();
		// 读文本内容
		PDFTextStripper stripper=new PDFTextStripper();
		// 设置按顺序输出
		stripper.setSortByPosition(true);
		stripper.setStartPage(1);
		stripper.setEndPage(pages);
		String content = stripper.getText(document);
		System.out.println(content);

		//可选，将pdf内容写入到txt文本文件
//            FileWriter fileWriter = new FileWriter(new File("pdf.txt"));
//            fileWriter.write(content);
//            fileWriter.flush();
//            fileWriter.close();

	} catch (Exception e) {
		System.out.println("读取PDF文件" + file.getAbsolutePath() + "失败！" + e);
		e.printStackTrace();
	} finally {
		if (in != null) {
			try {
				in.close();
			} catch (IOException e1) {
			}
		}
	}
}
```

# util
## 时间戳
- 判断两个时间戳是否是同一天
```java
public static boolean isSameDay(int beginTimestamp,int endTimestamp) {
	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
	String ds1 = sdf.format(beginTimestamp * 1000l);
	String ds2 = sdf.format(endTimestamp * 1000l);
	if (ds1.equals(ds2)) {
		return true;
	} else {
		return false;
	}
}
```

- 计算两个时间戳差值(秒级时间戳)
```java
public static int getTimeDifference(int beginTimestamp, int endTimestamp, int type) {
	int result = 0;
	if (beginTimestamp == 0 || endTimestamp == 0) {
		return 0;
	}
	try {
		int millisecond = endTimestamp - beginTimestamp;
		/**
		 * Math.abs((int)(millisecond/1000)); 绝对值 1秒 = 1000毫秒
		 * millisecond/1000 --> 秒
		 * millisecond/1000*60 - > 分钟
		 * millisecond/(1000*60*60) -- > 小时
		 * millisecond/(1000*60*60*24) --> 天
		 * */
		switch (type) {
			case 0: // second
				return  millisecond;
			case 1: // minute
				return (millisecond / 60);
			case 2: // hour
				return  (millisecond / (60 * 60));
			case 3: // day
				return (millisecond / (60 * 60 * 24));
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
	return result;
}
```

- 字符串排序
```
/**
 * @author: wjy
 * @description: 字符串根据ASCII码表进行升序排列
 */
public static String ASCIISort(String str) {
	char[] test = new char[str.length()];
	StringBuilder sb = new StringBuilder();
	while (true) {
		String a = str;//直接读取这行当中的字符串。
		for (int i = 0; i < str.length(); i++) {
			test[i] = a.charAt(i);//字符串处理每次读取一位。
		}
		Arrays.sort(test);
		for (int i = 0; i < test.length; i++) {
			sb.append(test[i]);
		}
		String trim = sb.toString().trim();
		return trim;
	}
}
```

# Lambda表达式
Lambda 表达式是一种匿名函数，简单地说，它是没有声明的方法，也即没有访问修饰符、返回值声明和名字。
它可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使 Java 语言的表达能力得到了提升。
Lambda表达式是Java SE 8中一个重要的新特性，允许你通过表达式来代替功能接口。就和方法一样，它提供了一个正常的参数列表和一个使用这些参数的主体(body，可以是一个表达式或一个代码块)。Lambda表达式还增强了集合库。
Java SE 8添加了2个对集合数据进行批量操作的包：java.util.function 包以及 java.util.stream 包。
流(stream)就如同迭代器(iterator)，但附加了许多额外的功能。
总的来说，lambda表达式和 stream 是自Java语言添加泛型(Generics)和注解(annotation)以来最大的变化。

## 环境准备
jdk1.8以上

## Lambda表达式的语法
基本语法:
(parameters) -> expression
或
(parameters) ->{ statements; }

简单例子：
// 1. 不需要参数,返回值为 5
() -> 5

// 2. 接收一个参数(数字类型)，返回其2倍的值
x -> 2 * x

// 3. 接受2个参数(数字)，并返回他们的差值
(x, y) -> x – y

// 4. 接收2个int型整数，返回他们的和
(int x, int y) -> x + y

// 5. 接受一个 string 对象，并在控制台打印，不返回任何值(看起来像是返回void)
(String s) -> System.out.print(s)

## 什么是函数式接口
在对上面进行举例说明之前，必须先来理解下函数式接口，因为Lambda是建立在函数式接口的基础上的。
- 只包含一个抽象方法的接口，称为函数式接口。
- 你可以通过 Lambda 表达式来创建该接口的对象。
- 我们可以在任意函数式接口上使用 @FunctionalInterface 注解，这样做可以检测它是否是一个函数式接口，同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。在实际开发者🈶️两个比较常见的函数式接口：Runnable接口，Comparator接口

Runnable接口相关
```java
public class Test {
    
    public static void main(String[] args) {
        
        // 1.1使用匿名内部类  
        new Thread(new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("Hello world !");  
            }  
        }).start();  
          
        // 1.2使用 lambda 获得Runnable接口对象  
        new Thread(() -> System.out.println("Hello world !")).start();  
        
//=============================================================================
        
        // 2.1使用匿名内部类  
        Runnable race1 = new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("Hello world !");  
            }  
        };  
          
        // 2.2使用 lambda直接获得接口对象 
        Runnable race2 = () -> System.out.println("Hello world !");          
        
        // 直接调用 run 方法(没开新线程哦!)  
        race1.run();  
        race2.run();  
    }
}
/*输出结果
 * Hello world !
 * Hello world !
 * Hello world !
 * Hello world !
 *／
```

使用Lambda对数组排序
```java
public class TestArray {
    
    public static void main(String[] args) {
        String[] players = {"zhansgan", "lisi", "wangwu", "zhaoliu",  "wangmazi"};  

        // 1.1 使用匿名内部类根据 surname 排序 players  
        Arrays.sort(players, new Comparator<String>() {  
            @Override  
            public int compare(String s1, String s2) {  
                return (s1.compareTo(s2));  
            }  
        });  
        
        // 1.2 使用 lambda 排序,根据 surname  
        Arrays.sort(players, (String s1, String s2) ->  s1.compareTo(s2));  
         
//================================================================================================
          
        // 2.1 使用匿名内部类根据 name lenght 排序 players  
        Arrays.sort(players, new Comparator<String>() {  
            @Override  
            public int compare(String s1, String s2) {  
                return (s1.length() - s2.length());  
            }  
        });  

        // 2.2使用Lambda,根据name length  
        Arrays.sort(players, (String s1, String s2) -> (s1.length() - s2.length()));  
    
//==================================================================================================    
        
        // 3.1 使用匿名内部类排序 players, 根据最后一个字母  
        Arrays.sort(players, new Comparator<String>() {  
            @Override  
            public int compare(String s1, String s2) {  
                return (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
            }  
        });  

        // 3.2 使用Lambda,根据最后一个字母
        Arrays.sort(players, (String s1, String s2) -> (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1)));  
    }
}
```
通过上面例子我们再来思考为什么Lambda表达式需要函数式接口？其实很简单目的就是为来保证唯一。
你的Runnable接口只要一个抽象方法，那么我用() -> System.out.println("Hello world !")，就只能代表run方法，如果你下面还有一个抽象方法，那我使用Lambda表达式，那鬼才知道要调用哪个抽象方法呢。

## 方法引用
函数式接口的实例可以通过 lambda 表达式、 方法引用、构造方法引用来创建。方法引用是 lambda 表达式的语法糖，任何用方法引用的地方都可由lambda表达式替换，但是并不是所有的lambda表达式都可以用方法引用来替换。
举例，这就是一个打印集合所有元素的例子，value -> System.out.println(value) 是一个Consumer函数式接口， 这个函数式接口可以通过方法引用来替换。
```
public class TestArray {
    
    public static void main(String[] args) {
         List<String> list = Arrays.asList("xuxiaoxiao", "xudada", "xuzhongzhong");
           list.forEach(value -> System.out.println(value));
    }
    /* 输出：
     * xuxiaoxiao
     * xudada
     * xuzhongzhong
     */
}
```
使用方法引用的方式，和上面的输出是一样的，方法引用使用的是双冒号（::）
`list.forEach(System.out::println);`

分类

|类别|使用形式|
|-|-|
|静态方法引用|类名 :: 静态方法名|
|实例方法引用|对象名(引用名) :: 实例方法名|
|类方法引用|类名 :: 实例方法名|
|构造方法引用|类名 :: new|

- 静态方法引用
```java
public class Apple {

    private String name;
    private String color;
    private double weight;

    public Apple(String name, String color, double weight) {
        this.name = name;
        this.color = color;
        this.weight = weight;
    }

    public static int compareByWeight(Apple a1, Apple a2) {
        double diff = a1.getWeight() - a2.getWeight();
        return new Double(diff).intValue();
    }

    //还有getter setter toString
}
```
有一个苹果的List，现在需要根据苹果的重量进行排序。List 的 sort 函数接收一个 Comparator 类型的参数，Comparator 是一个函数式接口，接收两个参数，返回一个int值。
Apple的静态方法compareByWeight正好符合Comparator函数式接口，所以可以使用：
Apple::compareByWeight 静态方法引用来替代lambda表达式
```java
public class LambdaTest {

    public static void main(String[] args) {

        Apple apple1 = new Apple("红富士", "Red", 280);
        Apple apple2 = new Apple("冯心", "Yello", 470);
        Apple apple3 = new Apple("大牛", "Red", 320);
        Apple apple4 = new Apple("小小", "Green", 300);

        List<Apple> appleList = Arrays.asList(apple1, apple2, apple3, apple4);

        //lambda 表达式形式
        //appleList.sort((Apple a1, Apple a2) -> {
        //    return new Double(a1.getWeight() - a2.getWeight()).intValue();
        //});

        //静态方法引用形式（可以看出引用方法比上面的更加简单
        appleList.sort(Apple::compareByWeight);

        appleList.forEach(apple -> System.out.println(apple));

    }
}
输出：
Apple{category='红富士', color='Red', weight=280.0}
Apple{category='小小', color='Green', weight=300.0}
Apple{category='大牛', color='Red', weight=320.0}
Apple{category='冯心', color='Yello', weight=470.0}
```
Apple.compareByWeight是方法的调用，而Apple::compareByWeight方法引用，这两者完全不是一回事。

- 实例方法引用
这个compareByWeight是一个实例方法
```
public class AppleComparator {

    public int compareByWeight(Apple a1, Apple a2) {
        double diff = a1.getWeight() - a2.getWeight();
        return new Double(diff).intValue();
    }
}
```
下面的例子通过实例对象的方法引用 comparator::compareByWeight 来代替lambda表达式
```java
public class LambdaTest {

    public static void main(String[] args) {

        Apple apple1 = new Apple("红富士", "Red", 280);
        Apple apple2 = new Apple("冯心", "Yello", 470);
        Apple apple3 = new Apple("哈哈", "Red", 320);
        Apple apple4 = new Apple("小小", "Green", 300);


        List<Apple> appleList = Arrays.asList(apple1, apple2, apple3, apple4);

        //lambda 表达式形式
        //appleList.sort((Apple a1, Apple a2) -> {
        //    return new Double(a1.getWeight() - a2.getWeight()).intValue();
        //});

        //实例方法引用
        AppleComparator comparator = new AppleComparator();
        appleList.sort(comparator::compareByWeight);

        appleList.forEach(apple -> System.out.println(apple));

    }
}
输出：
Apple{category='红富士', color='Red', weight=280.0}
Apple{category='小小', color='Green', weight=300.0}
Apple{category='哈哈', color='Red', weight=320.0}
Apple{category='冯心', color='Yello', weight=470.0}
```
通过上面两个例子可以看到，静态方法引用和实例方法引用都是比较好理解的。

- 类方法引用
一般来说，同类型对象的比较，应该当前调用方法的对象与另外一个对象进行比较，好的设计应该像下面： 
```java
public class Apple {

    private String category;
    private String color;
    private double weight;

    public Apple(String category, String color, double weight) {
        this.category = category;
        this.color = color;
        this.weight = weight;
    }
//这里和上面静态方式唯一区别就是这个参数就一个，需要实例对象调这个方法
    public int compareByWeight(Apple other) {
        double diff = this.getWeight() - other.getWeight();
        return new Double(diff).intValue();
    }

    //getter setter toString
}
```
还是之前List排序的例子，看看使用类方法引用如何写：
```java
public class LambdaTest {

    public static void main(String[] args) {

        Apple apple1 = new Apple("红富士", "Red", 280);
        Apple apple2 = new Apple("黄元帅", "Yello", 470);
        Apple apple3 = new Apple("红将军", "Red", 320);
        Apple apple4 = new Apple("国光", "Green", 300);


        List<Apple> appleList = Arrays.asList(apple1, apple2, apple3, apple4);

        //lambda 表达式形式
        //appleList.sort((Apple a1, Apple a2) -> {
        //    return new Double(a1.getWeight() - a2.getWeight()).intValue();
        //});

        //这里是类方法引用
        appleList.sort(Apple::compareByWeight);

        appleList.forEach(apple -> System.out.println(apple));

    }
}
输出：
Apple{category='红富士', color='Red', weight=280.0}
Apple{category='国光', color='Green', weight=300.0}
Apple{category='红将军', color='Red', weight=320.0}
Apple{category='黄元帅', color='Yello', weight=470.0}
```
这里使用的是：类名::实例方法名。首先要说明的是，方法引用不是方法调用。compareByWeight一定是某个实例调用的，就是lambda表达式的第一个参数，然后lambda表达式剩下的参数作为 
compareByWeight的参数，这样compareByWeight正好符合lambda表达式的定义。
或者也可以这样理解：
(Apple a1, Apple a2) -> { return new Double(a1.getWeight() - a2.getWeight()).intValue(); }
int compareByWeight(Apple other) 需要当前对象调用，然后与另外一个对象比较，并且返回一个int值。可以理解为lambda表达式的第一个参数 a1 赋值给当前对象， 然后 a2 赋值给 other对象，然后返回int值。

- 构造方法引用
```
public class ConstructionMethodTest {

    public String getString(Supplier<String> supplier) {
        return supplier.get();
    }

    public static void main(String[] args) {

        ConstructionMethodTest test = new ConstructionMethodTest();

        //lambda表达式形式
        System.out.println(test.getString(() -> { return new String();}));

        //构造方法引用形式
        System.out.println(test.getString(String::new));

    }
}
```
getString 方法接收一个Supplier类型的参数，Supplier 不接收参数，返回一个String。lambda表达式应该这样写：
`() -> { return new String();}`
替换成方法引用的形式如下： 实际上调用的是String 无参构造方法。
`String::new`

## 基本的Lambda例子
假设有一个玩家List，程序员可以使用 for循环来遍历，在Java SE 8中可以转换为另一种形式：
```
String[] atp = {"小明","小红", "小黑", "小白","小兰", "小强","小绿", "小小"};
List<String> players =  Arrays.asList(atp);

//以前的循环方式
for (String player : players) {
	System.out.print(player + "; ");
}

//使用lambda表达式以及函数操作(functional operation)
players.forEach((player) -> System.out.println(player + ";"));

//在JAVA 8 中使用双冒号操作符(double colon operator)
players.forEach(System.out::println);
```

```
//下面是使用lambdas 来实现 Runnable接口 的示例:
// 1.1使用匿名内部类
new Thread(new Runnable() {
	@Override
	public void run() {
		System.out.println("Hello world !");
	}
}).start();

// 1.2使用 lambda expression
new Thread(() -> System.out.println("Hello world !")).start();

// 2.1使用匿名内部类
Runnable race1 = new Runnable() {
	@Override
	public void run() {
		System.out.println("Hello world !");
	}
};

// 2.2使用 lambda expression
Runnable race2 = () -> System.out.println("Hello world !");

// 直接调用 run 方法(没开新线程哦!)
race1.run();
race2.run();
```

## 使用Lambda对集合进行排序
在java中，Comparator 类被用来排序集合，在下面的例子中，我们将根据球员的 name, surname, name 长度以及最后一个字母。和前面的示例一样，先使用匿名内部类来排序，然后再使用lambda表达式精简我们的代码。
在第一个例子中,我们将根据name来排序list。使用旧的方式,代码如下所示：

`String[] atp = {"小明","小红", "小黑", "小白","小兰", "小强","小绿", "小小"};`
```
// 1.1 使用匿名内部类根据 name 排序 players 
Arrays.sort(players, new Comparator<String>() { 
    @Override 
    public int compare(String s1, String s2) { 
        return (s1.compareTo(s2));  
    } 
});
```
使用lambdas，可以通过下面的代码实现同样的功能：
```
// 1.2 使用 lambda expression 排序 players 
Comparator<String> sortByName = (String s1, String s2) -> (s1.compareTo(s2)); 
Arrays.sort(players, sortByName); 
  
// 1.3 也可以采用如下形式: 
Arrays.sort(players, (String s1, String s2) -> (s1.compareTo(s2)));
```

其他的排序如下所示。和上面的示例一样，代码分别通过匿名内部类和一些lambda表达式来实现Comparator：
```
// 1.1 使用匿名内部类根据 surname 排序 players 
Arrays.sort(players, new Comparator<String>() { 
    @Override 
    public int compare(String s1, String s2) { 
        return (s1.substring(s1.indexOf(" ")).compareTo(s2.substring(s2.indexOf(" "))));  
    } 
}); 
  
// 1.2 使用 lambda expression 排序,根据 surname 
Comparator<String> sortBySurname = (String s1, String s2) -> 
    ( s1.substring(s1.indexOf(" ")).compareTo( s2.substring(s2.indexOf(" ")) ) ); 
Arrays.sort(players, sortBySurname); 
  
// 1.3 或者这样,怀疑原作者是不是想错了,括号好多... 
Arrays.sort(players, (String s1, String s2) -> 
      ( s1.substring(s1.indexOf(" ")).compareTo( s2.substring(s2.indexOf(" ")) ) ) 
    );
  
// 2.1 使用匿名内部类根据 name lenght 排序 players
Arrays.sort(players, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return (s1.length() - s2.length());  
    }
});
  
// 2.2 使用 lambda expression 排序,根据 name lenght
Comparator<String> sortByNameLenght = (String s1, String s2) -> (s1.length() - s2.length());
Arrays.sort(players, sortByNameLenght);
  
// 2.3 or this
Arrays.sort(players, (String s1, String s2) -> (s1.length() - s2.length()));
  
// 3.1 使用匿名内部类排序 players, 根据最后一个字母
Arrays.sort(players, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
    }
});
  
// 3.2 使用 lambda expression 排序,根据最后一个字母
Comparator<String> sortByLastLetter = 
    (String s1, String s2) ->
        (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
Arrays.sort(players, sortByLastLetter);
  
// 3.3 or this
Arrays.sort(players, (String s1, String s2) -> (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1)));
```

## 使用Lambdas和Streams
Stream是对集合的包装，通常和lambda一起使用。 使用lambdas可以支持许多操作，
如 map、filter、limit、sorted、count、min、max、sum、collect等等。
同样，tream使用懒运算，他们并不会真正地读取所有数据，遇到像getFirst()这样的方法就会结束链式语法。
在接下来的例子中,我们将探索lambdas和streams 能做什么。我们创建了一个Person类并使用这个类来添加一些数据到list中，将用于进一步流操作。

Person只是一个简单的POJO类：
```
public class Person {
  
private String firstName, lastName, job, gender;
private int salary, age;
  
public Person(String firstName, String lastName, String job,
                String gender, int age, int salary)       {  
          this.firstName = firstName;  
          this.lastName = lastName;  
          this.gender = gender;  
          this.age = age;  
          this.job = job;  
          this.salary = salary;  
}
// Getter and Setter
// . . . . .
}
```
接下来，我们将创建两个list，都用来存放Person对象：
```
List<Person> javaProgrammers = new ArrayList<Person>() {
  {
    add(new Person("Elsdon", "Jaycob", "Java programmer", "male", 43, 2000));
    add(new Person("Tamsen", "Brittany", "Java programmer", "female", 23, 1500));
    add(new Person("Floyd", "Donny", "Java programmer", "male", 33, 1800));
    add(new Person("Sindy", "Jonie", "Java programmer", "female", 32, 1600));
    add(new Person("Vere", "Hervey", "Java programmer", "male", 22, 1200));
    add(new Person("Maude", "Jaimie", "Java programmer", "female", 27, 1900));
    add(new Person("Shawn", "Randall", "Java programmer", "male", 30, 2300));
    add(new Person("Jayden", "Corrina", "Java programmer", "female", 35, 1700));
    add(new Person("Palmer", "Dene", "Java programmer", "male", 33, 2000));
    add(new Person("Addison", "Pam", "Java programmer", "female", 34, 1300));
  }
};
  
List<Person> phpProgrammers = new ArrayList<Person>() {
  {
    add(new Person("Jarrod", "Pace", "PHP programmer", "male", 34, 1550));
    add(new Person("Clarette", "Cicely", "PHP programmer", "female", 23, 1200));
    add(new Person("Victor", "Channing", "PHP programmer", "male", 32, 1600));
    add(new Person("Tori", "Sheryl", "PHP programmer", "female", 21, 1000));
    add(new Person("Osborne", "Shad", "PHP programmer", "male", 32, 1100));
    add(new Person("Rosalind", "Layla", "PHP programmer", "female", 25, 1300));
    add(new Person("Fraser", "Hewie", "PHP programmer", "male", 36, 1100));
    add(new Person("Quinn", "Tamara", "PHP programmer", "female", 21, 1000));
    add(new Person("Alvin", "Lance", "PHP programmer", "male", 38, 1600));
    add(new Person("Evonne", "Shari", "PHP programmer", "female", 40, 1800));
  }
};
```
现在我们使用forEach方法来迭代输出上述列表：
```
System.out.println("所有程序员的姓名:");
javaProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
phpProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
```
我们同样使用forEach方法，增加程序员的工资5%：
```
System.out.println("给程序员加薪 5% :");
Consumer<Person> giveRaise = e -> e.setSalary(e.getSalary() / 100 * 5 + e.getSalary());
  
javaProgrammers.forEach(giveRaise);
phpProgrammers.forEach(giveRaise);
```
另一个有用的方法是过滤器filter()，让我们显示月薪超过1400美元的PHP程序员：
```
System.out.println("下面是月薪超过 $1,400 的PHP程序员:")
phpProgrammers.stream()
          .filter((p) -> (p.getSalary() > 1400))  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
```
我们也可以定义过滤器，然后重用它们来执行其他操作：
```
// 定义 filters
Predicate<Person> ageFilter = (p) -> (p.getAge() > 25);
Predicate<Person> salaryFilter = (p) -> (p.getSalary() > 1400);
Predicate<Person> genderFilter = (p) -> ("female".equals(p.getGender()));
  
System.out.println("下面是年龄大于 24岁且月薪在$1,400以上的女PHP程序员:");
phpProgrammers.stream()
          .filter(ageFilter)  
          .filter(salaryFilter)  
          .filter(genderFilter)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  
// 重用filters
System.out.println("年龄大于 24岁的女性 Java programmers:");
javaProgrammers.stream()
          .filter(ageFilter)  
          .filter(genderFilter)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```
使用limit方法，可以限制结果集的个数：
```
System.out.println("最前面的3个 Java programmers:");
javaProgrammers.stream()
          .limit(3)  
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
		  
System.out.println("最前面的3个女性 Java programmers:");
javaProgrammers.stream()
          .filter(genderFilter)
          .limit(3)
          .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
```
排序呢? 我们在stream中能处理吗? 答案是肯定的。在下面的例子中,我们将根据名字和薪水排序Java程序员，放到一个list中,然后显示列表：
```
System.out.println("根据 name 排序,并显示前5个 Java programmers:");
List<Person> sortedJavaProgrammers = javaProgrammers
          .stream()  
          .sorted((p, p2) -> (p.getFirstName().compareTo(p2.getFirstName())))  
          .limit(5)  
          .collect(toList());  
  
sortedJavaProgrammers.forEach((p) -> System.out.printf("%s %s; %n", p.getFirstName(), p.getLastName()));
   
System.out.println("根据 salary 排序 Java programmers:");
sortedJavaProgrammers = javaProgrammers
          .stream()  
          .sorted( (p, p2) -> (p.getSalary() - p2.getSalary()) )  
          .collect( toList() );  
  
sortedJavaProgrammers.forEach((p) -> System.out.printf("%s %s; %n", p.getFirstName(), p.getLastName())); 
```
如果我们只对最低和最高的薪水感兴趣，比排序后选择第一个/最后一个更快的是min和max方法：
```
System.out.println("工资最低的 Java programmer:");
Person pers = javaProgrammers
          .stream()  
          .min((p1, p2) -> (p1.getSalary() - p2.getSalary()))  
          .get()  
  
System.out.printf("Name: %s %s; Salary: $%,d.", pers.getFirstName(), pers.getLastName(), pers.getSalary())
  
System.out.println("工资最高的 Java programmer:");
Person person = javaProgrammers
          .stream()  
          .max((p, p2) -> (p.getSalary() - p2.getSalary()))  
          .get()  
  
System.out.printf("Name: %s %s; Salary: $%,d.", person.getFirstName(), person.getLastName(), person.getSalary())
```
上面的例子中我们已经看到 collect 方法是如何工作的。结合 map 方法，我们可以使用 collect 方法来将我们的结果集放到一个字符串，一个 Set 或一个TreeSet中：
```
System.out.println("将 PHP programmers 的 first name 拼接成字符串:");
String phpDevelopers = phpProgrammers
          .stream()  
          .map(Person::getFirstName)  
          .collect(joining(" ; ")); // 在进一步的操作中可以作为标记(token)     
  
System.out.println("将 Java programmers 的 first name 存放到 Set:");
Set<String> javaDevFirstName = javaProgrammers
          .stream()  
          .map(Person::getFirstName)  
          .collect(toSet());  
  
System.out.println("将 Java programmers 的 first name 存放到 TreeSet:");
TreeSet<String> javaDevLastName = javaProgrammers
          .stream()  
          .map(Person::getLastName)  
          .collect(toCollection(TreeSet::new));
```
Streams 还可以是并行的(parallel)。示例如下：
```
System.out.println("计算付给 Java programmers 的所有money:");
int totalSalary = javaProgrammers
          .parallelStream()  
          .mapToInt(p -> p.getSalary())  
          .sum();  
```
我们可以使用summaryStatistics方法获得stream中元素的各种汇总数据。接下来，我们可以访问这些方法，比如getMax，getMin，getSum或getAverage：
```
//计算 count, min, max, sum, and average for numbers
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
IntSummaryStatistics stats = numbers
          .stream()  
          .mapToInt((x) -> x)  
          .summaryStatistics();  
  
System.out.println("List中最大的数字 : " + stats.getMax());
System.out.println("List中最小的数字 : " + stats.getMin());
System.out.println("所有数字的总和   : " + stats.getSum());
System.out.println("所有数字的平均值 : " + stats.getAverage()); 
``` 
[参考](https://www.cnblogs.com/franson-2016/p/5593080.html)
---
title: Exception
tags: java
date: 2019-05-07 17:46:43
---

# Java异常简介
**Java异常是Java提供的一种识别及相应错误的一致性机制**。

Java异常机制可以使程序中异常处理代码和正常业务代码分离，保证程序代码更加优雅，并提高程序健壮性。在有效使用异常的情况下，异常能清晰的回答what、where、why这3个问题：异常类型回答了“什么”被抛出，异常堆栈跟踪回答了“在哪“抛出，异常信息回答了“为什么“会抛出。

Java异常机制用到的几个关键字：try、catch、finally、throw、throws。
- try：用于监听。将要被监听的代码(可能抛出异常的代码)放在try语句块之内，当try语句块内发生异常时，异常就被抛出。
- catch：用于捕获异常。catch用来捕获try语句块中发生的异常。
- finally：finally语句块总是会被执行。它主要用于回收在try块里打开的物理资源(如数据库连接、网络连接和磁盘文件)。只有finally块，执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。
- throw：用于抛出异常。
- throws：用在方法签名中，用于声明该方法可能抛出的异常。

# 异常简单示例
## try和catch基本用法
```java
public class Demo1 {

    public static void main(String[] args) {
        try {
            int i = 10/0;
              System.out.println("i="+i); 
        } catch (ArithmeticException e) {
              System.out.println("Caught Exception"); 
            System.out.println("e.getMessage(): " + e.getMessage()); 
            System.out.println("e.toString(): " + e.toString()); 
            System.out.println("e.printStackTrace():");
            e.printStackTrace(); 
        }
    }
}
```
运行结果：
```
Caught Exception
e.getMessage(): / by zero
e.toString(): java.lang.ArithmeticException: / by zero
e.printStackTrace():
java.lang.ArithmeticException: / by zero
    at Demo1.main(Demo1.java:6)
```
结果说明：在try语句块中有除数为0的操作，该操作会抛出java.lang.ArithmeticException异常。通过catch，对该异常进行捕获。
观察结果我们发现，并没有执行System.out.println("i="+i)。这说明try语句块发生异常之后，try语句块中的剩余内容就不会再被执行了。

## finally的基本用法
在"示例一"的基础上，我们添加finally语句。
```java
public class Demo2 {

    public static void main(String[] args) {
        try {
            int i = 10/0;
              System.out.println("i="+i); 
        } catch (ArithmeticException e) {
              System.out.println("Caught Exception"); 
            System.out.println("e.getMessage(): " + e.getMessage()); 
            System.out.println("e.toString(): " + e.toString()); 
            System.out.println("e.printStackTrace():");
            e.printStackTrace(); 
        } finally {
            System.out.println("run finally");
        }
    }
}
```
运行结果：
```
Caught Exception
e.getMessage(): / by zero
e.toString(): java.lang.ArithmeticException: / by zero
e.printStackTrace():
java.lang.ArithmeticException: / by zero
    at Demo2.main(Demo2.java:6)
run finally
```
结果说明：最终执行了finally语句块。

## throws和throw的基本用法
throws是用于声明抛出的异常，而throw是用于抛出异常。
```java
class MyException extends Exception {
    public MyException() {}
    public MyException(String msg) {
        super(msg);
    }
}

public class Demo3 {

    public static void main(String[] args) {
        try {
            test();
        } catch (MyException e) {
            System.out.println("Catch My Exception");
            e.printStackTrace();
        }
    }
    public static void test() throws MyException{
        try {
            int i = 10/0;
              System.out.println("i="+i); 
        } catch (ArithmeticException e) {
            throw new MyException("This is MyException"); 
        }
    }
}
```
运行结果：
```
Catch My Exception
MyException: This is MyException
    at Demo3.test(Demo3.java:24)
    at Demo3.main(Demo3.java:13)
```
结果说明：
　　MyException是继承于Exception的子类。test()的try语句块中产生ArithmeticException异常(除数为0)，并在catch中捕获该异常；接着抛出MyException异常。main()方法对test()中抛出的MyException进行捕获处理。

# Java异常框架结构
![](Exception/1.jpg)
1. Throwable
　　Throwable是 Java 语言中所有错误或异常的超类。
　　Throwable包含两个子类: Error 和 Exception。它们通常用于指示发生了异常情况。
　　Throwable包含了其线程创建时线程执行堆栈的快照，它提供了printStackTrace()等接口用于获取堆栈跟踪数据等信息。

2. Exception
　　Exception及其子类是 Throwable 的一种形式，它指出了合理的应用程序想要捕获的条件。

3. RuntimeException 
　　RuntimeException是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。
　　编译器不会检查RuntimeException异常。例如，除数为零时，抛出ArithmeticException异常。RuntimeException是ArithmeticException的超类。当代码发生除数为零的情况时，倘若既"没有通过throws声明抛出ArithmeticException异常"，也"没有通过try...catch...处理该异常"，也能通过编译。这就是我们所说的"编译器不会检查RuntimeException异常"！
　　如果代码会产生RuntimeException异常，则需要通过修改代码进行避免。例如，若会发生除数为零的情况，则需要通过代码避免该情况的发生！

4. Error
　　和Exception一样，Error也是Throwable的子类。它用于指示合理的应用程序不应该试图捕获的严重问题，大多数这样的错误都是异常条件。
　　和RuntimeException一样，编译器也不会检查Error。

Java将可抛出(Throwable)的结构分为三种类型：被检查的异常(Checked Exception)，运行时异常(RuntimeException)和错误(Error)。

- 运行时异常
定义: RuntimeException及其子类都被称为运行时异常。
特点: Java编译器不会检查它。也就是说，当程序中可能出现这类异常时，倘若既"没有通过throws声明抛出它"，也"没有用try-catch语句捕获它"，还是会编译通过。例如，除数为零时产生的ArithmeticException异常，数组越界时产生的IndexOutOfBoundsException异常，fail-fail机制产生的ConcurrentModificationException异常等，都属于运行时异常。
　　虽然Java编译器不会检查运行时异常，但是我们也可以通过throws进行声明抛出，也可以通过try-catch对它进行捕获处理。
　　如果产生运行时异常，则需要通过修改代码来进行避免。例如，若会发生除数为零的情况，则需要通过代码避免该情况的发生！

- 被检查的异常
定义: Exception类本身，以及Exception的子类中除了"运行时异常"之外的其它子类都属于被检查异常。
特点: Java编译器会检查它。此类异常，要么通过throws进行声明抛出，要么通过try-catch进行捕获处理，否则不能通过编译。例如，CloneNotSupportedException就属于被检查异常。当通过clone()接口去克隆一个对象，而该对象对应的类没有实现Cloneable接口，就会抛出CloneNotSupportedException异常。
　　被检查异常通常都是可以恢复的。

- 错误
定义: Error类及其子类。
特点: 和运行时异常一样，编译器也不会对错误进行检查。
　　当资源不足、约束失败、或是其它程序无法继续运行的条件发生时，就产生错误。程序本身无法修复这些错误的。例如，VirtualMachineError就属于错误。
　　按照Java惯例，我们是不应该是实现任何新的Error子类的！

对于上面的3种结构，我们在抛出异常或错误时，到底该哪一种？《Effective Java》中给出的建议是：对于可以恢复的条件使用被检查异常，对于程序错误使用运行时异常

# 《Effective Java》中关于处理异常的建议
下面的内容对应原书中"第8章 异常"部分的第39-47条。

## 只针对不正常的情况才使用异常
建议：异常只应该被用于不正常的条件，它们永远不应该被用于正常的控制流。
通过比较下面的两份代码进行说明。
代码1
```java
try {
    int i=0;
    while (true) {
        arr[i]=0;
        i++;
    }
} catch (IndexOutOfBoundsException e) {
}
```

代码2
```
for (int i=0; i<arr.length; i++) {
    arr[i]=0;
}
```
两份代码的作用都是遍历arr数组，并设置数组中每一个元素的值为0。代码1的是通过异常来终止，看起来非常难懂，代码2是通过数组边界来终止。我们应该避免使用代码1这种方式，主要原因有三点：
- 异常机制的设计初衷是用于不正常的情况，所以很少会会JVM实现试图对它们的性能进行优化。所以，创建、抛出和捕获异常的开销是很昂贵的。
- 把代码放在try-catch中返回阻止了JVM实现本来可能要执行的某些特定的优化。
- 对数组进行遍历的标准模式并不会导致冗余的检查，有些现代的JVM实现会将它们优化掉。

实际上，基于异常的模式比标准模式要慢得多。测试代码如下：
```java
public class Advice1 {

    private static int[] arr = new int[]{1,2,3,4,5};
    private static int SIZE = 10000;

    public static void main(String[] args) {

        long s1 = System.currentTimeMillis();
        for (int i=0; i<SIZE; i++)
            endByRange(arr);
        long e1 = System.currentTimeMillis();
        System.out.println("endByRange time:"+(e1-s1)+"ms" );

        long s2 = System.currentTimeMillis();
        for (int i=0; i<SIZE; i++)
            endByException(arr);
        long e2 = System.currentTimeMillis();
        System.out.println("endByException time:"+(e2-s2)+"ms" );
    }

    // 遍历arr数组: 通过异常的方式
    private static void endByException(int[] arr) {
        try {
            int i=0;
            while (true) {
                arr[i]=0;
                i++;
                //System.out.println("endByRange: arr["+i+"]="+arr[i]);
            }
        } catch (IndexOutOfBoundsException e) {
        }
    }

    // 遍历arr数组: 通过边界的方式
    private static void endByRange(int[] arr) {
        for (int i=0; i<arr.length; i++) {
            arr[i]=0;
            //System.out.println("endByException: arr["+i+"]="+arr[i]);
        }
    }
}
```
运行结果：
```
endByRange time:0ms
endByException time:25ms
```
结果说明：通过异常遍历的速度比普通方式遍历数组慢很多。

## 对于可恢复的条件使用被检查的异常，对于程序错误使用运行时异常
- 运行时异常     -- RuntimeException类及其子类都被称为运行时异常。
- 被检查的异常 -- Exception类本身，以及Exception的子类中除了"运行时异常"之外的其它子类都属于被检查异常。

它们的区别是：Java编译器会对"被检查的异常"进行检查，而对"运行时异常"不会检查。也就是说，对于被检查的异常，要么通过throws进行声明抛出，要么通过try-catch进行捕获处理，否则不能通过编译。而对于运行时异常，倘若既"没有通过throws声明抛出它"，也"没有用try-catch语句捕获它"，还是会编译通过。当然，虽说Java编译器不会检查运行时异常，但是，我们同样可以通过throws对该异常进行说明，或通过try-catch进行捕获。
ArithmeticException(例如，除数为0)，IndexOutOfBoundsException(例如，数组越界)等都属于运行时异常。对于这种异常，我们应该通过修改代码进行避免它的产生。而对于被检查的异常，则可以通过处理让程序恢复运行。例如，假设因为一个用户没有存储足够数量的钱，所以他在企图在一个收费电话上进行呼叫就会失败；于是就将一个被检查异常抛出。

## 避免不必要的使用被检查的异常
"被检查的异常"是Java语言的一个很好的特性。与返回代码不同，"被检查的异常"会强迫程序员处理例外的条件，大大提高了程序的可靠性。
但是，过分使用被检查异常会使API用起来非常不方便。如果一个方法抛出一个或多个被检查的异常，那么调用该方法的代码则必须在一个或多个catch语句块中处理这些异常，或者必须通过throws声明抛出这些异常。 无论是通过catch处理，还是通过throws声明抛出，都给程序员添加了不可忽略的负担。
适用于"被检查的异常"必须同时满足两个条件：第一，即使正确使用API并不能阻止异常条件的发生。第二，一旦产生了异常，使用API的程序员可以采取有用的动作对程序进行处理。

## 尽量使用标准的异常
代码重用是值得提倡的，这是一条通用规则，异常也不例外。重用现有的异常有几个好处：
- 它使得你的API更加易于学习和使用，因为它与程序员原来已经熟悉的习惯用法是一致的。
- 对于用到这些API的程序而言，它们的可读性更好，因为它们不会充斥着程序员不熟悉的异常。
- 异常类越少，意味着内存占用越小，并且转载这些类的时间开销也越小。

Java标准异常中有几个是经常被使用的异常。如下表格：

异常 | 使用场景
-- | --
IllegalArgumentException | 非法参数
IllegalStateException | 非法参数状态
NullPointerException | 在null被禁止的情况下参数值为null(空指针)
IndexOutOfBoundsException | 数组下标越界
ConcurrentModificationException | 在禁止并发修改的情况下，对象检测到并发修改
UnsupportedOperationException | 对象不支持客户请求的方法

虽然它们是Java平台库迄今为止最常被重用的异常，但是，在许可的条件下，其它的异常也可以被重用。例如，如果你要实现诸如复数或者矩阵之类的算术对象，那么重用ArithmeticException和NumberFormatException将是非常合适的。如果一个异常满足你的需要，则不要犹豫，使用就可以，不过你一定要确保抛出异常的条件与该异常的文档中描述的条件一致。这种重用必须建立在语义的基础上，而不是名字的基础上。
最后，一定要清楚，选择重用哪一种异常并没有必须遵循的规则。例如，考虑纸牌对象的情形，假设有一个用于发牌操作的方法，它的参数(handSize)是发一手牌的纸牌张数。假设调用者在这个参数中传递的值大于整副牌的剩余张数。那么这种情形既可以被解释为IllegalArgumentException(handSize的值太大)，也可以被解释为IllegalStateException(相对客户的请求而言，纸牌对象的纸牌太少)。

## 抛出的异常要适合于相应的抽象
如果一个方法抛出的异常与它执行的任务没有明显的关联关系，这种情形会让人不知所措。当一个方法传递一个由低层抽象抛出的异常时，往往会发生这种情况。这种情况发生时，不仅让人困惑，而且也"污染"了高层API。
为了避免这个问题，高层实现应该捕获低层的异常，同时抛出一个可以按照高层抽象进行介绍的异常。这种做法被称为"异常转译(exception translation)"。

例如，在Java的集合框架AbstractSequentialList的get()方法如下(基于JDK1.7.0_40)：
```java
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}
```
listIterator(index)会返回ListIterator对象，调用该对象的next()方法可能会抛出NoSuchElementException异常。而在get()方法中，抛出NoSuchElementException异常会让人感到困惑。所以，get()对NoSuchElementException进行了捕获，并抛出了IndexOutOfBoundsException异常。即，相当于将NoSuchElementException转译成了IndexOutOfBoundsException异常。

## 每个方法抛出的异常都要有文档
要单独的声明被检查的异常，并且利用Javadoc的@throws标记，准确地记录下每个异常被抛出的条件。
如果一个类中的许多方法处于同样的原因而抛出同一个异常，那么在该类的文档注释中对这个异常做文档，而不是为每个方法单独做文档，这是可以接受的。

 

## 在细节消息中包含失败 -- 捕获消息
简而言之，当我们自定义异常或者抛出异常时，应该包含失败相关的信息。
当一个程序由于一个未被捕获的异常而失败的时候，系统会自动打印出该异常的栈轨迹。在栈轨迹中包含该异常的字符串表示。典型情况下它包含该异常类的类名，以及紧随其后的细节消息。

## 努力使失败保持原子性
当一个对象抛出一个异常之后，我们总期望这个对象仍然保持在一种定义良好的可用状态之中。对于被检查的异常而言，这尤为重要，因为调用者通常期望从被检查的异常中恢复过来。
一般而言，一个失败的方法调用应该保持使对象保持在"它在被调用之前的状态"。具有这种属性的方法被称为具有"失败原子性(failure atomic)"。可以理解为，失败了还保持着原子性。对象保持"失败原子性"的方式有几种：
- 设计一个非可变对象。
- 对于在可变对象上执行操作的方法，获得"失败原子性"的最常见方法是，在执行操作之前检查参数的有效性。如下(Stack.java中的pop方法)：
```java
public Object pop() {
    if (size==0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```
- 与上一种方法类似，可以对计算处理过程调整顺序，使得任何可能会失败的计算部分都发生在对象状态被修改之前。 
- 编写一段恢复代码，由它来解释操作过程中发生的失败，以及使对象回滚到操作开始之前的状态上。
- 在对象的一份临时拷贝上执行操作，当操作完成之后再把临时拷贝中的结果复制给原来的对象。
虽然"保持对象的失败原子性"是期望目标，但它并不总是可以做得到。例如，如果多个线程企图在没有适当的同步机制的情况下，并发的访问一个对象，那么该对象就有可能被留在不一致的状态中。

即使在可以实现"失败原子性"的场合，它也不是总被期望的。对于某些操作，它会显著的增加开销或者复杂性。
总的规则是：作为方法规范的一部分，任何一个异常都不应该改变对象调用该方法之前的状态，如果这条规则被违反，则API文档中应该清楚的指明对象将会处于什么样的状态。

## 不要忽略异常
当一个API的设计者声明一个方法会抛出某个异常的时候，他们正在试图说明某些事情。所以，请不要忽略它！忽略异常的代码如下：
```java
try {
    ...
} catch (SomeException e) {
}
```
空的catch块会使异常达不到应有的目的，异常的目的是强迫你处理不正常的条件。忽略一个异常，就如同忽略一个火警信号一样 -- 若把火警信号器关闭了，那么当真正的火灾发生时，就没有人看到火警信号了。所以，至少catch块应该包含一条说明，用来解释为什么忽略这个异常是合适的。

# 《Java Puzzles》中关于异常的几个问题
## 优柔寡断
下面的程序打印什么？
```java
public class Indecisive {

    public static void main(String[] args) {
        System.out.println(decision());
    }

    private static boolean decision() {
        try {
            return true;
        } finally {
            return false;
        }
    }
}
```
运行结果：
false

结果说明：
在一个 try-finally 语句中，finally 语句块总是在控制权离开 try 语句块时执行的。无论 try 语句块是正常结束的，还是意外结束的, 情况都是如此。
一条语句或一个语句块在它抛出了一个异常，或者对某个封闭型语句执行了一个 break 或 continue，或是象这个程序一样在方法中执行了一个return 时，将发生意外结束。它们之所以被称为意外结束,是因为它们阻止程序去按顺序执行下面的语句。当 try 语句块和 finally 语句块都意外结束时，try 语句块中引发意外结束的原因将被丢弃，而整个 try-finally 语句意外结束的原因将于 finally 语句块意外结束的原因相同。在这个程序中，在 try 语句块中的 return 语句所引发的意外结束将被丢弃, try-finally 语句意外结束是由 finally 语句块中的 return 而造成的。

简单地讲，程序尝试着 (try) (return) 返回 true，但是它最终 (finally) 返回(return)的是 false。丢弃意外结束的原因几乎永远都不是你想要的行为，因为意外结束的最初原因可能对程序的行为来说会显得更重要。对于那些在 try 语句块中执行 break、continue 或 return 语句，只是为了使其行为被 finally 语句块所否决掉的程序,要理解其行为是特别困难的。总之，每一个 finally 语句块都应该正常结束，除非抛出的是不受检查的异常。 千万不要用一个 return、break、continue 或 throw 来退出一个 finally 语句块，并且千万不要允许将一个受检查的异常传播到一个 finally 语句块之外去。对于语言设计者，也许应该要求 finally 语句块在未出现不受检查的异常时必须正常结束。朝着这个目标,try-finally 结构将要求 finally 语句块可以正常结束。return、break 或 continue 语句把控制权传递到 finally 语句块之外应该是被禁止的，任何可以引发将被检查异常传播到 finally 语句块之外的语句也同样应该是被禁止的。

## 不可思议
下面的三个程序每一个都会打印些什么? 不要假设它们都可以通过编译。
第一个程序
```java
import java.io.IOException;

public class Arcane1 {

    public static void main(String[] args) {
        try {
            System.out.println("Hello world");
        } catch(IOException e) {
            System.out.println("I've never seen println fail!");
        }
    }
}
```

第二个程序
```java
public class Arcane2 {
    public static void main(String[] args) {
        try {
            // If you have nothing nice to say, say nothing
        } catch(Exception e) {
            System.out.println("This can't happen");
        }
    }
}
```

第三个程序
```java
interface Type1 {
    void f() throws CloneNotSupportedException;
}

interface Type2 {
    void f() throws InterruptedException;
}

interface Type3 extends Type1, Type2 {
}

public class Arcane3 implements Type3 {
    public void f() {
        System.out.println("Hello world");
    }
    public static void main(String[] args) {
        Type3 t3 = new Arcane3();
        t3.f();
    }
}
```
运行结果：
- 第一个程序编译出错！
```
Arcane1.java:9: exception java.io.IOException is never thrown in body of corresponding try statement
        } catch(IOException e) {
          ^
1 error
```
- 第二个程序能正常编译和运行。
- 第三个程序能正常编译和运行。输出结果是: Hello world

结果说明：
- Arcane1展示了被检查异常的一个基本原则。它看起来应该是可以编译的:try 子句执行 I/O,并且 catch 子句捕获 IOException 异常。但是这个程序不能编译,因为 println 方法没有声明会抛出任何被检查异常,而IOException 却正是一个被检查异常。语言规范中描述道:如果一个 catch 子句要捕获一个类型为 E 的被检查异常, 而其相对应的 try 子句不能抛出 E 的某种子类型的异常,那么这就是一个编译期错误。

- 基于同样的理由,第二个程序,Arcane2,看起来应该是不可以编译的,但是它却可以。它之所以可以编译,是因为它唯一的 catch 子句检查了 Exception。尽管在这一点上十分含混不清,但是捕获 Exception 或 Throwble 的 catch 子句是合法的,不管与其相对应的 try 子句的内容为何。尽管 Arcane2 是一个合法的程序,但是 catch 子句的内容永远的不会被执行,这个程序什么都不会打印。

- 第三个程序,Arcane3,看起来它也不能编译。方法 f 在 Type1 接口中声明要抛出被检查异常 CloneNotSupportedException,并且在 Type2 接口中声明要抛出被检查异常 InterruptedException。Type3 接口继承了 Type1 和 Type2,因此, 看起来在静态类型为 Type3 的对象上调用方法 f 时, 有潜在可能会抛出这些异常。一个方法必须要么捕获其方法体可以抛出的所有被检查异常, 要么声明它将抛出这些异常。Arcane3 的 main 方法在静态类型为 Type3 的对象上调用了方法 f,但它对 CloneNotSupportedException 和 InterruptedExceptioin 并没有作这些处理。那么,为什么这个程序可以编译呢? 
　　上述分析的缺陷在于对“Type3.f 可以抛出在 Type1.f 上声明的异常和在 Type2.f 上声明的异常”所做的假设。这并不正确,因为每一个接口都限制了方法 f 可以抛出的被检查异常集合。一个方法可以抛出的被检查异常集合是它所适用的所有类型声明要抛出的被检查异常集合的交集,而不是合集。因此,静态类型为 Type3 的对象上的 f 方法根本就不能抛出任何被检查异常。因此,Arcane3可以毫无错误地通过编译,并且打印 Hello world。

## 不受欢迎的宾客
下面的程序会打印出什么呢?
```java
public class UnwelcomeGuest {
    public static final long GUEST_USER_ID = -1;
    private static final long USER_ID;

    static {
        try {
            USER_ID = getUserIdFromEnvironment();
        } catch (IdUnavailableException e) {
            USER_ID = GUEST_USER_ID;
            System.out.println("Logging in as guest");
        }
    }

    private static long getUserIdFromEnvironment() 
        throws IdUnavailableException {
        throw new IdUnavailableException();
    }

    public static void main(String[] args) {
        System.out.println("User ID: " + USER_ID);
    }
}

class IdUnavailableException extends Exception {
}
```
运行结果：
```
UnwelcomeGuest.java:10: variable USER_ID might already have been assigned
            USER_ID = GUEST_USER_ID;
            ^
1 error
```
结果说明：
该程序看起来很直观。对 getUserIdFromEnvironment 的调用将抛出一个异常, 从而使程序将 GUEST_USER_ID(-1L)赋值给 USER_ID, 并打印 Loggin in as guest。 然后 main 方法执行,使程序打印 User ID: -1。表象再次欺骗了我们,该程序并不能编译。如果你尝试着去编译它, 你将看到和一条错误信息。

问题出在哪里了?USER_ID 域是一个空 final(blank final),它是一个在声明中没有进行初始化操作的 final 域。很明显,只有在对 USER_ID 赋值失败时,才会在 try 语句块中抛出异常,因此,在 catch 语句块中赋值是相 当安全的。不管怎样执行静态初始化操作语句块,只会对 USER_ID 赋值一次,这正是空 final 所要求的。为什么编译器不知道这些呢? 要确定一个程序是否可以不止一次地对一个空 final 进行赋值是一个很困难的问题。事实上,这是不可能的。这等价于经典的停机问题,它通常被认为是不可能解决的。为了能够编写出一个编译器,语言规范在这一点上采用了保守的方式。在程序中,一个空 final 域只有在它是明确未赋过值的地方才可以被赋值。规范长篇大论,对此术语提供了一个准确的但保守的定义。 因为它是保守的,所以编译器必须拒绝某些可以证明是安全的程序。这个谜题就展示了这样的一个程序。幸运的是, 你不必为了编写 Java 程序而去学习那些骇人的用于明确赋值的细节。通常明确赋值规则不会有任何妨碍。如果碰巧你编写了一个真的可能会对一个空final 赋值超过一次的程序,编译器会帮你指出的。只有在极少的情况下,就像本谜题一样, 你才会编写出一个安全的程序, 但是它并不满足规范的形式化要求。编译器的抱怨就好像是你编写了一个不安全的程序一样,而且你必须修改你的程序以满足它。

解决这类问题的最好方式就是将这个烦人的域从空 final 类型改变为普通的final 类型,用一个静态域的初始化操作替换掉静态的初始化语句块。实现这一点的最佳方式是重构静态语句块中的代码为一个助手方法:

```java
public class UnwelcomeGuest {
    public static final long GUEST_USER_ID = -1;
    private static final long USER_ID = getUserIdOrGuest();
    private static long getUserIdOrGuest() {
        try {
            return getUserIdFromEnvironment();
        } catch (IdUnavailableException e) {
            System.out.println("Logging in as guest");
            return GUEST_USER_ID;
        }
    }

    private static long getUserIdFromEnvironment() 
        throws IdUnavailableException {
        throw new IdUnavailableException();
    }

    public static void main(String[] args) {
        System.out.println("User ID: " + USER_ID);
    }
}

class IdUnavailableException extends Exception {
}
```
程序的这个版本很显然是正确的,而且比最初的版本根据可读性,因为它为了域值的计算而增加了一个描述性的名字, 而最初的版本只有一个匿名的静态初始化操作语句块。将这样的修改作用于程序,它就可以如我们的期望来运行了。总之,大多数程序员都不需要学习明确赋值规则的细节。该规则的作为通常都是正确的。如果你必须重构一个程序,以消除由明确赋值规则所引发的错误,那么你应该考虑添加一个新方法。这样做除了可以解决明确赋值问题,还可以使程序的可读性提高。

## 您好,再见!
下面的程序将会打印出什么呢?
```java
public class HelloGoodbye {
    public static void main(String[] args) {
        try {
            System.out.println("Hello world");
            System.exit(0);
        } finally {
            System.out.println("Goodbye world");
        }
    }
}
```
运行结果:
`Hello world`

结果说明：
这个程序包含两个 println 语句: 一个在 try 语句块中, 另一个在相应的 finally语句块中。try 语句块执行它的 println 语句,并且通过调用 System.exit 来提前结束执行。在此时,你可能希望控制权会转交给 finally 语句块。然而,如果你运行该程序,就会发现它永远不会说再见:它只打印了 Hello world。这是否违背了"Indecisive示例" 中所解释的原则呢? 不论 try 语句块的执行是正常地还是意外地结束, finally 语句块确实都会执行。然而在这个程序中,try 语句块根本就没有结束其执行过程。System.exit 方法将停止当前线程和所有其他当场死亡的线程。finally 子句的出现并不能给予线程继续去执行的特殊权限。
当 System.exit 被调用时,虚拟机在关闭前要执行两项清理工作。首先,它执行所有的关闭挂钩操作,这些挂钩已经注册到了 Runtime.addShutdownHook 上。这对于释放 VM 之外的资源将很有帮助。务必要为那些必须在 VM 退出之前发生的行为关闭挂钩。下面的程序版本示范了这种技术,它可以如我们所期望地打印出 Hello world 和 Goodbye world:

```java
public class HelloGoodbye1 {
    public static void main(String[] args) {
        System.out.println("Hello world");
        Runtime.getRuntime().addShutdownHook(
        new Thread() {
            public void run() {
            System.out.println("Goodbye world");
            }
        });
        System.exit(0);
    }
}
```
VM 执行在 System.exit 被调用时执行的第二个清理任务与终结器有关。如果System.runFinalizerOnExit 或它的魔鬼双胞胎 Runtime.runFinalizersOnExit被调用了,那么 VM 将在所有还未终结的对象上面调用终结器。这些方法很久以前就已经过时了,而且其原因也很合理。无论什么原因,永远不要调用System.runFinalizersOnExit 和 Runtime.runFinalizersOnExit: 它们属于 Java类库中最危险的方法之一[ThreadStop]。调用这些方法导致的结果是,终结器会在那些其他线程正在并发操作的对象上面运行, 从而导致不确定的行为或导致死锁。

总之，System.exit 将立即停止所有的程序线程,它并不会使 finally 语句块得到调用,但是它在停止 VM 之前会执行关闭挂钩操作。当 VM 被关闭时，请使用关闭挂钩来终止外部资源。通过调用 System.halt 可以在不执行关闭挂钩的情况下停止 VM，但是这个方法很少使用。

## 不情愿的构造器
下面的程序将打印出什么呢?
```java
public class Reluctant {
    private Reluctant internalInstance = new Reluctant();
    public Reluctant() throws Exception {
        throw new Exception("I'm not coming out");
    }
    public static void main(String[] args) {
        try {
            Reluctant b = new Reluctant();
            System.out.println("Surprise!");
        } catch (Exception ex) {
            System.out.println("I told you so");
        }
    }
}
```
运行结果：
```
Exception in thread "main" java.lang.StackOverflowError
    at Reluctant.<init>(Reluctant.java:3)
    ...
```
结果说明：
main 方法调用了 Reluctant 构造器,它将抛出一个异常。你可能期望 catch 子句能够捕获这个异常,并且打印 I told you so。凑近仔细看看这个程序就会发现,Reluctant 实例还包含第二个内部实例,它的构造器也会抛出一个异常。无论抛出哪一个异常,看起来 main 中的 catch 子句都应该捕获它,因此预测该程序将打印 I told you 应该是一个安全的赌注。但是当你尝试着去运行它时,就会发现它压根没有去做这类的事情:它抛出了 StackOverflowError 异常,为什么呢?

与大多数抛出 StackOverflowError 异常的程序一样,本程序也包含了一个无限递归。当你调用一个构造器时,实例变量的初始化操作将先于构造器的程序体而运行[JLS 12.5]。在本谜题中, internalInstance 变量的初始化操作递归调用了构造器,而该构造器通过再次调用 Reluctant 构造器而初始化该变量自己的 internalInstance 域,如此无限递归下去。这些递归调用在构造器程序体获得执行机会之前就会抛出 StackOverflowError 异常,因为 StackOverflowError 是 Error 的子类型而不是 Exception 的子类型,所以 catch 子句无法捕获它。对于一个对象包含与它自己类型相同的实例的情况,并不少见。例如,链接列表节点、树节点和图节点都属于这种情况。你必须非常小心地初始化这样的包含实例,以避免 StackOverflowError 异常。

至于本谜题名义上的题目:声明将抛出异常的构造器,你需要注意,构造器必须声明其实例初始化操作会抛出的所有被检查异常。

## 域和流
下面的方法将一个文件拷贝到另一个文件,并且被设计为要关闭它所创建的每一个流,即使它碰到 I/O 错误也要如此。遗憾的是,它并非总是能够做到这一点。为什么不能呢,你如何才能订正它呢?
```java
static void copy(String src, String dest) throws IOException {
    InputStream in = null;
    OutputStream out = null;
    try {
        in = new FileInputStream(src);
        out = new FileOutputStream(dest);
        byte[] buf = new byte[1024];
        int n;
        while ((n = in.read(buf)) > 0)
            out.write(buf, 0, n);
    } finally {
        if (in != null) in.close();
        if (out != null) out.close();
    }
}
```
谜题分析：
这个程序看起来已经面面俱到了。其流域(in 和 out)被初始化为 null,并且新的流一旦被创建,它们马上就被设置为这些流域的新值。对于这些域所引用的流,如果不为空,则 finally 语句块会将其关闭。即便在拷贝操作引发了一个 IOException 的情况下,finally 语句块也会在方法返回之前执行。出什么错了呢?
问题在 finally 语句块自身中。close 方法也可能会抛出 IOException 异常。如果这正好发生在 in.close 被调用之时,那么这个异常就会阻止 out.close 被调用,从而使输出流仍保持在开放状态。请注意,该程序违反了"优柔寡断" 的建议:对 close 的调用可能会导致 finally 语句块意外结束。遗憾的是,编译器并不能帮助你发现此问题,因为 close 方法抛出的异常与 read 和 write 抛出的异常类型相同,而其外围方法(copy)声明将传播该异常。解决方式是将每一个 close 都包装在一个嵌套的 try 语句块中。

下面的 finally 语句块的版本可以保证在两个流上都会调用 close:
```java
try {
    // 和之前一样
} finally {
    if (in != null) {
        try {
            in.close();
        } catch (IOException ex) {
            // There is nothing we can do if close fails
        }
    }

    if (out != null) {
        try {
            out.close();
        } catch (IOException ex) {
            // There is nothing we can do if close fails
        }
    }
}
```
总之,当你在 finally 语句块中调用 close 方法时,要用一个嵌套的 try-catch 语句来保护它,以防止 IOException 的传播。更一般地讲,对于任何在 finally 语句块中可能会抛出的被检查异常都要进行处理,而不是任其传播。

## 异常为循环而抛
下面的程序会打印出什么呢?
```java
public class Loop {
    public static void main(String[] args) {
        int[][] tests = { { 6, 5, 4, 3, 2, 1 }, { 1, 2 },
            { 1, 2, 3 }, { 1, 2, 3, 4 }, { 1 } };
        int successCount = 0;
        try {
            int i = 0;
            while (true) {
                if (thirdElementIsThree(tests[i++]))
                    successCount ++;
            }
        } catch(ArrayIndexOutOfBoundsException e) {
            // No more tests to process
        }
        System.out.println(successCount);
    }
    private static boolean thirdElementIsThree(int[] a) {
        return a.length >= 3 & a[2] == 3;
    }
}
```
运行结果：
`0`

结果说明：
该程序主要说明了两个问题。
- 不应该使用异常作为终止循环的手段！
该程序用 thirdElementIsThree 方法测试了 tests 数组中的每一个元素。遍历这个数组的循环显然是非传统的循环:它不是在循环变量等于数组长度的时候终止,而是在它试图访问一个并不在数组中的元素时终止。尽管它是非传统的,但是这个循环应该可以工作。
如果传递给 thirdElementIsThree 的参数具有 3 个或更多的元素,并且其第三个元素等于 3,那么该方法将返回 true。对于 tests中的 5 个元素来说,有 2 个将返回 true,因此看起来该程序应该打印 2。如果你运行它,就会发现它打印的时 0。肯定是哪里出了问题,你能确定吗? 事实上,这个程序犯了两个错误。第一个错误是该程序使用了一种可怕的循环惯用法,该惯用法依赖的是对数组的访问会抛出异常。这种惯用法不仅难以阅读, 而且运行速度还非常地慢。不要使用异常来进行循环控制;应该只为异常条件而使用异常。为了纠正这个错误,可以将整个 try-finally 语句块替换为循环遍历数组的标准惯用法:
```
for (int i = 0; i < test.length; i++)
    if (thirdElementIsThree(tests[i]))
        successCount++;
```
如果你使用的是 5.0 或者是更新的版本,那么你可以用 for 循环结构来代替:
```java
for (int[] test : tests)
    if(thirdElementIsThree(test))
        successCount++;
```

- 主要比较"&操作符" 和 "&&操作符"的区别。注意示例中的操作符是&，这是按位进行"与"操作。

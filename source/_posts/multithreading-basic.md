---
title: multithreading-basic
tags: multithreading
date: 2019-02-04 18:21:30
---

# 线程状态
线程状态图：
![](multithreading-basic/1.jpg)

线程共包括以下五种状态：
1. 新建(new)：线程对象被创建后，进入新建状态。如：Thread thread = new Thread();
2. 就绪(Runnable)：可执行状态。线程对象被创建后，其他线程调用了该对象的start方法，来启动该线程。如：thread.start()。处于就绪状态的线程，随时可能被CPU调度执行。
3. 运行(Running)：线程获取CPU权限进行执行。注意：线程只能从就绪状态进入到运行状态。
4. 阻塞(Blocked)：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会继续运行。阻塞的情况分三种：
    - 等待阻塞：通过调用线程的wait()方法，让线程等待某工作完成。
    - 同步阻塞：线程获取synchronized同步锁失败(因为锁被其他线程占用)，它会进入同步阻塞状态。
    - 其他阻塞：通过调用线程的sleep()或join()或发出I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或超时、I/O处理完毕，线程会重新进入就绪状态。
5. 死亡(Dead)：线程执行完了或因异常退出了run()方法，该线程结束生命周期。

这5种状态涉及到的内容包括Object类，Thread和synchronized关键字。
Object类，定义了wait()，notify()，notifyAll()等休眠/唤醒函数。
Thread类，定义了一些列的线程操作函数。例如，sleep()休眠函数，interrupt()中断函数，getName()获取线程名称等。
synchronized，是关键字，它区分为synchronized代码块和synchronized方法。synchronized的作用是让线程获取对象的同步锁。
在后面详细介绍wait()，notify()等方法时，我们会分析为什么wait()，notify()等方法要定义在Object类，而不是Thread类中。

# 实现多线程的方式
常用的实现多线程的方式包括：`Thread`和`Runnable`，还可以通过`java.util.concurrent`包中的线程池来实现多线程。

## Thread和Runnable简介
Runnable 是一个接口，该接口中只包含了一个run()方法。它的定义如下：
```java
public interface Runnable {
    public abstract void run();
}
```
Runnable用来实现多线程。我们可以定义一个类A实现Runnable接口，然后，通过new Thread(new A())等方式新建线程。

Thread 是一个类。Thread本身就实现了Runnable接口。它的声明如下：
`public class Thread implements Runnable {}`
Thread也可以用来实现多线程。

## Thread和Runnable的异同点
不同点：
Thread 是类，而Runnable是接口；Thread本身是实现了Runnable接口的类。我们知道“一个类只能有一个父类，但是却能实现多个接口”，因此Runnable具有更好的扩展性。
此外，Runnable还可以用于“资源的共享”。即，多个线程都是基于某一个Runnable对象建立的，它们会共享Runnable对象上的资源。
通常，建议通过`Runnable`实现多线程。

## Thread多线程示例
```java
// ThreadTest.java 源码
class MyThread extends Thread{
    private int ticket=10;
    public void run(){
        for(int i=0;i<20;i++){
            if(this.ticket>0){
                System.out.println(this.getName()+" 卖票：ticket"+this.ticket--);
            }
        }
    }
};

public class ThreadTest {
    public static void main(String[] args) {
        // 启动3个线程t1,t2,t3；每个线程各卖10张票
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
    }
}
```
运行结果：
```
Thread-1 卖票：ticket10
Thread-1 卖票：ticket9
Thread-1 卖票：ticket8
Thread-1 卖票：ticket7
Thread-1 卖票：ticket6
Thread-1 卖票：ticket5
Thread-1 卖票：ticket4
Thread-1 卖票：ticket3
Thread-1 卖票：ticket2
Thread-1 卖票：ticket1
Thread-2 卖票：ticket10
Thread-2 卖票：ticket9
Thread-2 卖票：ticket8
Thread-2 卖票：ticket7
Thread-2 卖票：ticket6
Thread-2 卖票：ticket5
Thread-2 卖票：ticket4
Thread-2 卖票：ticket3
Thread-2 卖票：ticket2
Thread-2 卖票：ticket1
Thread-0 卖票：ticket10
Thread-0 卖票：ticket9
Thread-0 卖票：ticket8
Thread-0 卖票：ticket7
Thread-0 卖票：ticket6
Thread-0 卖票：ticket5
Thread-0 卖票：ticket4
Thread-0 卖票：ticket3
Thread-0 卖票：ticket2
Thread-0 卖票：ticket1
```
说明：
- MyThread继承于Thread，它是个自定义线程。每个MyThread都会卖出10张票。
- 主线程main创建并启动3个MyThread子线程。每个子线程都各自卖出了10张票。

## Runnable多线程示例
下面，我们对上面的程序进行修改。通过Runnable实现一个接口，从而实现多线程。
```java
// RunnableTest.java 源码
class MyThread implements Runnable{
    private int ticket=10;
    public void run(){
        for(int i=0;i<20;i++){
            if(this.ticket>0){
                System.out.println(Thread.currentThread().getName()+" 卖票：ticket"+this.ticket--);
            }
        }
    }
};

public class RunnableTest {
    public static void main(String[] args) {
        MyThread mt=new MyThread();
        // 启动3个线程t1,t2,t3(它们共用一个Runnable对象)，这3个线程一共卖10张票
        Thread t1=new Thread(mt);
        Thread t2=new Thread(mt);
        Thread t3=new Thread(mt);
        t1.start();
        t2.start();
        t3.start();
    }
}
```

结果：
```
Thread-1 卖票：ticket10
Thread-1 卖票：ticket9
Thread-1 卖票：ticket8
Thread-1 卖票：ticket7
Thread-1 卖票：ticket6
Thread-1 卖票：ticket5
Thread-1 卖票：ticket4
Thread-0 卖票：ticket10
Thread-0 卖票：ticket3
Thread-0 卖票：ticket2
Thread-0 卖票：ticket1
```

说明：
- 和上面“MyThread继承于Thread”不同；这里的MyThread实现了Runnable接口。
- 主线程main创建并启动3个子线程，而且这3个子线程都是基于“mt这个Runnable对象”而创建的。运行结果是这3个子线程一共卖出了10张票。这说明它们是共享了MyThread接口的。

需要注意，这里我们把main方法和线程类放在一个类中，三条线程从开始循环就会出现并发现象，如果把main方法和线程类分开写，三条线程也会并发，不过需要把循环次数都调大(100左右才能看到效果)。
至于为什么内部类开始就并发，分开写并发出现的比较晚，目前还不得而知。

# Thread中start和run的区别
## 区别说明
start()：启动一个新线程，新线程会执行相应的run()方法，start()不能被重复调用。
run()：和普通成员方法一样，可以被重复调用。若单独调用run()，会在当前线程中执行run()，而并不会启动新线程。

```
class MyThread extends Thread{
    public void run(){
        ...
    } 
};
MyThread mythread = new MyThread();
```
`mythread.start()`会启动一个新线程，并在新线程中运行run()方法。
而`mythread.run()`则会直接在当前线程中运行run()方法，并不会启动一个新线程来运行run()。

## 区别示例
```java
// Demo.java 的源码
class MyThread extends Thread{
    public MyThread(String name) {
        super(name);
    }

    public void run(){
        System.out.println(Thread.currentThread().getName()+" is running");
    } 
}; 

public class Demo {
    public static void main(String[] args) {
        Thread mythread=new MyThread("mythread");

        System.out.println(Thread.currentThread().getName()+" call mythread.run()");
        mythread.run();

        System.out.println(Thread.currentThread().getName()+" call mythread.start()");
        mythread.start();
    }
}
```
运行结果：
```
main call mythread.run()
main is running
main call mythread.start()
mythread is running
```

说明：
- `Thread.currentThread().getName()`指获取当前线程的名字。当前线程指正在CPU中执行的线程。
- `mythread.run()`是在`主线程main`中调用的，该run()方法直接运行在`主线程main`上。
- `mythread.start()`会启动`线程mythread`，`线程mythread`启动之后，会调用run()方法；此时的run()方法是运行在`线程mythread`上。

## start()和run()的源码分析
start()
```java
public synchronized void start() {
    // 如果线程不是"就绪状态"，则抛出异常！
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    // 将线程添加到ThreadGroup中
    group.add(this);

    boolean started = false;
    try {
        // 通过start0()启动线程
        start0();
        // 设置started标记
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
    }
}
```
说明：start()实际上是通过本地方法start0()启动线程的。而start0()会新运行一个线程，新线程会调用run()方法。
`private native void start0();`

run()
```java
public void run() {
    if (target != null) {
        target.run();
    }
}
```

说明：target是一个Runnable对象。run()就是直接调用Thread线程的Runnable成员的run()方法，并不会新建一个线程。


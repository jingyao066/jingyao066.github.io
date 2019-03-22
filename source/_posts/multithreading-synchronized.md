---
title: multithreading-synchronized
tags: multithreading
date: 2019-02-15 17:44:00
---

# synchronized原理
**在java中，每一个对象都有且仅有一个同步锁，这也意味着，同步锁是依赖对象而存在的。**
当调用对象的synchronized方法时，就获取了该对象的同步锁。如：synchronized(obj)，就获取了obj这个对象的同步锁。

不同线程对同步锁的访问是互斥的。即某个时间点，对象的同步锁只能被一个线程获取到。
通过同步锁，我们可以在多线程中实现对"对象/方法"的互斥访问。如：现在有两个线程a和b，它们都会访问对象obj的同步锁。
假设，在某一时刻，线程A获取到“obj的同步锁”并在执行一些操作；而此时，线程B也企图获取“obj的同步锁” —— 线程B会获取失败，它必须等待，直到线程A释放了“该对象的同步锁”之后线程B才能获取到“obj的同步锁”从而才可以运行。

# synchronized基本规则
现将synchronized的基本规则总结为下面3条，并通过实例对它们进行说明。
第一条: 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的该“synchronized方法”或者“synchronized代码块”的访问将被阻塞。
第二条: 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程仍然可以访问“该对象”的非同步代码块。
第三条: 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的其他的“synchronized方法”或者“synchronized代码块”的访问将被阻塞。


## 第一条
当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的该“synchronized方法”或者“synchronized代码块”的访问将被阻塞。
下面是“synchronized代码块”对应的演示程序。
```
public class MyRunable implements Runnable{
    @Override
    public void run() {
        synchronized (this){
            try {
                for (int i = 0; i < 5; i++) {
                    Thread.sleep(100);//休眠100ms
                    System.out.println(Thread.currentThread().getName() + " loop " + i);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

public class Demo1 {
    public static void main(String[] args) {
        Runnable demo = new MyRunable();
        Thread t1 = new Thread(demo, "t1");  // 新建“线程t1”, t1是基于demo这个Runnable对象
        Thread t2 = new Thread(demo, "t2");  // 新建“线程t2”, t2是基于demo这个Runnable对象
        t1.start();                                // 启动“线程t1”
        t2.start();                                //启动“线程t2”
    }
}
```
结果：
```
t1 loop 0
t1 loop 1
t1 loop 2
t1 loop 3
t1 loop 4
t2 loop 0
t2 loop 1
t2 loop 2
t2 loop 3
t2 loop 4
```
说明：
run()方法中存在“synchronized(this)代码块”，而且t1和t2都是基于"demo这个Runnable对象"创建的线程。
这就意味着，我们可以将synchronized(this)中的this看作是“demo这个Runnable对象”；因此，线程t1和t2共享“demo对象的同步锁”。
所以，当一个线程运行的时候，另外一个线程必须等待“运行线程”释放“demo的同步锁”之后才能运行。

如果你确认，你搞清楚这个问题了。那我们将上面的代码进行修改，然后再运行看看结果怎么样，看看你是否会迷糊。修改后的源码如下：
```
class MyThread extends Thread {
    
    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        synchronized(this) {
            try {  
                for (int i = 0; i < 5; i++) {
                    Thread.sleep(100); // 休眠100ms
                    System.out.println(Thread.currentThread().getName() + " loop " + i);  
                }
            } catch (InterruptedException ie) {  
            }
        }  
    }
}

public class Demo1_2 {

    public static void main(String[] args) {  
        Thread t1 = new MyThread("t1");  // 新建“线程t1”
        Thread t2 = new MyThread("t2");  // 新建“线程t2”
        t1.start();                          // 启动“线程t1”
        t2.start();                          // 启动“线程t2” 
    } 
}
```
代码说明：
比较Demo1_2 和 Demo1_1，我们发现，Demo1_2中的MyThread类是直接继承于Thread，而且t1和t2都是MyThread子线程。
幸运的是，在“Demo1_2的run()方法”也调用了synchronized(this)，正如“Demo1_1的run()方法”也调用了synchronized(this)一样！
那么，Demo1_2的执行流程是不是和Demo1_1一样呢？

结果：
```
t1 loop 0
t2 loop 0
t1 loop 1
t2 loop 1
t1 loop 2
t2 loop 2
t1 loop 3
t2 loop 3
t1 loop 4
t2 loop 4
```

结果说明：
如果这个结果一点也不令你感到惊讶，那么我相信你对synchronized和this的认识已经比较深刻了。否则的话，请继续阅读这里的分析。
synchronized(this)中的this是指“当前的类对象”，即synchronized(this)所在的类对应的当前对象。它的作用是获取“当前对象的同步锁”。
对于Demo1_2中，synchronized(this)中的this代表的是MyThread对象，而t1和t2是两个不同的MyThread对象，因此t1和t2在执行synchronized(this)时，
获取的是不同对象的同步锁。对于Demo1_1对而言，synchronized(this)中的this代表的是MyRunable对象；t1和t2共同一个MyRunable对象，因此，一个线程获取了对象的同步锁，会造成另外一个线程等待。

## 第二条

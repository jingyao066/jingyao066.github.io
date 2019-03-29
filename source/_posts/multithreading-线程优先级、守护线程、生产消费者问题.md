---
title: multithreading-线程优先级、守护线程、生产消费者问题
tags: multithreading
date: 2019-03-28 16:23:30
---

# 线程优先级
java 中的线程优先级的范围是1～10，默认的优先级是5。10表示最高优先级，1表示最低优先级，5是普通优先级。“高优先级线程”会优先于“低优先级线程”执行。java 中的线程优先级的范围是1～10，默认的优先级是5。“高优先级线程”会优先于“低优先级线程”执行。
java 中有两种线程：用户线程和守护线程。可以通过isDaemon()方法来区别它们：如果返回false，则说明该线程是“用户线程”；否则就是“守护线程”。
用户线程一般用户执行用户级任务，而守护线程也就是“后台线程”，一般用来执行后台任务，作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)。
用户线程和守护线程两者几乎没有区别，唯一的不同之处就在于虚拟机的离开。需要注意的是：Java虚拟机在“用户线程”都结束后会后退出。

JDK 中关于线程优先级和守护线程的介绍如下：
Every thread has a priority. Threads with higher priority are executed in preference to threads with lower priority. Each thread may or may not also be marked as a daemon. When code running in some thread creates a new Thread object, the new thread has its priority initially set equal to the priority of the creating thread, and is a daemon thread if and only if the creating thread is a daemon.

When a Java Virtual Machine starts up, there is usually a single non-daemon thread (which typically calls the method named main of some designated class). The Java Virtual Machine continues to execute threads until either of the following occurs:

The exit method of class Runtime has been called and the security manager has permitted the exit operation to take place.
All threads that are not daemon threads have died, either by returning from the call to the run method or by throwing an exception that propagates beyond the run method. 
Marks this thread as either a daemon thread or a user thread. The Java Virtual Machine exits when the only threads running are all daemon threads.
意思是：
每个线程都有一个优先级。“高优先级线程”会优先于“低优先级线程”执行。每个线程都可以被标记为一个守护进程或非守护进程。在一些运行的主线程中创建新的子线程时，子线程的优先级被设置为等于“创建它的主线程的优先级”，当且仅当“创建它的主线程是守护线程”时“子线程才会是守护线程”。

当Java虚拟机启动时，通常有一个单一的非守护线程（该线程通过是通过main()方法启动）。JVM会一直运行直到下面的任意一个条件发生，JVM就会终止运行：
(01) 调用了exit()方法，并且exit()有权限被正常执行。
(02) 所有的“非守护线程”都死了(即JVM中仅仅只有“守护线程”)。

每一个线程都被标记为“守护线程”或“用户线程”。当只有守护线程运行时，JVM会自动退出。

# 线程优先级示例
```
class MyThread extends Thread{
    public MyThread(String name) {
        super(name);
    }

    public void run(){
        for (int i=0; i<5; i++) {
            System.out.println(Thread.currentThread().getName()
                    +"("+Thread.currentThread().getPriority()+ ")"
                    +", loop "+i);
        }
    } 
}; 

public class Demo {
    public static void main(String[] args) {

        System.out.println(Thread.currentThread().getName()
                +"("+Thread.currentThread().getPriority()+ ")");

        Thread t1=new MyThread("t1");    // 新建t1
        Thread t2=new MyThread("t2");    // 新建t2
        t1.setPriority(1);                // 设置t1的优先级为1
        t2.setPriority(10);                // 设置t2的优先级为10
        t1.start();                        // 启动t1
        t2.start();                        // 启动t2
    }
}
```
运行结果：
```
main(5)
t1(1), loop 0
t1(1), loop 1
t1(1), loop 2
t1(1), loop 3
t1(1), loop 4
t2(10), loop 0
t2(10), loop 1
t2(10), loop 2
t2(10), loop 3
t2(10), loop 4
```
结果说明：
1.主线程main的优先级是5。
2.虽然t1的优先级被设为1，t2的优先级被设为10，经过多次运行发现，上边的结果是出现次数最多的，极少数时t1和t2会交替执行。
这说明了即使将优先级调高，也不能保证系统完全按照设定的优先级执行，还要取决于系统的线程规划器。

# 守护线程示例
```
// Demo.java
class MyThread extends Thread{
    public MyThread(String name) {
        super(name);
    }

    public void run(){
        try {
            for (int i=0; i<5; i++) {
                Thread.sleep(3);
                System.out.println(this.getName() +"(isDaemon="+this.isDaemon()+ ")" +", loop "+i);
            }
        } catch (InterruptedException e) {
        }
    }
};

class MyDaemon extends Thread{
    public MyDaemon(String name) {
        super(name);
    }

    public void run(){
        try {
            for (int i=0; i<10000; i++) {
                Thread.sleep(1);
                System.out.println(this.getName() +"(isDaemon="+this.isDaemon()+ ")" +", loop "+i);
            }
        } catch (InterruptedException e) {
        }
    }
}
public class Demo {
    public static void main(String[] args) {

        System.out.println(Thread.currentThread().getName()
                +"(isDaemon="+Thread.currentThread().isDaemon()+ ")");

        Thread t1=new MyThread("t1");    // 新建t1
        Thread t2=new MyDaemon("t2");    // 新建t2
        t2.setDaemon(true);                // 设置t2为守护线程
        t1.start();                        // 启动t1
        t2.start();                        // 启动t2
    }
}
```
运行结果：
```
main(isDaemon=false)
t1(isDaemon=false), loop 0
t2(isDaemon=true), loop 0
t2(isDaemon=true), loop 1
t1(isDaemon=false), loop 1
t1(isDaemon=false), loop 2
t2(isDaemon=true), loop 2
t1(isDaemon=false), loop 3
t2(isDaemon=true), loop 3
t2(isDaemon=true), loop 4
t2(isDaemon=true), loop 5
t1(isDaemon=false), loop 4
```
结果说明：

# 生产/消费者模型
生产/消费者问题是个非常典型的多线程问题，涉及到的对象包括“生产者”、“消费者”、“仓库”和“产品”。他们之间的关系如下：
(01) 生产者仅仅在仓储未满时候生产，仓满则停止生产。
(02) 消费者仅仅在仓储有产品时候才能消费，仓空则等待。
(03) 当消费者发现仓储没产品可消费时候会通知生产者生产。
(04) 生产者在生产出可消费产品时候，应该通知等待的消费者去消费。

# 生产/消费者实现
下面通过wait()/notify()方式实现该模型(后面在学习了线程池相关内容之后，再通过其它方式实现生产/消费者模型)。源码如下：
```
// Demo1.java
// 仓库
class Depot {
    private int capacity;    // 仓库的容量
    private int size;        // 仓库的实际数量

    public Depot(int capacity) {
        this.capacity = capacity;
        this.size = 0;
    }

    public synchronized void produce(int val) {
        try {
             // left 表示“想要生产的数量”(有可能生产量太多，需多此生产)
            int left = val;
            while (left > 0) {
                // 库存已满时，等待“消费者”消费产品。
                while (size >= capacity)
                    wait();
                // 获取“实际生产的数量”(即库存中新增的数量)
                // 如果“库存”+“想要生产的数量”>“总的容量”，则“实际增量”=“总的容量”-“当前容量”。(此时填满仓库)
                // 否则“实际增量”=“想要生产的数量”
                int inc = (size+left)>capacity ? (capacity-size) : left;
                size += inc;
                left -= inc;
                System.out.printf("%s produce(%3d) --> left=%3d, inc=%3d, size=%3d\n", 
                        Thread.currentThread().getName(), val, left, inc, size);
                // 通知“消费者”可以消费了。
                notifyAll();
            }
        } catch (InterruptedException e) {
        }
    } 

    public synchronized void consume(int val) {
        try {
            // left 表示“客户要消费数量”(有可能消费量太大，库存不够，需多此消费)
            int left = val;
            while (left > 0) {
                // 库存为0时，等待“生产者”生产产品。
                while (size <= 0)
                    wait();
                // 获取“实际消费的数量”(即库存中实际减少的数量)
                // 如果“库存”<“客户要消费的数量”，则“实际消费量”=“库存”；
                // 否则，“实际消费量”=“客户要消费的数量”。
                int dec = (size<left) ? size : left;
                size -= dec;
                left -= dec;
                System.out.printf("%s consume(%3d) <-- left=%3d, dec=%3d, size=%3d\n", 
                        Thread.currentThread().getName(), val, left, dec, size);
                notifyAll();
            }
        } catch (InterruptedException e) {
        }
    }

    public String toString() {
        return "capacity:"+capacity+", actual size:"+size;
    }
} 

// 生产者
class Producer {
    private Depot depot;
    
    public Producer(Depot depot) {
        this.depot = depot;
    }

    // 消费产品：新建一个线程向仓库中生产产品。
    public void produce(final int val) {
        new Thread() {
            public void run() {
                depot.produce(val);
            }
        }.start();
    }
}

// 消费者
class Customer {
    private Depot depot;
    
    public Customer(Depot depot) {
        this.depot = depot;
    }

    // 消费产品：新建一个线程从仓库中消费产品。
    public void consume(final int val) {
        new Thread() {
            public void run() {
                depot.consume(val);
            }
        }.start();
    }
}

public class Demo1 {
    public static void main(String[] args) {
        Depot mDepot = new Depot(100);
        Producer mPro = new Producer(mDepot);
        Customer mCus = new Customer(mDepot);

        mPro.produce(60);
        mPro.produce(120);
        mCus.consume(90);
        mCus.consume(150);
        mPro.produce(110);
    }
}
```
说明：
(01) Producer是“生产者”类，它与“仓库(depot)”关联。当调用“生产者”的produce()方法时，它会新建一个线程并向“仓库”中生产产品。
(02) Customer是“消费者”类，它与“仓库(depot)”关联。当调用“消费者”的consume()方法时，它会新建一个线程并消费“仓库”中的产品。
(03) Depot是“仓库”类，仓库中记录“仓库的容量(capacity)”以及“仓库中当前产品数目(size)”。
        “仓库”类的生产方法produce()和消费方法consume()方法都是synchronized方法，进入synchronized方法体，意味着这个线程获取到了该“仓库”对象的同步锁。这也就是说，同一时间，生产者和消费者线程只能有一个能运行。通过同步锁，实现了对“残酷”的互斥访问。
       对于生产方法produce()而言：当仓库满时，生产者线程等待，需要等待消费者消费产品之后，生产线程才能生产；生产者线程生产完产品之后，会通过notifyAll()唤醒同步锁上的所有线程，包括“消费者线程”，即我们所说的“通知消费者进行消费”。
      对于消费方法consume()而言：当仓库为空时，消费者线程等待，需要等待生产者生产产品之后，消费者线程才能消费；消费者线程消费完产品之后，会通过notifyAll()唤醒同步锁上的所有线程，包括“生产者线程”，即我们所说的“通知生产者进行生产”。

某一次运行结果：
```
Thread-0 produce( 60) --> left=  0, inc= 60, size= 60
Thread-4 produce(110) --> left= 70, inc= 40, size=100
Thread-2 consume( 90) <-- left=  0, dec= 90, size= 10
Thread-3 consume(150) <-- left=140, dec= 10, size=  0
Thread-1 produce(120) --> left= 20, inc=100, size=100
Thread-3 consume(150) <-- left= 40, dec=100, size=  0
Thread-4 produce(110) --> left=  0, inc= 70, size= 70
Thread-3 consume(150) <-- left=  0, dec= 40, size= 30
Thread-1 produce(120) --> left=  0, inc= 20, size= 50
```

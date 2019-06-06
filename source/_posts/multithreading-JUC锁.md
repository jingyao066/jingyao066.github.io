---
title: multithreading-JUC锁
tags: multithreading
date: 2019-03-28 17:51:23
---

根据锁的添加到Java中的时间，Java中的锁，可以分为"同步锁"和"JUC包中的锁"。

# 同步锁
即通过synchronized关键字来进行同步，实现对竞争资源的互斥访问的锁。Java 1.0版本中就已经支持同步锁了。
同步锁的原理是，对于每一个对象，有且仅有一个同步锁；不同的线程能共同访问该同步锁。但是，在同一个时间点，该同步锁能且只能被一个线程获取到。这样，获取到同步锁的线程就能进行CPU调度，从而在CPU上执行；而没有获取到同步锁的线程，必须进行等待，直到获取到同步锁之后才能继续运行。这就是多线程通过同步锁进行同步的原理。

# JUC包中的锁
相比同步锁，JUC包中的锁的功能更加强大，它为锁提供了一个框架，该框架允许更灵活地使用锁，只是它的用法更难罢了。
JUC包中的锁，包括：Lock接口，ReadWriteLock接口，LockSupport阻塞原语，Condition条件，
AbstractOwnableSynchronizer、AbstractQueuedSynchronizer、AbstractQueuedLongSynchronizer三个抽象类，ReentrantLock独占锁，ReentrantReadWriteLock读写锁。由于CountDownLatch，CyclicBarrier和Semaphore也是通过AQS来实现的；因此，我也将它们归纳到锁的框架中进行介绍。
先看看锁的框架图，如下所示。
![](multithreading-JUC锁/1.jpg)

- Lock接口
JUC包中的 Lock 接口支持那些语义不同(重入、公平等)的锁规则。所谓语义不同，是指锁可是有"公平机制的锁"、"非公平机制的锁"、"可重入的锁"等等。"公平机制"是指"不同线程获取锁的机制是公平的"，而"非公平机制"则是指"不同线程获取锁的机制是非公平的"，"可重入的锁"是指同一个锁能够被一个线程多次获取。

- ReadWriteLock
ReadWriteLock 接口以和Lock类似的方式定义了一些读取者可以共享而写入者独占的锁。JUC包只有一个类实现了该接口，即 ReentrantReadWriteLock，因为它适用于大部分的标准用法上下文。但程序员可以创建自己的、适用于非标准要求的实现。

- AbstractOwnableSynchronizer/AbstractQueuedSynchronizer/AbstractQueuedLongSynchronizer
AbstractQueuedSynchronizer就是被称之为AQS的类，它是一个非常有用的超类，可用来定义锁以及依赖于排队阻塞线程的其他同步器；ReentrantLock，ReentrantReadWriteLock，CountDownLatch，CyclicBarrier和Semaphore等这些类都是基于AQS类实现的。AbstractQueuedLongSynchronizer 类提供相同的功能但扩展了对同步状态的 64 位的支持。两者都扩展了类 AbstractOwnableSynchronizer（一个帮助记录当前保持独占同步的线程的简单类）。

- LockSupport
LockSupport提供“创建锁”和“其他同步类的基本线程阻塞原语”。 
LockSupport的功能和"Thread中的Thread.suspend()和Thread.resume()有点类似"，LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程。但是park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。

- Condition
Condition需要和Lock联合使用，它的作用是代替Object监视器方法，可以通过await(),signal()来休眠/唤醒线程。
Condition 接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用 Object.wait 访问的隐式监视器类似，但提供了更强大的功能。需要特别指出的是，单个 Lock 可能与多个 Condition 对象关联。为了避免兼容性问题，Condition 方法的名称与对应的 Object 版本中的不同。

- ReentrantLock
ReentrantLock是独占锁。所谓独占锁，是指只能被独自占领，即同一个时间点只能被一个线程锁获取到的锁。ReentrantLock锁包括"公平的ReentrantLock"和"非公平的ReentrantLock"。"公平的ReentrantLock"是指"不同线程获取锁的机制是公平的"，而"非公平的　　ReentrantLock"则是指"不同线程获取锁的机制是非公平的"，ReentrantLock是"可重入的锁"。
ReentrantLock的UML类图如下：
![](multithreading-JUC锁/2.jpg)

	- ReentrantLock实现了Lock接口。
	- ReentrantLock中有一个成员变量sync，sync是Sync类型；Sync是一个抽象类，而且它继承于AQS。
	- ReentrantLock中有"公平锁类"FairSync和"非公平锁类"NonfairSync，它们都是Sync的子类。ReentrantReadWriteLock中sync对象，是FairSync与NonfairSync中的一种，这也意味着ReentrantLock是"公平锁"或"非公平锁"中的一种，ReentrantLock默认是非公平锁。

7. ReentrantReadWriteLock
ReentrantReadWriteLock是读写锁接口ReadWriteLock的实现类，它包括子类ReadLock和WriteLock。ReentrantLock是共享锁，而WriteLock是独占锁。
ReentrantReadWriteLock的UML类图如下：
![](multithreading-JUC锁/3.jpg)

	- ReentrantReadWriteLock实现了ReadWriteLock接口。
	- ReentrantReadWriteLock中包含sync对象，读锁readerLock和写锁writerLock。读锁ReadLock和写锁WriteLock都实现了Lock接口。
	- 和"ReentrantLock"一样，sync是Sync类型；而且，Sync也是一个继承于AQS的抽象类。Sync也包括"公平锁"FairSync和"非公平锁"NonfairSync。


8. CountDownLatch
CountDownLatch是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。 
CountDownLatch的UML类图如下：
![](multithreading-JUC锁/4.jpg)

CountDownLatch包含了sync对象，sync是Sync类型。CountDownLatch的Sync是实例类，它继承于AQS。

9. CyclicBarrier
CyclicBarrier是一个同步辅助类，允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。
CyclicBarrier的UML类图如下：
![](multithreading-JUC锁/5.jpg)

CyclicBarrier是包含了"ReentrantLock对象lock"和"Condition对象trip"，它是通过独占锁实现的。
CyclicBarrier和CountDownLatch的区别是：
	- CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待。
	- CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。

10. Semaphore
Semaphore是一个计数信号量，它的本质是一个"共享锁"。
信号量维护了一个信号量许可集。线程可以通过调用acquire()来获取信号量的许可；当信号量中有可用的许可时，线程能获取该许可；否则线程必须等待，直到有可用的许可为止。 线程可以通过release()来释放它所持有的信号量许可。
Semaphore的UML类图如下：
![](multithreading-JUC锁/6.jpg)

和"ReentrantLock"一样，Semaphore包含了sync对象，sync是Sync类型；而且，Sync也是一个继承于AQS的抽象类。Sync也包括"公平信号量"FairSync和"非公平信号量"NonfairSync。

## 互斥锁ReentrantLock
ReentrantLock是一个可重入的互斥锁，又被称为独占锁。
顾名思义，ReentrantLock锁在同一个时间点只能被一个线程锁持有，可重入表示ReentrantLock锁可被单个线程多次获取。
ReentrantLock分为公平锁和非公平锁，它们的区别体现在获取锁的机制上是否公平。锁是为了保护竞争资源，防止多个线程同时操作同一对象而出错。
ReentrantLock在同一时间点只能被一个线程获取(当某线程获取到锁时，其他线程必须等待)。ReentrantLock是通过一个FIFO的等待队列来管理获取该锁的所有线程的。
在公平锁的机制下，线程一次排队获取锁，而非公平锁在锁是可获取状态时，不管自己是不是在队列的开头都会获取锁。

### ReentrantLock函数列表
```java
// 创建一个 ReentrantLock ，默认是“非公平锁”。
ReentrantLock()
// 创建策略是fair的 ReentrantLock。fair为true表示是公平锁，fair为false表示是非公平锁。
ReentrantLock(boolean fair)

// 查询当前线程保持此锁的次数。
int getHoldCount()
// 返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null。
protected Thread getOwner()
// 返回一个 collection，它包含可能正等待获取此锁的线程。
protected Collection<Thread> getQueuedThreads()
// 返回正等待获取此锁的线程估计数。
int getQueueLength()
// 返回一个 collection，它包含可能正在等待与此锁相关给定条件的那些线程。
protected Collection<Thread> getWaitingThreads(Condition condition)
// 返回等待与此锁相关的给定条件的线程估计数。
int getWaitQueueLength(Condition condition)
// 查询给定线程是否正在等待获取此锁。
boolean hasQueuedThread(Thread thread)
// 查询是否有些线程正在等待获取此锁。
boolean hasQueuedThreads()
// 查询是否有些线程正在等待与此锁有关的给定条件。
boolean hasWaiters(Condition condition)
// 如果是“公平锁”返回true，否则返回false。
boolean isFair()
// 查询当前线程是否保持此锁。
boolean isHeldByCurrentThread()
// 查询此锁是否由任意线程保持。
boolean isLocked()
// 获取锁。
void lock()
// 如果当前线程未被中断，则获取锁。
void lockInterruptibly()
// 返回用来与此 Lock 实例一起使用的 Condition 实例。
Condition newCondition()
// 仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
boolean tryLock()
// 如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁。
boolean tryLock(long timeout, TimeUnit unit)
// 试图释放此锁。
void unlock()
```

### ReentrantLock示例
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

// LockTest1.java
// 仓库
class Depot { 
    private int size;        // 仓库的实际数量
    private Lock lock;        // 独占锁

    public Depot() {
        this.size = 0;
        this.lock = new ReentrantLock();
    }

    public void produce(int val) {
        lock.lock();
        try {
            size += val;
            System.out.printf("%s produce(%d) --> size=%d\n", 
                    Thread.currentThread().getName(), val, size);
        } finally {
            lock.unlock();
        }
    }

    public void consume(int val) {
        lock.lock();
        try {
            size -= val;
            System.out.printf("%s consume(%d) <-- size=%d\n", 
                    Thread.currentThread().getName(), val, size);
        } finally {
            lock.unlock();
        }
    }
}; 

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

public class LockTest1 {  
    public static void main(String[] args) {  
        Depot mDepot = new Depot();
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
运行结果：
```
Thread-0 produce(60) --> size=60
Thread-1 produce(120) --> size=180
Thread-3 consume(150) <-- size=30
Thread-2 consume(90) <-- size=-60
Thread-4 produce(110) --> size=50
```

结果分析：
- Depot 是个仓库。通过produce()能往仓库中生产货物，通过consume()能消费仓库中的货物。通过独占锁lock实现对仓库的互斥访问：在操作(生产/消费)仓库中货品前，会先通过lock()锁住仓库，操作完之后再通过unlock()解锁。
- Producer是生产者类。调用Producer中的produce()函数可以新建一个线程往仓库中生产产品。
- Customer是消费者类。调用Customer中的consume()函数可以新建一个线程消费仓库中的产品。
- 在主线程main中，我们会新建1个生产者mPro，同时新建1个消费者mCus。它们分别向仓库中生产/消费产品。
根据main中的生产/消费数量，仓库最终剩余的产品应该是50。运行结果是符合我们预期的！

这个模型存在两个问题：
- 现实中，仓库的容量不可能为负数。但是，此模型中的仓库容量可以为负数，这与现实相矛盾！
- 现实中，仓库的容量是有限制的。但是，此模型中的容量确实没有限制的！
这两个问题，我们稍微会讲到如何解决。现在，先看个简单的示例2；通过对比“示例1”和“示例2”,我们能更清晰的认识lock(),unlock()的用途。

示例2
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

// LockTest2.java
// 仓库
class Depot { 
    private int size;        // 仓库的实际数量
    private Lock lock;        // 独占锁

    public Depot() {
        this.size = 0;
        this.lock = new ReentrantLock();
    }

    public void produce(int val) {
//        lock.lock();
//        try {
            size += val;
            System.out.printf("%s produce(%d) --> size=%d\n", 
                    Thread.currentThread().getName(), val, size);
//        } catch (InterruptedException e) {
//        } finally {
//            lock.unlock();
//        }
    }

    public void consume(int val) {
//        lock.lock();
//        try {
            size -= val;
            System.out.printf("%s consume(%d) <-- size=%d\n", 
                    Thread.currentThread().getName(), val, size);
//        } finally {
//            lock.unlock();
//        }
    }
}; 

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

public class LockTest2 {  
    public static void main(String[] args) {  
        Depot mDepot = new Depot();
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
(某一次)运行结果：
```
Thread-0 produce(60) --> size=-60
Thread-4 produce(110) --> size=50
Thread-2 consume(90) <-- size=-60
Thread-1 produce(120) --> size=-60
Thread-3 consume(150) <-- size=-60
```
结果说明：
“示例2”在“示例1”的基础上去掉了lock锁。在“示例2”中，仓库中最终剩余的产品是-60，而不是我们期望的50。原因是我们没有实现对仓库的互斥访问。

示例3
在“示例3”中，我们通过Condition去解决“示例1”中的两个问题：“仓库的容量不可能为负数”以及“仓库的容量是有限制的”。
解决该问题是通过Condition。Condition是需要和Lock联合使用的：通过Condition中的await()方法，能让线程阻塞[类似于wait()]；通过Condition的signal()方法，能让唤醒线程[类似于notify()]。
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Condition;

// LockTest3.java
// 仓库
class Depot {
    private int capacity;    // 仓库的容量
    private int size;        // 仓库的实际数量
    private Lock lock;        // 独占锁
    private Condition fullCondtion;            // 生产条件
    private Condition emptyCondtion;        // 消费条件

    public Depot(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        this.lock = new ReentrantLock();
        this.fullCondtion = lock.newCondition();
        this.emptyCondtion = lock.newCondition();
    }

    public void produce(int val) {
        lock.lock();
        try {
             // left 表示“想要生产的数量”(有可能生产量太多，需多此生产)
            int left = val;
            while (left > 0) {
                // 库存已满时，等待“消费者”消费产品。
                while (size >= capacity)
                    fullCondtion.await();
                // 获取“实际生产的数量”(即库存中新增的数量)
                // 如果“库存”+“想要生产的数量”>“总的容量”，则“实际增量”=“总的容量”-“当前容量”。(此时填满仓库)
                // 否则“实际增量”=“想要生产的数量”
                int inc = (size+left)>capacity ? (capacity-size) : left;
                size += inc;
                left -= inc;
                System.out.printf("%s produce(%3d) --> left=%3d, inc=%3d, size=%3d\n", 
                        Thread.currentThread().getName(), val, left, inc, size);
                // 通知“消费者”可以消费了。
                emptyCondtion.signal();
            }
        } catch (InterruptedException e) {
        } finally {
            lock.unlock();
        }
    }

    public void consume(int val) {
        lock.lock();
        try {
            // left 表示“客户要消费数量”(有可能消费量太大，库存不够，需多此消费)
            int left = val;
            while (left > 0) {
                // 库存为0时，等待“生产者”生产产品。
                while (size <= 0)
                    emptyCondtion.await();
                // 获取“实际消费的数量”(即库存中实际减少的数量)
                // 如果“库存”<“客户要消费的数量”，则“实际消费量”=“库存”；
                // 否则，“实际消费量”=“客户要消费的数量”。
                int dec = (size<left) ? size : left;
                size -= dec;
                left -= dec;
                System.out.printf("%s consume(%3d) <-- left=%3d, dec=%3d, size=%3d\n", 
                        Thread.currentThread().getName(), val, left, dec, size);
                fullCondtion.signal();
            }
        } catch (InterruptedException e) {
        } finally {
            lock.unlock();
        }
    }

    public String toString() {
        return "capacity:"+capacity+", actual size:"+size;
    }
}; 

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

public class LockTest3 {  
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
(某一次)运行结果：
```
Thread-0 produce( 60) --> left=  0, inc= 60, size= 60
Thread-1 produce(120) --> left= 80, inc= 40, size=100
Thread-2 consume( 90) <-- left=  0, dec= 90, size= 10
Thread-3 consume(150) <-- left=140, dec= 10, size=  0
Thread-4 produce(110) --> left= 10, inc=100, size=100
Thread-3 consume(150) <-- left= 40, dec=100, size=  0
Thread-4 produce(110) --> left=  0, inc= 10, size= 10
Thread-3 consume(150) <-- left= 30, dec= 10, size=  0
Thread-1 produce(120) --> left=  0, inc= 80, size= 80
Thread-3 consume(150) <-- left=  0, dec= 30, size= 50
```
代码中的已经包含了很详细的注释，这里就不再说明了。

## 公平锁
此处的公平锁指的是互斥锁的公平锁。

### 基本概念
- AQS：指AbstractQueuedSynchronizer类。
AQS是java中管理锁的抽象类，锁的许多公共方法都在这个类中实现。AQS是独占锁(如ReentrantLock)和共享锁(如Semaphore)的公共父类。

- AQS锁的类别：分为“独占锁”和“共享锁”两种。
独占锁：锁在一个时间点只能被一个线程占有。根据锁的获取机制，它又划分为公平锁和非公平锁。公平锁，是按照CLH等待线程按照先来先得的规则，公平的获取锁。
而非公平锁，当线程获取锁时，它会无视CLH等待队列而直接获取锁。独占锁的典型例子时ReentrantLock，此外，ReentrantReadWriteLock.WriteLock也是独占锁。

共享锁：能被多个线程同时拥有，能被共享的锁。JUC包中的ReentrantReadWriteLock.ReadLock，CyclicBarrier， CountDownLatch和Semaphore都是共享锁。

- CLH队列：Craig, Landin, and Hagersten lock queue
CLH队列是AQS中等待锁的线程队列。在多线程中，为了保护竞争资源不被多个线程同时操作而引起错误，我们通常需要锁来保护这些资源。
在独占锁中，竞争资源在一个时间点只能被一个线程锁访问，而其他线程则需要等待。CLH就是管理这些等待锁的线程的队列。
CLH是一个非阻塞的FIFO队列，也就是说往里面插入或移除一个节点的时候，在并发条件下不会阻塞，而是通过自旋锁和CAS保证节点插入和移除的原子性。

- CAS函数：Compare and Swap
CAS函数，是比较并交换函数，它是原子操作函数，即通过CAS操作的数据都是以原子方式进行的。例如：compareAndSetHead(), compareAndSetTail(), compareAndSetNext()等函数。

### ReentrantLock数据结构
ReentrantLock的UML类图
![](multithreading-JUC锁/7.jpg)

- ReentrantLock实现了Lock接口。
- ReentrantLock与sync是组合关系。ReentrantLock中，包含了Sync对象，而且，Sync是AQS的子类，更重要的是，Sync有两个子类FireSync(公平锁)和NonFireSync(非公平锁)。
ReentrantLock是一个独占锁，至于它到底是公平锁还是非公平锁，就取决于sync对象是"FairSync的实例"还是"NonFairSync的实例"。

### 获取公平锁
通过对互斥锁ReentrantLock的学习，我们知道可以通过lock()函数获取锁，下面，我们以lock()对获取公平锁的过程进行展开。

- lock()
lock()在ReentrantLock.java的FairSync类中实现，它的源码如下：
```java
final void lock() {
    acquire(1);
}
```
说明：
“当前线程”实际上是通过acquire(1)获取锁的。
这里说明一下“1”的含义，它是设置“锁的状态”的参数。对于“独占锁”而言，锁处于可获取状态时，它的状态值是0；锁被线程初次获取到了，它的状态值就变成了1。
由于ReentrantLock(公平锁/非公平锁)是可重入锁，所以“独占锁”可以被单个线程多此获取，每获取1次就将锁的状态+1。也就是说，初次获取锁时，通过acquire(1)将锁的状态值设为1；再次获取锁时，将锁的状态值设为2；依次类推...这就是为什么获取锁时，传入的参数是1的原因了。
可重入就是指锁可以被单个线程多次获取。

- acquire()
acquire()在AQS中实现的，它的源码如下：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
- “当前线程”首先通过tryAcquire()尝试获取锁。获取成功的话，直接返回；尝试失败的话，进入到等待队列排序等待(前面还有可能有需要线程在等待该锁)。
- “当前线程”尝试失败的情况下，先通过addWaiter(Node.EXCLUSIVE)来将“当前线程”加入到"CLH队列(非阻塞的FIFO队列)"末尾。CLH队列就是线程等待队列。
- 再执行完addWaiter(Node.EXCLUSIVE)之后，会调用acquireQueued()来获取锁。由于此时ReentrantLock是公平锁，它会根据公平性原则来获取锁。
- “当前线程”在执行acquireQueued()时，会进入到CLH队列中休眠等待，直到获取锁了才返回！如果“当前线程”在休眠等待过程中被中断过，acquireQueued会返回true，此时"当前线程"会调用selfInterrupt()来自己给自己产生一个中断。至于为什么要自己给自己产生一个中断，后面再介绍。

上面是对acquire()的概括性说明。下面，我们将该函数分为4部分来逐步解析。

#### tryAcquire()
- tryAcquire()
公平锁的tryAcquire()在ReentrantLock.java的FairSync类中实现，源码如下：
```java
protected final boolean tryAcquire(int acquires) {
    // 获取“当前线程”
    final Thread current = Thread.currentThread();
    // 获取“独占锁”的状态
    int c = getState();
    // c=0意味着“锁没有被任何线程锁拥有”，
    if (c == 0) {
        // 若“锁没有被任何线程锁拥有”，
        // 则判断“当前线程”是不是CLH队列中的第一个线程线程，
        // 若是的话，则获取该锁，设置锁的状态，并切设置锁的拥有者为“当前线程”。
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 如果“独占锁”的拥有者已经为“当前线程”，
        // 则将更新锁的状态。
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
说明：根据代码，我们可以分析出，tryAcquire()的作用就是尝试去获取锁。注意，这里只是尝试！
		尝试成功的话，返回true；尝试失败的话，返回false，后续再通过其它办法来获取该锁。后面我们会说明，在尝试失败的情况下，是如何一步步获取锁的。
		
- hasQueuedPredecessors()
hasQueuedPredecessors()在AQS中实现，源码如下：
```java
public final boolean hasQueuedPredecessors() {
    Node t = tail; 
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```
说明： 通过代码，能分析出，hasQueuedPredecessors() 是通过判断"当前线程"是不是在CLH队列的队首，来返回AQS中是不是有比“当前线程”等待更久的线程。下面对head、tail和Node进行说明。

- Node的源码
Node就是CLH队列的节点。Node在AQS中实现，它的数据结构如下：
```java
private transient volatile Node head;    // CLH队列的队首
private transient volatile Node tail;    // CLH队列的队尾

// CLH队列的节点
static final class Node {
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;

    // 线程已被取消，对应的waitStatus的值
    static final int CANCELLED =  1;
    // “当前线程的后继线程需要被unpark(唤醒)”，对应的waitStatus的值。
    // 一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
    static final int SIGNAL    = -1;
    // 线程(处在Condition休眠状态)在等待Condition唤醒，对应的waitStatus的值
    static final int CONDITION = -2;
    // (共享锁)其它线程获取到“共享锁”，对应的waitStatus的值
    static final int PROPAGATE = -3;

    // waitStatus为“CANCELLED, SIGNAL, CONDITION, PROPAGATE”时分别表示不同状态，
    // 若waitStatus=0，则意味着当前线程不属于上面的任何一种状态。
    volatile int waitStatus;

    // 前一节点
    volatile Node prev;

    // 后一节点
    volatile Node next;

    // 节点所对应的线程
    volatile Thread thread;

    // nextWaiter是“区别当前CLH队列是 ‘独占锁’队列 还是 ‘共享锁’队列 的标记”
    // 若nextWaiter=SHARED，则CLH队列是“独占锁”队列；
    // 若nextWaiter=EXCLUSIVE，(即nextWaiter=null)，则CLH队列是“共享锁”队列。
    Node nextWaiter;

    // “共享锁”则返回true，“独占锁”则返回false。
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    // 返回前一节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    // 构造函数。thread是节点所对应的线程，mode是用来表示thread的锁是“独占锁”还是“共享锁”。
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    // 构造函数。thread是节点所对应的线程，waitStatus是线程的等待状态。
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```
	说明：
	Node是CLH队列的节点，代表“等待锁的线程队列”。
	+ 每个Node都会一个线程对应。
	+ 每个Node会通过prev和next分别指向上一个节点和下一个节点，这分别代表上一个等待线程和下一个等待线程。
	+ Node通过waitStatus保存线程的等待状态。
	+ Node通过nextWaiter来区分线程是“独占锁”线程还是“共享锁”线程。如果是“独占锁”线程，则nextWaiter的值为EXCLUSIVE；如果是“共享锁”线程，则nextWaiter的值是SHARED。

- compareAndSetState()
compareAndSetState()在AQS中实现。它的源码如下：
```java
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```
说明： compareAndSwapInt() 是sun.misc.Unsafe类中的一个本地方法。对此，我们需要了解的是 compareAndSetState(expect, update) 是以原子的方式操作当前线程；若当前线程的状态为expect，则设置它的状态为update。

- setExclusiveOwnerThread()
setExclusiveOwnerThread()在AbstractOwnableSynchronizer.java中实现，它的源码如下：
```java
// exclusiveOwnerThread是当前拥有“独占锁”的线程
private transient Thread exclusiveOwnerThread;
protected final void setExclusiveOwnerThread(Thread t) {
    exclusiveOwnerThread = t;
}
```
说明：setExclusiveOwnerThread()的作用就是，设置线程t为当前拥有“独占锁”的线程。

- getState(), setState()
getState()和setState()都在AQS中实现，源码如下：
```java
// 锁的状态
private volatile int state;
// 设置锁的状态
protected final void setState(int newState) {
    state = newState;
}
// 获取锁的状态
protected final int getState() {
    return state;
}
```
说明：state表示锁的状态，对于“独占锁”而已，state=0表示锁是可获取状态(即，锁没有被任何线程锁持有)。由于java中的独占锁是可重入的，state的值可以>1。

小结：tryAcquire()的作用就是让“当前线程”尝试获取锁。获取成功返回true，失败则返回false。

#### addWaiter(Node.EXCLUSIVE)
addWaiter(Node.EXCLUSIVE)的作用是，创建“当前线程”的Node节点，且Node中记录“当前线程”对应的锁是“独占锁”类型，并且将该节点添加到CLH队列的末尾。

- addWaiter()
addWaiter()在AQS中实现，源码如下：
```java
private Node addWaiter(Node mode) {
    // 新建一个Node节点，节点对应的线程是“当前线程”，“当前线程”的锁的模型是mode。
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    // 若CLH队列不为空，则将“当前线程”添加到CLH队列末尾
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 若CLH队列为空，则调用enq()新建CLH队列，然后再将“当前线程”添加到CLH队列中。
    enq(node);
    return node;
}
```
说明：对于“公平锁”而言，addWaiter(Node.EXCLUSIVE)会首先创建一个Node节点，节点的类型是“独占锁”(Node.EXCLUSIVE)类型。然后，再将该节点添加到CLH队列的末尾。

- compareAndSetTail()
compareAndSetTail()在AQS中实现，源码如下：
```java
private final boolean compareAndSetTail(Node expect, Node update) {
    return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```
说明：compareAndSetTail也属于CAS函数，也是通过“本地方法”实现的。compareAndSetTail(expect, update)会以原子的方式进行操作，它的作用是判断CLH队列的队尾是不是为expect，是的话，就将队尾设为update。

- enq()
enq()在AQS中实现，源码如下：
```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
说明： enq()的作用很简单。如果CLH队列为空，则新建一个CLH表头；然后将node添加到CLH末尾。否则，直接将node添加到CLH末尾。

小结：addWaiter()的作用，就是将当前线程添加到CLH队列中。这就意味着将当前线程添加到等待获取“锁”的等待线程队列中了。

#### acquireQueued()
前面，我们已经将当前线程添加到CLH队列中了。而acquireQueued()的作用就是逐步的去执行CLH队列的线程，如果当前线程获取到了锁，则返回；否则，当前线程进行休眠，直到唤醒并重新获取锁了才返回。下面，我们看看acquireQueued()的具体流程。

- acquireQueued()
acquireQueued()在AQS中实现，源码如下：
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        // interrupted表示在CLH队列的调度中，
        // “当前线程”在休眠时，有没有被中断过。
        boolean interrupted = false;
        for (;;) {
            // 获取上一个节点。
            // node是“当前线程”对应的节点，这里就意味着“获取上一个等待锁的线程”。
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
说明：acquireQueued()的目的是从队列中获取锁。

- shouldParkAfterFailedAcquire()
shouldParkAfterFailedAcquire()在AQS中实现，源码如下：
```java
// 返回“当前线程是否应该阻塞”
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 前继节点的状态
    int ws = pred.waitStatus;
    // 如果前继节点是SIGNAL状态，则意味这当前线程需要被unpark唤醒。此时，返回true。
    if (ws == Node.SIGNAL)
        return true;
    // 如果前继节点是“取消”状态，则设置 “当前节点”的 “当前前继节点”  为  “‘原前继节点’的前继节点”。
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 如果前继节点为“0”或者“共享锁”状态，则设置前继节点为SIGNAL状态。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
	说明：
	- 关于waitStatus请参考下表(中扩号内为waitStatus的值)，更多关于waitStatus的内容，可以参考前面的Node类的介绍。
	```
	CANCELLED[1]  -- 当前线程已被取消
	SIGNAL[-1]    -- “当前线程的后继线程需要被unpark(唤醒)”。一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
	CONDITION[-2] -- 当前线程(处在Condition休眠状态)在等待Condition唤醒
	PROPAGATE[-3] -- (共享锁)其它线程获取到“共享锁”
	[0]           -- 当前线程不属于上面的任何一种状态。
	```
	- shouldParkAfterFailedAcquire()通过以下规则，判断“当前线程”是否需要被阻塞。
	```
	规则1：如果前继节点状态为SIGNAL，表明当前节点需要被unpark(唤醒)，此时则返回true。
	规则2：如果前继节点状态为CANCELLED(ws>0)，说明前继节点已经被取消，则通过先前回溯找到一个有效(非CANCELLED状态)的节点，并返回false。
	规则3：如果前继节点状态为非SIGNAL、非CANCELLED，则设置前继的状态为SIGNAL，并返回false。
	如果“规则1”发生，即“前继节点是SIGNAL”状态，则意味着“当前线程”需要被阻塞。接下来会调用parkAndCheckInterrupt()阻塞当前线程，直到当前先被唤醒才从parkAndCheckInterrupt()中返回。
	```
- parkAndCheckInterrupt())
parkAndCheckInterrupt()在AQS中实现，源码如下：
```java
private final boolean parkAndCheckInterrupt() {
    // 通过LockSupport的park()阻塞“当前线程”。
    LockSupport.park(this);
    // 返回线程的中断状态。
    return Thread.interrupted();
}
```
说明：parkAndCheckInterrupt()的作用是阻塞当前线程，并且返回“线程被唤醒之后”的中断状态。它会先通过LockSupport.park()阻塞“当前线程”，然后通过Thread.interrupted()返回线程的中断状态。

这里介绍一下线程被阻塞之后如何唤醒。一般有2种情况：
第1种情况：unpark()唤醒。“前继节点对应的线程”使用完锁之后，通过unpark()方式唤醒当前线程。
第2种情况：中断唤醒。其它线程通过interrupt()中断当前线程。

补充：LockSupport()中的park(),unpark()的作用 和 Object中的wait(),notify()作用类似，是阻塞/唤醒。
它们的用法不同，park(),unpark()是轻量级的，而wait(),notify()是必须先通过Synchronized获取同步锁。
关于LockSupport，我们会在之后的章节再专门进行介绍。

- 再次tryAcquire()
了解了shouldParkAfterFailedAcquire()和parkAndCheckInterrupt()函数之后。我们接着分析acquireQueued()的for循环部分。
```java
final Node p = node.predecessor();
if (p == head && tryAcquire(arg)) {
    setHead(node);
    p.next = null; // help GC
    failed = false;
    return interrupted;
}
```
	说明：
	- 通过node.predecessor()获取前继节点。predecessor()就是返回node的前继节点，若对此有疑惑可以查看下面关于Node类的介绍。
	- p == head && tryAcquire(arg)
       首先，判断“前继节点”是不是CHL表头。如果是的话，则通过tryAcquire()尝试获取锁。
       其实，这样做的目的是为了“让当前线程获取锁”，但是为什么需要先判断p==head呢？理解这个对理解“公平锁”的机制很重要，因为这么做的原因就是为了保证公平性！
       (a) 前面，我们在shouldParkAfterFailedAcquire()我们判断“当前线程”是否需要阻塞；
       (b) 接着，“当前线程”阻塞的话，会调用parkAndCheckInterrupt()来阻塞线程。当线程被解除阻塞的时候，我们会返回线程的中断状态。而线程被解决阻塞，可能是由于“线程被中断”，也可能是由于“其它线程调用了该线程的unpark()函数”。
       (c) 再回到p==head这里。如果当前线程是因为其它线程调用了unpark()函数而被唤醒，那么唤醒它的线程，应该是它的前继节点所对应的线程(关于这一点，后面在“释放锁”的过程中会看到)。 OK，是前继节点调用unpark()唤醒了当前线程！
            此时，再来理解p==head就很简单了：当前继节点是CLH队列的头节点，并且它释放锁之后；就轮到当前节点获取锁了。然后，当前节点通过tryAcquire()获取锁；获取成功的话，通过setHead(node)设置当前节点为头节点，并返回。
       总之，如果“前继节点调用unpark()唤醒了当前线程”并且“前继节点是CLH表头”，此时就是满足p==head，也就是符合公平性原则的。否则，如果当前线程是因为“线程被中断”而唤醒，那么显然就不是公平了。这就是为什么说p==head就是保证公平性！

小结：acquireQueued()的作用就是“当前线程”会根据公平性原则进行阻塞等待，直到获取锁为止；并且返回当前线程在等待过程中有没有并中断过。

#### selfInterrupt()
selfInterrupt()是AQS中实现，源码如下：
```java
private static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```
说明：selfInterrupt()的代码很简单，就是“当前线程”自己产生一个中断。但是，为什么需要这么做呢？
这必须结合acquireQueued()进行分析。如果在acquireQueued()中，当前线程被中断过，则执行selfInterrupt()；否则不会执行。

在acquireQueued()中，即使是线程在阻塞状态被中断唤醒而获取到cpu执行权利；但是，如果该线程的前面还有其它等待锁的线程，根据公平性原则，该线程依然无法获取到锁。它会再次阻塞！ 该线程再次阻塞，直到该线程被它的前面等待锁的线程锁唤醒；线程才会获取锁，然后“真正执行起来”！
也就是说，在该线程“成功获取锁并真正执行起来”之前，它的中断会被忽略并且中断标记会被清除！ 因为在parkAndCheckInterrupt()中，我们线程的中断状态时调用了Thread.interrupted()。该函数不同于Thread的isInterrupted()函数，isInterrupted()仅仅返回中断状态，而interrupted()在返回当前中断状态之后，还会清除中断状态。 正因为之前的中断状态被清除了，所以这里需要调用selfInterrupt()重新产生一个中断！

小结：selfInterrupt()的作用就是当前线程自己产生一个中断。


总结
再回过头看看acquire()函数，它最终的目的是获取锁。
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
- 先是通过tryAcquire()尝试获取锁。获取成功的话，直接返回；尝试失败的话，再通过acquireQueued()获取锁。
- 尝试失败的情况下，会先通过addWaiter()来将“当前线程”加入到"CLH队列"末尾；然后调用acquireQueued()，在CLH队列中排序等待获取锁，在此过程中，线程处于休眠状态。直到获取锁了才返回。 如果在休眠等待过程中被中断过，则调用selfInterrupt()来自己产生一个中断。

## 释放公平锁(基于JDK1.7.0_40)
- unlock()
unlock()在ReentrantLock.java中实现的，源码如下：
```
public void unlock() {
    sync.release(1);
}
```
说明：
unlock()是解锁函数，它是通过AQS的release()函数来实现的。
在这里，“1”的含义和“获取锁的函数acquire(1)的含义”一样，它是设置“释放锁的状态”的参数。由于“公平锁”是可重入的，所以对于同一个线程，每释放锁一次，锁的状态-1。

关于AQS, ReentrantLock 和 sync的关系如下：
```java
public class ReentrantLock implements Lock, java.io.Serializable {

    private final Sync sync;

    abstract static class Sync extends AbstractQueuedSynchronizer {
        ...
    }

    ...
}
```
从中，我们发现：sync是ReentrantLock.java中的成员对象，而Sync是AQS的子类。

- release()
release()在AQS中实现的，源码如下：
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
说明：
release()会先调用tryRelease()来尝试释放当前线程锁持有的锁。成功的话，则唤醒后继等待线程，并返回true。否则，直接返回false。

- tryRelease()
tryRelease()在ReentrantLock.java的Sync类中实现，源码如下：
```java
protected final boolean tryRelease(int releases) {
    // c是本次释放锁之后的状态
    int c = getState() - releases;
    // 如果“当前线程”不是“锁的持有者”，则抛出异常！
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();

    boolean free = false;
    // 如果“锁”已经被当前线程彻底释放，则设置“锁”的持有者为null，即锁是可获取状态。
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    // 设置当前线程的锁的状态。
    setState(c);
    return free;
}
```
	说明：
	tryRelease()的作用是尝试释放锁。
	- 如果“当前线程”不是“锁的持有者”，则抛出异常。
	- 如果“当前线程”在本次释放锁操作之后，对锁的拥有状态是0(即，当前线程彻底释放该“锁”)，则设置“锁”的持有者为null，即锁是可获取状态。同时，更新当前线程的锁的状态为0。
	getState(), setState()在前一章已经介绍过，这里不再说明。

getExclusiveOwnerThread(), setExclusiveOwnerThread()在AQS的父类AbstractOwnableSynchronizer.java中定义，源码如下：
```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    // “锁”的持有线程
    private transient Thread exclusiveOwnerThread;

    // 设置“锁的持有线程”为t
    protected final void setExclusiveOwnerThread(Thread t) {
        exclusiveOwnerThread = t;
    }

    // 获取“锁的持有线程”
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
   
    ...
}
```

- unparkSuccessor()
在release()中“当前线程”释放锁成功的话，会唤醒当前线程的后继线程。
根据CLH队列的FIFO规则，“当前线程”(即已经获取锁的线程)肯定是head；如果CLH队列非空的话，则唤醒锁的下一个等待线程。
下面看看unparkSuccessor()的源码，它在AQS中实现。
```java
private void unparkSuccessor(Node node) {
    // 获取当前线程的状态
    int ws = node.waitStatus;
    // 如果状态<0，则设置状态=0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    //获取当前节点的“有效的后继节点”，无效的话，则通过for循环进行获取。
    // 这里的有效，是指“后继节点对应的线程状态<=0”
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 唤醒“后继节点对应的线程”
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
说明：
unparkSuccessor()的作用是“唤醒当前线程的后继线程”。后继线程被唤醒之后，就可以获取该锁并恢复运行了。关于node.waitStatus的说明，请参考“上一章关于Node类的介绍”。

总结
“释放锁”的过程相对“获取锁”的过程比较简单。释放锁时，主要进行的操作，是更新当前线程对应的锁的状态。如果当前线程对锁已经彻底释放，则设置“锁”的持有线程为null，设置当前线程的状态为空，然后唤醒后继线程。

## 非公平锁
### 获取非公平锁(基于JDK1.7.0_40)
非公平锁和公平锁在获取锁的方法上，流程是一样的；它们的区别主要表现在“尝试获取锁的机制不同”。
简单点说，“公平锁”在每次尝试获取锁时，都是采用公平策略(根据等待队列依次排序等待)；而“非公平锁”在每次尝试获取锁时，都是采用的非公平策略(无视等待队列，直接尝试获取锁，如果锁是空闲的，即可获取状态，则获取锁)。
前面已经详细介绍了获取公平锁的流程和机制；下面，通过代码分析以下获取非公平锁的流程。

- lock()
lock()在ReentrantLock.java的NonfairSync类中实现，它的源码如下：
```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```
	说明：
	lock()会先通过compareAndSet(0, 1)来判断“锁”是不是空闲状态。是的话，“当前线程”直接获取“锁”；否则的话，调用acquire(1)获取锁。
	- compareAndSetState()是CAS函数，它的作用是比较并设置当前锁的状态。若锁的状态值为0，则设置锁的状态值为1。
	- setExclusiveOwnerThread(Thread.currentThread())的作用是，设置“当前线程”为“锁”的持有者。

公平锁和非公平锁关于lock()的对比：
公平锁：公平锁的lock()函数，会直接调用acquire(1)。
非公平锁：非公平锁会先判断当前锁的状态是不是空闲，是的话，就不排队，而是直接获取锁。

- acquire()
acquire()在AQS中实现的，它的源码如下：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
	- “当前线程”首先通过tryAcquire()尝试获取锁。获取成功的话，直接返回；尝试失败的话，进入到等待队列依次排序，然后获取锁。
	- “当前线程”尝试失败的情况下，会先通过addWaiter(Node.EXCLUSIVE)来将“当前线程”加入到"CLH队列(非阻塞的FIFO队列)"末尾。
	- 然后，调用acquireQueued()获取锁。在acquireQueued()中，当前线程会等待它在“CLH队列”中前面的所有线程执行并释放锁之后，才能获取锁并返回。如果“当前线程”在休眠等待过程中被中断过，则调用selfInterrupt()来自己产生一个中断。
	
公平锁和非公平锁关于acquire()的对比：
公平锁和非公平锁，只有tryAcquire()函数的实现不同；即它们尝试获取锁的机制不同。这就是我们所说的“它们获取锁策略的不同所在之处”。
前面已经详细介绍了acquire()涉及到的各个函数。这里仅对它们有差异的函数tryAcquire()进行说明。	

非公平锁的tryAcquire()在ReentrantLock.java的NonfairSync类中实现，源码如下：
```
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

nonfairTryAcquire()在ReentrantLock.java的Sync类中实现，源码如下：
```java
final boolean nonfairTryAcquire(int acquires) {
    // 获取“当前线程”
    final Thread current = Thread.currentThread();
    // 获取“锁”的状态
    int c = getState();
    // c=0意味着“锁没有被任何线程锁拥有”
    if (c == 0) {
        // 若“锁没有被任何线程锁拥有”，则通过CAS函数设置“锁”的状态为acquires。
        // 同时，设置“当前线程”为锁的持有者。
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 如果“锁”的持有者已经是“当前线程”，
        // 则将更新锁的状态。
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
	说明：
	根据代码，我们可以分析出，tryAcquire()的作用就是尝试去获取锁。
	- 如果“锁”没有被任何线程拥有，则通过CAS函数设置“锁”的状态为acquires，同时，设置“当前线程”为锁的持有者，然后返回true。
	- 如果“锁”的持有者已经是当前线程，则将更新锁的状态即可。
	- 如果不术语上面的两种情况，则认为尝试失败。

公平锁和非公平锁关于tryAcquire()的对比：
公平锁和非公平锁，它们尝试获取锁的方式不同。
公平锁在尝试获取锁时，即使“锁”没有被任何线程锁持有，它也会判断自己是不是CLH等待队列的表头；是的话，才获取锁。
而非公平锁在尝试获取锁时，如果“锁”没有被任何线程持有，则不管它在CLH队列的何处，它都直接获取锁。

### 释放非公平锁(基于JDK1.7.0_40)
非公平锁和公平锁在释放锁的方法和策略上是一样的。
而在前面的“Java多线程系列--“JUC锁”04之 公平锁(二) ”中，已经对“释放公平锁”进行了介绍；这里就不再重复的进行说明。

总结
公平锁和非公平锁的区别，是在获取锁的机制上的区别。表现在，在尝试获取锁时 —— 公平锁，只有在当前线程是CLH等待队列的表头时，才获取锁；而非公平锁，只要当前锁处于空闲状态，则直接获取锁，而不管CLH等待队列中的顺序。
只有当非公平锁尝试获取锁失败的时候，它才会像公平锁一样，进入CLH等待队列排序等待。
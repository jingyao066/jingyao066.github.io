---
title: 'multithreading-yield+sleep+join'
tags: multithreading
date: 2019-03-26 15:07:56
---

# yield()介绍
yield()的作用是让步。它能让当前线程由`运行状态`进入`就绪状态`，从而让其他具有相同优先级的等待线程获取执行权。
但是，并不能保证当前线程调佣yield()之后，其他具有相同优先级的线程就一定能获得执行权，也有可能是当前线程又进入到运行状态继续运行。

# yield()示例
```java
// YieldTest.java的源码
class ThreadA extends Thread{
    public ThreadA(String name){ 
        super(name); 
    } 
    public synchronized void run(){ 
        for(int i=0; i <10; i++){ 
            System.out.printf("%s [%d]:%d\n", this.getName(), this.getPriority(), i); 
            // i整除4时，调用yield
            if (i%4 == 0)
                Thread.yield();
        } 
    } 
} 

public class YieldTest{ 
    public static void main(String[] args){ 
        ThreadA t1 = new ThreadA("t1"); 
        ThreadA t2 = new ThreadA("t2"); 
        t1.start(); 
        t2.start();
    } 
}
```
某一次的运行结果：
```
t1 [5]:0
t2 [5]:0
t1 [5]:1
t1 [5]:2
t1 [5]:3
t1 [5]:4
t1 [5]:5
t1 [5]:6
t1 [5]:7
t2 [5]:1
t2 [5]:2
t2 [5]:3
t2 [5]:4
t2 [5]:5
t2 [5]:6
t2 [5]:7
t2 [5]:8
t1 [5]:8
t2 [5]:9
t1 [5]:9
```
结果说明：
`线程t1`在能被4整数的时候，并没有切换到`线程t2`。这表明，yield()虽然可以让线程由`运行状态`进入到`就绪状态`；但是，它不一定会让其它线程获取CPU执行权(即，其它线程进入到`运行状态`)，即使这个`其它线程`与当前调用yield()的线程具有相同的优先级。

# yield() 与 wait()的比较
我们知道，wait()的作用是让当前线程由`运行状态`进入`等待(阻塞)状态`的同时，也会释放同步锁。而yield()的作用是让步，它也会让当前线程离开`运行状态`。它们的区别是：
- wait()是让线程由`运行状态`进入到`等待(阻塞)状态`，而yield()是让线程由`运行状态`进入到`就绪状态`。
- wait()是会线程释放它所持有对象的同步锁，而yield()方法不会释放锁。

下面通过示例演示yield()是不会释放锁的。
```java
// YieldLockTest.java 的源码
public class YieldLockTest{ 

    private static Object obj = new Object();

    public static void main(String[] args){ 
        ThreadA t1 = new ThreadA("t1"); 
        ThreadA t2 = new ThreadA("t2"); 
        t1.start(); 
        t2.start();
    } 

    static class ThreadA extends Thread{
        public ThreadA(String name){ 
            super(name); 
        } 
        public void run(){ 
            // 获取obj对象的同步锁
            synchronized (obj) {
                for(int i=0; i <10; i++){ 
                    System.out.printf("%s [%d]:%d\n", this.getName(), this.getPriority(), i); 
                    // i整除4时，调用yield
                    if (i%4 == 0)
                        Thread.yield();
                }
            }
        } 
    } 
}
```
某一次的运行结果：
```
t1 [5]:0
t1 [5]:1
t1 [5]:2
t1 [5]:3
t1 [5]:4
t1 [5]:5
t1 [5]:6
t1 [5]:7
t1 [5]:8
t1 [5]:9
t2 [5]:0
t2 [5]:1
t2 [5]:2
t2 [5]:3
t2 [5]:4
t2 [5]:5
t2 [5]:6
t2 [5]:7
t2 [5]:8
t2 [5]:9
```
结果说明：
主线程main中启动了两个线程t1和t2。t1和t2在run()会引用同一个对象的同步锁，即`synchronized(obj)`。在t1运行过程中，虽然它会调用`Thread.yield()`；但是，t2是不会获取cpu执行权的。因为，t1并没有释放`obj所持有的同步锁`

# sleep()介绍
sleep() 定义在Thread.java中。
sleep() 的作用是让当前线程休眠，即当前线程会从`运行状态`进入到`休眠(阻塞)状态`。sleep()会指定休眠时间，线程休眠的时间会大于/等于该休眠时间；在线程重新被唤醒时，它会由`阻塞状态`变成`就绪状态`，从而等待cpu的调度执行。

# sleep()示例
```java
// SleepTest.java的源码
class ThreadA extends Thread{
    public ThreadA(String name){
        super(name);
    }
    public synchronized void run() {
        try {
            for(int i=0; i <10; i++){
                System.out.printf("%s: %d\n", this.getName(), i);
                // i能被4整除时，休眠1000毫秒
                if (i%4 == 0)
                    Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Demo{
    public static void main(String[] args){
        ThreadA t1 = new ThreadA("t1");
        t1.start();
    }
}
```
运行结果：
```
t1: 0(会休眠1妙)
t1: 1
t1: 2
t1: 3
t1: 4(会休眠1妙)
t1: 5
t1: 6
t1: 7
t1: 8(会休眠1妙)
t1: 9
```
结果说明：
在主线程main中启动线程t1。t1启动之后，当t1中的计算i能被4整除时，t1会通过`Thread.sleep(1000)`休眠1000毫秒，可以明显的看到3次停顿。

# sleep()和wait()比较
我们知道，wait()的作用是让当前线程由`运行状态`进入`等待(阻塞)状态`的同时，也会释放同步锁。而sleep()的作用是也是让当前线程由`运行状态`进入到`休眠(阻塞)状态`。
但是，wait()会释放对象的同步锁，而sleep()则不会释放锁。

下面通过示例演示sleep()是不会释放锁的。
```java
// SleepLockTest.java的源码
public class Demo {

    private static Object obj = new Object();

    public static void main(String[] args) {
        ThreadA t1 = new ThreadA("t1");
        ThreadA t2 = new ThreadA("t2");
        t1.start();
        t2.start();
    }

    static class ThreadA extends Thread {
        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            // 获取obj对象的同步锁
            synchronized (obj) {
                try {
                    for (int i = 0; i < 10; i++) {
                        System.out.printf("%s: %d\n", this.getName(), i);
                        // i能被4整除时，休眠100毫秒
                        if (i % 4 == 0)
                            Thread.sleep(100);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
运行结果：
```
t1: 0
t1: 1
t1: 2
t1: 3
t1: 4
t1: 5
t1: 6
t1: 7
t1: 8
t1: 9
t2: 0
t2: 1
t2: 2
t2: 3
t2: 4
t2: 5
t2: 6
t2: 7
t2: 8
t2: 9
```
结果说明：
主线程main中启动了两个线程t1和t2。t1和t2在run()会引用同一个对象的同步锁，即synchronized(obj)。在t1运行过程中，虽然它会调用`Thread.sleep(100)`；但是，t2是不会获取cpu执行权的。因为，t1并没有释放`obj所持有的同步锁`。
注意，若我们注释掉`synchronized(obj)`后再次执行该程序，t1和t2是可以相互切换的。下面是注释调`synchronized(obj)`之后的源码：
```java
// SleepLockTest.java的源码(注释掉synchronized(obj))
public class SleepLockTest{ 

    private static Object obj = new Object();

    public static void main(String[] args){ 
        ThreadA t1 = new ThreadA("t1"); 
        ThreadA t2 = new ThreadA("t2"); 
        t1.start(); 
        t2.start();
    } 

    static class ThreadA extends Thread{
        public ThreadA(String name){ 
            super(name); 
        } 
        public void run(){ 
            // 获取obj对象的同步锁
//            synchronized (obj) {
                try {
                    for(int i=0; i <10; i++){ 
                        System.out.printf("%s: %d\n", this.getName(), i); 
                        // i能被4整除时，休眠100毫秒
                        if (i%4 == 0)
                            Thread.sleep(100);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
//            }
        } 
    } 
}
```

# join()介绍
`join()`定义在Thread.java中。
`join()`的作用：让`主线程`等待`子线程`结束之后才能继续运行。这句话可能有点晦涩，我们还是通过例子去理解。
```java
// 主线程
public class Father extends Thread {
    public void run() {
        Son s = new Son();
        s.start();
        s.join();
        ...
    }
}
// 子线程
public class Son extends Thread {
    public void run() {
        ...
    }
}
```
说明：
上面的有两个类`Father(主线程类)`和`Son(子线程类)`。因为Son是在Father中创建并启动的，所以，Father是主线程类，Son是子线程类。
在Father主线程中，通过`new Son()`新建`子线程s`。接着通过`s.start()`启动`子线程s`，并且调用`s.join()`。在调用`s.join()`之后，Father主线程会一直等待，直到`子线程s`运行完毕；在`子线程s`运行完毕之后，Father主线程才能接着运行。 这也就是我们所说的`join()`的作用：`让主线程等待子线程结束之后才能继续运行`

# join()源码(基于JDK1.8.0_201-b09)
```java
public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```
说明：
当`millis==0`时，会进入`while(isAlive())`循环；即只要子线程是活的，主线程就不停的等待。

问题：
虽然`s.join()`被调用的地方是发生在`Father主线程`中，但是`s.join()`是通过`子线程s`去调用的`join()`。那么，`join()`方法中的`isAlive()`应该是判断`子线程s`是不是Alive状态；对应的`wait(0)`也应该是`让子线程s`等待才对。但如果是这样的话，`s.join()`的作用怎么可能是让主线程等待，直到子线程s完成为止呢，应该是让子线程等待才对(因为调用子线程对象s的wait方法嘛)？
答案：`wait()`的作用是让`当前线程`等待，而这里的`当前线程`是指当前在CPU上运行的线程。所以，虽然是调用子线程的`wait()`方法，但是它是通过`主线程`去调用的；所以，休眠的是主线程，而不是`子线程`。

# join()示例
```
// JoinTest.java的源码
public class JoinTest{ 

    public static void main(String[] args){ 
        try {
            ThreadA t1 = new ThreadA("t1"); // 新建“线程t1”

            t1.start();                     // 启动“线程t1”
            t1.join();                        // 将“线程t1”加入到“主线程main”中，并且“主线程main()会等待它的完成”
            System.out.printf("%s finish\n", Thread.currentThread().getName()); 
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    } 

    static class ThreadA extends Thread{

        public ThreadA(String name){ 
            super(name); 
        } 
        public void run(){ 
            System.out.printf("%s start\n", this.getName()); 

            // 延时操作
            for(int i=0; i <1000000; i++)
               ;

            System.out.printf("%s finish\n", this.getName()); 
        } 
    } 
}
```
运行结果：
```
t1 start
t1 finish
main finish
```
结果说明：
运行流程如图 
(01) 在“主线程main”中通过 new ThreadA("t1") 新建“线程t1”。 接着，通过 t1.start() 启动“线程t1”，并执行t1.join()。
(02) 执行t1.join()之后，“主线程main”会进入“阻塞状态”等待t1运行结束。“子线程t1”结束之后，会唤醒“主线程main”，“主线程”重新获取cpu执行权，继续运行。

![](multithreading-yield+sleep+join/1.png)

---
title: multithreading-JUC原子类
tags: multithreading
date: 2019-03-28 17:14:01
---

# JUC概述
JDK1.5新增了`java.util.concurrent`包(简称JUC)，就是并发包，用于解决并发编程的一些问题。

根据修改的数据类型，可以将JUC包中的原子操作类可以分为4类。
1. 基本类型: AtomicInteger, AtomicLong, AtomicBoolean ;
2. 数组类型: AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray ;
3. 引用类型: AtomicReference, AtomicStampedRerence, AtomicMarkableReference ;
4. 对象的属性修改类型: AtomicIntegerFieldUpdater, AtomicLongFieldUpdater, AtomicReferenceFieldUpdater 

这些类存在的目的是对相应的数据进行原子操作。所谓原子操作，是指**操作过程不会被中断，保证数据操作是以原子方式进行的。**

# AtomicLong原子类
AtomicInteger, AtomicLong和AtomicBoolean这3个基本类型的原子类的原理和用法相似，现只介绍AtomicLong。
AtomicLong是作用是对长整形进行原子操作。
在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。

## AtomicLong函数列表
```
// 构造函数
AtomicLong()
// 创建值为initialValue的AtomicLong对象
AtomicLong(long initialValue)
// 以原子方式设置当前值为newValue。
final void set(long newValue) 
// 获取当前值
final long get() 
// 以原子方式将当前值减 1，并返回减1后的值。等价于“--num”
final long decrementAndGet() 
// 以原子方式将当前值减 1，并返回减1前的值。等价于“num--”
final long getAndDecrement() 
// 以原子方式将当前值加 1，并返回加1后的值。等价于“++num”
final long incrementAndGet() 
// 以原子方式将当前值加 1，并返回加1前的值。等价于“num++”
final long getAndIncrement()
// 以原子方式将delta与当前值相加，并返回相加后的值。
final long addAndGet(long delta) 
// 以原子方式将delta添加到当前值，并返回相加前的值。
final long getAndAdd(long delta) 
// 如果当前值 == expect，则以原子方式将该值设置为update。成功返回true，否则返回false，并且不修改原值。
final boolean compareAndSet(long expect, long update)
// 以原子方式设置当前值为newValue，并返回旧值。
final long getAndSet(long newValue)
// 返回当前值对应的int值
int intValue() 
// 获取当前值对应的long值
long longValue()
// 以 float 形式返回当前值
float floatValue()
// 以 double 形式返回当前值
double doubleValue()
// 最后设置为给定值。延时设置变量值，这个等价于set()方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能延时（尽管可以忽略），所以如果不是想立即读取设置的新值，允许在“后台”修改值，那么此方法就很有用。如果还是难以理解，这里就类似于启动一个后台线程如执行修改新值的任务，原线程就不等待修改结果立即返回（这种解释其实是不正确的，但是可以这么理解）。
final void lazySet(long newValue)
// 如果当前值 == 预期值，则以原子方式将该设置为给定的更新值。JSR规范中说：以原子方式读取和有条件地写入变量但不 创建任何 happen-before 排序，因此不提供与除 weakCompareAndSet 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是等效的，都调用了unsafe.compareAndSwapInt()完成操作。
final boolean weakCompareAndSet(long expect, long update)
```
## AtomicLong源码解析

## AtomicLong示例
```
// LongTest.java的源码
import java.util.concurrent.atomic.AtomicLong;

public class LongTest {
    
    public static void main(String[] args){

        // 新建AtomicLong对象
        AtomicLong mAtoLong = new AtomicLong();

        mAtoLong.set(0x0123456789ABCDEFL);
        System.out.printf("%20s : 0x%016X\n", "get()", mAtoLong.get());
        System.out.printf("%20s : 0x%016X\n", "intValue()", mAtoLong.intValue());
        System.out.printf("%20s : 0x%016X\n", "longValue()", mAtoLong.longValue());
        System.out.printf("%20s : %s\n", "doubleValue()", mAtoLong.doubleValue());
        System.out.printf("%20s : %s\n", "floatValue()", mAtoLong.floatValue());

        System.out.printf("%20s : 0x%016X\n", "getAndDecrement()", mAtoLong.getAndDecrement());
        System.out.printf("%20s : 0x%016X\n", "decrementAndGet()", mAtoLong.decrementAndGet());
        System.out.printf("%20s : 0x%016X\n", "getAndIncrement()", mAtoLong.getAndIncrement());
        System.out.printf("%20s : 0x%016X\n", "incrementAndGet()", mAtoLong.incrementAndGet());

        System.out.printf("%20s : 0x%016X\n", "addAndGet(0x10)", mAtoLong.addAndGet(0x10));
        System.out.printf("%20s : 0x%016X\n", "getAndAdd(0x10)", mAtoLong.getAndAdd(0x10));

        System.out.printf("\n%20s : 0x%016X\n", "get()", mAtoLong.get());

        System.out.printf("%20s : %s\n", "compareAndSet()", mAtoLong.compareAndSet(0x12345679L, 0xFEDCBA9876543210L));
        System.out.printf("%20s : 0x%016X\n", "get()", mAtoLong.get());
    }
}
```
运行结果：
```
get() : 0x0123456789ABCDEF
          intValue() : 0x0000000089ABCDEF
         longValue() : 0x0123456789ABCDEF
       doubleValue() : 8.1985529216486896E16
        floatValue() : 8.1985531E16
   getAndDecrement() : 0x0123456789ABCDEF
   decrementAndGet() : 0x0123456789ABCDED
   getAndIncrement() : 0x0123456789ABCDED
   incrementAndGet() : 0x0123456789ABCDEF
     addAndGet(0x10) : 0x0123456789ABCDFF
     getAndAdd(0x10) : 0x0123456789ABCDFF

               get() : 0x0123456789ABCE0F
     compareAndSet() : false
               get() : 0x0123456789ABCE0F
```

# AtomicLongArray原子类
AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray这3个数组类型的原子类的原理和用法相似，现只介绍AtomicLongArray。
AtomicLongArray的作用则是对"长整形数组"进行原子操作。

## AtomicLongArray函数列表
```
// 创建给定长度的新 AtomicLongArray。
AtomicLongArray(int length)
// 创建与给定数组具有相同长度的新 AtomicLongArray，并从给定数组复制其所有元素。
AtomicLongArray(long[] array)

// 以原子方式将给定值添加到索引 i 的元素。
long addAndGet(int i, long delta)
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean compareAndSet(int i, long expect, long update)
// 以原子方式将索引 i 的元素减1。
long decrementAndGet(int i)
// 获取位置 i 的当前值。
long get(int i)
// 以原子方式将给定值与索引 i 的元素相加。
long getAndAdd(int i, long delta)
// 以原子方式将索引 i 的元素减 1。
long getAndDecrement(int i)
// 以原子方式将索引 i 的元素加 1。
long getAndIncrement(int i)
// 以原子方式将位置 i 的元素设置为给定值，并返回旧值。
long getAndSet(int i, long newValue)
// 以原子方式将索引 i 的元素加1。
long incrementAndGet(int i)
// 最终将位置 i 的元素设置为给定值。
void lazySet(int i, long newValue)
// 返回该数组的长度。
int length()
// 将位置 i 的元素设置为给定值。
void set(int i, long newValue)
// 返回数组当前值的字符串表示形式。
String toString()
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean    weakCompareAndSet(int i, long expect, long update)
```
## AtomicLongArray使用示例
```
// LongArrayTest.java的源码
import java.util.concurrent.atomic.AtomicLongArray;

public class LongArrayTest {
    
    public static void main(String[] args){

        // 新建AtomicLongArray对象
        long[] arrLong = new long[] {10, 20, 30, 40, 50};
        AtomicLongArray ala = new AtomicLongArray(arrLong);

        ala.set(0, 100);
        for (int i=0, len=ala.length(); i<len; i++) 
            System.out.printf("get(%d) : %s\n", i, ala.get(i));

        System.out.printf("%20s : %s\n", "getAndDecrement(0)", ala.getAndDecrement(0));
        System.out.printf("%20s : %s\n", "decrementAndGet(1)", ala.decrementAndGet(1));
        System.out.printf("%20s : %s\n", "getAndIncrement(2)", ala.getAndIncrement(2));
        System.out.printf("%20s : %s\n", "incrementAndGet(3)", ala.incrementAndGet(3));

        System.out.printf("%20s : %s\n", "addAndGet(100)", ala.addAndGet(0, 100));
        System.out.printf("%20s : %s\n", "getAndAdd(100)", ala.getAndAdd(1, 100));

        System.out.printf("%20s : %s\n", "compareAndSet()", ala.compareAndSet(2, 31, 1000));
        System.out.printf("%20s : %s\n", "get(2)", ala.get(2));
    }
}
```
运行结果：
```
get(0) : 100
get(1) : 20
get(2) : 30
get(3) : 40
get(4) : 50
  getAndDecrement(0) : 100
  decrementAndGet(1) : 19
  getAndIncrement(2) : 30
  incrementAndGet(3) : 41
      addAndGet(100) : 199
      getAndAdd(100) : 19
     compareAndSet() : true
              get(2) : 1000
```

# AtomicReference原子类
AtomicReference是作用是对"对象"进行原子操作。

## AtomicReference函数列表
```
// 使用 null 初始值创建新的 AtomicReference。
AtomicReference()
// 使用给定的初始值创建新的 AtomicReference。
AtomicReference(V initialValue)

// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean compareAndSet(V expect, V update)
// 获取当前值。
V get()
// 以原子方式设置为给定值，并返回旧值。
V getAndSet(V newValue)
// 最终设置为给定值。
void lazySet(V newValue)
// 设置为给定值。
void set(V newValue)
// 返回当前值的字符串表示形式。
String toString()
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean weakCompareAndSet(V expect, V update)
```

## AtomicReference示例
```
// AtomicReferenceTest.java的源码
import java.util.concurrent.atomic.AtomicReference;

public class AtomicReferenceTest {
    
    public static void main(String[] args){

        // 创建两个Person对象，它们的id分别是101和102。
        Person p1 = new Person(101);
        Person p2 = new Person(102);
        // 新建AtomicReference对象，初始化它的值为p1对象
        AtomicReference ar = new AtomicReference(p1);
        // 通过CAS设置ar。如果ar的值为p1的话，则将其设置为p2。
        ar.compareAndSet(p1, p2);

        Person p3 = (Person)ar.get();
        System.out.println("p3 is "+p3);
        System.out.println("p3.equals(p1)="+p3.equals(p1));
    }
}

class Person {
    volatile long id;
    public Person(long id) {
        this.id = id;
    }
    public String toString() {
        return "id:"+id;
    }
}
```
运行结果：
```
p3 is id:102
p3.equals(p1)=false
```
结果说明：
结果说明：
新建AtomicReference对象ar时，将它初始化为p1。
紧接着，通过CAS函数对它进行设置。如果ar的值为p1的话，则将其设置为p2。
最后，获取ar对应的对象，并打印结果。p3.equals(p1)的结果为false，这是因为Person并没有覆盖equals()方法，而是采用继承自Object.java的equals()方法；而Object.java中的equals()实际上是调用"=="去比较两个对象，即比较两个对象的地址是否相等。

# AtomicLongFieldUpdater原子类
AtomicIntegerFieldUpdater, AtomicLongFieldUpdater和AtomicReferenceFieldUpdater这3个修改类的成员的原子类型的原理和用法相似。
AtomicLongFieldUpdater可以对指定"类的 'volatile long'类型的成员"进行原子更新。它是基于反射原理实现的。

## AtomicLongFieldUpdater函数列表
```
// 受保护的无操作构造方法，供子类使用。
protected AtomicLongFieldUpdater()

// 以原子方式将给定值添加到此更新器管理的给定对象的字段的当前值。
long addAndGet(T obj, long delta)
// 如果当前值 == 预期值，则以原子方式将此更新器所管理的给定对象的字段设置为给定的更新值。
abstract boolean compareAndSet(T obj, long expect, long update)
// 以原子方式将此更新器管理的给定对象字段当前值减 1。
long decrementAndGet(T obj)
// 获取此更新器管理的在给定对象的字段中保持的当前值。
abstract long get(T obj)
// 以原子方式将给定值添加到此更新器管理的给定对象的字段的当前值。
long getAndAdd(T obj, long delta)
// 以原子方式将此更新器管理的给定对象字段当前值减 1。
long getAndDecrement(T obj)
// 以原子方式将此更新器管理的给定对象字段的当前值加 1。
long getAndIncrement(T obj)
// 将此更新器管理的给定对象的字段以原子方式设置为给定值，并返回旧值。
long getAndSet(T obj, long newValue)
// 以原子方式将此更新器管理的给定对象字段当前值加 1。
long incrementAndGet(T obj)
// 最后将此更新器管理的给定对象的字段设置为给定更新值。
abstract void lazySet(T obj, long newValue)
// 为对象创建并返回一个具有给定字段的更新器。
static <U> AtomicLongFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName)
// 将此更新器管理的给定对象的字段设置为给定更新值。
abstract void set(T obj, long newValue)
// 如果当前值 == 预期值，则以原子方式将此更新器所管理的给定对象的字段设置为给定的更新值。
abstract boolean weakCompareAndSet(T obj, long expect, long update)
```

## AtomicLongFieldUpdater示例
```
// LongTest.java的源码
import java.util.concurrent.atomic.AtomicLongFieldUpdater;

public class LongFieldTest {
    
    public static void main(String[] args) {

        // 获取Person的class对象
        Class cls = Person.class; 
        // 新建AtomicLongFieldUpdater对象，传递参数是“class对象”和“long类型在类中对应的名称”
        AtomicLongFieldUpdater mAtoLong = AtomicLongFieldUpdater.newUpdater(cls, "id");
        Person person = new Person(12345678L);

        // 比较person的"id"属性，如果id的值为12345678L，则设置为1000。
        mAtoLong.compareAndSet(person, 12345678L, 1000);
        System.out.println("id="+person.getId());
    }
}

class Person {
    volatile long id;
    public Person(long id) {
        this.id = id;
    }
    public void setId(long id) {
        this.id = id;
    }
    public long getId() {
        return id;
    }
}
```
运行结果：
`id=100`



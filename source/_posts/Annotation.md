---
title: Annotation
tags: 
    - java
date: 2019-04-30 16:36:38
---

# Annotation架构
![](Annotation/1.jpg)
从中，我们可以看出：
- 1个Annotation 和 1个RetentionPolicy关联。
可以理解为：每1个Annotation对象，都会有唯一的RetentionPolicy属性。
- 1个Annotation 和 1~n个ElementType关联。
可以理解为：对于每1个Annotation对象，可以有若干个ElementType属性。
- Annotation 有许多实现类，包括：Deprecated，Documented，Inherited，Override等等。
Annotation 的每一个实现类，都`和1个RetentionPolicy关联`并且`和1~n个ElementType关联`。

下面，我先介绍框架图的左半边，即Annotation，RetentionPolicy，ElementType。

# Annotation组成部分
java annotation 的组成中，有3个非常重要的主干类。它们分别是：
- Annotation.java
```java
package java.lang.annotation;
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```

- ElementType.java
```java
package java.lang.annotation;

public enum ElementType {
    TYPE,               /* 类、接口（包括注释类型）或枚举声明  */

    FIELD,              /* 字段声明（包括枚举常量）  */

    METHOD,             /* 方法声明  */

    PARAMETER,          /* 参数声明  */

    CONSTRUCTOR,        /* 构造方法声明  */

    LOCAL_VARIABLE,     /* 局部变量声明  */

    ANNOTATION_TYPE,    /* 注释类型声明  */

    PACKAGE             /* 包声明  */
}
```

- RetentionPolicy.java
```java
package java.lang.annotation;
public enum RetentionPolicy {
    SOURCE,  /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */

    CLASS,  /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */

    RUNTIME  /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
}
```
说明：
- Annotation 就是个接口。
每1个Annotation都与1个RetentionPolicy关联，并且与1～n个ElementType关联。
可以通俗的理解为：每1个Annotation对象，都会有唯一的RetentionPolicy属性；至于ElementType属性，则有1~n个。

- ElementType 是Enum枚举类型，它用来指定Annotation的类型。
每1个Annotation都与1～n个ElementType关联。当Annotation与某个ElementType关联时，就意味着：Annotation有了某种用途。
例如，若一个Annotation对象是METHOD类型，则该Annotation只能用来修饰方法。

- RetentionPolicy 是Enum枚举类型，它用来指定Annotation的策略。通俗点说，就是不同RetentionPolicy类型的Annotation的作用域不同。
每1个Annotation都与1个RetentionPolicy关联。
    + 若Annotation的类型为 SOURCE，则意味着：Annotation仅存在于编译器处理期间，编译器处理完之后，该Annotation就没用了。
    例如，`@Override`标志就是一个Annotation。当它修饰一个方法的时候，就意味着该方法覆盖父类的方法；并且在编译期间会进行语法检查，编译器处理完后，`@Override`就没有任何作用了。
    +  若Annotation的类型为 CLASS，则意味着：编译器将Annotation存储于类对应的.class文件中，它是Annotation的默认行为。
    + 若Annotation的类型为 RUNTIME，则意味着：编译器将Annotation存储于class文件中，并且可由JVM读入。

这时，只需要记住每1个Annotation都与1个RetentionPolicy关联，并且与1～n个ElementType关联。学完后面的内容之后，再回头看这些内容，会更容易理解。

# java自带的Annotation
## Annotation通用定义
```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation1 {
}
```
说明：
上面的作用是定义一个Annotation，它的名字是MyAnnotation1。定义了MyAnnotation1之后，我们可以在代码中通过`@MyAnnotation1`来使用它。
其它的，@Documented，@Target，@Retention，@interface都是来修饰MyAnnotation1的。下面分别说说它们的含义：
- @interface
使用@interface定义注解时，意味着它实现了java.lang.annotation.Annotation接口，即该注解就是一个Annotation。
定义Annotation时，@interface是必须的。
注意：它和我们通常的implemented实现接口的方法不同。Annotation接口的实现细节都由编译器完成。通过@interface定义注解后，该注解不能继承其他的注解或接口。

- @Documented 
类和方法的Annotation在缺省情况下是不出现在javadoc中的。如果使用@Documented修饰该Annotation，则表示它可以出现在javadoc中。
定义Annotation时，@Documented可有可无；若没有定义，则Annotation不会出现在javadoc中。

- @Target(ElementType.TYPE)
前面我们说过，ElementType 是Annotation的类型属性。而@Target的作用，就是来指定Annotation的类型属性。
@Target(ElementType.TYPE) 的意思就是指定该Annotation的类型是ElementType.TYPE。这就意味着，MyAnnotation1是来修饰`类、接口（包括注释类型）或枚举声明`的注解。
定义Annotation时，@Target可有可无。若有@Target，则该Annotation只能用于它所指定的地方；若没有@Target，则该Annotation可以用于任何地方。

- @Retention(RetentionPolicy.RUNTIME)
前面我们说过，RetentionPolicy 是Annotation的策略属性，而@Retention的作用，就是指定Annotation的策略属性。
@Retention(RetentionPolicy.RUNTIME) 的意思就是指定该Annotation的策略是RetentionPolicy.RUNTIME。这就意味着，编译器会将该Annotation信息保留在.class文件中，并且能被虚拟机读取。
定义Annotation时，@Retention可有可无。若没有@Retention，则默认是RetentionPolicy.CLASS。

## java自带的Annotation
通过上面的示例，我们能理解：
`@interface`：用来声明Annotation
`@Documented`：用来表示该Annotation是否会出现在javadoc中
`@Target`：用来指定Annotation的类型
`@Retention`：用来指定Annotation的策略

理解这一点之后，我们就很容易理解java中自带的Annotation的实现类，即Annotation架构图的右半边。

java 常用的Annotation：

`@Deprecated`：@Deprecated 所标注内容，不建议使用。
`@Override`：@Override 只能标注方法，表示该方法覆盖父类中的方法。
`@Documented`：@Documented 所标注内容，可以出现在javadoc中。
`@Inherited`：@Inherited只能被用来标注`Annotation类型`，它所标注的Annotation具有继承性。
`@Retention`：@Retention只能被用来标注`Annotation类型`，用来指定Annotation的RetentionPolicy属性。
`@Target`：@Target只能被用来标注`Annotation类型`，用来指定Annotation的ElementType属性。
`@SuppressWarnings`：@SuppressWarnings 所标注内容产生的警告，编译器会对这些警告保持静默。

由于`@Deprecated和@Override`类似，`@Documented, @Inherited, @Retention, @Target`类似，下面，我们只对@Deprecated, @Inherited, @SuppressWarnings 这3个Annotation进行说明。

## @Deprecated
@Deprecated 的定义如下：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Deprecated {
}
```
说明：
- @interface -- 它的用来修饰Deprecated，意味着Deprecated实现了java.lang.annotation.Annotation接口；即Deprecated就是一个注解。
- @Documented -- 它的作用是说明该注解能出现在javadoc中。
- @Retention(RetentionPolicy.RUNTIME)：
它的作用是指定Deprecated的策略是`RetentionPolicy.RUNTIME`。这就意味着，编译器会将Deprecated的信息保留在.class文件中，并且能被虚拟机读取。
- @Deprecated 所标注内容，不再被建议使用。

例如，若某个方法被 @Deprecated 标注，则该方法不再被建议使用。如果有开发人员试图使用或重写被@Deprecated标示的方法，编译器会给相应的提示信息。示例如下:
![](Annotation/2.jpg)

源码如下(DeprecatedTest.java)：
```java
package com.skywang.annotation;

import java.util.Date;
import java.util.Calendar;

public class DeprecatedTest {
    // @Deprecated 修饰 getString1(),表示 它是建议不被使用的函数
    @Deprecated
    private static void getString1(){
        System.out.println("Deprecated Method");
    }
    
    private static void getString2(){
        System.out.println("Normal Method");
    }
    
    // Date是日期/时间类。java已经不建议使用该类了
    private static void testDate() {
        Date date = new Date(113, 8, 25);
        System.out.println(date.getYear());
    }
    // Calendar是日期/时间类。java建议使用Calendar取代Date表示“日期/时间”
    private static void testCalendar() {
        Calendar cal = Calendar.getInstance();
        System.out.println(cal.get(Calendar.YEAR));
    }
    
    public static void main(String[] args) {
        getString1(); 
        getString2();
        testDate(); 
        testCalendar();
    }
}
```
说明：
说明：
上面是eclipse中的截图，比较类中getString1() 和 getString2()以及testDate() 和 testCalendar()。
- getString1() 被@Deprecated标注，意味着建议不再使用getString1()；所以getString1()的定义和调用时，都会一横线。这一横线是eclipse()对@Deprecated方法的处理。
getString2() 没有被@Deprecated标注，它的显示正常。
- testDate() 调用了Date的相关方法，而java已经建议不再使用Date操作日期/时间。因此，在调用Date的API时，会产生警告信息，途中的warnings。
testCalendar() 调用了Calendar的API来操作日期/时间，java建议用Calendar取代Date。因此，操作Calendar不会产生warning。

## @Inherited
@Inherited 的定义如下：
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```
说明：
- @interface -- 它的用来修饰Inherited，意味着Inherited实现了java.lang.annotation.Annotation接口；即Inherited就是一个注解。
- @Documented -- 它的作用是说明该注解能出现在javadoc中。
- @Retention(RetentionPolicy.RUNTIME) -- 它的作用是指定Inherited的策略是RetentionPolicy.RUNTIME。这就意味着，编译器会将Inherited的信息保留在.class文件中，并且能被虚拟机读取。
- @Target(ElementType.ANNOTATION_TYPE) -- 它的作用是指定Inherited的类型是ANNOTATION_TYPE。这就意味着，@Inherited只能被用来标注“Annotation类型”。
- @Inherited 的含义是，它所标注的Annotation将具有继承性。

假设，我们定义了某个Annotaion，它的名称是MyAnnotation，并且MyAnnotation被标注为@Inherited。现在，某个类Base使用了MyAnnotation，则Base具有了`具有了注解MyAnnotation`；现在，Sub继承了Base，由于MyAnnotation是@Inherited的(具有继承性)，所以，Sub也`具有了注解MyAnnotation`。

@Inherited的使用示例
源码如下(InheritableSon.java)：
```java
/**
 * @Inherited 演示示例
 * 
 * @author skywang
 * @email kuiwu-wang@163.com
 */
package com.skywang.annotation;

import java.lang.annotation.Target;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Inherited;

/**
 * 自定义的Annotation。
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@interface Inheritable
{
}

@Inheritable
class InheritableFather
{
    public InheritableFather() {
        // InheritableBase是否具有 Inheritable Annotation
        System.out.println("InheritableFather:"+InheritableFather.class.isAnnotationPresent(Inheritable.class));
    }
}

/**
 * InheritableSon 类只是继承于 InheritableFather，
 */
public class InheritableSon extends InheritableFather
{
    public InheritableSon() {
        super();    // 调用父类的构造函数
        // InheritableSon类是否具有 Inheritable Annotation
        System.out.println("InheritableSon:"+InheritableSon.class.isAnnotationPresent(Inheritable.class));
    }
    
    public static void main(String[] args)
    {
        InheritableSon is = new InheritableSon();
    }
}
```
运行结果：
```
InheritableFather:true
InheritableSon:true
```
现在，我们对InheritableSon.java进行修改：注释掉`Inheritable的@Inherited注解`。
源码如下(InheritableSon.java)：
```java
/**
 * @Inherited 演示示例
 * 
 * @author skywang
 * @email kuiwu-wang@163.com
 */
package com.skywang.annotation;

import java.lang.annotation.Target;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Inherited;

/**
 * 自定义的Annotation。
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
//@Inherited
@interface Inheritable
{
}

@Inheritable
class InheritableFather
{
    public InheritableFather() {
        // InheritableBase是否具有 Inheritable Annotation
        System.out.println("InheritableFather:"+InheritableFather.class.isAnnotationPresent(Inheritable.class));
    }
}

/**
 * InheritableSon 类只是继承于 InheritableFather，
 */
public class InheritableSon extends InheritableFather
{
    public InheritableSon() {
        super();    // 调用父类的构造函数
        // InheritableSon类是否具有 Inheritable Annotation
        System.out.println("InheritableSon:"+InheritableSon.class.isAnnotationPresent(Inheritable.class));
    }
    
    public static void main(String[] args)
    {
        InheritableSon is = new InheritableSon();
    }
}
```
运行结果：
```
InheritableFather:true
InheritableSon:false
```
对比上面的两个结果，我们发现：当注解Inheritable被@Inherited标注时，它具有继承性。否则，没有继承性。

## @SuppressWarnings
@SuppressWarnings 的定义如下：
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {

    String[] value();

}
```


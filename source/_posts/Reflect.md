---
title: Reflect
tags: java
date: 2019-05-06 16:21:35
---

# Reflect概述
Java 反射机制。通俗来讲呢，就是在运行状态中，我们可以根据`已经得的到类的部分信息`来还原`类的全部的信息`。这里`已经得的到类的部分信息`，可以是`类名或类的对象`等信息。`类的全部信息`就是指`类的属性，方法，继承关系和Annotation注解`等内容。

举个简单的例子：假设对于类ReflectionTest.java，我们知道的唯一信息是它的类名是`com.skywang.Reflection`。这时，我们想要知道ReflectionTest.java的其它信息(比如它的构造函数，它的成员变量等等)，要怎么办呢？
这就需要用到反射。通过反射，我们可以解析出ReflectionTest.java的完整信息，包括它的构造函数，成员变量，继承关系等等。

在了解了java反射机制的概念之后，接下来思考一个问题：如何根据类的类名，来获取类的完整信息呢？

这个过程主要分为两步：
1. 根据类名来获取对应类的Class对象。
2. 通过Class对象的函数接口，来读取`类的构造函数，成员变量`等信息。

下面，我们根据示例来加深对这个概念的理解。示例如下(Demo1.java):
```java
package com.skywang.test;

import java.lang.Class;

public class Demo1 {

    public static void main(String[] args) {

        try {
            // 根据“类名”获取 对应的Class对象
            Class<?> cls = Class.forName("com.skywang.test.Person");

            // 新建对象。newInstance()会调用类不带参数的构造函数
            Object obj = cls.newInstance();

            System.out.println("cls="+cls);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class Person {
    public Person() {
        System.out.println("create Person");
    }
}
```
运行结果：
```
create Person
cls=class com.skywang.test.Person
```

说明：
- Person类的完整包名是"com.skywang.test.Person"。而 Class.forName("com.skywang.test.Person"); 这一句的作用是，就是根据Person的包名来获取Person的Class对象。
- 接着，我们调用Class对象的newInstance()方法，创建Person对象。


现在，我们知道了“java反射机制”的概念以及它的原理。有了这个总体思想之后，接下来，我们可以开始对反射进行深入研究了。

# Class 详细说明
## 获取Class对象的方法

我这里总结了4种常用的`获取Class对象`的方法：
- Class.forName("类名字符串")(注意：类名字符串必须是全称，包名+类名)
- 类名.class
- 实例对象.getClass()
- "类名字符串".getClass()

下面，我们通过示例演示这4种方法。示例如下(Demo2.java):
```java
package com.skywang.test;

import java.lang.Class;

public class Demo2 {

    public static void main(String[] args) {

        try {
            // 方法1：Class.forName("类名字符串")  （注意：类名字符串必须是全称，包名+类名）
            //Class cls1 = Class.forName("com.skywang.test.Person");
            Class<?> cls1 = Class.forName("com.skywang.test.Person");
            //Class<Person> cls1 = Class.forName("com.skywang.test.Person");

            // 方法2：类名.class
            Class cls2 = Person.class; 

            // 方法3：实例对象.getClass()
            Person person = new Person();
            Class cls3 = person.getClass();

            // 方法4："类名字符串".getClass()
            String str = "com.skywang.test.Person"; 
            Class cls4 = str.getClass();

            System.out.printf("cls1=%s, cls2=%s, cls3=%s, cls4=%s\n", cls1, cls2, cls3, cls4);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class Person {
    public Person() {
        System.out.println("create Person");
    }
}
```
运行结果：
```
create Person
cls1=class com.skywang.test.Person, cls2=class com.skywang.test.Person, cls3=class com.skywang.test.Person, cls4=class java.lang.String
```

## Class的API说明
Class的全部API如下:
```java
public static Class    forName(String className)
public static Class    forName(String name, boolean initialize, ClassLoader loader)
public Constructor    getConstructor(Class[] parameterTypes)
public Constructor[]    getConstructors()
public Constructor    getDeclaredConstructor(Class[] parameterTypes)
public Constructor[]    getDeclaredConstructors()
public Constructor    getEnclosingConstructor()
public Method    getMethod(String name, Class[] parameterTypes)
public Method[]    getMethods()
public Method    getDeclaredMethod(String name, Class[] parameterTypes)
public Method[]    getDeclaredMethods()
public Method    getEnclosingMethod()
public Field    getField(String name)
public Field[]    getFields()
public Field    getDeclaredField(String name)
public Field[]    getDeclaredFields()
public Type[]    getGenericInterfaces()
public Type    getGenericSuperclass()
public Annotation<A>    getAnnotation(Class annotationClass)
public Annotation[]    getAnnotations()
public Annotation[]    getDeclaredAnnotations()
public boolean    isAnnotation()
public boolean    isAnnotationPresent(Class annotationClass)
public boolean    isAnonymousClass()
public boolean    isArray()
public boolean    isAssignableFrom(Class cls)
public boolean    desiredAssertionStatus()
public Class<U>    asSubclass(Class clazz)
public Class    getSuperclass()
public Class    getComponentType()
public Class    getDeclaringClass()
public Class    getEnclosingClass()
public Class[]    getClasses()
public Class[]    getDeclaredClasses()
public Class[]    getInterfaces()
public boolean    isEnum()
public boolean    isInstance(Object obj)
public boolean    isInterface()
public boolean    isLocalClass()
public boolean    isMemberClass()
public boolean    isPrimitive()
public boolean    isSynthetic()
public String    getSimpleName()
public String    getName()
public String    getCanonicalName()
public String    toString()
public ClassLoader    getClassLoader()
public Package    getPackage()
public int    getModifiers()
public ProtectionDomain    getProtectionDomain()
public URL    getResource(String name)
public InputStream    getResourceAsStream(String name)
public Object    cast(Object obj)
public Object    newInstance()
public Object[]    getSigners()
public Object[]    getEnumConstants()
public TypeVariable[]    getTypeParameters()
```
我们根据类的特性，将Class中的类分为4部分进行说明：构造函数，成员方法，成员变量，类的其它信息(如注解、包名、类名、继承关系等等)。Class中涉及到Annotation(注解)的相关API，可以点击查看前一章节关于Annotation的详细介绍。

### 构造函数
相关API
```java
// 获取“参数是parameterTypes”的public的构造函数
public Constructor    getConstructor(Class[] parameterTypes)
// 获取全部的public的构造函数
public Constructor[]    getConstructors()
// 获取“参数是parameterTypes”的，并且是类自身声明的构造函数，包含public、protected和private方法。
public Constructor    getDeclaredConstructor(Class[] parameterTypes)
// 获取类自身声明的全部的构造函数，包含public、protected和private方法。
public Constructor[]    getDeclaredConstructors()
// 如果这个类是“其它类的构造函数中的内部类”，调用getEnclosingConstructor()就是这个类所在的构造函数；若不存在，返回null。
public Constructor    getEnclosingConstructor()
```


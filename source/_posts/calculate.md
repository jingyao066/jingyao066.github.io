---
title: calculate
tags: java
date: 2019-09-22 20:34:00
---

# BigDecimal
## BigDecimal初始化
这里对比了两种形式，第一种直接value写数字的值，第二种用string来表示
```java
BigDecimal num1 = new BigDecimal(0.005);
BigDecimal num2 = new BigDecimal(1000000);
BigDecimal num3 = new BigDecimal(-1000000);
//尽量用字符串的形式初始化
BigDecimal num12 = new BigDecimal("0.005");
BigDecimal num22 = new BigDecimal("1000000");
BigDecimal num32 = new BigDecimal("-1000000");
```
我们对其进行加减乘除绝对值的运算。
```java
//加法
BigDecimal result1 = num1.add(num2);
BigDecimal result12 = num12.add(num22);

//减法
BigDecimal result2 = num1.subtract(num2);
BigDecimal result22 = num12.subtract(num22);

//乘法
BigDecimal result3 = num1.multiply(num2);
BigDecimal result32 = num12.multiply(num22);

//绝对值
BigDecimal result4 = num3.abs();
BigDecimal result42 = num32.abs();

//除法
BigDecimal result5 = num2.divide(num1,20,BigDecimal.ROUND_HALF_UP);
BigDecimal result52 = num22.divide(num12,20,BigDecimal.ROUND_HALF_UP);
```
我把result全部输出可以看到结果，这里出现了差异，这也是为什么初始化建议使用string的原因。

※ 注意：
- System.out.println()中的数字默认是double类型的，double类型小数计算不精准。
- 使用BigDecimal类构造方法传入double类型时，计算的结果也是不精确的！
因为不是所有的浮点数都能够被精确的表示成一个double 类型值，有些浮点数值不能够被精确的表示成 double 类型值，因此它会被表示成与它最接近的 double 类型的值。必须改用传入String的构造方法。这一点在BigDecimal类的构造方法注释中有说明。

## 除法divide()参数使用
使用除法函数在divide的时候要设置各种参数，要精确的小数位数和舍入模式，不然会出现报错。
我们可以看到divide函数配置的参数如下：
`即为 （BigDecimal divisor 除数， int scale 精确小数位，  int roundingMode 舍入模式）`
可以看到舍入模式有很多种BigDecimal.ROUND_XXXX_XXX, 具体都是什么意思呢。

八种舍入模式解释如下
- ROUND_UP
舍入远离零的舍入模式。
在丢弃非零部分之前始终增加数字(始终对非零舍弃部分前面的数字加1)。
注意，此舍入模式始终不会减少计算值的大小。

- ROUND_DOWN
接近零的舍入模式。
在丢弃某部分之前始终不增加数字(从不对舍弃部分前面的数字加1，即截短)。
注意，此舍入模式始终不会增加计算值的大小。

- ROUND_CEILING
接近正无穷大的舍入模式。
如果 BigDecimal 为正，则舍入行为与 ROUND_UP 相同;
如果为负，则舍入行为与 ROUND_DOWN 相同。
注意，此舍入模式始终不会减少计算值。

- ROUND_FLOOR
接近负无穷大的舍入模式。
如果 BigDecimal 为正，则舍入行为与 ROUND_DOWN 相同;
如果为负，则舍入行为与 ROUND_UP 相同。
注意，此舍入模式始终不会增加计算值。

- ROUND_HALF_UP
向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。
如果舍弃部分 >= 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同。
注意，这是我们大多数人在小学时就学过的舍入模式(四舍五入)。

- ROUND_HALF_DOWN
向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为上舍入的舍入模式。
如果舍弃部分 > 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同(五舍六入)。

- ROUND_HALF_EVEN
向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。
如果舍弃部分左边的数字为奇数，则舍入行为与 ROUND_HALF_UP 相同;
如果为偶数，则舍入行为与 ROUND_HALF_DOWN 相同。
注意，在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。
此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况。
如果前一位为奇数，则入位，否则舍去。
以下例子为保留小数点1位，那么这种舍入方式下的结果。
1.15>1.2 1.25>1.2

- ROUND_UNNECESSARY
断言请求的操作具有精确的结果，因此不需要舍入。
如果对获得精确结果的操作指定此舍入模式，则抛出ArithmeticException。
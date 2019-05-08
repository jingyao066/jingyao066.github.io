---
title: SimpleDateFormat
tags: java
date: 2019-05-07 11:38:40
---

# DateFormat 介绍
DateFormat 的作用是 格式化并解析“日期/时间”。实际上，它是Date的格式化工具，它能帮助我们格式化Date，进而将Date转换成我们想要的String字符串供我们使用
不过DateFormat的格式化Date的功能有限，没有SimpleDateFormat强大；但DateFormat是SimpleDateFormat的父类。所以，我们先对DateFormat有个整体了解，然后再学习SimpleDateFormat。
DateFormat 的作用是格式化Date。它支持格式化风格包括 FULL、LONG、MEDIUM 和 SHORT 共4种：
- DateFormat.SHORT 
	完全为数字，如 12.13.52 或 3:30pm
- DateFormat.MEDIUM
	较长，如 Jan 12, 1952
- DateFormat.LONG 
	更长，如 January 12, 1952 或 3:30:32pm
- DateFormat.FULL 
	是完全指定，如 Tuesday、April 12、1952 AD 或 3:30:42pm PST。

# DateFormat 的定义如下
`public abstract class NumberFormat extends Format {}`

# DateFormat 的函数接口
```java
// 默认构造函数
DateFormat()

// 非构造函数
Object                   clone()
boolean                  equals(Object object)
abstract StringBuffer    format(Date date, StringBuffer buffer, FieldPosition field)
final StringBuffer       format(Object object, StringBuffer buffer, FieldPosition field)
final String             format(Date date)
static Locale[]          getAvailableLocales()
Calendar                 getCalendar()
final static DateFormat     getInstance()
final static DateFormat     getDateInstance()
final static DateFormat     getDateInstance(int style)
final static DateFormat     getDateInstance(int style, Locale locale)
final static DateFormat     getTimeInstance()
final static DateFormat     getTimeInstance(int style)
final static DateFormat     getTimeInstance(int style, Locale locale)
final static DateFormat     getDateTimeInstance()
final static DateFormat     getDateTimeInstance(int dateStyle, int timeStyle)
final static DateFormat     getDateTimeInstance(int dateStyle, int timeStyle, Locale locale)
NumberFormat     getNumberFormat()
TimeZone         getTimeZone()
int              hashCode()
boolean          isLenient()
Date             parse(String string)
abstract Date    parse(String string, ParsePosition position)
Object           parseObject(String string, ParsePosition position)
void             setCalendar(Calendar cal)
void             setLenient(boolean value)
void             setNumberFormat(NumberFormat format)
void             setTimeZone(TimeZone timezone)
```
注意：DateFormat是一个抽象类。

当我们通过DateFormat的 getInstance(), getDateInstance()和getDateTimeInstance() 获取DateFormat实例时；实际上是返回的SimpleDateFormat对象。 
下面的函数实际上都是返回的SimpleDateFormat对象。
```java
final static DateFormat getInstance()
final static DateFormat getTimeInstance()
final static DateFormat getTimeInstance(int style)
final static DateFormat getTimeInstance(int style, Locale locale)
final static DateFormat getDateInstance()
final static DateFormat getDateInstance(int style)
final static DateFormat getDateInstance(int style, Locale locale)
final static DateFormat getDateTimeInstance()
final static DateFormat getDateTimeInstance(int dateStyle, int timeStyle)
final static DateFormat getDateTimeInstance(int dateStyle, int timeStyle, Locale locale)
```
这些函数在SimpleDateFormat.java中的定义如下：
```java
public static final int FULL = 0;
public static final int LONG = 1;
public static final int MEDIUM = 2;
public static final int SHORT = 3;
public static final int DEFAULT = MEDIUM;

public final static DateFormat getInstance() {
    return getDateTimeInstance(SHORT, SHORT);
}

public final static DateFormat getTimeInstance()
{
    return get(DEFAULT, 0, 1, Locale.getDefault());
}

public final static DateFormat getTimeInstance(int style)
{
    return get(style, 0, 1, Locale.getDefault());
}

public final static DateFormat getTimeInstance(int style,
                                             Locale aLocale)
{
    return get(style, 0, 1, aLocale);
}

public final static DateFormat getDateInstance()
{
    return get(0, DEFAULT, 2, Locale.getDefault());
}

public final static DateFormat getDateInstance(int style)
{
    return get(0, style, 2, Locale.getDefault());
}

public final static DateFormat getDateInstance(int style,
                                             Locale aLocale)
{
    return get(0, style, 2, aLocale);
}

public final static DateFormat getDateTimeInstance()
{
    return get(DEFAULT, DEFAULT, 3, Locale.getDefault());
}

public final static DateFormat getDateTimeInstance(int dateStyle,
                                                   int timeStyle)
{
    return get(timeStyle, dateStyle, 3, Locale.getDefault());
}

public final static DateFormat
    getDateTimeInstance(int dateStyle, int timeStyle, Locale aLocale)
{
    return get(timeStyle, dateStyle, 3, aLocale);
}

/**
 * 获取DateFormat实例，实际上是返回SimpleDateFormat对象。
 * 
 * timeStyle -- 值可以为“FULL”或“LONG”或“MEDIUM”或“SHORT”
 * dateStyle -- 值可以为“FULL”或“LONG”或“MEDIUM”或“SHORT”
 * flags     -- 值可以为“1”或“2”或“3”。
 *       1 表示获取“时间样式”
 *       2 表示获取“日期样式”
 *       3 表示获取“时间和日期样式”
 * loc       -- locale对象，表示“区域”
 */
private static DateFormat get(int timeStyle, int dateStyle,
                              int flags, Locale loc) {
    if ((flags & 1) != 0) {
        if (timeStyle < 0 || timeStyle > 3) {
            throw new IllegalArgumentException("Illegal time style " + timeStyle);
        }
    } else {
        timeStyle = -1;
    }
    if ((flags & 2) != 0) {
        if (dateStyle < 0 || dateStyle > 3) {
            throw new IllegalArgumentException("Illegal date style " + dateStyle);
        }
    } else {
        dateStyle = -1;
    }
    try {
        // Check whether a provider can provide an implementation that's closer 
        // to the requested locale than what the Java runtime itself can provide.
        LocaleServiceProviderPool pool =
            LocaleServiceProviderPool.getPool(DateFormatProvider.class);
        if (pool.hasProviders()) {
            DateFormat providersInstance = pool.getLocalizedObject(
                                                DateFormatGetter.INSTANCE,
                                                loc, 
                                                timeStyle,
                                                dateStyle,
                                                flags);
            if (providersInstance != null) {
                return providersInstance;
            }
        }

        return new SimpleDateFormat(timeStyle, dateStyle, loc);
    } catch (MissingResourceException e) {
        return new SimpleDateFormat("M/d/yy h:mm a");
    }
}
```
通过上面的代码，我们能够进一步的认识到：DateFormat的作用是格式化Date；帮助我们将Date转换成我们需要的String字符串。DateFormat提供的功能非常有限，它只能支持FULL、LONG、MEDIUM 和 SHORT 这4种格式。而且，我们获取DateFormat实例时，实际上是返回的SimpleDateFormat对象。

# DateFormat 实例
下面，我们通过实例学习使用DateFormat的常用API。
源码如下(DateFormatTest.java): 
```java
import java.util.Date;
import java.util.Locale;
import java.text.DateFormat;
import java.text.FieldPosition;

/**
 * DateFormat 的API测试程序
 *
 * @author skywang
 * @email kuiwu-wang@163.com
 */
public class DateFormatTest {
    
    public static void main(String[] args) {

        // 只显示“时间”：调用getTimeInstance()函数
        testGetTimeInstance() ;

        // 只显示“日期”：调用getDateInstance()函数
        testGetDateInstance() ;

        // 显示“日期”+“时间”：调用getDateTimeInstance()函数
        testGetDateTimeInstance() ;
        
        // 测试format()函数
        testFormat();
    }

    /**
     * 测试DateFormat的getTimeInstance()函数
     * 它共有3种重载形式：
     * (01) getTimeInstance()
     * (02) getTimeInstance(int style)
     * (03) getTimeInstance(int style, Locale locale)
     *
     * @author skywang
     */
    private static void testGetTimeInstance() {
        Date date = new Date(); 

        //Locale locale = new Locale("fr", "FR");
        Locale locale = new Locale("zh", "CN"); 

        // 等价于 DateFormat.getTimeInstance( DateFormat.MEDIUM); 
        DateFormat short0  = DateFormat.getTimeInstance( ); 

        // 参数是：“时间的显示样式”
        DateFormat short1  = DateFormat.getTimeInstance( DateFormat.SHORT); 
        DateFormat medium1 = DateFormat.getTimeInstance( DateFormat.MEDIUM); 
        DateFormat long1   = DateFormat.getTimeInstance( DateFormat.LONG); 
        DateFormat full1   = DateFormat.getTimeInstance( DateFormat.FULL); 

        // 参数是：“时间的显示样式” 和 “地区”
        DateFormat short2  = DateFormat.getTimeInstance( DateFormat.SHORT, locale); 
        DateFormat medium2 = DateFormat.getTimeInstance( DateFormat.MEDIUM, locale); 
        DateFormat long2   = DateFormat.getTimeInstance( DateFormat.LONG, locale); 
        DateFormat full2   = DateFormat.getTimeInstance( DateFormat.FULL, locale); 

        System.out.println("\n----getTimeInstance ----\n"
                + "(1.0) Empty Param   : " + short0.format(date) +"\n"
                + "(2.1) One Param(s)  : " + short1.format(date) +"\n"
                + "(2.2) One Param(m)  : " + medium1.format(date) +"\n"
                + "(2.3) One Param(l)  : " + long1.format(date) +"\n"
                + "(2.4) One Param(f)  : " + full1.format(date) +"\n"
                + "(3.1) One Param(s,l): " + short2.format(date) +"\n"
                + "(3.2) One Param(m,l): " + medium2.format(date) +"\n"
                + "(3.3) One Param(l,l): " + long2.format(date) +"\n"
                + "(3.4) One Param(f,l): " + full2.format(date) +"\n"
                ); 
    }

    /**
     * 测试DateFormat的getDateTimeInstance()函数
     * 它共有3种重载形式：
     * (01) getDateInstance()
     * (02) getDateInstance(int style)
     * (03) getDateInstance(int style, Locale locale)
     */
    public static void testGetDateTimeInstance() {
        Date date = new Date(); 

        Locale locale = new Locale("zh", "CN"); 

        // 等价于 DateFormat.getDateTimeInstance( DateFormat.MEDIUM); 
        DateFormat short0  = DateFormat.getDateTimeInstance( ); 

        DateFormat short1  = DateFormat.getDateTimeInstance( DateFormat.SHORT, DateFormat.SHORT); 
        DateFormat medium1 = DateFormat.getDateTimeInstance( DateFormat.MEDIUM, DateFormat.MEDIUM); 
        DateFormat long1   = DateFormat.getDateTimeInstance( DateFormat.LONG, DateFormat.LONG); 
        DateFormat full1   = DateFormat.getDateTimeInstance( DateFormat.FULL, DateFormat.FULL); 

        DateFormat short2  = DateFormat.getDateTimeInstance( DateFormat.SHORT, DateFormat.SHORT, locale); 
        DateFormat medium2 = DateFormat.getDateTimeInstance( DateFormat.MEDIUM, DateFormat.MEDIUM, locale); 
        DateFormat long2   = DateFormat.getDateTimeInstance( DateFormat.LONG, DateFormat.LONG, locale); 
        DateFormat full2   = DateFormat.getDateTimeInstance( DateFormat.FULL, DateFormat.FULL, locale); 

        System.out.println("\n----getDateTimeInstance ----\n"
                + "(1.0) Empty Param   : " + short0.format(date) +"\n"
                + "(2.1) One Param(s)  : " + short1.format(date) +"\n"
                + "(2.2) One Param(m)  : " + medium1.format(date) +"\n"
                + "(2.3) One Param(l)  : " + long1.format(date) +"\n"
                + "(2.4) One Param(f)  : " + full1.format(date) +"\n"
                + "(3.1) One Param(s,l): " + short2.format(date) +"\n"
                + "(3.2) One Param(m,l): " + medium2.format(date) +"\n"
                + "(3.3) One Param(l,l): " + long2.format(date) +"\n"
                + "(3.4) One Param(f,l): " + full2.format(date) +"\n"
                ); 
    }

    /**
     * 测试DateFormat的getDateInstance()函数
     * 它共有3种重载形式：
     * (01) getDateTimeInstance()
     * (02) getDateTimeInstance(int dateStyle, int timeStyle)
     * (03) getDateTimeInstance(int dateStyle, int timeStyle, Locale locale)
     */
    public static void testGetDateInstance() {
        Date date = new Date(); 

        //Locale locale = new Locale("en", "US"); 
        Locale locale = new Locale("zh", "CN"); 

        // 等价于 DateFormat.getDateInstance( DateFormat.MEDIUM); 
        DateFormat short0  = DateFormat.getDateInstance( ); 

        DateFormat short1  = DateFormat.getDateInstance( DateFormat.SHORT); 
        DateFormat medium1 = DateFormat.getDateInstance( DateFormat.MEDIUM); 
        DateFormat long1   = DateFormat.getDateInstance( DateFormat.LONG); 
        DateFormat full1   = DateFormat.getDateInstance( DateFormat.FULL); 

        DateFormat short2  = DateFormat.getDateInstance( DateFormat.SHORT, locale); 
        DateFormat medium2 = DateFormat.getDateInstance( DateFormat.MEDIUM, locale); 
        DateFormat long2   = DateFormat.getDateInstance( DateFormat.LONG, locale); 
        DateFormat full2   = DateFormat.getDateInstance( DateFormat.FULL, locale); 

        System.out.println("\n----getDateInstance ----\n"
                + "(1.0) Empty Param   : " + short0.format(date) +"\n"
                + "(2.1) One Param(s)  : " + short1.format(date) +"\n"
                + "(2.2) One Param(m)  : " + medium1.format(date) +"\n"
                + "(2.3) One Param(l)  : " + long1.format(date) +"\n"
                + "(2.4) One Param(f)  : " + full1.format(date) +"\n"
                + "(3.1) One Param(s,l): " + short2.format(date) +"\n"
                + "(3.2) One Param(m,l): " + medium2.format(date) +"\n"
                + "(3.3) One Param(l,l): " + long2.format(date) +"\n"
                + "(3.4) One Param(f,l): " + full2.format(date) +"\n"
                ); 

    }


    /**
     * 测试DateFormat的format()函数
     */
    public static void testFormat() {
        Date date = new Date(); 
        StringBuffer sb = new StringBuffer();
        FieldPosition field = new FieldPosition(DateFormat.YEAR_FIELD);
        DateFormat format = DateFormat.getDateTimeInstance();

        sb =  format.format(date, sb, field);
        System.out.println("\ntestFormat"); 
        System.out.printf("sb=%s\n", sb);
    }
}
```

运行结果：
```
----getTimeInstance ----
(1.0) Empty Param   : 4:54:22 PM
(2.1) One Param(s)  : 4:54 PM
(2.2) One Param(m)  : 4:54:22 PM
(2.3) One Param(l)  : 4:54:22 PM CST
(2.4) One Param(f)  : 4:54:22 PM CST
(3.1) One Param(s,l): 下午4:54
(3.2) One Param(m,l): 16:54:22
(3.3) One Param(l,l): 下午04时54分22秒
(3.4) One Param(f,l): 下午04时54分22秒 CST


----getDateInstance ----
(1.0) Empty Param   : Jan 23, 2014
(2.1) One Param(s)  : 1/23/14
(2.2) One Param(m)  : Jan 23, 2014
(2.3) One Param(l)  : January 23, 2014
(2.4) One Param(f)  : Thursday, January 23, 2014
(3.1) One Param(s,l): 14-1-23
(3.2) One Param(m,l): 2014-1-23
(3.3) One Param(l,l): 2014年1月23日
(3.4) One Param(f,l): 2014年1月23日 星期四


----getDateTimeInstance ----
(1.0) Empty Param   : Jan 23, 2014 4:54:23 PM
(2.1) One Param(s)  : 1/23/14 4:54 PM
(2.2) One Param(m)  : Jan 23, 2014 4:54:23 PM
(2.3) One Param(l)  : January 23, 2014 4:54:23 PM CST
(2.4) One Param(f)  : Thursday, January 23, 2014 4:54:23 PM CST
(3.1) One Param(s,l): 14-1-23 下午4:54
(3.2) One Param(m,l): 2014-1-23 16:54:23
(3.3) One Param(l,l): 2014年1月23日 下午04时54分23秒
(3.4) One Param(f,l): 2014年1月23日 星期四 下午04时54分23秒 CST


testFormat
sb=Jan 23, 2014 4:54:23 PM
```

# SimpleDateFormat 介绍
SimpleDateFormat 是一个格式化Date 以及 解析日期字符串 的工具。它的最常用途是，能够按照指定的格式来对Date进行格式化，然后我们使用可以格式化Date后得到的字符串。
更严格的说，SimpleDateFormat 是一个以与语言环境有关的方式来格式化和解析日期的具体类。它允许进行格式化（日期 -> 文本）、解析（文本 -> 日期）和规范化。

## SimpleDateFormat的构造函数：
```java
// 构造函数
SimpleDateFormat()
SimpleDateFormat(String pattern)
SimpleDateFormat(String template, DateFormatSymbols value)
SimpleDateFormat(String template, Locale locale)

// 非构造函数
void                             applyLocalizedPattern(String template)
void                             applyPattern(String template)
Object                           clone()
boolean                          equals(Object object)
StringBuffer                     format(Date date, StringBuffer buffer, FieldPosition fieldPos)
AttributedCharacterIterator      formatToCharacterIterator(Object object)
Date                             get2DigitYearStart()
DateFormatSymbols                getDateFormatSymbols()
int                              hashCode()
Date                             parse(String string, ParsePosition position)
void                             set2DigitYearStart(Date date)
void                             setDateFormatSymbols(DateFormatSymbols value)
String                           toLocalizedPattern()
String                           toPattern()
```

## SimpleDateFormat 简单示范：
```java
// 新建date对象，时间是2013-09-19
Date date = new Date(113,8,19); 
// 新建“SimpleDateFormat对象”，并设置 sdf 的“格式化模式”
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
// 用 sdf 格式化 date，并返回字符串。
String str = sdf.format(date); 
```

# SimpleDateFormat 相关格式说明
日期和时间模式
日期和时间格式由日期和时间模式 字符串指定。在日期和时间模式字符串中，未加引号的字母 'A' 到 'Z' 和 'a' 到 'z' 被解释为模式字母，用来表示日期或时间字符串元素。文本可以使用单引号 (') 引起来，以免进行解释。"''" 表示单引号。所有其他字符均不解释；只是在格式化时将它们简单复制到输出字符串，或者在解析时与输入字符串进行匹配。
定义了以下模式字母（所有其他字符 'A' 到 'Z' 和 'a' 到 'z' 都被保留）：

字母 | 日期或时间元素 | 表示 | 示例
-- | -- | -- | --
G |	Era 标志符 | Text |	AD
y |	年	|Year | 1996;96
M |	年中的月份 |	Month |	July; Jul; 07
w |	年中的周数 |	Number | 27
W |	月份中的周数 |	Number | 2
D |	年中的天数 |	Number | 189
d |	月份中的天数 |	Number | 10
F |	月份中的星期 |	Number | 2
E |	星期中的天数 |	Text | Tuesday; Tue
a |	Am/pm 标记 |	Text |	PM
H |	一天中的小时数（0-23） |	Number | 0
k |	一天中的小时数（1-24） |	Number | 24
K |	am/pm 中的小时数（0-11） |	Number | 0
h |	am/pm 中的小时数（1-12） |	Number | 12
m |	小时中的分钟数 |	Number | 30
s |	分钟中的秒数 |	Number | 55
S |	毫秒数 | Number | 978
z |	时区 |	General time zone | Pacific Standard Time; PST; GMT-08:00
Z |	时区 |	RFC 822 time zone | -0800

模式字母通常是重复的，其数量确定其精确表示：

Text: 对于格式化来说，如果模式字母的数量大于等于 4，则使用完全形式；否则，在可用的情况下使用短形式或缩写形式。对于解析来说，两种形式都是可接受的，与模式字母的数量无关。
Number: 对于格式化来说，模式字母的数量是最小的数位，如果数位不够，则用 0 填充以达到此数量。对于解析来说，模式字母的数量被忽略，除非必须分开两个相邻字段。
Year: 如果格式器的 Calendar 是格里高利历，则应用以下规则。
Month: 如果模式字母的数量为 3 或大于 3，则将月份解释为 text；否则解释为 number。
对于格式化来说，如果模式字母的数量为 2，则年份截取为 2 位数,否则将年份解释为 number。
对于解析来说，如果模式字母的数量大于 2，则年份照字面意义进行解释，而不管数位是多少。因此使用模式 "MM/dd/yyyy"，将 "01/11/12" 解析为公元 12 年 1 月 11 日。
在解析缩写年份模式（"y" 或 "yy"）时，SimpleDateFormat 必须相对于某个世纪来解释缩写的年份。这通过将日期调整为 SimpleDateFormat 实例创建之前的 80 年和之后 20 年范围内来完成。例如，在 "MM/dd/yy" 模式下，如果 SimpleDateFormat 实例是在 1997 年 1 月 1 日创建的，则字符串 "01/11/12" 将被解释为 2012 年 1 月 11 日，而字符串 "05/04/64" 将被解释为 1964 年 5 月 4 日。在解析时，只有恰好由两位数字组成的字符串（如 Character#isDigit(char) 所定义的）被解析为默认的世纪。其他任何数字字符串将照字面意义进行解释，例如单数字字符串，3 个或更多数字组成的字符串，或者不都是数字的两位数字字符串（例如"-1"）。因此，在相同的模式下， "01/02/3" 或 "01/02/003" 解释为公元 3 年 1 月 2 日。同样，"01/02/-3" 解析为公元前 4 年 1 月 2 日。
否则，则应用日历系统特定的形式。对于格式化和解析，如果模式字母的数量为 4 或大于 4，则使用日历特定的 long form。否则，则使用日历特定的 short or abbreviated form。
SimpleDateFormat 还支持本地化日期和时间模式 字符串。在这些字符串中，以上所述的模式字母可以用其他与语言环境有关的模式字母来替换。SimpleDateFormat 不处理除模式字母之外的文本本地化；而由类的客户端来处理。
示例

以下示例显示了如何在美国语言环境中解释日期和时间模式。给定的日期和时间为美国太平洋时区的本地时间 2001-07-04 12:08:56。

日期和时间模式 | 结果
 -- | --
"yyyy.MM.dd G 'at' HH:mm:ss z" | 2001.07.04 AD at 12:08:56 PDT
"EEE, MMM d, ''yy" | Wed, Jul 4, '01
"h:mm a" | 12:08 PM
"hh 'o''clock' a, zzzz" | 12 o'clock PM, Pacific Daylight Time
"K:mm a, z" | 0:08 PM, PDT
"yyyyy.MMMMM.dd GGG hh:mm aaa" | 02001.July.04 AD 12:08 PM
"EEE, d MMM yyyy HH:mm:ss Z" | Wed, 4 Jul 2001 12:08:56 -0700
"yyMMddHHmmssZ" | 010704120856-0700
"yyyy-MM-dd'T'HH:mm:ss.SSSZ" |	2001-07-04T12:08:56.235-0700

日期格式是不同步的。建议为每个线程创建独立的格式实例。如果多个线程同时访问一个格式，则它必须是外部同步的。

# SimpleDateFormat 示例 
下面，我们通过实例学习如何使用SimpleDateFormat。
源码如下(SimpleDateFormatTest.java)：
```java
import java.util.Date;
import java.util.Locale;
import java.util.Calendar;
import java.text.DateFormat;
import java.text.SimpleDateFormat;

/**
 * SimpleDateFormat 的API测试程序
 *
 * @author skywang
 * @email kuiwu-wang@163.com
 */
public class SimpleDateFormatTest {
    
    public static void main(String[] args) {

        // 通过SimpleDateFormat 获取日期/时间：有多种格式
        testSimpleDateFormats() ;

        // 通过DateFormat 获取日期/时间
        superTest() ;
    }

    /**
     * 通过SimpleDateFormat 获取日期/时间。有多种格式可以选择
     */
    private static void testSimpleDateFormats() {
        String[] formats = new String[] {
            "HH:mm",                                // 14:22
            "h:mm a",                               // 2:22 下午
            "HH:mm z",                              // 14:22 CST
            "HH:mm Z",                              // 14:22 +0800
            "HH:mm zzzz",                           // 14:22 中国标准时间
            "HH:mm:ss",                             // 14:22:30
            "yyyy-MM-dd",                           // 2013-09-19
            "yyyy-MM-dd HH:mm",                     // 2013-09-19 14:22
            "yyyy-MM-dd HH:mm:ss",                  // 2013-09-19 14:22:30
            "yyyy-MM-dd HH:mm:ss zzzz",             // 2013-09-19 14:22:30 中国标准时间
            "EEEE yyyy-MM-dd HH:mm:ss zzzz",        // 星期四 2013-09-19 14:22:30 中国标准时间
            "yyyy-MM-dd HH:mm:ss.SSSZ",             // 2013-09-19 14:22:30.000+0800
            "yyyy-MM-dd'T'HH:mm:ss.SSSZ",           // 2013-09-19T14:22:30.000+0800
            "yyyy.MM.dd G 'at' HH:mm:ss z",         // 2013.09.19 公元 at 14:22:30 CST
            "K:mm a",                               // 2:22 下午, CST
            "EEE, MMM d, ''yy",                     // 星期四, 九月 19, '13
            "hh 'o''clock' a, zzzz",                // 02 o'clock 下午, 中国标准时间
            "yyyyy.MMMMM.dd GGG hh:mm aaa",         // 02013.九月.19 公元 02:22 下午
            "EEE, d MMM yyyy HH:mm:ss Z",           // 星期四, 19 九月 2013 14:22:30 +0800
            "yyMMddHHmmssZ",                        // 130919142230+0800
            "yyyy-MM-dd'T'HH:mm:ss.SSSZ",           // 2013-09-19T14:22:30.000+0800
            "EEEE 'DATE('yyyy-MM-dd')' 'TIME('HH:mm:ss')' zzzz",        // 星期四 2013-09-19 14:22:30 中国标准时间
        };

        //Date date = (new Date(0));                    // date为1970-01-01 07:00:00
        //Date date = Calendar.getInstance().getTime(); // date为当前时间
        Date date = new Date(113, 8, 19, 14, 22, 30);   // date为2013-09-19 14:22:30
        for (String format : formats) {
            SimpleDateFormat sdf = new SimpleDateFormat(format, Locale.SIMPLIFIED_CHINESE);
            //SimpleDateFormat sdf = new SimpleDateFormat(format);
            System.out.format("%30s    %s\n", format, sdf.format(date));
         }
    }

    /**
     * 通过DateFormat 获取日期/时间
     */
    private static void superTest() {
        // 新建date对象，时间是2013-09-19 14:22:30
        // (01) 年=“‘目标年’ - 1900”，
        // (02) 月。 0是一月，1是二月，依次类推。
        // (03) 日。 1-31之间的数
        Date mDate = new Date(113, 8, 19, 14, 22, 30);
        Locale locale = new Locale("zh", "CN"); 

        // 14:22:30
        String time = DateFormat.getTimeInstance( DateFormat.MEDIUM, Locale.SIMPLIFIED_CHINESE).format(mDate);
        // 2013-09-19
        String date = DateFormat.getDateInstance( DateFormat.MEDIUM, Locale.SIMPLIFIED_CHINESE).format(mDate);
        // 2013-09-19 14:22:30
        String datetime = DateFormat.getDateTimeInstance( DateFormat.MEDIUM, DateFormat.MEDIUM, Locale.SIMPLIFIED_CHINESE).format(mDate);

        System.out.printf("\ntime=%s\ndate=%s\ndatetime=%s\n",time,date,datetime); 
    }
}
```

运行结果：
```
HH:mm    14:22
                        h:mm a    2:22 下午
                       HH:mm z    14:22 CST
                       HH:mm Z    14:22 +0800
                    HH:mm zzzz    14:22 中国标准时间
                      HH:mm:ss    14:22:30
                    yyyy-MM-dd    2013-09-19
              yyyy-MM-dd HH:mm    2013-09-19 14:22
           yyyy-MM-dd HH:mm:ss    2013-09-19 14:22:30
      yyyy-MM-dd HH:mm:ss zzzz    2013-09-19 14:22:30 中国标准时间
 EEEE yyyy-MM-dd HH:mm:ss zzzz    星期四 2013-09-19 14:22:30 中国标准时间
      yyyy-MM-dd HH:mm:ss.SSSZ    2013-09-19 14:22:30.000+0800
    yyyy-MM-dd'T'HH:mm:ss.SSSZ    2013-09-19T14:22:30.000+0800
  yyyy.MM.dd G 'at' HH:mm:ss z    2013.09.19 公元 at 14:22:30 CST
                        K:mm a    2:22 下午
              EEE, MMM d, ''yy    星期四, 九月 19, '13
         hh 'o''clock' a, zzzz    02 o'clock 下午, 中国标准时间
  yyyyy.MMMMM.dd GGG hh:mm aaa    02013.九月.19 公元 02:22 下午
    EEE, d MMM yyyy HH:mm:ss Z    星期四, 19 九月 2013 14:22:30 +0800
                 yyMMddHHmmssZ    130919142230+0800
    yyyy-MM-dd'T'HH:mm:ss.SSSZ    2013-09-19T14:22:30.000+0800
EEEE 'DATE('yyyy-MM-dd')' 'TIME('HH:mm:ss')' zzzz    星期四 DATE(2013-09-19) TIME(14:22:30) 中国标准时间

time=14:22:30
date=2013-9-19
datetime=2013-9-19 14:22:30
```

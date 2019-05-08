---
title: locale and timezone
tags: java
date: 2019-05-07 14:13:54
---
# Locale 介绍
Locale 表示地区。每一个Locale对象都代表了一个特定的地理、政治和文化地区。
在操作 Date, Calendar等表示日期/时间的对象时，经常会用到；因为不同的区域，时间表示方式都不同。

下面说说Locale对象的3种常用创建方式。
## 获取默认的Locale
使用方法：
`Locale locale = Locale.getDefault()`
 
## 直接使用Locale的静态对象
Locale.java中提供了以下静态对象
```java
public static final Locale CANADA
public static final Locale CANADA_FRENCH
public static final Locale CHINA
public static final Locale CHINESE
public static final Locale ENGLISH
public static final Locale FRANCE
public static final Locale FRENCH
public static final Locale GERMAN
public static final Locale GERMANY
public static final Locale ITALIAN
public static final Locale ITALY
public static final Locale JAPAN
public static final Locale JAPANESE
public static final Locale KOREA
public static final Locale KOREAN
public static final Locale PRC
public static final Locale ROOT
public static final Locale SIMPLIFIED_CHINESE
public static final Locale TAIWAN
public static final Locale TRADITIONAL_CHINESE
public static final Locale UK
public static final Locale US
```
使用方法：下面的Locale对象是对应 “中国(大陆)”的
`Locale locale = Locale.SIMPLIFIED_CHINESE`

## 通过Locale的构造函数创建Locale对象
Locale的构造函数共有3个。如下：
```java
Locale(String language)
Locale(String language, String country)
Locale(String language, String country, String variant)
```

使用方法：
`Locale local = new Locale("zh", "CN");`

Locale类支持非常多的国家和地区。我们可以通过以下方法，查看Locale支持的全部区域：
```java
Locale[] ls = Locale.getAvailableLocales();

for (Locale locale:ls) {
    System.out.println("locale :"+locale);
}
```
输入结果如下：
All Locales: ja_JP, es_PE, en, ja_JP_JP, es_PA, sr_BA, mk, es_GT, ar_AE, no_NO, sq_AL, bg, ar_IQ, ar_YE, hu, pt_PT, el_CY, ar_QA, mk_MK, sv, de_CH, en_US, fi_FI, is, cs, en_MT, sl_SI, sk_SK, it, tr_TR, zh, th, ar_SA, no, en_GB, sr_CS, lt, ro, en_NZ, no_NO_NY, lt_LT, es_NI, nl, ga_IE, fr_BE, es_ES, ar_LB, ko, fr_CA, et_EE, ar_KW, sr_RS, es_US, es_MX, ar_SD, in_ID, ru, lv, es_UY, lv_LV, iw, pt_BR, ar_SY, hr, et, es_DO, fr_CH, hi_IN, es_VE, ar_BH, en_PH, ar_TN, fi, de_AT, es, nl_NL, es_EC, zh_TW, ar_JO, be, is_IS, es_CO, es_CR, es_CL, ar_EG, en_ZA, th_TH, el_GR, it_IT, ca, hu_HU, fr, en_IE, uk_UA, pl_PL, fr_LU, nl_BE, en_IN, ca_ES, ar_MA, es_BO, en_AU, sr, zh_SG, pt, uk, es_SV, ru_RU, ko_KR, vi, ar_DZ, vi_VN, sr_ME, sq, ar_LY, ar, zh_CN, be_BY, zh_HK, ja, iw_IL, bg_BG, in, mt_MT, es_PY, sl, fr_FR, cs_CZ, it_CH, ro_RO, es_PR, en_CA, de_DE, ga, de_LU, de, es_AR, sk, ms_MY, hr_HR, en_SG, da, mt, pl, ar_OM, tr, th_TH_TH, el, ms, sv_SE, da_DK, es_HN

下面选择其中的两个进行说明，如何利用它们来创建Locale对象：
例如，第一个输出是“ja_JP”。

其中，ja代表“语言”，这里指日语；“JP”代表国家，这里指日本。
我们可以通过如下方法，创建“语言是日语，国家是日本的Locale对象”。
`Locale locale = new Locale("ja", "JP");`

例如，第三个输出是“en”。

其中，en代表“语言”，这里指英语。
我们可以通过如下方法，创建“语言是英文的Locale对象”。
`Locale locale = new Locale("en");`

# Locale 函数列表
```java
// Locale的构造函数
Locale(String language)
Locale(String language, String country)
Locale(String language, String country, String variant)

Object                         clone()
boolean                     equals(Object object)
static Locale[]             getAvailableLocales()
String                         getCountry()
static Locale                 getDefault()
String                         getDisplayCountry(Locale locale)
final String                 getDisplayCountry()
final String                 getDisplayLanguage()
String                         getDisplayLanguage(Locale locale)
String                         getDisplayName(Locale locale)
final String                 getDisplayName()
final String                 getDisplayVariant()
String                         getDisplayVariant(Locale locale)
String                         getISO3Country()
String                         getISO3Language()
static String[]             getISOCountries()
static String[]             getISOLanguages()
String                         getLanguage()
String                         getVariant()
synchronized int             hashCode()
synchronized static void     setDefault(Locale locale)
final String                 toString()
```

# Locale示例
下面通过示例演示在Date中使用Locale的。
参考代码如下(LocaleTest.java)：
```java
import java.util.Locale;
import java.util.Date;
import java.util.Calendar;
import java.text.SimpleDateFormat;
import java.text.DateFormat;

/**
 * Locale 的测试程序
 *
 * @author skywang
 * @email kuiwu-wang@163.com
 */
public class LocaleTest {

    public static void main(String[] args) {
        // 2种不同的Locale的创建方法
        testDiffDateLocales();

        // 显示所有的Locales
        testAllLocales();
    }


    /**
     *  2种不同的Locale的创建方法
     */
    private static void testDiffDateLocales() {
        // date为2013-09-19 14:22:30
        Date date = new Date(113, 8, 19, 14, 22, 30);

        // 创建“简体中文”的Locale
        Locale localeCN = Locale.SIMPLIFIED_CHINESE;
        // 创建“英文/美国”的Locale
        Locale localeUS = new Locale("en", "US");

        // 获取“简体中文”对应的date字符串
        String cn = DateFormat.getDateInstance(DateFormat.MEDIUM, localeCN).format(date);
        // 获取“英文/美国”对应的date字符串
        String us = DateFormat.getDateInstance(DateFormat.MEDIUM, localeUS).format(date);

        System.out.printf("cn=%s\nus=%s\n", cn, us);
    }

    /**
     *  显示所有的Locales
     */
    private static void testAllLocales() {
        Locale[] ls = Locale.getAvailableLocales();

        System.out.print("All Locales: ");
        for (Locale locale:ls) {
            System.out.printf(locale+", ");
        }
        System.out.println();
    }
}
```

# TimeZone简介
TimeZone 表示时区偏移量，也可以计算夏令时。
在操作 Date, Calendar等表示日期/时间的对象时，经常会用到TimeZone；因为不同的时区，时间不同。

下面说说TimeZone对象的 2种常用创建方式。

- 获取默认的TimeZone对象
使用方法：
`TimeZone tz = TimeZone.getDefault()`

- 使用 getTimeZone(String id) 方法获取TimeZone对象
使用方法：
```java
// 获取 “GMT+08:00”对应的时区
TimeZone china = TimeZone.getTimeZone("GMT+:08:00");
// 获取 “中国/重庆”对应的时区
TimeZone chongqing = TimeZone.getTimeZone("Asia/Chongqing");
```

关于 getTimeZone(String id) 这种方式支持的全部id参数的取值，可以通过以下方式查找：
```java
String[] ids = TimeZone.getAvailableIDs();
for (String id:ids) 
    System.out.printf(id+", ");
```
输出结果略...

例如，创建上面第2个打印值“Etc/GMT+11”对应的TimeZone。方法如下：
`TimeZone tz = TimeZone.getTimeZone("Etc/GMT+11");`

# TimeZone的函数接口
```java
// 构造函数
TimeZone()

Object                           clone()
synchronized static String[]     getAvailableIDs()
synchronized static String[]     getAvailableIDs(int offsetMillis)
int                              getDSTSavings()
synchronized static TimeZone     getDefault()
final String                     getDisplayName(Locale locale)
String                           getDisplayName(boolean daylightTime, int style, Locale locale)
final String                     getDisplayName()
final String                     getDisplayName(boolean daylightTime, int style)
String                           getID()
abstract int                     getOffset(int era, int year, int month, int day, int dayOfWeek, int timeOfDayMillis)
int                              getOffset(long time)
abstract int                     getRawOffset()
synchronized static TimeZone     getTimeZone(String id)
boolean                          hasSameRules(TimeZone timeZone)
abstract boolean                 inDaylightTime(Date time)
synchronized static void         setDefault(TimeZone timeZone)
void                             setID(String id)
abstract void                    setRawOffset(int offsetMillis)
abstract boolean                 useDaylightTime()
```

# TimeZone示例
下面通过示例演示在Date中使用TimeZone。
参考代码如下(TimeZoneTest.java)：
```java
import java.text.DateFormat;
import java.util.Date;
import java.util.TimeZone;

/**
 * TimeZone的测试程序
 *
 * @author skywang
 * @email kuiwu-wang@163.com
 */
public class TimeZoneTest {

    public static void main(String[] args) {

        // 测试创建TimeZone对象的3种方法
        showUsageOfTimeZones() ;

        // 测试TimeZone的其它API
        testOtherAPIs() ;

        // 打印getTimeZone(String id)支持的所有id
        //printAllTimeZones() ;
    }


    /**
     * 测试创建TimeZone对象的3种方法
     */
    public static void showUsageOfTimeZones() {
        TimeZone tz;

        // (01) 默认时区
        tz = TimeZone.getDefault();
        printDateIn(tz) ;

        // (02) 设置时区为"GMT+08:00"
        tz = TimeZone.getTimeZone("GMT+08:00");
        printDateIn(tz) ;

        // (03) 设置时区为""
        tz = TimeZone.getTimeZone("Asia/Chongqing");
        printDateIn(tz) ;
    }

    /**
     * 打印 tz对应的日期/时间
     */
    private static void printDateIn(TimeZone tz) {
        // date为2013-09-19 14:22:30
        Date date = new Date(113, 8, 19, 14, 22, 30);
        // 获取默认的DateFormat，用于格式化Date
        DateFormat df = DateFormat.getInstance();
        // 设置时区为tz
        df.setTimeZone(tz);
        // 获取格式化后的字符串
        String str = df.format(date);

        System.out.println(tz.getID()+" :"+str);
    }

    /**
     * 测试TimeZone的其它API
     */
    public static void testOtherAPIs() {
        // 默认时区
        TimeZone tz = TimeZone.getDefault();

        // 获取“id”
        String id = tz.getID();

        // 获取“显示名称”
        String name = tz.getDisplayName();

        // 获取“时间偏移”。相对于“本初子午线”的偏移，单位是ms。
        int offset = tz.getRawOffset();
        // 获取“时间偏移” 对应的小时
        int gmt = offset/(3600*1000);

        System.out.printf("id=%s, name=%s, offset=%s(ms), gmt=%s\n",
                id, name, offset, gmt);
    }

    /**
     * 打印getTimeZone(String id)支持的所有id
     */
    public static void printAllTimeZones() {

        String[] ids = TimeZone.getAvailableIDs();
        for (String id:ids) {
            //int offset = TimeZone.getTimeZone(avaIds[i]).getRawOffset();
            //System.out.println(i+"  "+avaIds[i]+" "+offset / (3600 * 1000) + "\t");
            System.out.printf(id+", ");
        }
        System.out.println();
    }
}
```


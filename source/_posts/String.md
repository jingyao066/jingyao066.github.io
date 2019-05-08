---
title: String
tags: java
date: 2019-05-07 15:30:07
---

# String简介
String 是java中的字符串，它继承于CharSequence。
String类所包含的API接口非常多。为了便于今后的使用，我对String的API进行了分类，并都给出的演示程序。

String 和 CharSequence 关系：
String 继承于CharSequence，也就是说String也是CharSequence类型。
CharSequence是一个接口，它只包括length(), charAt(int index), subSequence(int start, int end)这几个API接口。除了String实现了CharSequence之外，StringBuffer和StringBuilder也实现了CharSequence接口。
需要说明的是，CharSequence就是字符序列，String、StringBuilder和StringBuffer本质上都是通过字符数组实现的。

 

StringBuilder 和 StringBuffer 的区别：
StringBuilder 和 StringBuffer都是可变的字符序列。它们都继承于AbstractStringBuilder，实现了CharSequence接口。
但是，StringBuilder是非线程安全的，而StringBuffer是线程安全的。

它们之间的关系图如下： 
![](String/1.jpg)

# String函数列表
```java
public    String()
public    String(String original)
public    String(char[] value)
public    String(char[] value, int offset, int count)
public    String(byte[] bytes)
public    String(byte[] bytes, int offset, int length)
public    String(byte[] ascii, int hibyte)
public    String(byte[] ascii, int hibyte, int offset, int count)
public    String(byte[] bytes, String charsetName)
public    String(byte[] bytes, int offset, int length, String charsetName)
public    String(byte[] bytes, Charset charset)
public    String(byte[] bytes, int offset, int length, Charset charset)
public    String(int[] codePoints, int offset, int count)
public    String(StringBuffer buffer)
public    String(StringBuilder builder)

public char    charAt(int index)
public int    codePointAt(int index)
public int    codePointBefore(int index)
public int    codePointCount(int beginIndex, int endIndex)
public int    compareTo(String anotherString)
public int    compareToIgnoreCase(String str)
public String    concat(String str)
public boolean    contains(CharSequence s)
public boolean    contentEquals(StringBuffer sb)
public boolean    contentEquals(CharSequence cs)
public static String    copyValueOf(char[] data, int offset, int count)
public static String    copyValueOf(char[] data)
public boolean    endsWith(String suffix)
public boolean    equals(Object anObject)
public boolean    equalsIgnoreCase(String anotherString)
public static String    format(String format, Object[] args)
public static String    format(Locale l, String format, Object[] args)
public int    hashCode()
public int    indexOf(int ch)
public int    indexOf(int ch, int fromIndex)
public int    indexOf(String str)
public int    indexOf(String str, int fromIndex)
public String    intern()
public int    lastIndexOf(int ch)
public int    lastIndexOf(int ch, int fromIndex)
public int    lastIndexOf(String str)
public int    lastIndexOf(String str, int fromIndex)
public int    length()
public boolean    matches(String regex)
public int    offsetByCodePoints(int index, int codePointOffset)
public boolean    regionMatches(int toffset, String other, int ooffset, int len)
public boolean    regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len)
public String    replace(char oldChar, char newChar)
public String    replace(CharSequence target, CharSequence replacement)
public String    replaceAll(String regex, String replacement)
public String    replaceFirst(String regex, String replacement)
public String[]    split(String regex, int limit)
public String[]    split(String regex)
public boolean    startsWith(String prefix, int toffset)
public boolean    startsWith(String prefix)
public CharSequence    subSequence(int beginIndex, int endIndex)
public String    substring(int beginIndex)
public String    substring(int beginIndex, int endIndex)
public char[]    toCharArray()
public String    toLowerCase(Locale locale)
public String    toLowerCase()
public String    toString()
public String    toUpperCase(Locale locale)
public String    toUpperCase()
public String    trim()
public static String    valueOf(Object obj)
public static String    valueOf(char[] data)
public static String    valueOf(char[] data, int offset, int count)
public static String    valueOf(boolean b)
public static String    valueOf(char c)
public static String    valueOf(int i)
public static String    valueOf(long l)
public static String    valueOf(float f)
public static String    valueOf(double d)
public void    getBytes(int srcBegin, int srcEnd, byte[] dst, int dstBegin)
public byte[]    getBytes(String charsetName)
public byte[]    getBytes(Charset charset)
public byte[]    getBytes()
public void    getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
public boolean    isEmpty()
```

此处省略CharSequence和String源码。

# CharSequence示例
下面通过示例，演示CharSequence的使用方法！
源码如下(CharSequenceTest.java):
```java
/**
 * CharSequence 演示程序
 *
 * @author skywang
 */
import java.nio.charset.Charset;
import java.io.UnsupportedEncodingException;

public class CharSequenceTest {

    public static void main(String[] args) {
        testCharSequence();
    }

    /**
     * CharSequence 测试程序
     */
    private static void testCharSequence() {
        System.out.println("-------------------------------- testCharSequence -----------------------------");

        // 1. CharSequence的子类String
        String str = "abcdefghijklmnopqrstuvwxyz";
        System.out.println("1. String");
        System.out.printf("   %-30s=%d\n", "str.length()", str.length());
        System.out.printf("   %-30s=%c\n", "str.charAt(5)", str.charAt(5));
        String substr = (String)str.subSequence(0,5);
        System.out.printf("   %-30s=%s\n", "str.subSequence(0,5)", substr.toString());

        // 2. CharSequence的子类StringBuilder
        StringBuilder strbuilder = new StringBuilder("abcdefghijklmnopqrstuvwxyz");
        System.out.println("2. StringBuilder");
        System.out.printf("   %-30s=%d\n", "strbuilder.length()", strbuilder.length());
        System.out.printf("   %-30s=%c\n", "strbuilder.charAt(5)", strbuilder.charAt(5));
        // 注意：StringBuilder的subSequence()返回的是，实际上是一个String对象！
        String substrbuilder = (String)strbuilder.subSequence(0,5);
        System.out.printf("   %-30s=%s\n", "strbuilder.subSequence(0,5)", substrbuilder.toString());

        // 3. CharSequence的子类StringBuffer
        StringBuffer strbuffer = new StringBuffer("abcdefghijklmnopqrstuvwxyz");
        System.out.println("3. StringBuffer");
        System.out.printf("   %-30s=%d\n", "strbuffer.length()", strbuffer.length());
        System.out.printf("   %-30s=%c\n", "strbuffer.charAt(5)", strbuffer.charAt(5));
        // 注意：StringBuffer的subSequence()返回的是，实际上是一个String对象！
        String substrbuffer = (String)strbuffer.subSequence(0,5);
        System.out.printf("   %-30s=%s\n", "strbuffer.subSequence(0,5)", substrbuffer.toString());

        System.out.println();
    }
}
```

运行结果：
```
-------------------------------- testCharSequence -----------------------------
1. String
   str.length()                  =26
   str.charAt(5)                 =f
   str.subSequence(0,5)          =abcde
2. StringBuilder
   strbuilder.length()           =26
   strbuilder.charAt(5)          =f
   strbuilder.subSequence(0,5)   =abcde
3. StringBuffer
   strbuffer.length()            =26
   strbuffer.charAt(5)           =f
   strbuffer.subSequence(0,5)    =abcde
```

# String 构造函数
下面通过示例，演示String的各种构造函数的使用方法！
源码如下(StringContructorTest.java):
```java
/**
 * String 构造函数演示程序
 *
 * @author skywang
 */
import java.nio.charset.Charset;
import java.io.UnsupportedEncodingException;

public class StringContructorTest {

    public static void main(String[] args) {
        testStringConstructors() ;
    }

    /**
     * String 构造函数测试程序
     */
    private static void testStringConstructors() {
        try {
            System.out.println("-------------------------------- testStringConstructors -----------------------");

            String str01 = new String();
            String str02 = new String("String02");
            String str03 = new String(new char[]{'s','t','r','0','3'});
            String str04 = new String(new char[]{'s','t','r','0','4'}, 1, 3);          // 1表示起始位置，3表示个数
            String str05 = new String(new byte[]{0x61, 0x62, 0x63, 0x64, 0x65});       // 0x61在ASC表中，对应字符"a"; 1表示起始位置，3表示长度
            String str06 = new String(new byte[]{0x61, 0x62, 0x63, 0x64, 0x65}, 1, 3); // 0x61在ASC表中，对应字符"a"; 1表示起始位置，3表示长度
            String str07 = new String(new byte[]{0x61, 0x62, 0x63, 0x64, 0x65}, 0);       // 0x61在ASC表中，对应字符"a";0，表示“高字节”
            String str08 = new String(new byte[]{0x61, 0x62, 0x63, 0x64, 0x65}, 0, 1, 3); // 0x61在ASC表中，对应字符"a"; 0，表示“高字节”；1表示起始位置，3表示长度
            String str09 = new String(new byte[]{(byte)0xe5, (byte)0xad, (byte)0x97, /* 字-对应的utf-8编码 */ 
                                                 (byte)0xe7, (byte)0xac, (byte)0xa6, /* 符-对应的utf-8编码 */ 
                                                 (byte)0xe7, (byte)0xbc, (byte)0x96, /* 编-对应的utf-8编码 */ 
                                                 (byte)0xe7, (byte)0xa0, (byte)0x81, /* 码-对应的utf-8编码 */ }, 
                                      0, 12, "utf-8");  // 0表示起始位置，12表示长度。
            String str10 = new String(new byte[]{(byte)0x5b, (byte)0x57, /* 字-对应的utf-16编码 */ 
                                                 (byte)0x7b, (byte)0x26, /* 符-对应的utf-16编码 */ 
                                                 (byte)0x7f, (byte)0x16, /* 编-对应的utf-16编码 */ 
                                                 (byte)0x78, (byte)0x01, /* 码-对应的utf-16编码 */ }, 
                                      0, 8, "utf-16");  // 0表示起始位置，8表示长度。
            String str11 = new String(new byte[]{(byte)0xd7, (byte)0xd6, /* 字-对应的gb2312编码  */ 
                                                 (byte)0xb7, (byte)0xfb, /* 符-对应的gb2312编码 */ 
                                                 (byte)0xb1, (byte)0xe0, /* 编-对应的gb2312编码 */ 
                                                 (byte)0xc2, (byte)0xeb, /* 码-对应的gb2312编码 */ }, 
                                      Charset.forName("gb2312")); 
            String str12 = new String(new byte[]{(byte)0xd7, (byte)0xd6, /* 字-对应的gbk编码 */ 
                                                 (byte)0xb7, (byte)0xfb, /* 符-对应的gbk编码 */ 
                                                 (byte)0xb1, (byte)0xe0, /* 编-对应的gbk编码 */ 
                                                 (byte)0xc2, (byte)0xeb, /* 码-对应的gbk编码 */ }, 
                                      0, 8, Charset.forName("gbk")); 
            String str13 = new String(new int[] {0x5b57, 0x7b26, 0x7f16, 0x7801}, 0, 4);  // "字符编码"(\u5b57是‘字’的unicode编码)。0表示起始位置，4表示长度。
            String str14 = new String(new StringBuffer("StringBuffer"));
            String str15 = new String(new StringBuilder("StringBuilder"));

            System.out.printf(" str01=%s \n str02=%s \n str03=%s \n str04=%s \n str05=%s \n str06=%s \n str07=%s \n str08=%s\n str09=%s\n str10=%s\n str11=%s\n str12=%s\n str13=%s\n str14=%s\n str15=%s\n",
                    str01, str02, str03, str04, str05, str06, str07, str08, str09, str10, str11, str12, str13, str14, str15);


            System.out.println();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：
```
-------------------------------- testStringConstructors -----------------------
 str01= 
 str02=String02 
 str03=str03 
 str04=tr0 
 str05=abcde 
 str06=bcd 
 str07=abcde 
 str08=bcd
 str09=字符编码
 str10=字符编码
 str11=字符编码
 str12=字符编码
 str13=字符编码
 str14=StringBuffer
 str15=StringBuilder
```

# String 将各种对象转换成String的API
源码如下(StringValueTest.java):
```java
/**
 * String value相关示例
 *
 * @author skywang
 */
import java.util.HashMap;

public class StringValueTest {
    
    public static void main(String[] args) {
        testValueAPIs() ;
    }

    /**
     * String 的valueOf()演示程序
     */
    private static void testValueAPIs() {
        System.out.println("-------------------------------- testValueAPIs --------------------------------");
        // 1. String    valueOf(Object obj)
        //  实际上，返回的是obj.toString();
        HashMap map = new HashMap();
        map.put("1", "one");
        map.put("2", "two");
        map.put("3", "three");
        System.out.printf("%-50s = %s\n", "String.valueOf(map)", String.valueOf(map));

        // 2.String    valueOf(boolean b)
        System.out.printf("%-50s = %s\n", "String.valueOf(true)", String.valueOf(true));

        // 3.String    valueOf(char c)
        System.out.printf("%-50s = %s\n", "String.valueOf('m')", String.valueOf('m'));

        // 4.String    valueOf(int i)
        System.out.printf("%-50s = %s\n", "String.valueOf(96)", String.valueOf(96));

        // 5.String    valueOf(long l)
        System.out.printf("%-50s = %s\n", "String.valueOf(12345L)", String.valueOf(12345L));

        // 6.String    valueOf(float f)
        System.out.printf("%-50s = %s\n", "String.valueOf(1.414f)", String.valueOf(1.414f));

        // 7.String    valueOf(double d)
        System.out.printf("%-50s = %s\n", "String.valueOf(3.14159d)", String.valueOf(3.14159d));

        // 8.String    valueOf(char[] data)
        System.out.printf("%-50s = %s\n", "String.valueOf(new char[]{'s','k','y'})", String.valueOf(new char[]{'s','k','y'}));

        // 9.String    valueOf(char[] data, int offset, int count)
        System.out.printf("%-50s = %s\n", "String.valueOf(new char[]{'s','k','y'}, 0, 2)", String.valueOf(new char[]{'s','k','y'}, 0, 2));

        System.out.println();
    }
}
```
运行结果：
```
-------------------------------- testValueAPIs --------------------------------
String.valueOf(map)                                = {3=three, 2=two, 1=one}
String.valueOf(true)                               = true
String.valueOf('m')                                = m
String.valueOf(96)                                 = 96
String.valueOf(12345L)                             = 12345
String.valueOf(1.414f)                             = 1.414
String.valueOf(3.14159d)                           = 3.14159
String.valueOf(new char[]{'s','k','y'})            = sky
String.valueOf(new char[]{'s','k','y'}, 0, 2)      = sk
```

# String 中index相关的API
源码如下(StringIndexTest.java):
```java
/**
 * String 中index相关API演示
 *
 * @author skywang
 */

public class StringIndexTest {

    public static void main(String[] args) {
        testIndexAPIs() ;
    }

    /**
     * String 中index相关API演示
     */
    private static void testIndexAPIs() {
        System.out.println("-------------------------------- testIndexAPIs --------------------------------");

        String istr = "abcAbcABCabCaBcAbCaBCabc";
        System.out.printf("istr=%s\n", istr);

        // 1. 从前往后，找出‘a’第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.indexOf((int)'a')", istr.indexOf((int)'a'));

        // 2. 从位置5开始，从前往后，找出‘a’第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.indexOf((int)'a', 5)", istr.indexOf((int)'a', 5));

        // 3. 从后往前，找出‘a’第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.lastIndexOf((int)'a')", istr.lastIndexOf((int)'a'));

        // 4. 从位置10开始，从后往前，找出‘a’第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.lastIndexOf((int)'a', 10)", istr.lastIndexOf((int)'a', 10));


        // 5. 从前往后，找出"bc"第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.indexOf(\"bc\")", istr.indexOf("bc"));

        // 6. 从位置5开始，从前往后，找出"bc"第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.indexOf(\"bc\", 5)", istr.indexOf("bc", 5));

        // 7. 从后往前，找出"bc"第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.lastIndexOf(\"bc\")", istr.lastIndexOf("bc"));

        // 8. 从位置4开始，从后往前，找出"bc"第一次出现的位置
        System.out.printf("%-30s = %d\n", "istr.lastIndexOf(\"bc\", 4)", istr.lastIndexOf("bc", 4));

        System.out.println();
    }
}
```
运行结果：
```
-------------------------------- testIndexAPIs --------------------------------
istr=abcAbcABCabCaBcAbCaBCabc
istr.indexOf((int)'a')         = 0
istr.indexOf((int)'a', 5)      = 9
istr.lastIndexOf((int)'a')     = 21
istr.lastIndexOf((int)'a', 10) = 9
istr.indexOf("bc")             = 1
istr.indexOf("bc", 5)          = 22
istr.lastIndexOf("bc")         = 22
istr.lastIndexOf("bc", 4)      = 4
```

# String “比较”操作的API
源码如下(StringCompareTest.java):
```java
/**
 * String 中比较相关API演示
 *
 * @author skywang
 */

public class StringCompareTest {

    public static void main(String[] args) {
        testCompareAPIs() ;
    }

    /**
     * String 中比较相关API演示
     */
    private static void testCompareAPIs() {
        System.out.println("-------------------------------- testCompareAPIs ------------------------------");

        //String str = "abcdefghijklmnopqrstuvwxyz";
        String str = "abcAbcABCabCAbCabc";
        System.out.printf("str=%s\n", str);

        // 1. 比较“2个String是否相等”
        System.out.printf("%-50s = %b\n", 
                "str.equals(\"abcAbcABCabCAbCabc\")", 
                str.equals("abcAbcABCabCAbCabc"));

        // 2. 比较“2个String是否相等(忽略大小写)”
        System.out.printf("%-50s = %b\n", 
                "str.equalsIgnoreCase(\"ABCABCABCABCABCABC\")", 
                str.equalsIgnoreCase("ABCABCABCABCABCABC"));

        // 3. 比较“2个String的大小”
        System.out.printf("%-40s = %d\n", "str.compareTo(\"abce\")", str.compareTo("abce"));

        // 4. 比较“2个String的大小(忽略大小写)”
        System.out.printf("%-40s = %d\n", "str.compareToIgnoreCase(\"ABC\")", str.compareToIgnoreCase("ABC"));

        // 5. 字符串的开头是不是"ab"
        System.out.printf("%-40s = %b\n", "str.startsWith(\"ab\")", str.startsWith("ab"));

        // 6. 字符串的从位置3开头是不是"ab"
        System.out.printf("%-40s = %b\n", "str.startsWith(\"Ab\")", str.startsWith("Ab", 3));

        // 7. 字符串的结尾是不是"bc"
        System.out.printf("%-40s = %b\n", "str.endsWith(\"bc\")", str.endsWith("bc"));

        // 8. 字符串的是不是包含"ABC"
        System.out.printf("%-40s = %b\n", "str.contains(\"ABC\")", str.contains("ABC"));

        // 9. 比较2个字符串的部分内容
        String region1 = str.substring(2, str.length());    // 获取str位置3(包括)到末尾(不包括)的子字符串
        // 将“str中从位置2开始的字符串”和“region1中位置0开始的字符串”进行比较，比较长度是5。
        System.out.printf("regionMatches(%s) = %b\n", region1, 
                str.regionMatches(2, region1, 0, 5));

        // 10. 比较2个字符串的部分内容(忽略大小写)
        String region2 = region1.toUpperCase();    // 将region1转换为大写
        String region3 = region1.toLowerCase();    // 将region1转换为小写
        System.out.printf("regionMatches(%s) = %b\n", region2, 
                str.regionMatches(2, region2, 0, 5));
        System.out.printf("regionMatches(%s) = %b\n", region3, 
                str.regionMatches(2, region3, 0, 5));

        // 11. 比较“String”和“StringBuffer”的内容是否相等
        System.out.printf("%-60s = %b\n", 
                "str.contentEquals(new StringBuffer(\"abcAbcABCabCAbCabc\"))", 
                str.contentEquals(new StringBuffer("abcAbcABCabCAbCabc")));

        // 12. 比较“String”和“StringBuilder”的内容是否相等
        System.out.printf("%-60s = %b\n", 
                "str.contentEquals(new StringBuilder(\"abcAbcABCabCAbCabc\"))", 
                str.contentEquals(new StringBuilder("abcAbcABCabCAbCabc")));

        // 13. match()测试程序
        // 正则表达式 xxx.xxx.xxx.xxx，其中xxx中x的取值可以是0～9，xxx中有1～3位。
        String reg_ipv4 = "[0-9]{3}(\\.[0-9]{1,3}){3}";    

        String ipv4addr1 = "192.168.1.102";
        String ipv4addr2 = "192.168";
        System.out.printf("%-40s = %b\n", "ipv4addr1.matches()", ipv4addr1.matches(reg_ipv4));
        System.out.printf("%-40s = %b\n", "ipv4addr2.matches()", ipv4addr2.matches(reg_ipv4));

        System.out.println();
    }
}
```
运行结果：
```
-------------------------------- testCompareAPIs ------------------------------
str=abcAbcABCabCAbCabc
str.equals("abcAbcABCabCAbCabc")                   = true
str.equalsIgnoreCase("ABCABCABCABCABCABC")         = true
str.compareTo("abce")                    = -36
str.compareToIgnoreCase("ABC")           = 15
str.startsWith("ab")                     = true
str.startsWith("Ab")                     = true
str.endsWith("bc")                       = true
str.contains("ABC")                      = true
regionMatches(cAbcABCabCAbCabc) = true
regionMatches(CABCABCABCABCABC) = false
regionMatches(cabcabcabcabcabc) = false
str.contentEquals(new StringBuffer("abcAbcABCabCAbCabc"))    = true
str.contentEquals(new StringBuilder("abcAbcABCabCAbCabc"))   = true
ipv4addr1.matches()                      = true
ipv4addr2.matches()                      = false
```

# String “修改(追加/替换/截取/分割)”操作的API
源码如下(StringModifyTest.java):
```java
/**
 * String 中 修改(追加/替换/截取/分割)字符串的相关API演示
 *
 * @author skywang
 */

public class StringModifyTest {
    
    public static void main(String[] args) {
        testModifyAPIs() ;
    }

    /**
     * String 中 修改(追加/替换/截取/分割)字符串的相关API演示
     */
    private static void testModifyAPIs() {
        System.out.println("-------------------------------- testModifyAPIs -------------------------------");

        String str = " abcAbcABCabCAbCabc ";
        System.out.printf("str=%s, len=%d\n", str, str.length());

        // 1.追加
        // 将"123"追加到str之后
        System.out.printf("%-30s = %s\n", "str.concat(\"123\")", 
                str.concat("123"));

        // 2.截取
        // 截取str中从位置7(包括)开始的元素。
        System.out.printf("%-30s = %s\n", "str.substring(7)", str.substring(7));
        // 截取str中从位置7(包括)到位置10(不包括)之间的元素。
        System.out.printf("%-30s = %s\n", "str.substring(7, 10)", str.substring(7, 10));
        // 删除str中首位的空格，并返回。
        System.out.printf("%-30s = %s, len=%d\n", "str.trim()", str.trim(), str.trim().length());

        // 3.替换
        // 将str中的 “字符‘a’” 全部替换为 “字符‘_’”
        System.out.printf("%-30s = %s\n", "str.replace('a', 'M')", str.replace('a', '_'));
        // 将str中的第一次出现的“字符串“a”” 替换为 “字符串“###””
        System.out.printf("%-30s = %s\n", "str.replaceFirst(\"a\", \"###\")", str.replaceFirst("a", "###"));
        // 将str中的 “字符串“a”” 全部替换为 “字符串“$$$””
        System.out.printf("%-30s = %s\n", "str.replace(\"a\", \"$$$\")", str.replace("a", "$$$"));

        // 4.分割
        // 以“b”作为分隔符，对str进行分割
        String[] splits = str.split("b");
        for (int i=0; i<splits.length; i++) {
            System.out.printf("splits[%d]=%s\n", i, splits[i]);
        }

        System.out.println();
    }
}
```
运行结果：
```
-------------------------------- testModifyAPIs -------------------------------
str= abcAbcABCabCAbCabc , len=20
str.concat("123")              =  abcAbcABCabCAbCabc 123
str.substring(7)               = ABCabCAbCabc 
str.substring(7, 10)           = ABC
str.trim()                     = abcAbcABCabCAbCabc, len=18
str.replace('a', 'M')          =  _bcAbcABC_bCAbC_bc 
str.replaceFirst("a", "###")   =  ###bcAbcABCabCAbCabc 
str.replace("a", "$$$")        =  $$$bcAbcABC$$$bCAbC$$$bc 
splits[0]= a
splits[1]=cA
splits[2]=cABCa
splits[3]=CA
splits[4]=Ca
splits[5]=c
```

# String 操作Unicode的API
源码如下(StringUnicodeTest.java):
```java
/**
 * String 中与unicode相关的API
 *
 * @author skywang
 */

public class StringUnicodeTest {
    
    public static void main(String[] args) {
        testUnicodeAPIs() ;
    }

    /**
     * String 中与unicode相关的API
     */
    private static void testUnicodeAPIs() {
        System.out.println("-------------------------------- testUnicodeAPIs ------------------------------");

        String ustr = new String(new int[] {0x5b57, 0x7b26, 0x7f16, 0x7801}, 0, 4);  // "字符编码"(\u5b57是‘字’的unicode编码)。0表示起始位置，4表示长度。
        System.out.printf("ustr=%s\n", ustr);

        //  获取位置0的元素对应的unciode编码
        System.out.printf("%-30s = 0x%x\n", "ustr.codePointAt(0)", ustr.codePointAt(0));

        // 获取位置2之前的元素对应的unciode编码
        System.out.printf("%-30s = 0x%x\n", "ustr.codePointBefore(2)", ustr.codePointBefore(2));

        // 获取位置1开始偏移2个代码点的索引
        System.out.printf("%-30s = %d\n", "ustr.offsetByCodePoints(1, 2)", ustr.offsetByCodePoints(1, 2));

        // 获取第0~3个元素之间的unciode编码的个数
        System.out.printf("%-30s = %d\n", "ustr.codePointCount(0, 3)", ustr.codePointCount(0, 3));

        System.out.println();
    }
}
```
运行结果：
```
-------------------------------- testUnicodeAPIs ------------------------------
ustr=字符编码
ustr.codePointAt(0)            = 0x5b57
ustr.codePointBefore(2)        = 0x7b26
ustr.offsetByCodePoints(1, 2)  = 3
ustr.codePointCount(0, 3)      = 3
```

# String 剩余的API
源码如下(StringOtherTest.java):
```java
/**
 * String 中其它的API
 *
 * @author skywang
 */

public class StringOtherTest {
    
    public static void main(String[] args) {
        testOtherAPIs() ;
    }

    /**
     * String 中其它的API
     */
    private static void testOtherAPIs() {
        System.out.println("-------------------------------- testOtherAPIs --------------------------------");

        String str = "0123456789";
        System.out.printf("str=%s\n", str);

        // 1. 字符串长度
        System.out.printf("%s = %d\n", "str.length()", str.length());

        // 2. 字符串是否为空
        System.out.printf("%s = %b\n", "str.isEmpty()", str.isEmpty());

        // 3. [字节] 获取字符串对应的字节数组
        byte[] barr = str.getBytes();
        for (int i=0; i<barr.length; i++) {
               System.out.printf("barr[%d]=0x%x ", i, barr[i]);
        }
        System.out.println();

        // 4. [字符] 获取字符串位置4的字符
        System.out.printf("%s = %c\n", "str.charAt(4)", str.charAt(4));

        // 5. [字符] 获取字符串对应的字符数组
        char[] carr = str.toCharArray();
        for (int i=0; i<carr.length; i++) {
               System.out.printf("carr[%d]=%c ", i, carr[i]);
        }
        System.out.println();

        // 6. [字符] 获取字符串中部分元素对应的字符数组
        char[] carr2 = new char[3];
        str.getChars(6, 9, carr2, 0);
        for (int i=0; i<carr2.length; i++) {
               System.out.printf("carr2[%d]=%c ", i, carr2[i]);
        }
        System.out.println();

        // 7. [字符] 获取字符数组对应的字符串
        System.out.printf("%s = %s\n", 
                "str.copyValueOf(new char[]{'a','b','c','d','e'})", 
                String.copyValueOf(new char[]{'a','b','c','d','e'}));

        // 8. [字符] 获取字符数组中部分元素对应的字符串
        System.out.printf("%s = %s\n", 
                "str.copyValueOf(new char[]{'a','b','c','d','e'}, 1, 4)", 
                String.copyValueOf(new char[]{'a','b','c','d','e'}, 1, 4));

        // 9. format()示例，将对象数组按指定格式转换为字符串
        System.out.printf("%s = %s\n", 
                "str.format()", 
                String.format("%s-%d-%b", "abc", 3, true));

        System.out.println();
    }
}
```
运行结果：
```
-------------------------------- testOtherAPIs --------------------------------
str=0123456789
str.length() = 10
str.isEmpty() = false
barr[0]=0x30 barr[1]=0x31 barr[2]=0x32 barr[3]=0x33 barr[4]=0x34 barr[5]=0x35 barr[6]=0x36 barr[7]=0x37 barr[8]=0x38 barr[9]=0x39 
str.charAt(4) = 4
carr[0]=0 carr[1]=1 carr[2]=2 carr[3]=3 carr[4]=4 carr[5]=5 carr[6]=6 carr[7]=7 carr[8]=8 carr[9]=9 
carr2[0]=6 carr2[1]=7 carr2[2]=8 
str.copyValueOf(new char[]{'a','b','c','d','e'}) = abcde
str.copyValueOf(new char[]{'a','b','c','d','e'}, 1, 4) = bcde
str.format() = abc-3-true
```

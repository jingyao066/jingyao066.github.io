---
title: mysql数据类型
tags: mysql
date: 2018-12-25 11:31:38
---

# mysql数据类型
主要包括以下五大类：
```
整数类型：BIT、BOOL、TINY INT、SMALL INT、MEDIUM INT、 INT、 BIG INT

浮点数类型：FLOAT、DOUBLE、DECIMAL

字符串类型：CHAR、VARCHAR、TINY TEXT、TEXT、MEDIUM TEXT、LONGTEXT、TINY BLOB、BLOB、MEDIUM BLOB、LONG BLOB

日期类型：Date、DateTime、TimeStamp、Time、Year

其他数据类型：BINARY、VARBINARY、ENUM、SET、Geometry、Point、MultiPoint、LineString、MultiLineString、Polygon、GeometryCollection等
```

## 整型
MySQL数据类型 | 含义（有符号）
-- | --
tinyint(m) | 1个字节  范围(-128~127)
smallint(m) | 2个字节  范围(-32768~32767)
mediumint(m) | 3个字节  范围(-8388608~8388607)
int(m) | 4个字节  范围(-2147483648~2147483647)
bigint(m) | 8个字节  范围(+-9.22*10的18次方)

取值范围如果加了unsigned，则最大值翻倍，如tinyint unsigned的取值范围为(0~256)。
int(m)里的m是表示SELECT查询结果集中的显示宽度，并不影响实际的取值范围。也不影响插入的内容，如：
设置字段test_length为int(1)，即显示宽度为1，不影响插入内容，即：
`insert into test (test_length) values(1000000)`
是可以正常插入的。
但是，想要真正影响显示宽度，还需要一个属性，zerofill，即Navicat中的`填充零`选项，数据长度不足的，将会以0填充在前边。
设置zerofill之后，如果设置int(3)，即显示长度为3，插入10，select出的结果为：010，若不设置zerofill，结果为：10
超出设置长度，则正常显示。

## 浮点型与定点型
MySQL数据类型 | 含义
-- | --
float(m,d) | 单精度浮点型    8位精度(4字节)     m总个数，d小数位
double(m,d) | 双精度浮点型    16位精度(8字节)    m总个数，d小数位
DECIMAL(m,d) | 

1.MySQL中可以指定浮点数和定点数的精度。其基本形式如下：数据类型(M,D)，M参数称为精度，是数据的总长度，小数点也占位置(MYSQL5.8)。D参数称为标度，是指小数点后面的长度是D
例：float(4,2)的含义数据是float型，数据长度是4，小数点后保留2位。所以，12.34是符合要求的。4指数据总长度是4，包含小数点的长度，如果插入123.45，就会报错：
`Out of range value for column`
意思是超出了范围值。
而且，float(m,d)，d不能比m大，m只能 >= d， 这是在数据表创建时规定的，否则会报
`M must >= D`
若m=d，即float(4,4)，则只能插入比1小的数，即0.1234；
d表示小数点位数，若数据超出指定长度：
.float和double类型，会直接截断指定长度之后的数据
.decimal类型，则遵守四舍五入
这就是为什么float和double类型会损失精度，所以尽量少去使用这两种类型。

2.上述指定的小数精度的方法虽然都适用于浮点数和定点数，但不是浮点数的标准用法。建议在定义浮点数时，如果不是实际情况需要，最好不要使用，如果使用了，可能会影响数据库的迁移。

3.如果插入值的精度高于实际定义的精度，系统会自动进行四舍五入处理，使值的精度达到要求。

4.浮点型在数据库中存放的是近似值，而定点类型在数据库中存放的是精确值

5.未指定精度：
.float和double默认会保存实际精度，但这与操作系统和硬件的精度有关。
.decimal型的默认整数位为10，小数位为0，即默认为整数。

6.在MySQL中，定点数以字符串形式存储，因此，其精度比浮点数要高，而且浮点数会出现误差，这是浮点数一直存在的缺陷。
如果要对数据的精度要求比较高，比如订单价格、商品售价等等，还是选择定点数decimal比较安全。

## 字符串(char,varchar,_text)
MySQL数据类型 | 含义
-- | --
char(n) | n个字节，1 <= n <= 255，固定长度，最多255个字符
varchar(n) | 可变长度，最多65535个字符
tinytext | 可变长度，最多255个字符
text | 可变长度，最多65535个字符
mediumtext | 可变长度，最多2的24次方-1个字符
longtext | 可变长度，最多2的32次方-1个字符

### char
`char(n)`
1.若存入字符数小于n，则以空格补于其后，查询之时再将空格去掉。所以char类型存储的字符串末尾不能有空格
2.存入字符数不能大于n,若超出，则会报：
`Data too long for column`
3.固定长度，占用n(指定长度)个字节

### varchar
1.varchar是存入的实际字符数+1个字节

### 速度对比
char > varchar > text
char、varchar可直接创建索引，text创建索引要指定前多少个字符。varchar查询速度快于text,在都创建索引的情况下，text的索引似乎不起作用。

#### 关于varchar的长度
有必要单独说明，比较复杂：

1.编码长度限制
字符类型若为latin1，每个字符最多占1个字节，最大长度不能超过65535;
字符类型若为gbk，每个字符最多占2个字节，最大长度不能超过32766;
字符类型若为utf8，每个字符最多占3个字节，最大长度不能超过21845。
若定义的时候超过上述限制，则varchar字段会被强行转为text类型，并产生warning。

2.存储限制
varchar最多能存储65535个字节的数据。varchar 的最大长度受限于最大行长度（max row size，65535bytes）。65535并不是一个很精确的上限，可以继续缩小这个上限。
65535个字节包括所有字段的长度，变长字段的长度标识（每个变长字段额外使用1或者2个字节记录实际数据长度）、NULL标识位的累计。

NULL标识位：
如果varchar字段定义中带有default null允许列空,则需要需要1bit来标识，每8个bits的标识组成一个字段。一张表中存在N个varchar字段，那么需要（N+7）/8 （取整）bytes存储所有的NULL标识位。

最大行长度：
MySQL要求一个行的定义长度不能超过65535。若定义的表长度超过这个值，则提示：
`ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs。`
即：一行中所有字段加起来的长度，不能超过65535。
如：一行有两个字段，一个char(100)，那么另外varchar字段的长度只能是(65435):65535 - 100,超出会报上面的错误。

如果数据表只有一个varchar字段且该字段DEFAULT NULL，那么该varchar字段的最大长度为65532个字节，即65535-2-1=65532 byte。
为什么 -2 -1 ？
字符集是latin1，占用1个字节。
null标识位，( 1 + 7 ) / 8 =1,所以null标识位占用1个字节。
因为varchar类型存储变长字段的字符类型，与char类型不同的是，其存储时需要在前缀长度列表加上实际存储的字符，当存储的字符串长度小于255字节时，其需要1字节的空间，当大于255字节时，需要2字节的空间。

3.行长度限制
即最大行长度限制。

##### 举例说明实际长度的计算
1.若一个表只有一个varchar类型，如定义为
create table t4(c varchar(N)) charset=gbk;
则此处N的最大值为(65535-1-2)/2= 32766。
减1的原因是实际行存储从第二个字节开始;
减2的原因是varchar头部的2个字节表示长度;
除2的原因是字符编码是gbk。

2.若一个表定义为
create table t4(c int, c2 char(30), c3 varchar(N)) charset=utf8;
则此处N的最大值为 (65535-1-2-4-30*3)/3=21812
减1和减2与上例相同;
减4的原因是int类型的c占4个字节;
减30*3的原因是char(30)占用90个字节，编码是utf8。
如果被varchar超过上述的b规则，被强转成text类型，则每个字段占用定义长度为11字节，当然这已经不是varchar了。
则此处N的最大值为 (65535-1-2-4-30*3)/3=21812

3.
```
CREATE TABLE t6 (
id int,
a VARCHAR(100) DEFAULT NULL,
b VARCHAR(100) DEFAULT NULL,
c VARCHAR(100) DEFAULT NULL,
d VARCHAR(100) DEFAULT NULL,
e VARCHAR(100) DEFAULT NULL,
f VARCHAR(100) DEFAULT NULL,
g VARCHAR(100) DEFAULT NULL,
h VARCHAR(100) DEFAULT NULL,
i VARCHAR(N) DEFAULT NULL
) CHARSET=utf8;
```
每个NULL字段用1bit标识，10个字段都是default null，那么需要用(10+7)/8bit = 2 bytes存储NULL标识位。int占用4个 byte。
(65535 - 1 - 2*8  -4 - 100*3*8 -2)/3=21037

我想，这样应该能够很好的说明varchar的长度问题了。
参考内容：<<MySQL技术内幕--InnoDB引擎第二版>>

## 日期时间类型
MySQL数据类型 | 含义
-- | --
date | 日期 '2008-12-2'
time | 时间 '12:25:36'
datetime | 日期时间 '2008-12-2 22:06:44'
timestamp | 自动存储记录修改时间

若定义一个字段为timestamp，这个字段里的时间数据会随其他字段修改的时候自动刷新，所以这个数据类型的字段可以存放这条记录最后被修改的时间。

## 数据类型的属性
MySQL关键字 | 含义
-- | --
NULL | 数据列可包含NULL值
NOT NULL | 数据列不允许包含NULL值
DEFAULT | 默认值
PRIMARY KEY | 主键
AUTO_INCREMENT | 自动递增，适用于整数类型
UNSIGNED | 无符号
CHARACTER SET name | 指定一个字符集

## 数据类型长度和范围一览

数据类型 | 字节长度 | 范围或用法
-- | -- | --
Bit | 1 | 无符号[0,255]，有符号[-128,127]，BIT和BOOL布尔型都占用1字节
TinyInt | 1 | 整数[0,255]
SmallInt | 2 | 无符号[0,65535]，有符号[-32768,32767]
MediumInt | 3 | 无符号[0,2^24-1]，有符号[-2^23,2^23-1]]
Int | 4 | 无符号[0,2^32-1]，有符号[-2^31,2^31-1]
BigInt | 8 | 无符号[0,2^64-1]，有符号[-2^63 ,2^63 -1]
Float(M,D) | 4 | 单精度浮点数
Double(M,D) | 8 | 双精度浮点。
Decimal(M,D) | M+1或M+2 | 未打包的浮点数，用法类似于FLOAT和DOUBLE
Date | 3 | 以YYYY-MM-DD的格式显示，比如：2009-07-19
Date Time | 8 | 以YYYY-MM-DD HH:MM:SS的格式显示，比如：2009-07-19 11：22：30
TimeStamp | 4 | 以YYYY-MM-DD的格式显示，比如：2009-07-19
Time | 3 | 以HH:MM:SS的格式显示。比如：11：22：30
Year | 1 | 以YYYY的格式显示。比如：2009
Char(M) | M | 定长字符串。
VarChar(M) | M | 变长字符串，要求M<=255
Binary(M) | M | 类似Char的二进制存储，特点是插入定长不足补0
VarBinary(M) | M | 类似VarChar的变长二进制存储，特点是定长不补0
Tiny Text | Max:255 | 大小写不敏感
Text | Max:64K | 大小写不敏感
Medium Text | Max:16M | 大小写不敏感
Long Text | Max:4G | 大小写不敏感
TinyBlob | Max:255 | 大小写敏感
Blob | Max:64K | 大小写敏感
MediumBlob | Max:16M | 大小写敏感
LongBlob | Max:4G | 大小写敏感
Enum | 1或2 | 最大可达65535个不同的枚举值
Set | 可达8 | 最大可达64个不同的值
Geometry |   |  
Point |   |  
LineString |   |  
Polygon |   |  
MultiPoint |   |  
MultiLineString |   |  
MultiPolygon |   |  
GeometryCollection |   |  


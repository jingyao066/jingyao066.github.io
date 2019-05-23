---
title: mysql基础
tags: mysql
date: 2018-12-06 18:12:58
---

# 增删改查
```
插入(增)：
insert into 表名字(列名1，列名2...) values(值1，值2...)
或者：
insert into 表名字 values(值1，值2...)
若不指定列名，则需要插入全部字段。
```

```
删除：
delete from 表名字 where id = 1
若不加where条件，则删除数据表中所有数据
```

```
修改：
update 表名字 set 列1=值1，列2=值2... where id = 1
若不加where条件，则更新数据表中所有数据
```

```
查询：
select * from 表名字
星号代表查询数据表中所有字段
select id,stu_name,age, from 表名字
查询多个字段，用逗号进行分割
```

# 数据类型
主要包括以下五大类：
整数类型：BIT、BOOL、TINY INT、SMALL INT、MEDIUM INT、 INT、 BIG INT
浮点数类型：FLOAT、DOUBLE、DECIMAL
字符串类型：CHAR、VARCHAR、TINY TEXT、TEXT、MEDIUM TEXT、LONGTEXT、TINY BLOB、BLOB、MEDIUM BLOB、LONG BLOB
日期类型：Date、DateTime、TimeStamp、Time、Year
其他数据类型：BINARY、VARBINARY、ENUM、SET、Geometry、Point、MultiPoint、LineString、MultiLineString、Polygon、GeometryCollection等

# 库相关
- 创建数据库:
`create database[if not exists] db_name`
- 删除数据库:
`drop database[if exists] db_name`
- 查看当前所有数据库
`show databases`

注：需要将`if exists`旁边的括号去掉，中括号表示可选参数。
`if not exists`：如果不存在
`if exists`：如果存在
加入这个参数可以避免sql报错。

# 表相关
- 创建数据表:
```sql
create table[if not exists] table_name(
    列名 类型 是否为空
    列名 类型 是否为空
)
```
示例：
```sql
CREATE TABLE `grade` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `grade_name` varchar(10) DEFAULT NULL COMMENT '年级名字',
  `create_time` int(11) DEFAULT NULL COMMENT '创建时间',
  `version` int(11) DEFAULT '1' COMMENT '版本号',
  `sort` int(11) DEFAULT NULL COMMENT '排序',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='年级表';
```

字段参数说明：
` id int(11) NOT NULL AUTO_INCREMENT`
```
id：字段名称
int(11)：字段类型及长度
NOT NULL：是否可为空，可为空则为null
AUTO_INCREMENT：自动递增，一般此属性只应用在主键上
COMMENT：字段说明
```

- 删除表
`DROP TABLE [if exists] table_name`

- 显示表的数据结构
`desc table_name`

- 查看数据库中所有的表
`show tables`

- 修改数据表

```
ALTER TABLE <表名>
[ALTER COLUMN <列名> <新数据类型>]
|[ADD <列名> <数据类型> [属性]
|[DROP COLUMN <列名>]
|[ADD [constraint] [约束名] 约束定义
|[DROP [constraint] 约束名]
```

- 新增字段
`alter table table_name add column_name varchar(200) not null default 0 COMMENT "描述";`

- 修改字段
`ALTER TABLE table_name MODIFY column_name VARCHAR(200) NULL DEFAULT null COMMENT '这是字段描述';`

- 删除列
`ALTER TABLE 表名 DROP 列名`

- 复制表
结构、数据都复制：
`CREATE TABLE 新表名 SELECT * FROM 旧表名`
仅结构
`CREATE TABLE 新表名 LIKE 旧表名`

- A表数据导入B表：
`INSERT INTO 表名 SELECT * FROM 有数据的表`

# 主外键
主键：`primary_key`
- 保证数据唯一性
- 一个数据表只能有一个主键
- 主键不可以为空，并且不可以重复
	
外键：`foreign key`
- 保证数据的完整性，一致性
- 外键列和参照列的数据类型必须一致
- 参照列的表为父表，外键列的表为子表
- 实现一对多，多对一的关系
- 外键列和参照列必须创建索引，如不创建，mySQL会自动创建
```
create table test(
	id int primary_key auto_increment,
	foreign key(test1_id) references  test2(id)
	//外键列为test1_id(当前表的列),参照了test2表的id列
)
```
尽量不要使用外键，我们一般人为的人为表与表之间是相关联的。如：
班级表class为主表，在学生表student中加入class_id字段，表示班级id，这样就人为的认为student为class的字表，两张表就关联起来了。
使用外键在删除表或修改表结构时，会造成很大麻烦。

# where条件
- 可以跟在`select、update、delete`后
- 多个where条件，并且的关系，用and链接，或者的关系，用or链接
- `group by`：把查询结构进行分组，分组结果可以升序(asc)或降序(desc)，默认升序，用于去重
- `having`：如果要将group by的结果进行删选，需要用到having关键字，不可以用where
- `order by`：把查询结果进行排序，可升(asc)可降(desc),默认升序，可以进行多次排序，用逗号分割排序字段，越靠前，优先级越高
- `limit`：限制查询返回的结果(用于分页功能)是select语句的最后一个关键字
	参数1：从第几条结果开始获取
	参数2：获取几条结果

select语句结构：	
```sql
select * from table_name
	where
	 gruop by
	  having
	   order by
	    limit
```
关键字顺序不可以颠倒

## where条件参数介绍
以下参数必须跟在where关键字后面，作为where的修饰
```
=：相等
>(=)：大于(等于)
<(=)：小于(等于)
is [not] null：为null(不为null)
[not] like：模糊查询
[not] between ... and ..：(不包含)包含在范围内
[not] in：包含在指定范围值内
```

## null
使用NULL条件检索时不能使用=号，必须使用is
`is null`，不能是`= null`

## like(模糊查询)
`select * from student where stu_name like '%小%';`
`%`表示可随意替换
`小%`表示必须以小开头，后边随意
`%小`表示必须以小结尾，前边随意
`%小%`表示前后都随意，中间包含小

## [not]between ... and ... 
范围获取值：
`select * from user where id between 1 and 15 ;`
属于where关键字，可用大于号和小于号来实现相同的功能

## [not]in
判断一个字段的值是否与括号内的值相同，括号内是or的关系，即满足任何一个条件，都会显示出来
`select * from user where id in(1,2,3) ;`

# 多条件组合AND 和 OR 操作符
查询中使用多个条件组合时，可以使用`and`或者`or`
and：且(必须全部满足)
or：或(只需满足一个)

# 字符函数
 以下字符函数都必须跟在`select`语句后面
`concat(str1,str2,...)`：拼接字符串
`length(str)`：字符串长度
`lower(str)`：字符串转小写
`upper(str)`：字符串转大写
`trim(str)`：去除字符串前后的空格
`right(str,1)`：右侧开始截取，参数2位截取几位
`left(str,1)`：左侧开始截取字符串，参数2为截取几位
`replace(参数1，参数2，参数3):`：1.要处理的字符串2.要替换的字符3.替换成什么
`substring(参数1，参数2，参数3)`：1.要截取的字符串2.从第几位开始截取3.截取的长度

问题：
已知一个数字字段，内容不规律，最长有10位，最短有4位，
想把不够10位的全部在前边用0补齐，怎么办?

SQL：
`update 表名字 set num = right(concat('000000',num),10);`
解：因为最短有4位，在每个字符串左侧都拼接6个0，然后从右侧开始截取10个字符

# 聚合函数
聚合函数必须跟在`select`语句后面
一列的和：`select sum(列名) from 表名 where ...条件`
一列的最大值：`select max(列名) from 表名 where ...条件`
一列的最小值：`select min(列名) from 表名 where ...条件`
一列的数量：`select count(列名/一般都用id) from 表名 where ...条件`
一列的平均值：`select avg(列名) from 表名 where ...条件`

# 数值函数
`ceil()`：向上取整
`floor()`：向下取整
`round()`：四舍五入
`power()`：幂运算

# 日期时间函数
`now()`：当前时间(年月日-时分秒)
`curdate()`：当前日期(年月日)
`curtime()`：当前时间(时分秒)
`unix_timestamp(now())`：当前时间戳，如：1557729824，一共十位数，精确到秒，精确到毫秒需要十三位

`date_add()`：日期加减算法
参数1：要进行运算的日期
参数2：加或减的量
参数3：加或减的日期单位(小时，天，周，月，年)

`datediff()`：日期差值计算
参数1：日期类型
参数2：日期类型
		
`date_format()`：日期格式化
将指定日期，安装指定规则进行格式化(具体请百度该函数)
`date_format('2017-1-1','%y/%m/%d/')--->2017/1/1`
注：该函数会导致索引失效

# case when
```
select case(
	when sex='0' then '女'
	when sex='1' then '男'
	end
)
from 表名
```
`case when`用于查询时替换表中数据，如性别字段，里面有0和1，查询时可分别把0和1替换成女和男

# 表连接
需要注意：
- 查询结果在哪个表
- 链接哪个表
- 链接条件
- where条件

## 表连接类型
- `cross join`：交叉连接
- `inner join`：内连接
- 外链接分为两种：
    `left join`：左外连接
	`right join`：右外链接
- `union 和 union all`：联合查询
- `full join`：全连接

示例表1：
```sql
CREATE TABLE t1 (
  t1_id int(11) NOT NULL AUTO_INCREMENT,
  t1_str varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8;
```

填充数据
```sql
INSERT INTO test.t1(t1_id, t1_str) VALUES (1, '1');
INSERT INTO test.t1(t1_id, t1_str) VALUES (2, '2');
INSERT INTO test.t1(t1_id, t1_str) VALUES (3, '3');
```

示例表2：
```sql
CREATE TABLE t2 (
  t2_id int(11) NOT NULL AUTO_INCREMENT,
  t2_str varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8;
```
填充数据：
```wql
INSERT INTO test.t2(id,t2_str) VALUES (2, 'a');
INSERT INTO test.t2(id,t2_str) VALUES (3, 'b');
```

## 交叉连接
语法：
`select * from t1 cross join t2;`
在mysql中，上述查询语句查询出的结果与如下语句相同:
`select * from t1,t2;`
上述查询的结果：
```
t1_id	t1_str	t2_id	t2_str
1	1	2	a
1	1	3	b
2	2	2	a
2	2	3	b
3	3	2	a
3	3	3	b
```
观察数据发现：
t1表的每一行记录，都与t2表中的任意一条数据关联，反之，t2表的每一行记录，都与t1表中的任意一条数据关联。
换句话说：两张表中的数据被交叉连接在了一起。
我们把上述**没有任何条件的连接**称为交叉连接，该连接得到的结果与线性代数中**笛卡尔积**一样。

不难发现，使用交叉连接时，任意一张表中的记录多出一行，`交叉连接`的数量都会增长很多，而且得到的数据没有意义，所以通过交叉连接进行多表查询这种方法，应尽量避免使用。

## 内连接
概念：
两张表中同时符合某种条件的数据的组合。

语法：
`SELECT * FROM t1,t2 where t1.t1_id = t2.t2_id`
另一种写法：
`SELECT * FROM t1 inner join t2 on t1.t1_id = t2.t2_id`
inner字段可以省略。

查询结果：
```
t1_id	t1_str	t2_id	t2_str
2	2	2	a
3	3	3	b
```

上面的sql语句查询出了t1表与t2表中id号相同的记录，并把两表中id号相同的记录连接在了一起。
其中，where或on后边的语句就是"某种条件"，查询结果就是"数据的组合"，这就是内连接，官方建议使用第二种写法。

mysql中，内连接与交叉连接的不同之处就是，内连接比交叉连接有更多的限制条件。经过试验证明(试验代码略)，无论是给内连接(inner join)去掉限制条件(where、on)，
还是给交叉连接加上限制条件(where、on)，得到的结果都是一样的。即，双方加条件结果一致，不加条件结果也一致。
那么两者可以互相代替么？(待补充)

注意：内连接只是根据条件查询出了两张表中符合条件的数据的组合，如：t1表中t1_id号为1的数据就没有被查出来。
刚才我们使用的是**等值连接**，限制条件通过等号相连，
那么相应的还有**不等连接**，即条件不是通过等号相连。如：
`SELECT * FROM t1 inner join t2 on t1.t1_id > t2.t2_id`
查询结果：
```
t1_id	t1_str	t2_id	t2_str
3	3	2	a
```
应该灵活使用等值or不等值连接。

注意：不能用**交集**的概念去解释内连接，
交集：同时属于集合A、B的元素所组成的集合称为交集。
但上面结果中，id号为1的数据只属于t1表，t2表中不存在。所以内连接定义为：
**同时满足条件的数据的组合**

## 外链接：left join , right join
外链接分为两种，左外连接和右外链接。

语法：
`SELECT * FROM t1 left outer join t2 on t1.t1_id = t2.t2_id`

结果如下：
```
t1_id	t1_str	t2_id	t2_str
2	2	2	a
3	3	3	b
1	1
```

通过与内连接的查询结果对比，发现左外连接查出的数据更多一些，多出的一行记录由t1表中的id号为1的记录和一条"空记录"组成。
可是t2表中并不存在id号为1的记录啊，为什么不符合连接条件的记录也会出现在查询结果中呢？这就是左外连接的特性。

左外连接不仅会查询出两表中同时符合条件的记录的组合，同时还会将"left outer join"左侧的表中的不符合条件的记录同时展示出来，由于左侧表中的这一部分记录并不符合连接条件，所以这一部分记录使用"空记录"进行连接。
换句话说，左外连接"左侧的表"中的所有记录都会被展示出来，左侧表中符合条件的记录将会与右侧表中符合条件的记录相互连接组合，左侧表中不符合条件的记录将会与右侧表中的"空记录"进行连接。
上述示例中的t1表就是"left outer join"左侧的表，t2表就是"left outer join"右侧的表，连接条件就是t1id=t2id,虽然t1表中id号为1的记录不满足连接条件，但是仍然会被展示出来，t2表中会使用"空记录"与其进行连接，表示t1表中对应的记录是不满足连接条件的记录。

了解了左外连接，那么右外链接与之相反，右表中所有记录会被展示，右表中符合条件的记录会与左表中符合条件的记录相互连接组合，右表中不符合条件的记录会与左表中的空记录进行连接。
那么在此就不对右外链接做过多说明了。

其实，左外连接一般简称为左连接，left outer join 简写为 left join，右外链接简称为右连接。简写为 right join。

## 联合查询：union 和 union all
语法：
`select column_name(s) from table_name1 union select column_name(s) from table_name2`

结果：
```
t1_id	t1_str
1	1
2	2
3	3
2	a
3	b
```
两张表的结果被集中显示到了一起。union前表的数据在前边显示，union后的后边显示。
注：使用union连接两个语句时，两个语句查询出的字段数量必须相同，否则无法使用union进行联合查询。

使用union将两个结果集集中显示时，重复的数据会被合并为一条，如果想显示全部数据，请使用union all，这也是union和union all的唯一区别。

## 全连接：full join
在sql标准中，有一种被称为"全连接"的多表查询方式，但是mysql并不支持全连接。
不过，我们可以变相的实现"全连接",在mysql中，我们可以使用"left join"、"union"、"right join"的组合实现所谓的"全连接"。

实现方式：
```
SELECT * FROM t1 left join t2 on t1.t1_id = t2.t2_id
union
SELECT * FROM t1 right outer join t2 on t1.t1_id = t2.t2_id
```

结果：
```
t1_id	t1_str	t2_id	t2_str
2	2	2	a
3	3	3	b
1	1
```

# 子查询
语法：
```
select column_name from table_name
where id in (select ...)
```
所谓子查询，就是嵌套在其他查询语句中的查询，子查询的形式有非常多，可以写在：
- `select`后，作为一个列(column)
- `where`后，作为条件
等等...

# 一些问题
数据库字段中时常包含`描述`字段，意思是作为xxx的描述信息，但是英文单词`describe`以及`desc`都是mysql的关键字，谨慎使用，可以使用`description`

# mysql删除数据出现问题
```sql
delete from table where id in (
	select a.id from table where name like 'xxx'
)
```
错误：You can't specify target table 'xxx' for update in FROM clause
意思是：不能先select同一个表的某些值，然后再update这个表。

解决办法：临时表
```
delete from table where id in (
	select * from(
		select a.id from table where name like 'xxx'
	)a
)
```
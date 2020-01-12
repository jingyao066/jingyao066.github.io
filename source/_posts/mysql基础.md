---
title: mysql基础
tags: mysql
date: 2018-12-06 18:12:58
---

# 增删改查
```
增：
insert into 表名字(列名1，列名2...) values(值1，值2...)
或：
insert into 表名字 values(值1，值2...)
若不指定列名，则需要插入全部字段。
```

```
删：
delete from 表名字 where id = 1
若不加where条件，则删除数据表中所有数据
```

```
改：
update 表名字 set 列1=值1，列2=值2... where id = 1
若不加where条件，则更新数据表中所有数据
```

```
查：
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
id：字段名称
int(11)：字段类型及长度
DEFAULT NULL：默认为空，不可空是not null
AUTO_INCREMENT：自动递增，一般此属性只应用在主键上
COMMENT：字段说明

- 修改数据表

```
ALTER TABLE <表名>
[ALTER COLUMN <列名> <新数据类型>]
|[ADD <列名> <数据类型> [属性]
|[DROP COLUMN <列名>]
|[ADD [constraint] [约束名] 约束定义
|[DROP [constraint] 约束名]
```

- 删除表
`DROP TABLE [if exists] table_name`

- 显示表的数据结构
`desc table_name`

- 查看数据库中所有的表
`show tables`

- 新增字段
`alter table table_name add column_name varchar(200) not null default 0 COMMENT "描述";`

- 修改字段属性
`alter table table_name modify column_name VARCHAR(200) NULL DEFAULT null COMMENT '这是字段描述';`

- 修改字段名
`alter table <表名> change <字段名> <字段新名称> <字段的类型>`

- 删除列
`ALTER TABLE 表名 DROP 列名`

- 复制表
结构、数据都复制：
`CREATE TABLE 新表名 SELECT * FROM 旧表名`
仅结构
`CREATE TABLE 新表名 LIKE 旧表名`

- A表数据导入B表：
`INSERT INTO 表名 SELECT * FROM 有数据的表`

- 仅复制表结构
`create table my_table_copy2 like my_table;`

- 复制表结构和数据
`create table my_table_copy1 select * from my_table;`
注意：该语句只是按select语句执行的结果新建表，并不会复制表的主键、索引等信息。

- 完全复制表
`create table my_table_copy2 like my_table;`
`insert into my_table_copy2 select * from my_table;`

- 复制表，同时重新定义字段名
`create table my_table_copy3`
`select id,username yhm,realname xm,email dzyj,address dz from my_table;`

- 复制表，同时定义字段信息
`create table my_table_copy4 (id INTEGER not null auto_increment PRIMARY KEY) select * from my_table;`

- 检查数据表是否存在
`SELECT table_name FROM information_schema.tables WHERE table_name=#{tableName, jdbcType=VARCHAR}`

- 检查表字段是否存在
`SELECT column_name FROM information_schema.columns WHERE table_name=#{tableName, jdbcType=VARCHAR} AND column_name=#{columnName, jdbcType=VARCHAR}`

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
```sql
create table test(
	id int primary_key auto_increment,
	foreign key(test1_id) references  test2(id)
	//外键列为test1_id(当前表的列),参照了test2表的id列
)
```
尽量不要使用外键，我们一般人为的人为表与表之间是相关联的。如：
班级表class为主表，在学生表student中加入class_id字段，表示班级id，这样就人为的认为student为class的字表，两张表就关联起来了。使用外键在删除表或修改表结构时，会造成很大麻烦。

# where条件
- 可以跟在`select、update、delete`后
- 多个where条件，并且的关系，用and链接，或者的关系，用or链接
- `group by`：把查询结构进行分组，分组结果可以升序(asc)或降序(desc)，默认升序，用于去重
- `having`：如果要将group by的结果进行删选，需要用到having关键字，不可以用where
- `order by`：把查询结果进行排序，可升(asc)可降(desc)，默认升序，可以进行多次排序，用逗号分割排序字段，越靠前，优先级越高
- `limit`：限制查询返回的结果(用于分页功能)是select语句的最后一个关键字
	参数1：从第几条结果开始获取
	参数2：获取几条结果
	只写一个参数：获取几条结果

select语句结构：	
```sql
select * from table_name
	join table_name_b on ...
	where
	 gruop by
	  having
	   order by
	    limit
```
关键字顺序不可以颠倒

## where条件参数介绍
以下参数必须跟在where关键字后面，作为where的修饰
`=`：相等
`>=`：大于等于
`<=`：小于等于
`is [not] null`：为null(不为null)
`[not] like`：模糊查询
`[not] between ... and ..`：(不包含)包含在范围内
`[not] in`：包含在指定范围值内

## null
使用NULL条件检索时不能使用=号，必须使用is：`is null`，不能是`= null`

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
`unix_timestamp()`：当前时间戳，如：1557729824，一共十位数，精确到秒，精确到毫秒需要十三位
`unix_timestamp(now())`：带参数，可以是一个DATE字符串，一个DATETIME字符串，一个TIMESTAMP或者一个当地时间的YYMMDD或YYYYMMDD格式的数字，表示自'1970-01-01 00:00:00'与指定时间的秒数差

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
在mysql中，上述查询语句查询出的结果与如下语句相同：
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
概念：两张表中同时符合某种条件的数据的组合。
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

mysql中，内连接与交叉连接的不同之处就是，内连接比交叉连接有更多的限制条件。经过试验证明(试验代码略)，
无论是给内连接(inner join)去掉限制条件(where、on)，还是给交叉连接加上限制条件(where、on)，得到的结果都是一样的。
即，双方加条件结果一致，不加条件结果也一致。那么两者可以互相代替么？(待补充)

注意：内连接只是根据条件查询出了两张表中符合条件的数据的组合，如：t1表中t1_id号为1的数据就没有被查出来。
刚才我们使用的是**等值连接**，限制条件通过等号相连，
那么相应的还有**不等连接**，即条件不是通过等号相连。如：
`SELECT * FROM t1 inner join t2 on t1.t1_id > t2.t2_id`
查询结果：
```
t1_id	t1_str	t2_id	t2_str
3	3	2	a
```
应该灵活使用等值或不等值连接。

注意：不能用**交集**的概念去解释内连接。
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
上述示例中的t1表就是"left outer join"左侧的表，t2表就是"left outer join"右侧的表，连接条件就是t1id=t2id，虽然t1表中id号为1的记录不满足连接条件，但是仍然会被展示出来，t2表中会使用"空记录"与其进行连接，表示t1表中对应的记录是不满足连接条件的记录。

了解了左外连接，那么右外链接与之相反，右表中所有记录会被展示，右表中符合条件的记录会与左表中符合条件的记录相互连接组合，右表中不符合条件的记录会与左表中的空记录进行连接。那么在此就不对右外链接做过多说明了。
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
不过，我们可以变相的实现"全连接"，在mysql中，我们可以使用"left join"、"union"、"right join"的组合实现所谓的"全连接"。

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

# 问题
- 数据库字段中时常包含`描述`字段，意思是作为xxx的描述信息，但是英文单词`describe`以及`desc`都是mysql的关键字，谨慎使用，可以使用`description`

- mysql删除数据出现问题
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

- tinyint
mysql字段类型为tinyint，tinyint(1)会在查询的时候转换成true false，tinyint(2)以及以上是不会的。这个和mybatis没关系

- this is incompatible with sql_mode=only_full_group_by
这个错误的原因是高版本mysql（客户服务器版本是5.7.18）默认的sql_mode包含ONLY_FULL_GROUP_BY，这个属性保证了select到的列都在group by中出现。 
查看sql_mode的语句如下：
`select @@GLOBAL.sql_mode;`

可以使用sql语句暂时修改sql_mode：
`set @@GLOBAL.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'`

还需要刷新一下：
`flush privileges;`

可能还会报错，你还需要重启服务。。。为什么？？

然而重启mysql数据库之后，ONLY_FULL_GROUP_BY又出现了：
所以需要修改mysql配置文件，通过手动添加sql_mode的方式强制指定不需要ONLY_FULL_GROUP_BY属性，my.cnf位于etc文件夹下，vim下光标移到最后，添加如下：
`sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`
重启后问题解决。

# 修改mysql时区
在配置文件加上：
`default-time_zone = '+8:00'`

# explain关键字
mysql官方介绍：
When EXPLAIN is used with an explainable statement, MySQL displays information from the optimizer about the statement execution plan. That is, MySQL explains how it would process the statement, including information about how tables are joined and in which order.
explain能解释mysql如何处理SQL语句，表的加载顺序，表是如何连接，以及索引使用情况。是SQL优化的重要工具。

explain的结果共包含12个字段，下面逐一解释。

## id
id相同，执行顺序从上之下
id不同，执行顺序从大到小
id相同不同，同时存在，遵守1、2规则

|id|select_type|table|
|-|-|-|
|1|PRIMARY|A|
|1|PRIMARY|B|
|2|SUBQUERY|B|

可以这样理解，执行顺序从大到小，先执行id为2的，然后执行id为1的（先A再B，规则1）；执行顺序为：第三行，第一行，第二行。

## select_type
查询中每个select的查询类型，如下：

- SIMPLE：简单select，不使用union和子查询
`explain select * from A`

- PRIMARY：查询中包含任何复杂的子部分，最外层的select被标记为PRIMARY
`explain select * from A where A.key_A = (select key_B from B where B.id = 1)`

- UNION：union中第二个后面的select语句

- DEPENDENT UNION：一般是子查询中的第二个select语句（取决于外查询，mysql内部也有些优化）

- UNION RESULT：union的结果
```
explain
select * from A
union
select * from B
```

- SUBQUERY：子查询中的第一个select
`explain select * from A where A.key_A = (select key_B from B where B.id = 1)`

- DEPENDENT SUBQUERY：子查询中第一个select，取决于外查询（在mysql中会有些优化，有些dependent会直接优化成simple）

- DERIVED：派生表的select（from子句的子查询），奇怪的是在5.7的版本中竟然只有一个SIMPLE
`explain select * from (select * from A where a.key_A = 1) t`

## table
显示数据来自于哪个表，有时不是真实的表的名字（虚拟表），虚拟表最后一位是数字，代表id为多少的查询。

## type
这个字段是我们优化要重点关注的字段，这个字段直接反映我们SQL的性能是否高效。
这个字段值较多，这里我只重点关注我们开发中经常用到的几个字段：system、const、eq_ref、ref、range、index、all
性能由好到差依次为：system->const->eq_ref->ref->range->index->all

- system：表只有一行记录，这个是const的特例，一般不会出现，可以忽略

- const：表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。
`explain select * from a where id = 1`

- eq_ref：唯一性索引扫描，表中只有一条记录与之匹配。一般是两表关联，关联条件中的字段是主键或唯一索引。
`explain select * from a,b where a.id = b.a_id`

- ref：非唯一行索引扫描，返回匹配某个单独值的所有行
`explain select * from a where a.key_A = 1`

- range：检索给定范围的行，一般条件查询中出现了>、<、in、between等查询
`explain select * from a where a.id > 1`

- index：遍历索引树。通常比ALL快，因为索引文件通常比数据文件小。all和index都是读全表，但index是从索引中检索的，而all是从硬盘中检索的。
`explain select id from a`

- all：遍历全表以找到匹配的行
`explain select * from a`

## possible_keys
显示可能应用在这张表中的索引，但**不一定被查询实际使用**。

## key
实际使用的索引。

## key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。一般来说，索引长度越长表示精度越高，效率偏低；长度越短，效率高，但精度就偏低。并不是真正使用索引的长度，是个预估值。

## ref
表示哪一列被使用了，常数表示这一列等于某个常数。

## rows
大致找到所需记录需要读取的行数。

## filter
表示选取的行和读取的行的百分比，100表示选取了100%，80表示读取了80%。

## extra
- Using filesort：使用外部的索引排序，而不是按照表内的索引顺序进行读取。（一般需要优化）
- Using temporary：使用了临时表保存中间结果。常见于排序order by和分组查询group by（最好优化）
- Using index：表示select语句中使用了覆盖索引，直接冲索引中取值，而不需要回行（从磁盘中取数据）
- Using where：使用了where过滤
- Using index condition：5.6之后新增的，表示查询的列有非索引的列，先判断索引的条件，以减少磁盘的IO
- Using join buffer：使用了连接缓存
- impossible where：where子句的值总是false
还有一些，基本上很少遇到，就不作说明了。

# mysql使用规范
表存储引擎必须使用InnoDB

表字符集默认使用utf8，必要时候使用utf8mb4
解读：
- 通用，无乱码风险，汉字3字节，英文1字节
- utf8mb4是utf8的超集，有存储4字节例如表情符号时，使用它

禁止使用存储过程，视图，触发器，Event
解读：
- 对数据库性能影响较大，互联网业务，能让站点层和服务层干的事情，不要交到数据库层
- 调试，排错，迁移都比较困难，扩展性较差

禁止在线上环境做数据库压力测试
测试，开发，线上数据库环境必须隔离

## 命名规范
库名，表名，列名必须用小写，采用下划线分隔
解读：abc，Abc，ABC都是给自己埋坑

库名，表名，列名必须见名知义，长度不要超过32字符
解读：tmp，wushan谁TM知道这些库是干嘛的

库备份必须以bak为前缀，以日期为后缀
从库必须以-s为后缀
备库必须以-ss为后缀

## 表设计规范
单实例表个数必须控制在2000个以内
单表分表个数必须控制在1024个以内
表必须有主键，推荐使用UNSIGNED整数为主键
潜在坑：删除无主键的表，如果是row模式的主从架构，从库会挂住

禁止使用外键，如果要保证完整性，应由代码实现
解读：外键使得表之间相互耦合，影响update/delete等SQL性能，有可能造成死锁，高并发情况下容易成为数据库瓶颈

建议将大字段，访问频度低的字段拆分到单独的表中存储，分离冷热数据

## 列设计规范
根据业务区分使用tinyint/int/bigint，分别会占用1/4/8字节
根据业务区分使用char/varchar

解读：
- 字段长度固定，或者长度近似的业务场景，适合使用char，能够减少碎片，查询性能高
- 字段长度相差较大，或者更新较少的业务场景，适合使用varchar，能够减少空间

根据业务区分使用datetime/timestamp
解读：前者占用5个字节，后者占用4个字节，存储年使用YEAR，存储日期使用DATE，存储时间使用datetime

必须把字段定义为NOT NULL并设默认值
解读：
- NULL的列使用索引，索引统计，值都更加复杂，MySQL更难优化
- NULL需要更多的存储空间
- NULL只能采用IS NULL或者IS NOT NULL，而在=/!=/in/not in时有大坑

使用INT UNSIGNED存储IPv4，不要用char(15)

使用varchar(20)存储手机号，不要使用整数
解读：
- 牵扯到国家代号，可能出现+/-/()等字符，例如+86
- 手机号不会用来做数学运算
- varchar可以模糊查询，例如like ‘138%’

使用TINYINT来代替ENUM
解读：ENUM增加新值要进行DDL操作

## 索引规范
唯一索引使用uniq_[字段名]来命名
非唯一索引使用idx_[字段名]来命名
单张表索引数量建议控制在5个以内
解读：
- 互联网高并发业务，太多索引会影响写性能
- 生成执行计划时，如果索引太多，会降低性能，并可能导致MySQL选择不到最优索引
- 异常复杂的查询需求，可以选择ES等更为适合的方式存储

组合索引字段数不建议超过5个
解读：如果5个字段还不能极大缩小row范围，八成是设计有问题

不建议在频繁更新的字段上建立索引
非必要不要进行JOIN查询，如果要进行JOIN查询，被JOIN的字段必须类型相同，并建立索引
解读：踩过因为JOIN字段类型不一致，而导致全表扫描的坑么？

理解组合索引最左前缀原则，避免重复建设索引，如果建立了(a,b,c)，相当于建立了(a), (a,b), (a,b,c)

## SQL规范
禁止使用select *，只获取必要字段
解读：
- select *会增加cpu/io/内存/带宽的消耗
- 指定字段能有效利用索引覆盖
- 指定字段查询，在表结构变更时，能保证对应用程序无影响

insert必须指定字段，禁止使用insert into T values()
解读：指定字段插入，在表结构变更时，能保证对应用程序无影响

隐式类型转换会使索引失效，导致全表扫描

禁止在where条件列使用函数或者表达式
解读：导致不能命中索引，全表扫描

禁止负向查询以及%开头的模糊查询
解读：导致不能命中索引，全表扫描

禁止大表JOIN和子查询
同一个字段上的OR必须改写问IN，IN的值必须少于50个
应用程序必须捕获SQL异常
解读：方便定位线上问题

## 数据库三大范式
- 1NF，即第一范式，指同一列中不能有多个值，(即实体中的某个属性不能有多个值,或者不能有重复的属性)，简而言之，第一范式就是**无重复的列**
- 2NF，第二范式，要求表中的每一行(每一条数据)，必须可以被唯一的区分(即唯一id，主键)
- 3NF，第三范式，要求一个表中不包含，已在其他表中包含的，非主关键字(主键)信息。
如：现有员工表，包含员工的姓名、年龄等，这时不应该将员工所属部门的信息放在员工表中，而应该新建一张部门表，在员工标中通过部门id与部门表对应。

# group by 的简单使用
我们先准备一张表和一些记录
创建学生的成绩表courses:
```
CREATE TABLE `courses` (
	`id` INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '自增id',
	`student` VARCHAR(255) DEFAULT NULL COMMENT '学生',
	`class` VARCHAR(255) DEFAULT NULL COMMENT '课程',
	`score` INT(255) DEFAULT NULL COMMENT '分数',
PRIMARY KEY (`id`),
UNIQUE KEY `course` (`student`, `class`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
该表记录了学生某节课的考试分数。向courses表中插入记录：
```
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('A', 'Math', 90);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('A', 'Chinese', 80);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('A', 'English', 70);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('A', 'History', 80);

INSERT INTO `courses`(`student`, `class`, `score`) VALUES('B', 'Math', 73);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('B', 'Chinese', 60);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('B', 'English', 70);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('B', 'History', 90);

INSERT INTO `courses`(`student`, `class`, `score`) VALUES('C', 'Math', 70);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('C', 'Chinese', 50);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('C', 'English', 20);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('C', 'History', 10);

INSERT INTO `courses`(`student`, `class`, `score`) VALUES('D', 'Math', 53);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('D', 'Chinese', 32);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('D', 'English', 99);
INSERT INTO `courses`(`student`, `class`, `score`) VALUES('D', 'History', 100);
```
使用几个简单的例子看一下group by的作用:
`SELECT * FROM courses GROUP BY class;`

|id|student|class|score|
|-|-|-|-|
|2|A|Chinese|80|
|3|A|English|70|
|4|A|History|80|
|1|A|Math|90|

类似地，我们按照score对记录进行分组：
`SELECT * FROM courses GROUP BY score;`

|id|student|class|score|
|-|-|-|-|
|12|C|History|10|
|11|C|English|20|
|14|D|Chinese|32|
|10|C|Chinese|50|
|13|D|Math|53|
|6|B|Chinese|60|
|3|A|English|70|
|5|B|Math|73|
|2|A|Chinese|80|
|11|A|Math|90|
|15|D|English|99|
|16|D|History|100|

我们甚至可以对多个字段进行group by：
`SELECT * FROM courses GROUP BY class,student;`

|id	|student|class|score|
|-|-|-|-|
|2	|A	|Chinese	|80|
|6	|B	|Chinese	|60|
|10	|C	|Chinese	|50|
|14	|D	|Chinese	|32|
|3	|A	|English	|70|
|7	|B	|English	|70|
|11	|C	|English	|20|
|15	|D	|English	|99|
|4	|A	|History	|80|
|8	|B	|History	|90|
|12	|C	|History	|10|
|16	|D	|History	|100|
|1	|A	|Math	|90|
|5	|B	|Math	|73|
|9	|C	|Math	|70|
|13	|D	|Math	|53|

最后，我们交换字段顺序对记录进行分组：
`SELECT * FROM courses GROUP BY student,class;`

|id	|student|class|	score|
|-|-|-|-|
|2	|A	|Chinese	|80|
|3	|A	|English	|70|
|4	|A	|History	|80|
|1	|A	|Math	|90|
|6	|B	|Chinese	|60|
|7	|B	|English	|70|
|8	|B	|History	|90|
|5	|B	|Math	|73|
|10	|C	|Chinese	|50|
|11	|C	|English	|20|
|12	|C	|History	|10|
|9	|C	|Math	|70|
|14	|D	|Chinese	|32|
|15	|D	|English	|99|
|16	|D	|History	|100
|13	|D	|Math	|53|

这样的结果可能会使人困惑，我们以第一个sql为例，解释下sql执行的过程：
![](mysql基础/1.png)
该sql首先会按照class进行分组得到四张中间表，然后输出的时候将每一个分组的第一个记录组合在一起形成了最终的结果。我们还可以发现，最终的记录是按照class进行排序的。这样的顺序并不可靠，具体形成的原因恐怕需要在Mysql的底层原理中找到答案。

## 其他用法
- 与order by结合在一起使用
我们需要学生的成绩表，且每个学生每科的成绩按照由大到小的顺序排列
我们可以很自然的写出下面的sql:
`SELECT * FROM courses GROUP BY student,class ORDER BY score DESC;`
然而，执行的结果貌似并不是我们想要的：

|id	|student	|class	|score|
|-|-|-|-|
|16	|D	|History	|100|
|15	|D	|English	|99|
|1	|A	|Math	|90|
|8	|B	|History	|90|
|2	|A	|Chinese	|80|
|4	|A	|History	|80|
|5	|B	|Math	|73|
|9	|C	|Math	|70|
|3	|A	|English	|70|
|7	|B	|English	|70|
|6	|B	|Chinese	|60|
|13	|D	|Math	|53|
|10	|C	|Chinese	|50|
|14	|D	|Chinese	|32|
|11	|C	|English	|20|
|12	|C	|History	|10|

通过观察，我们可以发现，事实上，这个sql是将所有的记录按照score由大到小的顺序排列了，为什么会出现这样的结果呢？
事实上，这个取决于整个sql的执行顺序，真正的执行顺序是 from ... where ... group by ... order by ... select，order by 作用在整个记录，而不是每个分组上。
那么，怎么样能够得到我们期望的结果呢？这里给出我的sql实现：
`SELECT * FROM courses GROUP BY student,class ORDER BY student,score DESC;`

|id	|student	|class	|score|
|-|-|-|-|
|1	|A	|Math	|90|
|2	|A	|Chinese	|80|
|4	|A	|History	|80|
|3	|A	|English	|70|
|8	|B	|History	|90|
|5	|B	|Math	|73|
|7	|B	|English	|70|
|6	|B	|Chinese	|60|
|9	|C	|Math	|70|
|10	|C	|Chinese	|50|
|11	|C	|English	|20|
|12	|C	|History	|10|
|16	|D	|History	|100|
|15	|D	|English	|99|
|13	|D	|Math	|53|
|14	|D	|Chinese	|32|

- 与having结合在一起使用
我们需要得到所有功课平均分达到60分的同学和他们的均分：
```
SELECT `student`, AVG(`score`) AS`avg_score`
FROM `courses`
GROUP BY `student`
HAVING AVG(`score`) >= 60
ORDER BY `avg_score` DESC;
```

|student	|avg_score|
|-|-|
|A	|80.0000|
|B	|73.2500|
|D	|71.0000|
这里需要注意一个问题：where 与 having的区别。where作用于所有的记录，而having则作用于一个分组。
举例说明：
假设我们这里需要得到所有功课（除历史课）平均分达到60分的同学和他们的均分：
```
SELECT `student`, AVG(`score`) AS `avg_score`
FROM `courses`
WHERE `class` <> 'History'
GROUP BY `student`
HAVING AVG(`score`) >= 60
ORDER BY `avg_score` DESC;
```
|student|avg_score|
|-|-|
|A	|80.0000|
|B	|67.6667|
|D	|61.3333|

- Group By与Limit
我们需要列出均分最高的三门课：
```
SELECT `class`, AVG(`score`) AS `avg_score`
FROM `courses`
GROUP By `class`
ORDER BY `avg_score` DESC
LIMIT 3;
```

|class|avg_score|
|-|-|
|Math|71.5000|
|History|70.0000|
|English|64.7500|

我们需要理解的是：group by分组的依据，以及where过滤条件作用的粒度。

## group by 练习
- 获取所有课程
我们的第一反应是可以使用distinct关键字实现
`SELECT DISTINCT(class) FROM courses;`

|class|
|-|
|Chinese|
|English|
|History|
|Math|

事实上，我们同样可以使用group by来实现
`SELECT class FROM courses GROUP BY class;`
结果与上面相同。因此，我们可以发现group by的一个重要的作用就是枚举出所有满足group by条件的组合，间接达到了去重的效果。

- 所有的学生对应修读的课程
```
SELECT `student`,`class` 
FROM `courses` 
GROUP BY `student`, `class`; 
```
|student	|class|
|-|-|
|A	|Chinese|
|A	|English|
|A	|History|
|A	|Math|
|B	|Chinese|
|B	|English|
|B	|History|
|B	|Math|
|C	|Chinese|
|C	|English|
|C	|History|
|C	|Math|
|D	|Chinese|
|D	|English|
|D	|History|
|D	|Math|

- 每门课程的排名表
```
SELECT `class`, `student`, `score`
FROM `courses`
ORDER BY `class`, `score` DESC;
```
|class	|student	|score|
|-|-|-|
|Chinese	|A	|80
|Chinese	|B	|60|
|Chinese	|C	|50|
|Chinese	|D	|32|
|English	|D	|99|
|English	|A	|70|
|English	|B	|70|
|English	|C	|20|
|History	|D	|100|
|History	|B	|90|
|History	|A	|80|
|History	|C	|10|
|Math	|A	|90|
|Math	|B	|73|
|Math	|C	|70|
|Math	|D	|53|

- 参考每门课的最高分选择课代表
```
SELECT `c`.`class`, `c`.`student`, `c`.`score`
FROM (
SELECT `class`, MAX(`score`) AS `score`
FROM `courses`
GROUP BY `class`
)`T`
INNER JOIN `courses` `c` ON `T`.`class` = `c`.`class` AND `T`.`score` = `c`.`score`;
```

|class	|student	|score|
|-|-|-|
|Math	|A	|90|
|Chinese	|A	|80|
|English	|D	|99|
|History	|D	|100|

- 为每门课的前三名颁奖
我们容易写出这样的一个sql
```
SELECT `class`, `student`
FROM `courses`
GROUP BY `class`, `student`
ORDER BY `class`, `score`
LIMIT 3;
```
很显然，这并不是我们想要的结果，至于为什么结果是这个样子的，大家可以参考上面的order by，这里不再赘述。
正确的sql为
```
SELECT `c1`.`class`, `c1`.`student`, `c1`.`score`
FROM `courses` `c1`
  INNER JOIN `courses` `c2` ON `c1`.`class` = `c2`.`class` AND `c1`.`student` <> `c2`.`student` AND `c1`.`score` > `c2`.`score`
GROUP BY `c1`.`class`, `c1`.`student`
ORDER BY `c1`.`class`, `c1`.`score` DESC;
```
再提供一种通用的解法
```
SELECT `c1`.`class`, `c1`.`student`, `c1`.`score`
FROM `courses` `c1`
INNER JOIN `courses` `c2` ON `c1`.`class` = `c2`.`class` AND `c1`.`score` <= `c2`.`score`
GROUP BY `c1`.`class`, `c1`.`student`
HAVING COUNT(*) <= 3
ORDER BY `c1`.`class`, `c1`.`score` DESC;
```
上述两个sql执行的结果均为

|class	|student	|score|
|-|-|-|
|Chinese	|A	|80|
|Chinese	|B	|60|
|Chinese	|C	|50|
|English	|D	|99|
|English	|A	|70|
|English	|B	|70|
|History	|D	|100|
|History	|B	|90|
|History	|A	|80|
|Math	|A	|90|
|Math	|B	|73|
|Math	|C	|70|
两个sql中为什么一个条件是c1.score>c2.score而另一个是c1.score<=c2.score，以及count(*)<=3而不是count(*)<3,可以细细体会下。

- 了解每个学生最擅长的科目和最不擅长的科目
```
SELECT `T`.`student`, `c`.`class` AS `best_class`, `T`.`max_score` AS `max_score`, `c2`.`class` AS `worst_class`, `T`.`min_score`
FROM (
  SELECT `student`, MAX(`score`) AS `max_score`, MIN(`score`) AS `min_score`
  FROM `courses`
  GROUP BY `student`
) `T`
INNER JOIN `courses` `c` ON `T`.`student` = `c`.`student` AND `c`.`score` = `T`.`max_score`
INNER JOIN `courses` `c2` ON `T`.`student` = `c2`.`student` AND `c2`.`score` = `T`.`min_score`;
```

|student	|best_class	|max_score	|worst_class	|min_score|
|-|-|-|-|-|
|A	|Math	|90	|English	|70|
|B	|History	|90	|Chinese	|60|
|C	|Math	|70	|History	|10|
|D	|History	|100	|Chinese	|32|

[参考](https://www.cnblogs.com/hhe0/p/9556070.html)
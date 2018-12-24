---
title: mysql基础
tags: mysql
date: 2018-12-06 18:12:58
---

MySQL基础

## 1.增删改查
```
插入(增)：
insert into 表名字(列名1，列名2...) values(值1，值2...)
或者：
insert into 表名字 values(值1，值2...)
若不指定列名，则需要插入全部字段。
```

```
删除：
delete from 表名字
若不加where条件，则删除数据表中所有数据
```

```
修改：
update 表名字 set 列1=值1，列2=值2...
若不加where条件，则更新数据表中所有数据
```

```
查询：
select * from 表名字
星号代表查询数据表中所有字段
select id,stu_name,age, from 表名字
查询多个字段，用逗号进行分割
```

常用数据类型
```
int 整形
double 浮点型
datetime 日期型
varchar 字符型(要给长度，如20，即20个字符)
```	

1.创建数据库:
`create database[if not exists] db_name`

2.删除数据库:
`drop database[if exists] db_name`

3.创建数据表:
```
create table[if not exists] table_name(
	列名 类型 是否为空
	列名 类型 是否为空
)
```

## 2.主外键
主键：primary_key
	1.保证数据唯一性
	2.一个数据表只能有一个主键
	3.主键不可以为空，并且不可以重复
	
  外键:
	1.保证数据的完整性，一致性
	2.外键列和参照列的数据类型必须一致
	3.参照列的表为父表，外键列的表为子表
	4.实现一对多，多对一的关系
	5.外键列和参照列必须创建索引，如不创建，mySQL会自动创建
```
create table test(
	id int primary_key auto_increment,
	foreign key(test1_id) references  test2(id)
	//外键列为test1_id(当前表的列),参照了test2表的id列
)
```
尽量不要使用外键。

## 3.修改表sql
修改数据表
```
ALTER TABLE <表名>
[ALTER COLUMN <列名> <新数据类型>]
|[ADD <列名> <数据类型> [属性]
|[DROP COLUMN <列名>]
|[ADD [constraint] [约束名] 约束定义
|[DROP [constraint] 约束名]
```
```
create table test(
	id int PRIMARY key auto_increment, //创建id列，规定为主键列，自增
	test_name varchar(20),
	test2_id int
)
```

新增字段
`alter table wd_user add account_status varchar(255) not null default 0;`

修改字段
`ALTER TABLE wd_evaluate MODIFY cus_id VARCHAR(200) NULL DEFAULT null COMMENT '定制表id';`

查看表结构
`desc tabl_name;`

## 4.where条件
可以跟在select/update/delete后
多个where条件，并且的关系，用and链接，或者的关系，用or链接

group by:把查询结构进行分组，分组结果可以升序(asc)或降序(desc),默认升序,用于去重

having:如果要将group by的结果进行删选，需要用到having关键字，不可以用where

order by:把查询结果进行排序，可升(asc)可降(desc),默认升序，可以进行多次排序，用逗号分割排序字段

limit:限制查询返回的结果(用于分页功能)是select语句的最后一个关键字
	参数1：从第几条结果开始获取
	参数2：获取几条结果

select语句结构：	
```
select * from 表名字
	where
	 gruop by
	  having
	   order by
	    limit
```
关键字顺序不可以颠倒
    
## 5.字符函数
 以下字符函数都必须跟在select语句后面
`concat(str1,str2,...)：拼接字符串`

`left(str,1):左侧开始截取字符串，参数2为截取几位`

`right(str,1):右侧开始截取，参数2位截取几位`

`substring(参数1，参数2，参数3):1.要截取的字符串2.从第几位开始截取3.截取的长度`

`replace(参数1，参数2，参数3):1.要处理的字符串2.要替换的字符3.替换成什么`
	
`trim(str):去除字符串前后的空格`

`length(str):字符串长度`

`lower(str):字符串转小写`

`upper(str):字符串转大写`

	
问题：
已知一个数字字段，内容不规律，最长有10位，最短有4位，
想把不够10位的全部在前边用0补齐，怎么办?

SQL：
update 表名字 set num = right(concat('000000',num),10);	
解:因为最短有4位，在每个字符串左侧都拼接6个0，然后从右侧开始截取10个字符


## 6.聚合函数
聚合函数必须跟在select语句后面
`一列的和：select sum(列名) from 表名 where ...条件`

`一列的最大值：select max(列名) from 表名 where ...条件`

`一列的最小值：select min(列名) from 表名 where ...条件`

`一列的数量：select count(列名/一般都用id) from 表名 where ...条件`

`一列的平均值：select avg(列名) from 表名 where ...条件`
	 
## 7. case when
```
select case(
	when sex='0' then '女'
	when sex='1' then '男'
	end
)
from 表名
```
case when方法，该方法用于查询时替换表中数据，
如性别字段，里面有0和1，
查询时可分别把0和1替换成女和男

## 8.like(模糊查询)
```
select * from student where stu_name like '%小%';
%表示可随意替换
小%  表示必须以小开头，后边随意
%小  表示必须以小结尾，前边随意
%小% 表示前后都随意，中间包含小
```
## 9.数值函数
```
ceil():向上取整
floor():向下取整
round():四舍五入
power():幂运算
```	

以下方法必须跟在where关键字后面，作为where的修饰
[not] between ... and ...
区间范围，属于where关键字,可用大于号和小于号来实现相同的功能

[not] in(1,2,3,4,5)
判断一个字段的值是否与括号内的值相同，
括号内是or的关系，即满足任何一个条件，都会显示出来

is [not] null:判断字段是否为空

## 10..时间函数
now()：当前时间(年月日  时分秒)

curdate()：当前日期(年月日)

curtime()：当前时间(时分秒)

date_add：日期加减算法
参数1：要进行运算的日期
参数2：加或减的量
参数3：加或减的日期单位(小时，天，周，月，年)

datediff()：日期差值计算
参数1：日期类型
参数2：日期类型
		
date_format():日期格式化
将指定日期，安装指定规则进行格式化(具体请百度该函数)
date_format('2017-1-1','%y/%m/%d/')--->2017/1/1

## 11.表连接
1.查询结果在哪个表
2.链接哪个表
3.链接条件
4.where条件
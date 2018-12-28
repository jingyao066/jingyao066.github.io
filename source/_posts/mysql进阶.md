---
title: mysql进阶-视图
tags: mysql
date: 2018-12-26 18:27:30
---

## 视图
概念：
视图就是一条SELECT语句执行后返回的结果集。
视图（view）是一种虚拟存在的表，是一个逻辑表，本身并不包含数据。作为一个select语句保存在数据字典中的。

所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。

### 特性
视图是对若干张基本表的引用，一张虚表，查询语句执行的结果，不存储具体的数据(基本表数据发生了改变，视图也会跟着改变)
可以跟基本表一样，进行增删改查操作(ps:增删改操作有条件限制)；

### 作用
1.方便操作，特别是查询操作，减少复杂的sql语句，增强可读性。
2.更加安全，数据库授权命令不能限定到特定行和特定列，但是通过合理创建视图，就可以把权限限定到行列级别。
总而言之，使用视图的大部分情况是为了保障数据安全性，提高查询效率。

### 使用场景
1.权限控制的时候，不希望用户访问表中某些包含敏感信息的列，如salary...
2.信息来源于多张表关联，可以创建视图获取信息，省的每次都得重新写sql。

### 语法
语法：
```
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    VIEW view_name [(column_list)]
    AS select_statement
   [WITH [CASCADED | LOCAL] CHECK OPTION]
```

1.OR REPLACE：表示替换已有视图
2.ALGORITHM：表示视图选择算法，默认算法是UNDEFINED(未定义的)：MySQL自动选择要使用的算法 ；merge合并；temptable临时表
3.select_statement：表示select语句
4.[WITH [CASCADED | LOCAL] CHECK OPTION]：表示视图在更新时保证在视图的权限范围之内。
	cascade是默认值，表示更新视图的时候，要满足视图和表的相关条件。
	local表示更新视图的时候，要满足该视图定义的一个条件即可。
TIPS：推荐使用WHIT [CASCADED|LOCAL] CHECK OPTION选项，可以保证数据的安全性。

基本格式：
```
create view <视图名称>[(column_list)]
    as select语句
with check option;
```

### 创建单表视图
```
mysql> create view v_F_players(编号,名字,性别,电话)
    -> as
    -> select PLAYERNO,NAME,SEX,PHONENO from PLAYERS
    -> where SEX='F'
    -> with check option;
Query OK, 0 rows affected (0.00 sec)

mysql> desc v_F_players;
+--------+----------+------+-----+---------+-------+
| Field  | Type     | Null | Key | Default | Extra |
+--------+----------+------+-----+---------+-------+
| 编号    | int(11)  | NO   |     | NULL    |       |
| 名字    | char(15) | NO   |     | NULL    |       |
| 性别    | char(1)  | NO   |     | NULL    |       |
| 电话    | char(13) | YES  |     | NULL    |       |
+--------+----------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql> select * from  v_F_players;
+--------+-----------+--------+------------+
| 编号    | 名字      | 性别    | 电话        |
+--------+-----------+--------+------------+
|      8 | Newcastle | F      | 070-458458 |
|     27 | Collins   | F      | 079-234857 |
|     28 | Collins   | F      | 010-659599 |
|    104 | Moorman   | F      | 079-987571 |
|    112 | Bailey    | F      | 010-548745 |
+--------+-----------+--------+------------+
5 rows in set (0.02 sec)
```

### 创建多表视图
```
mysql> create view v_match
    -> as 
    -> select a.PLAYERNO,a.NAME,MATCHNO,WON,LOST,c.TEAMNO,c.DIVISION
    -> from 
    -> PLAYERS a,MATCHES b,TEAMS c
    -> where a.PLAYERNO=b.PLAYERNO and b.TEAMNO=c.TEAMNO;
Query OK, 0 rows affected (0.03 sec)

mysql> select * from v_match;
+----------+-----------+---------+-----+------+--------+----------+
| PLAYERNO | NAME      | MATCHNO | WON | LOST | TEAMNO | DIVISION |
+----------+-----------+---------+-----+------+--------+----------+
|        6 | Parmenter |       1 |   3 |    1 |      1 | first    |
|       44 | Baker     |       4 |   3 |    2 |      1 | first    |
|       83 | Hope      |       5 |   0 |    3 |      1 | first    |
|      112 | Bailey    |      12 |   1 |    3 |      2 | second   |
|        8 | Newcastle |      13 |   0 |    3 |      2 | second   |
+----------+-----------+---------+-----+------+--------+----------+
5 rows in set (0.04 sec)
```
1.如果创建视图时不明确指定视图的列名，那么列名就和定义视图的select子句中的列名完全相同。
2.如果显式的指定视图的列名就按照指定的列名。
注意：显示指定视图列名，要求视图名后面的列的数量必须匹配select子句中的列的数量。

### 查看视图
语法：
`show create view`

视图一旦创建完毕，就可以像一个普通表那样使用，视图主要用来查询
`select * from view_name;`

有关视图的信息记录在information_schema数据库中的views表中
```
select * from information_schema.views where TABLE_NAME='v_F_players';
```

### 视图的更改
#### 语法：
`CREATE OR REPLACE VIEW`

基本格式：
`create or replace view view_name as select语句;`
在视图存在的情况下可对视图进行修改，视图不在的情况下可创建视图

ALTER语句修改视图
```
ALTER
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = { user | CURRENT_USER }]
    [SQL SECURITY { DEFINER | INVOKER }]
VIEW view_name [(column_list)]
AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```
注意：修改视图是指修改数据库中已存在的表的定义，当基表的某些字段发生改变时，可以通过修改视图来保持视图和基本表之间一致

#### DML操作更新视图
DML:Data Manipulation Language 数据操纵语言，，以INSERT、UPDATE、DELETE三种指令为核心，也可加上select。CRUD
因为视图本身没有数据，因此对视图进行的dml操作最终都体现在基表中。
```
mysql> create view v_student as select * from student;

mysql> select * from v_student;
+--------+--------+------+
| 学号    | name   | sex  |
+--------+--------+------+
|      1 | 张三    | M    |
|      2 | 李四    | F    |
|      5 | 王五    | NULL |
+--------+--------+------+

mysql> update v_student set name='钱六' where 学号='1';

mysql> select * from student;
+--------+--------+------+
| 学号    | name   | sex  |
+--------+--------+------+
|      1 | 钱六    | M    |
|      2 | 李四    | F    |
|      5 | 王五    | NULL |
+--------+--------+------+
```

注意，不是所有视图都可以做dml操作。有下列内容之一，视图不能做DML操作：
```
①select子句中包含distinct

②select子句中包含组函数

③select语句中包含group by子句

④select语句中包含order by子句

⑤select语句中包含union 、union all等集合运算符

⑥where子句中包含相关子查询

⑦from子句中包含多个表

⑧如果视图中有计算列，则不能更新

⑨如果基表中有某个具有非空约束的列未出现在视图定义中，则不能做insert操作
```

#### 删除视图
删除视图是指删除数据库中已存在的视图，删除视图时，只能删除视图的定义，不会删除数据，也就是说不动基表，语法如下：
```
DROP VIEW [IF EXISTS]   
view_name [, view_name] ...
```

基本格式：
` drop view v_student;`
如果视图不存在，则抛出异常；使用IF EXISTS选项使得删除不存在的视图时不抛出异常。

### 使用WITH CHECK OPTION约束 
对于可以执行DML操作的视图，定义时可以带上WITH CHECK OPTION约束

作用：
对视图所做的DML操作的结果，不能违反视图的WHERE条件的限制。

示例：创建视图，包含1960年之前出生的所有球员（老兵）
```
mysql> create view v_veterans
    -> as
    -> select * from PLAYERS
    -> where birth_date < '1960-01-01'
    -> with check option;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from v_veterans;
+----------+---------+----------+------------+-----+--------+----------------+---------+----------+-----------+------------+----------+
| PLAYERNO | NAME    | INITIALS | BIRTH_DATE | SEX | JOINED | STREET         | HOUSENO | POSTCODE | TOWN      | PHONENO    | LEAGUENO |
+----------+---------+----------+------------+-----+--------+----------------+---------+----------+-----------+------------+----------+
|        2 | Everett | R        | 1948-09-01 | M   |   1975 | Stoney Road    | 43      | 3575NH   | Stratford | 070-237893 | 2411     |
|       39 | Bishop  | D        | 1956-10-29 | M   |   1980 | Eaton Square   | 78      | 9629CD   | Stratford | 070-393435 | NULL     |
|       83 | Hope    | PK       | 1956-11-11 | M   |   1982 | Magdalene Road | 16A     | 1812UP   | Stratford | 070-353548 | 1608     |
+----------+---------+----------+------------+-----+--------+----------------+---------+----------+-----------+------------+----------+
3 rows in set (0.02 sec)
```

此时，使用update对视图进行修改：
```
mysql> update v_veterans
    -> set BIRTH_DATE='1970-09-01'
    -> where PLAYERNO=39;
ERROR 1369 (HY000): CHECK OPTION failed 'TENNIS.v_veterans'
```
因为违反了视图中的WHERE birth_date < '1960-01-01'子句，所以抛出异常；

利用with check option约束限制，保证更新视图是在该视图的权限范围之内。

#### 嵌套视图
定义在另一个视图的上面的视图
```
mysql> create view v_ear_veterans
    -> as
    -> select * from v_veterans
　　-> where JOINED < 1980;
```
使用WITH CHECK OPTION约束时，(不指定选项则默认是CASCADED)

可以使用CASCADED或者 LOCAL选项指定检查的程度：

　　①WITH CASCADED CHECK OPTION：检查所有的视图

　　　　例如：嵌套视图及其底层的视图

　　②WITH LOCAL CHECK OPTION：只检查将要更新的视图本身

　　　　对嵌套视图不检查其底层的视图　

### 定义视图时的其他选项
```
CREATE [OR REPLACE]   
　　[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]  
　　[DEFINER = { user | CURRENT_USER }]  
　　[SQL SECURITY { DEFINER | INVOKER }]
VIEW view_name [(column_list)]  
AS select_statement  
　　[WITH [CASCADED | LOCAL] CHECK OPTION]
```

1、ALGORITHM选项：选择在处理定义视图的select语句中使用的方法

　　①UNDEFINED：MySQL将自动选择所要使用的算法

　　②MERGE：将视图的语句与视图定义合并起来，使得视图定义的某一部分取代语句的对应部分

　　③TEMPTABLE：将视图的结果存入临时表，然后使用临时表执行语句

缺省ALGORITHM选项等同于ALGORITHM = UNDEFINED

2、DEFINER选项：指出谁是视图的创建者或定义者

　　①definer= '用户名'@'登录主机'

　　②如果不指定该选项，则创建视图的用户就是定义者，指定关键字CURRENT_USER(当前用户)和不指定该选项效果相同

3、SQL SECURITY选项：要查询一个视图，首先必须要具有对视图的select权限。

　　但是，如果同一个用户对于视图所访问的表没有select权限，那会怎么样？

SQL SECURITY选项决定执行的结果：

　　①SQL SECURITY DEFINER：定义(创建)视图的用户必须对视图所访问的表具有select权限，也就是说将来其他用户访问表的时候以定义者的身份，此时其他用户并没有访问权限。

　　②SQL SECURITY INVOKER：访问视图的用户必须对视图所访问的表具有select权限。

缺省SQL SECURITY选项等同于SQL SECURITY DEFINER　

**视图权限总结：**
使用root用户定义一个视图(推荐使用第一种)：u1、u2
1）u1作为定义者定义一个视图，u1对基表有select权限，u2对视图有访问权限：u2是以定义者的身份访问可以查询到基表的内容；

2）u1作为定义者定义一个视图，u1对基表没有select权限，u2对视图有访问权限，u2对基表有select权限：u2访问视图的时候是以调用者的身份，此时调用者是u2，可以查询到基表的内容。

### 视图查询语句的处理
示例：所有有罚款的球员的信息

创建视图：
```
mysql> create view cost_raisers
    -> as
    -> select * from PLAYERS
    -> where playerno in (select playerno from PENALTIES);
```

查询视图：
```
mysql> select playerno from cost_raisers
    -> where town='Stratford';
+----------+
| PLAYERNO |
+----------+
|        6 |
+----------+ 
```

1、替代方法：

　　先把select语句中的视图名使用定义视图的select语句来替代；

　　再处理所得到的select语句。
```
mysql> select playerno from
　　 -> (
    ->   select * from PLAYERS
    ->   where playerno in->   　　(select playerno from PENALTIES)
    -> )as viewformula
    -> where town='Stratford';
+----------+
| PLAYERNO |
+----------+
|        6 |
+----------+
```

2、具体化方法：
先处理定义视图的select语句，这会生成一个中间的结果集；
然后，再在中间结果上执行select查询。

`mysql> select <列名> from <中间结果>;`

---
title: mysql进阶
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
语法：
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

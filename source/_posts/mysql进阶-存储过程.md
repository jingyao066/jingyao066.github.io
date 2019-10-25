---
title: mysql进阶-存储过程
tags: mysql
date: 2018-12-28 20:08:17
---

# 概述
SQL语句需要先编译然后执行，而存储过程（Stored Procedure）是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。
存储过程是可编程的函数，在数据库中(数据字典表)创建并保存，可以由SQL语句和控制结构组成。当想要在不同的应用程序或平台上执行相同的函数，或者封装特定功能时，存储过程是非常有用的。数据库中的存储过程可以看做是对编程中面向对象方法的模拟，它允许控制数据的访问方式。

# 优点
- 增强SQL语言的功能和灵活性：将重复性很高的一些操作，封装到一个存储过程中，简化了对这些SQL的调用
- 组件式编程：存储过程被创建后，可以在程序中被多次调用，而不必重新编写该存储过程的SQL语句。而且数据库专业人员可以随时对存储过程进行修改，对应用程序源代码毫无影响。
- 较快的执行速度：存储过程是预编译的。在首次运行一个存储过程时查询，优化器对其进行分析优化，并且给出最终被存储在系统表中的执行计划。而批处理的Transaction-SQL语句在每次运行时都要进行编译和优化，速度相对要慢一些。
- 减少网络流量：针对同一个数据库对象的操作（如查询、修改），如果这一操作所涉及的Transaction-SQL语句被组织进存储过程，那么当在客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而大大减少网络流量并降低了网络负载。
- 保证数据安全：通过对执行某一存储过程的权限进行限制，能够实现对相应的数据的访问权限的限制，避免了非授权用户对数据的访问，保证了数据的安全。

# 创建存储过程
完整语法：
```
CREATE
    [DEFINER = { user | CURRENT_USER }]
　PROCEDURE sp_name ([proc_parameter[,...]])
    [characteristic ...] routine_body

proc_parameter:
    [ IN | OUT | INOUT ] param_name type

characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }

routine_body:
　　Valid SQL routine statement

[begin_label:] BEGIN
　　[statement_list]
　　　　……
END [end_label]
```

简单语法：
```
CREATE 
	PROCEDURE  过程名
	([[IN|OUT|INOUT] 参数名 数据类型[,[IN|OUT|INOUT] 参数名 数据类型…]])
	[特性 ...]
	BEGIN
		过程体
	END
```

存储过程传入表名，mysql是不支持表名，列名做参数的。只能采用动态拼接的方法来生成sql语句。
参考：
```
DROP PROCEDURE IF EXISTS `pagePro`;
CREATE DEFINER = `root`@`localhost` PROCEDURE `pagePro`(in pageNo int,in pageSize int,in tableName varchar(50))
BEGIN
	DECLARE startIndex INT;
	set startIndex = pageSize * (pageNo-1);
	set @s = concat("select * from ",tableName," LIMIT ",startIndex,",",pageSize);
	prepare stmt from @s;
	execute stmt;
	DEALLOCATE prepare stmt;
END;
```

## 存储过程体
存储过程体包含了在过程调用时必须执行的语句，例如：dml、ddl语句，if-then-else和while-do语句、声明变量的declare语句等
过程体格式：以begin开始，以end结束(可嵌套)
```
BEGIN
　　BEGIN
　　　　BEGIN
　　　　　　statements; 
　　　　END
　　END
END
```
注意：每个嵌套块及其中的每条语句，必须以分号结束，表示过程体结束的begin-end块(又叫做复合语句compound statement)，则不需要分号。

## 为语句块贴标签
```
[begin_label:] BEGIN
　　[statement_list]
END [end_label]
```

示例：
```
label1: BEGIN
　　label2: BEGIN
　　　　label3: BEGIN
　　　　　　statements; 
　　　　END label3 ;
　　END label2;
END label1
```
标签的作用：
①增强代码的可读性
②在某些语句(例如:leave和iterate语句)，需要用到标签


简单示例：
```
DELIMITER //
  CREATE PROCEDURE myproc(OUT s int)
    BEGIN
      SELECT COUNT(*) INTO s FROM students;
    END
    //
DELIMITER;
```
## 分隔符
MySQL默认以";"为分隔符，如果没有声明分割符，则编译器会把存储过程当成SQL语句进行处理，因此编译过程会报错，所以要事先用“DELIMITER //”声明当前段分隔符，让编译器把两个"//"之间的内容当做存储过程的代码，不会执行这些代码；“DELIMITER ;”的意为把分隔符还原。

## 参数
存储过程根据需要可能会有输入、输出、输入输出参数，如果有多个参数用","分割开。MySQL存储过程的参数用在存储过程的定义，共有三种参数类型,IN,OUT,INOUT:

IN参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值
OUT：该值可在存储过程内部被改变，并可返回
INOUT：调用时指定，并且可被改变和返回

## 过程体
过程体的开始与结束使用BEGIN与END进行标识。

## in参数例子
```
DELIMITER //
  CREATE PROCEDURE in_param(IN p_in int)
    BEGIN
    SELECT p_in;
    SET p_in=2;
    SELECT p_in;
    END;
    //
DELIMITER ;
#调用
SET @p_in=1;
CALL in_param(@p_in);
SELECT @p_in;
```
结果：
```
p_in
1

p_in
2

@p_in
1
```
以上可以看出，p_in虽然在存储过程中被修改，但并不影响@p_id的值，因为前者为局部变量、后者为全局变量。

## OUT参数例子
```
#存储过程OUT参数
DELIMITER //
  CREATE PROCEDURE out_param(OUT p_out int)
    BEGIN
      SELECT p_out;
      SET p_out=2;
      SELECT p_out;
    END;
    //
DELIMITER ;
#调用
SET @p_out=1;
CALL out_param(@p_out);
SELECT @p_out;
```
结果：
```
p_out
(null)
#因为out是向调用者输出参数，不接收输入的参数，所以存储过程里的p_out为null

p_out
2

@p_out
2
#调用了out_param存储过程，输出参数，改变了p_out变量的值
```

## INOUT参数例子
```
#存储过程INOUT参数
DELIMITER //
  CREATE PROCEDURE inout_param(INOUT p_inout int)
    BEGIN
      SELECT p_inout;
      SET p_inout=2;
      SELECT p_inout;
    END;
    //
DELIMITER ;
#调用
SET @p_inout=1;
CALL inout_param(@p_inout) ;
SELECT @p_inout;
```
结果：
```
p_inout
1

p_inout
2

@p_inout
2
```

注意1：
①如果过程没有参数，也必须在过程名后面写上小括号
例：CREATE PROCEDURE sp_name ([proc_parameter[,...]]) ……
②确保参数的名字不等于列的名字，否则在过程体中，参数名被当做列名来处理

注意2：
输入值使用in参数；
返回值使用out参数；
inout参数就尽量的少用。


# 变量
语法：
`DECLARE 变量名1[,变量名2...] 数据类型 [默认值];`
数据类型为MySQL的数据类型。

## 变量赋值
语法：
`SET 变量名 = 变量值 [,变量名= 变量值 ...]`

## 用户变量
用户变量一般以@开头，请规范命名用户变量

### 示例
1.
```
SELECT 'Hello World' into @x;
SELECT @x;
```

示例2：
```
SET @y='Goodbye Cruel World';
SELECT @y;
```

示例3：
```
SET @z=1+2+3;
SELECT @z;
```

### 在存储过程中使用用户变量
```
CREATE PROCEDURE GreetWorld() SELECT CONCAT(@greeting,' World');
SET @greeting='Hello';
CALL GreetWorld();
```
结果：
`Hello World`

### 在存储过程间传递全局范围的用户变量
```
CREATE PROCEDURE p1() SET @last_proc='p1';
CREATE PROCEDURE p2() SELECT CONCAT('Last procedure was ',@last_proc);
CALL p1();
CALL p2();
```
结果：
`Last procedure was p1`

# 注释
MySQL存储过程可使用两种风格的注释：

双杠：--，该风格一般用于单行注释
C风格： 一般用于多行注释

# 调用
用call和你过程名以及一个括号，括号里面根据需要，加入参数，参数包括输入参数、输出参数、输入输出参数。

# 存储过程的查询
查询存储过程:
```
SELECT name FROM mysql.proc WHERE db='数据库名';
SELECT routine_name FROM information_schema.routines WHERE routine_schema='数据库名';
SHOW PROCEDURE STATUS WHERE db='数据库名';
```

查看存储过程详细信息:
`SHOW CREATE PROCEDURE 数据库.存储过程名;`

# 存储过程的修改
```
ALTER {PROCEDURE | FUNCTION} sp_name [characteristic ...]
characteristic:
{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
| SQL SECURITY { DEFINER | INVOKER }
| COMMENT 'string'
```
.sp_name参数表示存储过程或函数的名称；
.characteristic参数指定存储函数的特性。
.CONTAINS SQL表示子程序包含SQL语句，但不包含读或写数据的语句；
.NO SQL表示子程序中不包含SQL语句；
.READS SQL DATA表示子程序中包含读数据的语句；
.MODIFIES SQL DATA表示子程序中包含写数据的语句。
.SQL SECURITY { DEFINER | INVOKER }指明谁有权限来执行，DEFINER表示只有定义者自己才能够执行；INVOKER表示调用者可以执行。
.COMMENT 'string'是注释信息。

示例：
```
#将读写权限改为MODIFIES SQL DATA，并指明调用者可以执行。
ALTER  PROCEDURE  num_from_employee
  MODIFIES SQL DATA
  SQL SECURITY INVOKER ;
#将读写权限改为READS SQL DATA，并加上注释信息'FIND NAME'。
ALTER  PROCEDURE  name_from_employee
  READS SQL DATA
  COMMENT 'FIND NAME';
```

# 删除存储过程
`DROP PROCEDURE [过程1[,过程2…]]`
从MySQL的表格中删除一个或多个存储过程。

# 存储过程的控制语句
## 变量作用域
内部变量在其作用域范围内享有更高的优先权，当执行到end时，内部变量消失，不再可见了，在存储过程外再也找不到这个内部变量，但是可以通过out参数或者将其值指派给会话变量来保存其值。
```
DELIMITER //
  CREATE PROCEDURE proc()
    BEGIN
      DECLARE x1 VARCHAR(5) DEFAULT 'outer';
        BEGIN
          DECLARE x1 VARCHAR(5) DEFAULT 'inner';
          SELECT x1;
        END;
      SELECT x1;
    END;
    //
DELIMITER ;
#调用
CALL proc();
```
结果：
`inner`


## 条件语句
IF-THEN-ELSE语句
```
#条件语句IF-THEN-ELSE
DROP PROCEDURE IF EXISTS proc3;
DELIMITER //
CREATE PROCEDURE proc3(IN parameter int)
  BEGIN
    DECLARE var int;
    SET var=parameter+1;
    IF var=0 THEN
      INSERT INTO t VALUES (17);
    END IF ;
    IF parameter=0 THEN
      UPDATE t SET s1=s1+1;
    ELSE
      UPDATE t SET s1=s1+2;
    END IF ;
  END ;
  //
DELIMITER ;
```

CASE-WHEN-THEN-ELSE语句
```
#CASE-WHEN-THEN-ELSE语句
DELIMITER //
  CREATE PROCEDURE proc4 (IN parameter INT)
    BEGIN
      DECLARE var INT;
      SET var=parameter+1;
      CASE var
        WHEN 0 THEN
          INSERT INTO t VALUES (17);
        WHEN 1 THEN
          INSERT INTO t VALUES (18);
        ELSE
          INSERT INTO t VALUES (19);
      END CASE ;
    END ;
  //
DELIMITER ;
```

## 循环语句
WHILE-DO…END-WHILE
```
DELIMITER //
  CREATE PROCEDURE proc5()
    BEGIN
      DECLARE var INT;
      SET var=0;
      WHILE var<6 DO
        INSERT INTO t VALUES (var);
        SET var=var+1;
      END WHILE ;
    END;
  //
DELIMITER ;
```

REPEAT...END REPEAT
此语句的特点是执行操作后检查结果
```
DELIMITER //
  CREATE PROCEDURE proc6 ()
    BEGIN
      DECLARE v INT;
      SET v=0;
      REPEAT
        INSERT INTO t VALUES(v);
        SET v=v+1;
        UNTIL v>=5
      END REPEAT;
    END;
  //
DELIMITER ;
```

LOOP...END LOOP
```
DELIMITER //
  CREATE PROCEDURE proc7 ()
    BEGIN
      DECLARE v INT;
      SET v=0;
      LOOP_LABLE:LOOP
        INSERT INTO t VALUES(v);
        SET v=v+1;
        IF v >=5 THEN
          LEAVE LOOP_LABLE;
        END IF;
      END LOOP;
    END;
  //
DELIMITER;
```

LABLES标号
标号可以用在begin repeat while 或者loop 语句前，语句标号只能在合法的语句前面使用。可以跳出循环，使运行指令达到复合语句的最后一步。

ITERATE迭代
通过引用复合语句的标号,来从新开始复合语句
```
#ITERATE
DELIMITER //
  CREATE PROCEDURE proc8()
  BEGIN
    DECLARE v INT;
    SET v=0;
    LOOP_LABLE:LOOP
      IF v=3 THEN
        SET v=v+1;
        ITERATE LOOP_LABLE;
      END IF;
      INSERT INTO t VALUES(v);
      SET v=v+1;
      IF v>=5 THEN
        LEAVE LOOP_LABLE;
      END IF;
    END LOOP;
  END;
  //
DELIMITER;
```

# 函数
存储过程可使用的函数与mysql本身的函数一致。

# 游标
概述1:
在操作mysql的时候我们知道MySQL检索操作返回一组称为结果集的行。这组返回的行都是与 SQL语句相匹配的行（零行或多行）。
简单的SELECT语句,没有办法得到第一行、下一行或前 10行，也不存在每次一行地处理所有行的简单方法。
有时，需要在检索出来的行中前进或后退一行或多行。这就是使用游标的原因。
游标（ cursor）是一个存储在MySQL服务器上的数据库查询，它不是一条 SELECT语句，而是被该语句检索出来的结果集。
在存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或做出更改。

概述2：
游标的设计是一种数据缓冲区的思想，用来存放SQL语句执行的结果。游标是一种能从包括多条数据记录的结果集中每次提取一条记录的机制。
尽管游标能遍历结果中的所有行，但一次只指向一行。
游标的作用就是用于对查询数据库所返回的记录进行遍历，以便进行相应的操作。

简单来说就是查询出来的数据索引，通过对游标的操作（第一个位置、最后一个位置、上一个位置、下一个位置）可以遍历出数据。

## 游标的特性
属性：
1.不敏感（Asensitive）：数据库可以选择不复制结果集
2.只读（Read only）
3.不滚动（Nonscrollable）：游标只能向一个方向前进，并且不可以跳过任何一行数据。

优点：
针对行操作，对从数据库中SELECT查询得到的结果集的每一行，可以进行独立的、相同或不同的操作，是一种分离的思想。

缺点：
1.性能较低，只能一行一行的操作，在数据量大的情况下，速度过慢。
2.会产生死锁，影响其他业务操作。当数据量大时，使用游标会造成内存不足的现象。

## 游标的应用场景
1.存储过程
2.函数
3.触发器
4.事件

## 游标的应用
1.定义
`DECLARE cursor_name CURSOR FOR select_statement`

2.打开游标
`OPEN cursor_name;`

3.读取游标数据
`FETCH cursor_name INTO var_name [, var_name]...`

4.关闭游标
`CLOSE cursor_name;`

5.释放游标
`DEALLOCATE cursor_name;`

使用游标涉及几个明确的步骤。
1、在能够使用游标前，必须声明（定义）它。这个过程实际上没有检索数据，它只是定义要使用的 SELECT语句。
2、一旦声明后，必须打开游标以供使用。这个过程用前面定义的SELECT语句把数据实际检索出来。
3、对于填有数据的游标，根据需要取出（检索）各行。
4、在结束游标使用时，必须关闭游标。
在声明游标后，可根据需要频繁地打开和关闭游标。在游标打开后，可根据需要频繁地执行取操作。

## 游标使用示例
1.创建游标测试表
```
CREATE TABLE cursor_table
(
id INT ,
name VARCHAR(10),
age INT
)ENGINE=innoDB DEFAULT CHARSET=utf8;
insert into cursor_table values(1, '孙悟空', 500);
insert into cursor_table values(2, '猪八戒', 200);
insert into cursor_table values(3, '沙悟净', 100);
insert into cursor_table values(4, '唐僧', 20);
```

三种方式使用游标创建一个存储过程，统计年龄大于30的记录的数量。
2.Loop循环
```
CREATE  PROCEDURE getTotal()
BEGIN
    DECLARE total INT; 
    ##创建接收游标数据的变量
    DECLARE sid INT;
    DECLARE sname VARCHAR(10);
    #创建总数变量
    DECLARE sage INT;
    #创建结束标志变量
    DECLARE done INT DEFAULT false;
    #创建游标
    DECLARE cur CURSOR FOR SELECT id,name,age from cursor_table where age>30;
    #指定游标循环结束时的返回值
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true;
    #设置初始值 
    SET sage = 0;
    SET total=0;
    #打开游标
    OPEN cur;
    #开始循环游标里的数据
    read_loop:loop
    #根据游标当前指向的一条数据
    FETCH cur INTO sid,sname,sage;
    #判断游标的循环是否结束
    IF done THEN
        LEAVE read_loop;    #跳出游标循环  
    END IF;
    #获取一条数据时，将count值进行累加操作，这里可以做任意你想做的操作，
    SET total = total + 1;
    #结束游标循环
    END LOOP;
    #关闭游标
    CLOSE cur;

    #输出结果
    SELECT total;
END
```

```
#调用存储过程  
call getTotal();
```

3.While循环
```
CREATE  PROCEDURE getTotal()
BEGIN
    DECLARE total INT; 
    ##创建接收游标数据的变量
    DECLARE sid INT;
    DECLARE sname VARCHAR(10);
    #创建总数变量
    DECLARE sage INT;
    #创建结束标志变量
    DECLARE done INT DEFAULT false;
    #创建游标
    DECLARE cur CURSOR FOR SELECT id,name,age from cursor_table where age>30;
    #指定游标循环结束时的返回值
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true; 
    SET total = 0;
    OPEN cur;
    FETCH cur INTO sid, sname, sage;
    WHILE(NOT done) 
    DO
        SET total = total + 1;  
        FETCH cur INTO sid, sname, sage;  
    END WHILE;

    CLOSE cur;
    SELECT total;
END
```

4.Repeat循环
```
CREATE getTotal()
BEGIN
    DECLARE total INT; 
    ##创建接收游标数据的变量
    DECLARE sid INT;
    DECLARE sname VARCHAR(10);
    #创建总数变量
    DECLARE sage INT;
    #创建结束标志变量
    DECLARE done INT DEFAULT false;
    #创建游标
    DECLARE cur CURSOR FOR SELECT id,name,age from cursor_table where age > 30;
    #指定游标循环结束时的返回值
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = true; 
    SET total = 0;
    OPEN cur;
    REPEAT
    FETCH cur INTO sid, sname, sage;
    IF NOT done THEN
        SET total = total + 1;  
    END IF;
    UNTIL done END REPEAT;
    CLOSE cur;
    SELECT total;
END
```

5.使用游标批量更新数据
```
BEGIN
 DECLARE  no_more_record INT DEFAULT 0;
 DECLARE  pID BIGINT(20);
 DECLARE  pValue DECIMAL(15,5);
 DECLARE  cur_record CURSOR FOR   SELECT colA, colB from tableABC;  /*首先这里对游标进行定义*/
 DECLARE  CONTINUE HANDLER FOR NOT FOUND  SET  no_more_record = 1; /*这个是个条件处理,针对NOT FOUND的条件,当没有记录时赋值为1*/
 
 OPEN  cur_record; /*接着使用OPEN打开游标*/
 FETCH  cur_record INTO pID, pValue; /*把第一行数据写入变量中,游标也随之指向了记录的第一行*/
 
 WHILE no_more_record != 1 DO
 INSERT  INTO testTable(ID, Value)
 VALUES  (pID, pValue);
 FETCH  cur_record INTO pID, pValue;
 
 END WHILE;
 CLOSE  cur_record;  /*用完后记得用CLOSE把资源释放掉*/
END
```

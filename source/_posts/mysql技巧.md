---
title: mysql技巧
tags: mysql
date: 2018-12-06 18:13:55
---

# left join和 inner join 区别
left以 left join 左侧的表为主表
right 以 right join 右侧表为主表
inner join 查找的数据是左右两张表共有的

# 查找一个表中存在而另一个表中不存在的记录
```
select j.id,j.job_name,c.member_name as company_id,j.job_address,j.salary,j.flush_time
from hr_job j
left join HR_COMPANY c on c.id = j.company_id
left join hr_job_apply ja on ja.job_id = j.id
where not exists (select ja.id from HR_JOB_APPLY where ja.job_id = j.id)

select * from t1
where id not in(select id from t2)
```
# 时间相关 
```
获取今天到明天的数据
where date_format(create_date,'%y-%m-%d') 
BETWEEN date_format(curdate(),'%y-%m-%d') 
and DATE_SUB(curdate(),INTERVAL -1 DAY)

当天数据1
where date_format(create_date,'%y-%m-%d') = date_format(curdate(),'%y-%m-%d')

当天数据2
select * from 表名 where to_days(时间字段名) = to_days(now());
注：to_days 是mysql的函数

昨天数据
SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) <= 1

近7天
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(时间字段名)

近30天
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)

本月
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )

上一月
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1

查询本季度数据
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(now());

查询上季度数据
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));

查询本年数据
select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW());

查询上年数据
select * from `ht_invoice_information` where year(create_date)=year(date_sub(now(),interval 1 year));

查询当前这周的数据
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now());

查询上周的数据
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now())-1;

查询上个月的数据
select name,submittime from enterprise where date_format(submittime,'%Y-%m')=date_format(DATE_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m')

select * from user where DATE_FORMAT(pudate,'%Y%m') = DATE_FORMAT(CURDATE(),'%Y%m') ; 

select * from user where WEEKOFYEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = WEEKOFYEAR(now()) 

select * from user where MONTH(FROM_UNIXTIME(pudate,'%y-%m-%d')) = MONTH(now()) 

select * from user where YEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = YEAR(now()) and MONTH(FROM_UNIXTIME(pudate,'%y-%m-%d')) = MONTH(now()) 

select * from user where pudate between  上月最后一天  and 下月第一天

查询当前月份的数据 
select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(now(),'%Y-%m')

查询距离当前现在6个月的数据
select name,submittime from enterprise where submittime between date_sub(now(),interval 6 month) and now();
```

# 几种case-when的形式
```
CASE b.check_type
when '1' then b.real_name
when '2' then b.association_name
    else u2.name(有else)
end AS "user.name",
```	
```
CASE a.exh_type
when '1' then a.is_art
when '2' then c.is_art
end AS "isArt"
```
```
case (case后边没东西)
when a.end_time > now() then '1'
when a.end_time &lt; now() then '2'
end AS "isPass"
```
```
where a.is_home = '1'
and a.publish_status = '1'
and (
case a.exh_type
    when '1' then a.audit_status = '2'
        else 1 = 1
        end
)
```
	
# rand()
	随机函数效率很低
	order by RAND() ：随机返回结果

# REGEXP
	单字匹配多字，或反之
	例如：木雕中，标签
	活动,展讯,文化弘扬 为一条数据的标签，要求，可以匹配到标签为展讯的内容
```
AND x.label REGEXP
(
    SELECT
    replace(new_category,',','|')
    FROM
    wd_news a1
    WHERE
    a1.id = '1a94c3168229403aa11ea136596e335a'
)
```

# 并发插入、存在不插入，存在更新操作
方法1：通常会，先查再插，但是这样无法避免并发问题，不用考虑了。

2.存在即更新，不存在即插入
`REPLACE into table (c1,c2) values(1,5)`
如果发现表中已经有此行数据（根据主键或者唯一索引判断），
则先删除此行数据，然后插入新的数据，否则，直接插入新数据。

所以，插入时如果不插入主键(逐渐默认递增)，就需要保证一列是唯一索引，保证唯一，才行
```
创建唯一索引sql:(也可以通过navcat创建)
	CREATE UNIQUE INDEX sub_id ON table_name(sub_id); 
删除唯一索引sql:
	ALTER TABLE table_name DROP INDEX sub_id; 
```

3.存在不插入，不存在再插入(感觉这种方法是最好的)
mysql 不能在插入时带where条件，故用临时表处理
```
INSERT INTO demo_in(a,b,c) 
SELECT 123, 2, 4 FROM DUAL 
WHERE NOT EXISTS(SELECT c FROM demo_in WHERE c = 4);
```
用临时表DUAL来标记数据，然后插入到demo_in表中。
条件是c=4，并且not exists，也就是当c=4条件满足，则不插入。

# 视哪张表为主表，哪张为子表
对比两条SQL

SQL 1:
```
SELECT
	a.id,
	b.type,
	b.last_price,
	b.order_content,
	b.create_time
	FROM
	business a
	LEFT JOIN wallet_checkinout b ON a.user_id = b.user_id
	WHERE
		a.id = 1000039
		AND b.money_type = 3
		AND b.biz_type = 6
	ORDER BY
	 b.create_time desc
```
SQL 2:
```
SELECT
      a.id,
      a.type,
      a.last_price,
      a.order_info,
      a.order_content,
      a.create_time,
      a.close_status
    FROM
    wallet_checkinout a
    LEFT JOIN business b ON b.user_id = a.user_id
    WHERE
      b.id = 1000039
      AND a.type = 1
      AND a.money_type = 3
      AND a.biz_type = 6
		order by a.create_time desc
```
以上两条sql查出的结果是一样的。
目前，我认为：无论以哪张表为主表都可以。

# union和union all 区别
1.union 操作符会选取不同的值
   UNION ALL 允许重复的值(会将所有的值列出)

# union(union all)和order by 
1.每一个子句可以使用()包围，但是当第一个子句使用了()时，其它的子句都必须使用括号包围。
2. 每一个字句可以包含WHERE，GROUP BY，HAVING，JOIN，LIMIT，但是不能使用ORDER BY，如果需要使用ORDER BY，必须使用()包围子句。

# 给查询记录添加自然序号
表a，数据：
id name
12 张三
13 李四
15 王五

通过一条sql获取这三条记录，并且给每行前面加上从1开始的序号

1 12 张三
2 13 李四
3 14 王五
```
SELECT
@rownum := @rownum + 1 AS rownum,
a.* 
FROM
( SELECT @rownum := 0 ) r,
table a;
```

# MySQL单表多次查询和多表联合查询，哪个效率高？
很多高性能的应用都会对关联查询进行分解。
简单地，可以对每个表进行一次单表查询，然后将结果在应用程序中进行关联。例如，下面这个查询：
```
select * from tableA a
join tableB b on b.tableA_id = a.id
join tableC c on c.tableB_id = b.id
where a.name = '123';
```

可以分解成下面这些查询来代替：
```
select id from tableA where name = '123'; -- 结果A
select * from tableB where tableA_id = id(结果A) -- 结果B
select * from tableC where id in (结果B)
```

到底为什么要这样做？
咋一看，这样做并没有什么好处，原本一条查询，这里却变成了多条查询，返回结果又是一模一样。
事实上，用分解关联查询的方式重构查询具有如下优势：
1.让缓存效率更高
许多应用程序可以方便地缓存单表查询对应的结果对象。另外对于MySQL的查询缓存来说，如果关联中的某个表发生了变化，那么就无法使用查询缓存了，而拆分后，如果某个表很少改变，那么基于该表的查询就可以重复利用查询缓存结果了。

2.将查询分解后，执行单个查询可以减少锁的竞争。

3.在应用层做关联，可以更容易对数据库进行拆分，更容易做到高性能和可扩展。

4.查询本身效率也可能会有所提升

5.可以减少冗余记录的查询。

6.更进一步，这样做相当于在应用中实现了哈希关联，而不是使用MySQL的嵌套环关联，某些场景哈希关联的效率更高很多。

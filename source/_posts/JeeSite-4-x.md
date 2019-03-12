---
title: JeeSite_4.x
tags: 架构
date: 2019-03-11 11:46:11
---

# 概述
优点：
1.省的自己搭框架
2.生成增删改查(包括了页面)

缺点：
自定义较弱，扩展可能困难，修改困难。

适合：快速开发，外包

官网：https://jeesite.gitee.io/docs/

# 环境搭建
## 环境要求
1.Java SDK 1.8
2.Eclipse/IDEA
3.Apache Maven 3.3.0+
4.MySql 5.7.11+

## 拉取代码
`git clone https://gitee.com/thinkgem/jeesite4-cloud.git`

idea自动加载Maven依赖包，初次加载会比较慢，此时可以准备数据库。

## 初始化数据库
（下载最新的mysql脚本）： https://gitee.com/thinkgem/jeesite4/attach_files

1.配置 my.ini
1）打开 my.ini 给 [mysqld] 增加如下配置：
`sql_mode="ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"`

2）如果遇到 ERROR 1709 (HY000): Index column size too large. The maximum column size is 767 bytes.
错误，请加入如下配置：

```
innodb_large_prefix = ON 
innodb_file_format = Barracuda 
innodb_file_per_table = ON
```

并修改报错的建表语句后面加上：ENGINE=InnoDB row_format=DYNAMIC;

2.创建用户和授权
```
set global read_only=0;
set global optimizer_switch='derived_merge=off';
create user 'jeesite'@'%' identified by 'jeesite';
create database jeesite DEFAULT CHARSET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
grant all privileges on jeesite.* to 'jeesite'@'%' identified by 'jeesite';
flush privileges;
```

第三步(创建用户)时可能会报错，
`ERROR 1396 (HY000): Operation CREATE USER failed for ‘test’@’%’ `

查看是不是存在这个用户 `select user from user; `发现没有这个用户。 

记得上次有删除过这个用户。可能没有刷新权限 
flush privileges; 

之后还是不行报错`ERROR 1396 (HY000): Operation CREATE USER failed for ‘test’@’%’`

没办法再删除一次： 
drop user ‘test’@’%’; 
flush privileges; 

之后create user ‘test’@’%’ identified by ‘test’; 
成功。 

网上找了下原因： 
Assume the user is there, so drop the user 
After deleting the user, there is need to flush the mysql privileges 
Now create the user.

3.用idea打开刚才下载好的mysql执行项目`jeesite-web-v4.1.3-2019-2-15`

cmd进入到项目根目录下，执行：
`mvn clean package -Dmaven.test.skip=true`

然后在pom文件中加入以下插件
```
<plugin>  
        <groupId>org.apache.maven.plugins</groupId>  
        <artifactId>maven-surefire-plugin</artifactId>  
        <version>2.4.2</version>  
        <configuration>  
          <skipTests>true</skipTests>  
        </configuration>  
</plugin>
```

执行init-data.bat
如果没有报错，此时数据库应该就初始化完成了，最后显示build success

## 启动项目
配置 /jeesite-cloud-config/../cloud-config/application.yml 分布式统一配置文件 JDBC 和 Redis 等信息。

按顺序运行以下启动类的main方法（启动完成一个再启下一个）：
```
/jeesite-cloud-eureka/../EurekaApplication.java
/jeesite-cloud-config/../ConfigApplication.java
/jeesite-cloud-gateway/../GatewayApplication.java
/jeesite-cloud-module-core/../CoreApplication.java
/jeesite-cloud-module-test1/../Test1Application.java
/jeesite-cloud-module-test2/../Test2Application.java
```

以上都启动成功后，浏览器访问网关项目地址即可：
访问地址：http://127.0.0.1:8980/js system admin
若访问报错，请再等待一会，可能服务未完全启动完成

更多请参考项目中的readme.md


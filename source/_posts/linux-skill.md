---
title: linux skill
tags: linux
date: 2019-03-27 16:11:40
---

# apache下载文件
![](linux skill/1.png)
1.找到apache的安装目录,httpd目录
2.进入到conf.d目录下
3.删掉welcome.conf rm –rf welcome.conf
4.回到apache的安装目录下,进入到conf 文件夹下
5.查看httpd.conf文件(里面有默认访问页面的路径) vi httpd.conf
找到这个地方
DocumentRoot "/var/www/html"
这个是你apache默认的访问页面,进入到这个文件夹
这个文件夹下的内容会被显示到apahce的默认访问页里，把你想要下载的东西打包成压缩文件，传到这个文件夹下就好了。


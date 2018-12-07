---
title: nginx负载均衡的简单配置
tags: nginx
date: 2018-12-06 18:06:29
---
### 1.下载 http://nginx.org/
### 2.解压至英文路径
### 3.进入解压后文件夹
双击nginx.exe启动nginx,默认为80端口，nginx启动时一闪而过，启动后打开浏览器，输入localhost，看到欢迎页面证明启动成功，若看不到欢迎页面，可能是80端口被占用
解决方法：进入nginx文件夹下的conf文件夹
编辑nginx.conf，找到server{listen   80;}修改端口号，重启nginx即可
### 4.启动与停止
1.启动，windows下双击.exe文件
2.停止： nginx  –s  stop 即可
  若出现  nginx.pid 类似字样错误
        输入 nginx  –c  conf/nginx.conf
        输入后cmd会卡住，关闭即可，重新开启cmd，重复上述步骤停止nginx

### 5.配置简单的负载均衡
准备工作：
本机两个可以启动的tomcat，端口不同。
编辑conf/nginx.conf，添加如下配置：
![](https://github.com/ayanamiq/images/blob/master/images/nginx_conf1.png?raw=true)
weight为权重，权重越大越优先分配请求
继续编辑
![](https://github.com/ayanamiq/images/blob/master/images/nginx_conf2.png?raw=true)
proxy_pass配置是 为http://locahost 开启代理服务，如网站上线则换成网址
配置完之后，重启nginx
打开浏览器，输入locahost 则可看到两个tomcat互换的效果，证明配置成功
(ps:老毛子真的屌)

### 6.静态资源分离
静态资源为 css ,js,html,图片等资源
编辑conf/nginx.conf
![](https://github.com/ayanamiq/images/blob/master/images/nginx_conf3.png?raw=true)
此段配置为 当访问.jpg或.png时，会到D：/upload去匹配资源
要保证路径在本机存在
配置完成后，重启nginx
到该路径下放置一张图片
打开浏览器，输入localhost/文件名，可以访问到该图片证明配置成功
![](https://github.com/ayanamiq/images/blob/master/images/nginx_img.png?raw=true)
该配置可匹配多层路径，如在upload文件夹下创建一个新的文件夹images并放置一张图片
打开浏览器输入localhost/images/文件名  则可加载images文件夹下对应名字的图片


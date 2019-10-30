---
title: nginx
tags: nginx
date: 2018-12-06 18:06:29
---

# 配置
1.下载：http://nginx.org/
2.解压至英文路径
3.进入解压后文件夹，双击nginx.exe启动nginx，默认为80端口，nginx启动时一闪而过，启动后打开浏览器，输入localhost，看到欢迎页面证明启动成功，若看不到欢迎页面，可能是80端口被占用
解决方法：进入nginx文件夹下的conf文件夹
编辑nginx.conf，找到server{listen   80;}修改端口号，重启nginx即可

# 启动与停止
1.启动，windows下双击.exe文件
2.停止： nginx  –s  stop 即可
若出现  nginx.pid 类似字样错误，输入
`nginx  –c  conf/nginx.conf`
输入后cmd会卡住，关闭即可，重新开启cmd，重复上述步骤停止nginx

# 配置简单的负载均衡
准备工作：
本机两个可以启动的tomcat，端口不同。
编辑conf/nginx.conf，添加如下配置：
```
upstream localhost{
    server 192.168.3.110:8081 max_fails=2 fail_timeout=600s weight=10;
    server 192.168.3.110:8080 max_fails=2 fail_timeout=600s weight=5;
}
```
weight为权重，权重越大越优先分配请求
继续编辑
```
location / {
    root    html;
    index   index.html    index.htm;
    proxy_pass   http://localhost;
    proxy_redirect   default;
}
```
proxy_pass配置是 为http://locahost 开启代理服务，如网站上线则换成网址
配置完之后，重启nginx
打开浏览器，输入locahost 则可看到两个tomcat互换的效果，证明配置成功

# 静态资源分离
静态资源为css ，js，html，图片等资源
编辑conf/nginx.conf，在server的大括号最后面加入如下配置：
```
location ~ \.(jpg|png)${
    root D:/upload;
}
```
这段配置表示：当访问.jpg或.png时，会到D：/upload去匹配资源，要保证路径在本机存在，配置完成后，重启nginx，到该路径下放置一张图片，打开浏览器，输入localhost/文件名，可以访问到该图片证明配置成功。
该配置可匹配多层路径，如在upload文件夹下创建一个新的文件夹images并放置一张图片。
打开浏览器输入localhost/images/文件名  则可加载images文件夹下对应名字的图片。

# 配置端口转发
```
server {
	listen       80; #监听端口
	server_name  api.xxx.com; #你想转发的域名
	location / {
		proxy_pass http://api.xxx.com:8000;  #你想转发的地址，这里我想把本机的8000端口映射到80端口上，最好不要写localhost或127.0.0.1
		
		#下面几个参数可以不加，我也不知道干嘛的
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Scheme $scheme;
		proxy_pass_header Server;
		proxy_set_header Host $http_host;
		proxy_redirect off;
	}

	location ~.txt {
		root /usr/local/src; #静态资源目录
	}
}
```

# Nginx配置Http和Https共存
把`ssl on；`这行去掉，ssl写在443端口后面。这样http和https的链接都可以用。
注意nginx配置文件有两个`server`模块，本来`server listen 443`这个模块是注释掉的，
现在我们把`server listen 80`这个模块都注释掉，把`server listen 443`打开，并把配置全都放到`listen 443`这个模块中。
```
server {
	listen 80 default backlog=2048;
	listen       443 ssl;
	server_name  api.zjxk12.com;

	# ssl开头的都是证书配置
	ssl_certificate      /usr/local/nginx/cert/2998903_api.zjxk12.com.pem;
	ssl_certificate_key  /usr/local/nginx/cert/2998903_api.zjxk12.com.key;
	ssl_session_cache    shared:SSL:1m;
	ssl_session_timeout  5m;
	ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_prefer_server_ciphers  on;

	# 端口转发
	location / {
		proxy_pass http://api.zjxk12.com:8000;
		index  index.html index.htm;
	}

	location ~.txt{
		root /usr/local/src;
	}

	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
		root   html;
	}
}
```
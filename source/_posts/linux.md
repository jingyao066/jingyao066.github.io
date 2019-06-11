---
title: linux
tags: linux
date: 2019-03-27 16:11:40
---

# 软件安装
## mysql
官网下载：https://dev.mysql.com/downloads/mysql/
拉到页面下方，下拉Select Operating System，选择`Linux-Generic`，Select OS Version：选择64位。如果不想下载8.0版本的，可以选择右侧的：
Looking for the latest GA version?，会出现mysql5.7版本。下载最下边的`Linux - Generic (glibc 2.12) (x86, 64-bit), TAR`
下载完，上传到linux服务器`/usr/local`位置。

加压缩：`tar -zxvf mysql-5.7...`
重命名：`mv mysql-5.7... mysql`
在mysql根目录创建data文件夹：`mkdir data`
创建mysql的用户组和用户，并对mysql目录设置用户组和用户：
```
 groupadd mysql
 useradd mysql -g mysql
 cd mysql
 pwd:/usr/local/mysql
 chown -R mysql .
 chgrp -R mysql .
```

初始化mysql并启动mysql服务：
`cd /usr/local/mysql/bin`

初始化：
`./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data `

报错：
`error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory`

如果已经安装了libnuma.so.1，先
`yum remove libnuma.so.1`

然后安装64位版本：
`yum -y install numactl.x86_64`

还是上边的报错，执行：
`yum install -y libaio`
或执行：
`yum install -y libaio.so.1`

然后执行上边的初始化指令，最后提示如下初始化成功：
` A temporary password is generated for root@localhost: C(tSgbIuh4yy`
最后提示的是默认的密码。

然后修改下权限，把除了data外的所有mysql文件的权限都设置为root：
```
chown -R root .
chown -R mysql data
```

然后修改mysql配置文件（如果没有自己创建）：
`vi /etc/my.cnf`

直接修改默认配置：
```
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/tmp/mysql.sock
port=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[client]
socket=/usr/local/mysql/tmp/mysql.sock

[mysqld_safe]
#log-error=/var/log/mariadb/mariadb.log
#pid-file=/var/run/mariadb/mariadb.pid

basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/tmp/mysql.sock
port=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```
basedir就是mysql根目录
datadir就是上面在mysql根目录中新建的data文件夹
socket我在mysql根目录中新建了一个tmp文件夹，然后这里就指向了它，mysql.sock这个文件在我们启动mysql时会自动创建。所以我们只要新建tmp文件夹就行了。 

修改tmp文件夹权限：
`chown -R mysql:mysql tmp`

然后可以启动服务了：

启动和关闭mysql：
```
#/etc/init.d/mysql start   或者   serivce mysql start  或者  ./bin/mysqld_safe&  
#/etc/init.d/mysql stop    或者   service mysql stop   或者  ./bin/mysqladmin -u root -p shutdown
```

我一般使用`./bin/mysqld_safe&`命令启动mysql服务
启动后检查下`ps -ef|grep mysql`检查下是否启动

启动：
`./bin/mysqld_safe&`
见到如下内容说明启动成功：
```
[1] 30706
[root@VM_0_12_centos mysql-5.7]# Logging to '/usr/local/mysql-5.7/data/VM_0_12_centos.err'.
2019-06-09T01:55:28.047197Z mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql-5.7/data
```
输入bg后台运行，然后再运行`ps -ef|grep mysql`检查可以看到mysql已经启动了。

连接mysql：
`./bin/mysql -uroot -p`

报错：
`mysql: [ERROR] unknown variable 'symbolic-links=0'`

查看my.cnf文件，文件中
`# Disabling symbolic-links is recommended to prevent assorted security risks symbolic-links=0`
应该是分两行展示了，修改该行为一行显示。

再次链接Mysql:
`./bin/mysql -uroot -p`

上边自动生成的密码是可以复制的，此时直接右键空白处，然后回车，应该就成功进入了。

输入默认密码报错：
`Can't connect to local MySQL server through socket '/usr/local/mysql-5.7/tmp/mysql.sock' (2)`
此时需要检查配置文件`vi /etc/my.cnf`，socket文件路径是否正确，tmp文件夹是否已经创建。

成功登录mysql，执行命令报错：
`You must reset your password using ALTER USER statement before executing this statement.`

执行如下sql：
`alter user user() identified by "123456";`
`identified by`后边是密码
这部具体是什么操作？

此时还需要修改root的密码：
`update user set authentication_string=password('填入新密码”') where user='root';`

注意mysql5.7之前需要把`authentication_string`替换为：`password`

忘记root密码怎么办？
编辑配置文件：`vi /etc/my.cnf`

在[mysqld]下面添加一条命令：`skip-grant-tables`
保存退出，开始修改root密码：

进入MySql控制台（直接按回车，这时不需要输入root密码。）
`mysql -uroot -p`

切换到mysql数据库
`mysql>use mysql;`

修改mysql数据库中root的密码
`update user set authentication_string=password(“填入新密码”) where user=‘root’;`

刷新mysql权限
`flush privileges;`

退出
`exit;`

再次vi /etc/my.cnf。把skip-grant-tables删除掉。保存退出。完成MySql Root密码修改

为root用户赋予权限：
`grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;`
注意修改identified by后边的密码。

可能会报错：
`Your password does not satisfy the current policy requirements`

这时修改密码级别：
`set global validate_password_policy=0;`
`set global validate_password_length=4;`

然后执行赋权语句成功，刷新权限：
`flush privileges;`

重启mysql服务：
`service mysql restart`

## tomcat
官网下载：
https://tomcat.apache.org/download-90.cgi
找到Core下的tar.gz包下载并上传到/usr/local 目录下。

到usr/local目录下解压：
`tar -zxvf apache-tomcat-9.0.1.tar.gz`

tomcat默认端口是8080，这里为了防止和其他任务端口冲突，我们将端口修改为合适的端口。
cd到/tomcat/conf，`vi server.xml`，找到`Connector port="8080"`的位置，修改为恰当的端口。

cd到tomcat的bin目录下：
`cd ../bin`，启动tomcat`./startup.sh`，看到日志末尾`Tomcat started`，说明tomcat启动成功。

查看tomcat日志：
`tail -f logs/catalina.out`

若远程无法访问，可能需要在阿里云控制台添加端口白名单。或者是防火墙

## git
先确认是否安装了git：
`rpm -qa|grep git`
如果安装了需要先卸载：
`rpm -e --nodeps git  或者  rpm -e git`

安装：
`yum install git`
再使用 rpm -qa|grep git 来查看是否已经安装好了Git。

## maven
1. 到指定目录下载maven：
`wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz`

2. 解压：
`tar -zxvf apache-maven-3.3.9-bin.tar.gz `

3. 改下名，方便后面操作：
`mv apache-maven-3.3.9 maven`

4. 配置环境变量
`vi /etc/profile`，在合适的位置添加如下内容：
`M2_HOME=/opt/tyrone/maven （需要修改为自己maven的安装路径）`
`export PATH=${M2_HOME}/bin:${PATH}`

然后使配置文件生效：
`source /etc/profile`

5. 检查是否安装成功
`mvn -v`
出现版本号：`Apache Maven 3.3.9...`，证明安装成功。

## zookeeper
下载地址：
http://mirror.bit.edu.cn/apache/zookeeper/
下载stable版本。

将下载好的.gz解压并复制到想要的位置。/home/wjy/install

在zk的根目录创建两个文件夹data和data_log，分别用来存放zk的数据以及日志。然后进入到data目录，使用pwd指令查看并复制data目录的绝对路径。

进入zookeeper的conf目录下，复制zoo_sample.cfg为zoo.cfg
`cp zoo_sample.cfg zoo.cfg`

然后编辑zoo.cfg，修改或新增下面两行配置：
```
dataDir=/home/wjy/install/zookeeper-3.4.13/data
dataLogDir=/home/wjy/install/zookeeper-3.4.13/data_log
```
保存并退出。

zoo.cfg只存放了单机的配置，想配置集群需要在最后面加上其他服务器的ip和端口。

到zk的bin目录下启动：
`./zkServer.sh start`

查看状态
`./zkServer.sh status`
```
ZooKeeper JMX enabled by default
Using config: /home/wjy/install/zookeeper-3.4.13/bin/../conf/zoo.cfg
Mode: standalone
```
出现上边的状态说明启动成功。
standalone：单机

## redis
https://redis.io/
官网下载，解压并复制到想要的位置 /home/wjy/install

cd到redis根目录下，执行
`make`
报错：
```
net.c:36:23: fatal error: sys/types.h: No such file or directory(没有那个文件或目录)
...
adlist.c:32:20: fatal error: stdlib.h: No such file or directory
```
原因是缺少文件，执行以下命令后再次make：
`sudo apt-get install libc6-dev`

还是报错:
`zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory`

原因是libc不是默认的分配器，在make后添加参数解决：
`sudo make MALLOC=libc`

等待执行完成，到redis/src目录下，执行
`./redis-server`
出现端口号和进程PID，证明启动成功。

通过上边的指令启动redis，`ctrl+c`推出窗口，redis也会关闭，可以在指令后加上`&`
`./redis-server &`
按ctrl + C 可退出redis 启动窗口，此时redis　并不会关闭，而是会再后台运行，可通过命令查看: ps aux | grep redis

### 给redis添加密码
- 临时密码(redis重启之后会失效)
首先启动redis服务，然后进入redis：
`redis-cli -p 6379`

查看当前redis有没有设置密码：
`config get requirepass`
```
1) "requirepass"
2) ""
```

为以上显示说明没有密码，那么现在来设置密码：
`config set requirepass 123456`

提示ok，再次查看当前redis就提示需要密码：
```
127.0.0.1:6379> config get requirepass
(error) NOAUTH Authentication required.
127.0.0.1:6379>
```

- 永久方式
修改redis根目录下的`redis.conf`文件：
`vi redis.conf`

查找字符串：
命令模式下输入`“/字符串”`，例如“/Section 3”。
如果查找下一个，按“n”即可。
`/requirepass foobared`

在该行下面输入：
`requirepass 123456`
123456是你的密码

## nginx
1. 下载Nginx及相关组件
Linux系统是Centos 6.5 64位，如果不是root用户，切换到root用户下安装
`su root`
输入密码，密码不显示。

进入用户目录下载程序：
`cd /usr/local/src`

下载相关组件：
```
[root@localhost src]# wget http://nginx.org/download/nginx-1.10.2.tar.gz
省略安装内容...
[root@localhost src]# wget http://www.openssl.org/source/openssl-fips-2.0.10.tar.gz
省略安装内容...
[root@localhost src]# wget http://zlib.net/zlib-1.2.11.tar.gz
省略安装内容...
[root@localhost src]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
省略安装内容...
```
上边的安装包可能不是最新的，可以去官网找最新的安装包：
nginx：
http://nginx.org/download/
注意列表不是有序的，现在是2019年，我在页面搜索2019，就可以看到最近发布的安装包，然后把上边的`nginx-1.10.2.tar.gz`替换成新的安装包名字。

安装c++编译环境，如已安装可略过
```
[root@localhost src]# yum install gcc-c++
省略安装内容...
期间会有确认提示输入y回车
Is this ok [y/N]:y
省略安装内容...
```
2. 安装Nginx及相关组件

openssl安装
```
[root@localhost src]# tar zxvf openssl-fips-2.0.10.tar.gz
省略安装内容...
[root@localhost src]# cd openssl-fips-2.0.10
[root@localhost openssl-fips-2.0.10]# ./config && make && make install
省略安装内容...
```

pcre安装
```
[root@localhost src]# tar zxvf pcre-8.40.tar.gz
省略安装内容...
[root@localhost src]# cd pcre-8.40
[root@localhost pcre-8.40]# ./configure && make && make install
省略安装内容...
```

zlib安装
```
[root@localhost src]# tar zxvf zlib-1.2.11.tar.gz
省略安装内容...
[root@localhost src]# cd zlib-1.2.11
[root@localhost zlib-1.2.11]# ./configure && make && make install
省略安装内容...
```

nginx安装
```
[root@localhost src]# tar zxvf nginx-1.10.2.tar.gz
省略安装内容...
[root@localhost src]# cd nginx-1.10.2
[root@localhost nginx-1.10.2]# ./configure && make && make install
省略安装内容...
```

3. 启动nginx
先找一下nginx安装到什么位置上了
`whereis nginx`
`nginx: /usr/local/nginx`

进入nginx目录并启动：
```
cd /usr/local/nginx/sbin
nginx
```

参考地址：
https://www.cnblogs.com/taiyonghai/p/6728707.html

# 技巧
## apache下载文件
![](linux/1.png)
1. 找到apache的安装目录，httpd目录
2. 进入到conf.d目录下
3. 删掉welcome.conf
`rm –rf welcome.conf`
4. 回到apache的安装目录下，进入到conf 文件夹下
5. 查看httpd.conf文件(里面有默认访问页面的路径)
`vi httpd.conf`
找到这个：
`DocumentRoot "/var/www/html"`
这个是你apache默认的访问页面，进入到这个文件夹
这个文件夹下的内容会被显示到apahce的默认访问页里，把你想要下载的东西打包成压缩文件，传到这个文件夹下，就可以通过ip访问下载了。

# 直接用java -jar xxx.jar，当退出或关闭shell时，程序就会停止掉。
`java -jar xxx.jar &`
`nohup java -jar xxxx.jar &`

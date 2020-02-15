---
title: linux
tags: linux
date: 2019-03-27 16:11:40
---

# 环境搭建
## jdk
可以选择卸载系统自带的OpenJDK以及相关的java文件，不卸载也没关系，直接修改`/etc/profile`即可。
检查是否安装了jdk：
`java -version`
可能已经安装openJDK，这里需要先卸载掉：
`rpm -qa | grep java`
命令说明：
rpm：管理套件
-qa：使用询问模式，查询所有套件
grep：查找文件里符合条件的字符串
java：查找包含java字符串的文件

把在检查出的列表中前缀带`java`的文件全部删除。
完整的删除文件的命令，在命令窗口键入：
`rpm -e --nodeps java-1.8.0-openjdk-1.8.0.102-4.b14.el7.x86_64`
命令介绍：
rpm：管理套件
-e：删除指定的套件
--nodeps：不验证套件档的相互关联性

检查是否删除成功，再次输入：
`java -version`
`command not found`
代表删除成功，如果还没有删除，则用`yum -y remove`去删除他们。

### 普通方式安装JDK
[官网下载jdk1.8linux安装包](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
下载`Linux64...x64.tar.gz`这个包，只有这一个包符合我们的要求。
下载完通过FileZilla上传到服务器的理想位置，我上传到`/usr/local/src`。
然后再复制到`/usr/local`，这里是我们的安装位置。`cp jdk-8u144-linux-x64.tar.gz /usr/local`
解压：`tar -zxvf jdk-8u144-linux-x64.tar.gz`
删除JDK压缩包：`rm -f jdk-8u144-linux-x64.tar.gz`
编辑全局变量：`vim /etc/profile`
```
#java environment
export JAVA_HOME=/usr/local/jdk1.8.0_211
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
```
注意修改为自己的jdk安装路径，然后保存退出。
让刚刚设置的环境变量生效`source /etc/profile`
检查是否配置成功`java -version`，出现版本号证明成功。
注意：JDK安装在哪个用户下，就是给哪个用户使用。
[参考地址](https://www.cnblogs.com/sxdcgaq8080/p/7492426.html)

### docker安装JDK
1. 安装docker(过程略)
2. 下载centos镜像
`sudo docker pull centos:centos7`

3. [下载jdk1.8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，并上传到/usr/local/src目录，然后解压

```
$ sudo cd /usr/local/src
$ sudo tar zxf jdk-8u201-linux-x64.tar.gz
$ sudo ls
jdk1.8.0_201  jdk-8u201-linux-x64.tar.gz
```

4. 创建Dockerfile
先在/usr/local目录下创建jdk目录，并将/usr/local/src下的jdk-8u201-linux-x64.tar.gz复制到/usr/local/jdk目录下，然后在/usr/local/jdk目录下创建Dockerfile文件。
```
$ sudo mkdir /usr/local/jdk
$ sudo cd /usr/local/jdk
$ sudo cd /usr/local/jdk
$ sudo cp ../src/jdk-8u201-linux-x64.tar.gz ./
$ sudo ls
jdk-8u201-linux-x64.tar.gz
$ sudo vi Dockerfile
FROM centos:centos7
MAINTAINER tom
RUN mkdir /usr/local/jdk
WORKDIR /usr/local/jdk
ADD jdk-8u201-linux-x64.tar.gz /usr/local/jdk

ENV JAVA_HOME /usr/local/jdk/jdk1.8.0_201
ENV JRE_HOME /usr/local/jdk/jdk1.8.0_201/jre
ENV PATH $JAVA_HOME/bin:$PATH
```
注意：
- 修改上边指令中jdk的名称
- jdk拷贝到dockerfile同级目录，如果在其它目录拷贝的时候可能出现找不到目录错误
- 使用ADD指令会直接对jdk-8u144-linux-x64.tar.gz进行解压缩，不用再单独的tar解压jdk了
- 很多地方都是使用的Dockerfile这种固定名称，其实创建的时候可以通过 -f 来指定dockerfile

5. 使用Dockerfile创建镜像
`sudo docker build -t jdk1.8 .`

如果文件名不是`Dockerfile`，需要指定别名：
`docker build -t jdk-8u144：20180619 . -f jdkdockerfile`

注意：
- `-t`指定镜像的名称和tag； 
- 使用-f 指定要使用的dockerfile，如果不指定会寻找当前目录名为Dockerfile的文件
- 上面有个 . ,这个表示当前目录，必不可少的

6. 在镜像仓库中查看是否构建成功
`docker images`
可以看到仓库中已经有jdk镜像

7. 运行创建的镜像
`docker run -d -it jdk1.8 /bin/bash`

创建容器的时候一定要使用 -it /bin/bash这种方式，要不然jdk的容器起不来。

8. 进入jdk容器，查看是否安装正确（即查看安装之后的目录）
`docker exec -it jdk1.8 /bin/bash`

[其他参考](https://www.jianshu.com/p/59e0a8828b3b)

## mysql
### docker安装mysql
查看docker仓库中的mysql版本：
`docker search mysql`
[可从DockerHub上查看可下载的mysql版本](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=mysql&starCount=0)
[也可参照dockerHub上mysql的文档来安装](https://hub.docker.com/_/mysql/)

我们下载5.7版本：
`docker pull mysql:5.7`

[也可以通过Dockerfile构建mysql](https://www.runoob.com/docker/docker-install-mysql.html)

查看docker镜像：
`docker images | grep mysql`

根据镜像说明可知：
默认的配置文件是：/etc/mysql/my.cnf
默认的数据目录是：/var/lib/mysql

可以先简单构建一次mysql：
`docker run -d --name mysql --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7`

进入容器，查看my.cnf：
`docker exec -it mysql bash`
`cat /etc/mysql/my.cnf`

通过查看mysql容器中的my.cnf可以发现，主配置文件my.cnf加载了宿主机 /etc/mysql/conf.d 文件夹下所有的配置。
这个路径只是默认的，我们可以在构建容器时，手动指定data文件夹和conf文件的位置。
所以如果想要自定义配置，只需在我们理想的目录创建 .cnf 文件，并在构建容器时指定配置文件的路径即可(新建的文件可以任意起名，但保证后缀名是 .cnf)。

创建将要映射到容器中的目录以及.cnf文件，然后再创建容器。
在宿主机中创建文件夹 mysql，并分别创建 data 目录和 conf 目录。新建配置文件 my.cnf，并写入如下内容：
```
[mysqld]
server-id = 1 #服务Id唯一
port = 3306
log-error    = /var/log/mysql/error.log
#只能用IP地址
skip_name_resolve 
#数据库默认字符集
character-set-server = utf8mb4
#数据库字符集对应一些排序等规则 
collation-server = utf8mb4_general_ci
#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'
#最大连接数
max_connections = 1000
```

补一个比较完整的配置：
```
[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
#服务Id唯一
server-id = 1
port = 3306
log-error = /var/log/mysql/error.log

#只能用IP地址
skip_name_resolve

character-set-client-handshake = FALSE

#数据库默认字符集
character-set-server = utf8mb4

#数据库字符集对应一些排序等规则
collation-server = utf8mb4_general_ci

#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'

#最大连接数
max_connections = 1000

#解决only_full_group_by问题
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

#时区
default-time_zone = '+8:00'

#内存监控
performance_schema_instrument ='memory%=counted'

#慢查询日志
#slow_query_log= on
#long_quert_time= 2
#slow-query-log-file = /home/mysql-slow-log.log
```

接下来分别映射数据库目录和配置文件目录，启动容器：
```
docker run --name mysql -d \
-v /usr/local/mysql/conf:/etc/mysql/conf.d \
-v /usr/local/mysql/data:/var/lib/mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

命令说明：
`-p 3306:3306`：将容器的3306端口映射到主机的3306端口
`-v /usr/local/mysql/conf:/etc/mysql/conf.d`：将主机`/usr/local/mysql/conf`目录挂载到容器的`/etc/mysql/conf.d`
`-e MYSQL_ROOT_PASSWORD=123456`：初始化root用户的密码
`--rm`：容器退出时就能够自动清理容器内部的文件系统，就是如果你退出容器，这个容器就删除了，需要重新docker run ...
`-d`：后台运行容器，并返回容器ID
`imageID`: mysql镜像ID

使用容器客户端连接：
`docker exec -it mysql mysql -uroot -p123`

查看容器启动情况：
`docker ps -a`

如果服务器开放了3306端口，那么现在可以通过navicat远程连接mysql了。

### 普通方式安装
[官网下载mysql安装包](https://dev.mysql.com/downloads/mysql/)
拉到页面下方，下拉选`Select Operating System`，选择`Linux-Generic`，`Select OS Version`选择64位。如果不想下载8.0，
可以选择右侧的`Looking for the latest GA version?`，会出现mysql5.7版本。
下载最下边的`Linux - Generic (glibc 2.12) (x86, 64-bit), TAR`，下载完，上传到linux服务器`/usr/local/src`位置。
或使用wget获取安装包：
`wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.28-linux-glibc2.12-x86_64.tar.gz`

解压缩：`tar -zxvf mysql-5.7...`
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
`cd /usr/local/mysql`

初始化：
`./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data `

报错：
`error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory`

如果已经安装了libnuma.so.1，先执行
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
或是这么找：`find / -type f -name 'my.cnf'`

```
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/tmp/mysql.sock

# 取消密码验证，一会改完密码这段还要注释掉
skip-grant-tables

#服务Id唯一
server-id = 1
port = 3306
log-error = /var/log/mysql/error.log

#只能用IP地址
skip_name_resolve

character-set-client-handshake = FALSE

#数据库默认字符集
character-set-server = utf8mb4

#数据库字符集对应一些排序等规则
collation-server = utf8mb4_general_ci

#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'

#最大连接数
max_connections = 1000

#解决only_full_group_by问题
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

#时区
default-time_zone = '+8:00'

#内存监控
performance_schema_instrument ='memory%=counted'

#慢查询日志
#slow_query_log= on
#long_quert_time= 2
#slow-query-log-file = /home/mysql-slow-log.log

[client]
socket=/usr/local/mysql/tmp/mysql.sock
default-character-set = utf8mb4

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
basedir就是mysql根目录，datadir就是上面在mysql根目录中新建的data文件夹。
socket我在mysql根目录中新建了一个tmp文件夹，然后这里就指向了它，mysql.sock这个文件在启动mysql时会自动创建，所以只要新建tmp文件夹就行了。 

修改tmp文件夹权限：
`chown -R mysql:mysql tmp`

将mysql加入服务
`cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql`
开机启动
`chkconfig mysql on`
启动mysql
`service mysql start`

还有一些启动方式作为参考：
```
#/etc/init.d/mysql start  或者  ./bin/mysqld_safe&
#/etc/init.d/mysql stop   或者  ./bin/mysqladmin -u root -p shutdown
```
可以使用`./bin/mysqld_safe&`命令启动mysql服务，启动后检查下`ps -ef|grep mysql`检查下是否启动
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
`/usr/local/mysql/bin/mysql -u root -p`

报错：
`mysql: [ERROR] unknown variable 'symbolic-links=0'`

查看my.cnf文件，文件中
`# Disabling symbolic-links is recommended to prevent assorted security risks symbolic-links=0`
应该是分两行展示了，修改该行为一行显示。

再次链接Mysql(由于/etc/my.cnf中设置了取消密码验证，所以此处密码直接回车即可)：
`/usr/local/mysql/bin/mysql -u root -p`

输入默认密码报错：
`Can't connect to local MySQL server through socket '/usr/local/mysql-5.7/tmp/mysql.sock' (2)`
此时需要检查配置文件`vi /etc/my.cnf`，socket文件路径是否正确，tmp文件夹是否已经创建。

成功登录mysql，执行sql报错：
`You must reset your password using ALTER USER statement before executing this statement.`

设置密码：
操作mysql数据库
`use mysql;`

修改root用户的密码：
`update user set authentication_string=password('你的密码') where user='root';`
`flush privileges;`
`exit;`

将/etc/my.cnf中的skip-grant-tables删除，然后登录再次设置密码（不知道为啥如果不再次设置密码就操作不了数据库了）
`/usr/local/mysql/bin/mysql -u root -p`

执行如下sql：
`alter user 'root'@'localhost' identified by '修改后的密码';`
注意mysql_5.7之前的版本需要把`authentication_string`替换为：`password`

允许远程连接
```
/usr/local/mysql/bin/mysql -u root -p
use mysql;
update user set host='%' where user = 'root';
flush privileges;
eixt;
```

添加快捷方式(可选)
`ln -s /usr/local/mysql/bin/mysql /usr/bin`

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

再次vi /etc/my.cnf。把skip-grant-tables删除掉。保存退出。完成MySql root密码的修改。

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

[参考地址](https://www.cnblogs.com/daemon-/p/9009360.html)

### windows安装mysql_8.0
1. [官网下载](https://www.mysql.com/downloads/)
2. 点击MySQL Community Edition (GPL) 下面的download，位于页面最下边。
3. 点击MySQL Community Server (GPL)下面的download
4. 页面最下边的有三种安装包。
安装版  .msi结尾的
Windows (x86, 64-bit), ZIP Archive 免安装版，解压即可。
Windows (x86, 64-bit), ZIP Archive Debug Binaries & Test Suite，带工具包版本。
这里下载第二个，免安装版，下载之后解压到你想要的位置。

5. 在解压的根目录新建一个 my.ini 的文件，然后加入以下内容：（basedir datadir 这两个的值一定要改成你自己的目录地址）
```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8

[mysqld] 
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=D:\install\mysql-8.0.15-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\install\mysql-8.0.15-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

6. 添加环境变量
我的电脑右键属性-->高级系统设置-->环境变量-->系统变量中找到Path-->点击编辑-->新建 加入Mysql的bin目录路径；

7. 在mysql的bin目录以管理员身份运行cmd，以此输入
`mysqld --install`安装
`mysqld --initialize`初始化
`net start mysql`运行
初始化后在Mysql的根目录会生成 data 数据库文件夹，如果启动服务失败，很有可能就是这个data文件夹有问题，删掉重新执行上边的步骤。

8. 登陆mysql
`mysql -u root -p`
此时需要输入mysql的默认密码，在mysql的安装目录下搜索*.err，以文本形式打开可以看到一些内容：
搜索文本`A temporary password is generated for root@localhost: `
localhost冒号之后的就是mysql的默认密码，复制，在cmd窗口右键，将密码黏贴，回车，登录成功。

9. 登录成功后必须修改密码才能进行其他操作。
`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的新密码';`

10. 配置远程连接mysql：
在mysql的bin目录下执行：
`mysql -u root -p密码`
`use mysql;`
`update user set host='%' where user ='root';`修改连接权限
`update user set plugin='mysql_native_password' where user ='root';`更改加密方式：
`flush privileges;`执行刷新权限

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

或使用指令查看是否安装成功：
`git --version`

## maven
1. 到理想目录下载maven：
`wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz`
2. 解压：
`tar -zxvf apache-maven-3.3.9-bin.tar.gz `
3. 改下名，方便后面操作：
`mv apache-maven-3.3.9 maven`
4. 配置环境变量
`vim /etc/profile`，在合适的位置添加如下内容：
`M2_HOME=/usr/local/maven`
`export PATH=${M2_HOME}/bin:${PATH}`
修改为自己maven的安装路径

然后使配置文件生效：
`source /etc/profile`
5. 检查是否安装成功
`mvn -v`
出现版本号：`Apache Maven 3.3.9...`，证明安装成功。

### windows安装maven
提前安装好jdk。

[官网下载zip包](http://maven.apache.org/download.cgi)
解压到理想的路径

添加M2_HOME环境变量
值为maven的安装路径

添加MAVEN_HOME环境变量
值为maven的安装路径

编辑系统变量`PATH`，在末尾添加
`%M2_HOME%\bin`
`%MAVEN_HOME%\bin`

验证
打开cmd，`mvn -v`，显示版本号证明配置成功！

## zookeeper
[下载地址](http://mirror.bit.edu.cn/apache/zookeeper/)
下载stable版本。

将下载好的.gz解压并复制到想要的位置。如：/usr/local/install 或 /home/wjy/install

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

### docker安装
- 查看zookeeper镜像
`docker search zookeeper`
- 拉取镜像
`docker pull zookeeper`
- 构建zookeeper容器
`docker run --name=zookeeper -p 2181:2181 -d --privileged zookeeper`

--name：指定容器别名为zookeeper
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
-d: 后台运行容器，并返回容器ID
-v /mysoft/zookeeper/data/:/data/ ：将宿主机的目录挂在到该镜像

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

停止redis报错` NOAUTH Authentication required`

### 停止redis
通过源码安装的redis，只能通过这种方式停止，或使用`kill -9`
`./redis-cli -h 你的ip -p 6379 -a 你的密码 shutdown`

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

问题：修改并重启redis之后发现配置并没有生效，原来启动时还需要指定配置文件。
`./redis-server ../redis.conf &`，然后`bg`
这样才算是指定了刚才修改的配置文件。

至于windows下的redis配置文件，同理。

### 配置远程链接redis
- 将`redis.conf`中的`bind 127.0.0.1`注释掉，或改成`0.0.0.0`
- 找到`protected-mode`，后边默认是`yes`，改成`no`
重启redis。

在windows的cmd或mac的终端输入
`redis-cli -h 你的服务器ip -p 6379 -a 你的密码`

远程连接成功。如果还是连不上，请检查你服务器的安全组，是否开放了6379端口。

### docker安装redis
- 搜索redis镜像
`docker search redis`
- 拉取官方镜像
`docker pull redis`
- 检查是否成功
`docker images`
- 配置data、conf
`mkdir /usr/local/docker/redis/data`
`mkdir /usr/local/docker/redis/conf`

修改redis.conf
```
#bind 127.0.0.1
protected-mode no
appendonly yes
requirepass yourpassword
```
protected-mode：在没有显示定义 bind 地址（即监听全网断），又没有设置密码 requirepass时，只允许本地回环 127.0.0.1 访问

- 构建redis容器：
```
docker run \
-p 6379:6379 \ # 端口映射 宿主机:容器
-v /usr/local/docker/redis/data:/data \ # 映射数据目录 rw 为读写
-v /usr/local/docker/redis/conf/redis.conf:/etc/redis/redis.conf \ # 挂载配置文件 ro 为readonly
--privileged=true \ # 给与一些权限
--name redis \ # 给容器起个名字
-d docker.io/redis:latest redis-server /etc/redis/redis.conf # deamon 运行 服务使用指定的配置文件
```
- 查看运行是否成功
`docker ps -a`

- 进入Docker容器redis的客户端
`docker exec -it 容器ID redis-cli`
- 查看设置的密码是否成功
`auth 刚才设置的密码`

## nginx
下载Nginx及相关组件
Linux系统是Centos 6.5 64位，如果不是root用户，切换到root用户下安装
`su root`
输入密码，密码不显示。

首先安装所需依赖：
`yum -y install gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel`

进入理想目录准备下载：
`cd /usr/local/src`

下载nginx tar包：
`wget http://nginx.org/download/nginx-1.17.8.tar.gz`

上边的版本号可能不是最新的，可以去[官网](http://nginx.org/download/)找最新的安装包。
注意列表不是有序的，现在是2020年，我在页面搜索2020，就可以看到最近发布的安装包，然后把上边的`nginx-1.17.8.tar.gz`替换成新的安装包名字。

解包：
`tar -xvf nginx-1.17.8.tar.g`

进入nginx目录：
`cd nginx-1.10.2`

安装：
`./configure && make && make install`

执行完，会在源码同一层生成名为nginx的目录。

启动nginx
先找一下nginx安装到什么位置上了
`whereis nginx`
`nginx: /usr/local/nginx`

进入nginx目录并启动：
```
cd /usr/local/nginx/sbin
./nginx
```
停止nginx：`./nginx -s stop`

[参考1](https://www.cnblogs.com/taiyonghai/p/6728707.html)
[参考2](https://www.cnblogs.com/bluestorm/p/4574688.html)

make报错：
`make: *** No rule to make target `build', needed by `default'. Stop.`
更新yum
`yum update`

安装前置库
`yum install -y gcc pcre pcre-devel openssl openssl-devel gd gd-devel`

进入nginx根目录重新编译一次参数：
```
./configure \

--prefix=/usr/local/nginx \

--pid-path=/usr/local/nginx/run \

--user=nginx \

--group=nginx \

--with-http_ssl_module \

--with-http_flv_module \

--with-http_stub_status_module \

--with-http_gzip_static_module \

--with-pcre \

--with-http_image_filter_module \

--with-debug \
```
ok了。

[make报错参考](https://blog.csdn.net/cailongbiaoyuli/article/details/84348866)

## solr
[官网下载最新的solr](https://lucene.apache.org/solr/downloads.html)
下载Binary releases-zip包，下载后上传到服务器`/usr/local/src`，cp到`/usr/local`下，解压`unzip solr-8.2.0.zip`。
进入bin目录，执行`./solr -start`

提示：
```
*** [WARN] ***  Your Max Processes Limit is currently 63473. 
 It should be set to 65000 to avoid operational disruption. 
 If you no longer wish to see this warning, set SOLR_ULIMIT_CHECKS to false in your profile or solr.in.sh
WARNING: Starting Solr as the root user is a security risk and not considered best practice. Exiting.
         Please consult the Reference Guide. To override this check, start with argument '-force'
```
根据提示再次执行：
`./solr start -force`
最后提示：
`Started Solr server on port 8983 (pid=27222). Happy searching!`
证明启动成功，打开服务器访问：
`你的ip:8983`
就可以看到solr的web页面了。

如果不能访问，可能是你服务器的端口没开，请设置服务器的安全组。

### 创建core
选择左侧CoreAdmin，再选择Add Core，输入核心的名称和实例路径，后边三个默认就好。
点击Add Core提交信息，这里我遇到了一个错误信息，如下：
`Can't find resource 'solrconfig.xml' in classpath orc`
意思是在核心目录下找不到相应的配置文件，解决办法：将下面路径的conf文件夹copy到新创建的core目录下 /usr/local/solr-8.0.0/server/solr/new_core。
`/usr/local/solr-8.2.0/server/solr/configsets/_default`
执行复制命令
`cp -r conf ../../new_core/`
然后再去创建，就成功了。

### 给solr添加访问密码
昨天因为没给solr添加访问密码，导致服务器被挖矿病毒攻击，服务器无法访问，数据库无法访问。还好今天把数据恢复了。

创建并编辑文件
`vim /usr/local/solr-8.2.0/server/etc/verify.properties`
输入如下内容：
```
#用户名 密码 权限
user:pass,admin
```
也可以配置多用户：
```
#用户名 密码 权限
user: pass,admin
user1: pass,admin
user3: pass,admin
```
继续编辑文件
`vim /usr/local/solr-8.2.0/server/contexts/solr-jetty-context.xml`
初始文件内容如下:
```
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">
<Configure class="org.eclipse.jetty.webapp.WebAppContext">
  <Set name="contextPath"><Property name="hostContext" default="/solr"/></Set>
  <Set name="war"><Property name="jetty.base"/>/solr-webapp/webapp</Set>
  <Set name="defaultsDescriptor"><Property name="jetty.base"/>/etc/webdefault.xml</Set>
  <Set name="extractWAR">false</Set>
</Configure>
```
添加如下内容，注意要写在Configure标签中：
```
<Get name="securityHandler">
 <Set name="loginService">
	 <New class="org.eclipse.jetty.security.HashLoginService">
		<Set name="name">verify—name</Set>
		<Set name="config"><SystemProperty name="jetty.home" default="."/>/etc/verify.properties</Set>
	 </New>
 </Set>
</Get>
```
继续编辑文件
`vim /usr/local/solr-8.2.0/server/solr-webapp/webapp/WEB-INF/web.xml`

找到如下内容：
```
  <!-- Get rid of error message -->
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Disable TRACE</web-resource-name>
      <url-pattern>/</url-pattern>
      <http-method>TRACE</http-method>
    </web-resource-collection>
    <auth-constraint/>
  </security-constraint>
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Enable everything but TRACE</web-resource-name>
      <url-pattern>/</url-pattern>
      <http-method-omission>TRACE</http-method-omission>
    </web-resource-collection>
  </security-constraint>
```
在下边添加
```
<security-constraint>
<web-resource-collection>
  <web-resource-name>Solr</web-resource-name>
  <url-pattern>/</url-pattern>
</web-resource-collection>   
 <auth-constraint>      
	<role-name>admin</role-name> 
 </auth-constraint> 
</security-constraint>
 
<login-config>      
	<auth-method>BASIC</auth-method> 
	<realm-name>verify-name</realm-name>   
</login-config>
```
然后启动solr，就会让输入用户名密码了。

另外也可以通过solr地址中加入用户名密码的方式登录，直接进入主界面,地址如下：
`http://user:pass@localhost:8088/solr/`

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

# 环境部署步骤
1. 安装jdk
2. 安装docker
3. 通过docker安装mysql
4. 通过docker安装zookeeper
5. 安装redis
6. 安装tomcat(为了安装jenkins)
7. war包安装jenkins
8. 安装maven
9. 安装git
10. 安装nginx

# 阿里云升级https
## 申请证书
登录阿里云->控制台->产品与服务->安全（云盾）->SSL证书(应用安全)->购买证书->免费型DV SSL->购买并支付(这里支付0.00，就是不用花钱)
然后回到控制台->SSL证书(应用安全)，可以看到刚才购买的免费SSL证书。

- 点击右侧的证书申请，填写申请人的详细信息，然后点击下一步，进入证书验证信息页面。在这个界面可以看到三个值：记录类型、主机记录和记录值。
- 找到你的域名供应商控制台，我的是腾讯云，进入控制台，在左上角`云产品`的下拉列表搜索`云解析`，进入会看到你的域名，点击右侧的解析，点击左上角的`添加记录`
- 根据提示分别填入上边的三个信息，然后确认保存，解析记录添加成功。
- 回到阿里云，点击验证，成功会提示：已经成功提交到CA公司，请您保持电话畅通，并及时查阅邮箱中来自CA公司的电子邮件。
- 安静的等待审核通过，一般十分钟就通过了，审核通过后会在ssl证书页面看到证书的状态是`已签发`。
- 点击右侧的下载，根据自己的服务器类型选择证书下载，我下载nginx版本，顺便打开帮助文档，[按照文档来](https://help.aliyun.com/knowledge_detail/95491.html)。

- 在Nginx的安装目录下创建cert目录，并且将下载的全部文件拷贝到cert目录中，注意这里先解压再上传到服务器，应该是两个文件，分别是.key和.pem结尾的。
- 打开`nginx.conf`配置文件，做如下配置：
```
server {
	listen 80 default backlog=2048;
	listen       443 ssl;
	server_name  api.zjxk12.com;

	# ssl开头的都是证书配置
	ssl_certificate      /usr/local/nginx/cert/api.zjxk12.com.pem;
	ssl_certificate_key  /usr/local/nginx/cert/api.zjxk12.com.key;
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
ssl开头的都是证书配置，要按照规定写，其他参数根据自己需求填写。
填完之后启动nginx报错：
`nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:105`
原因也很简单，nginx缺少http_ssl_module模块，编译安装的时候带上--with-http_ssl_module配置就行了，但是现在的情况是我的nginx已经安装过了，怎么添加模块，其实也很简单，往下看： 做个说明：我的nginx的安装目录是/usr/local/nginx这个目录，我的源码包在/usr/local/src/nginx-1.6.2目录

Nginx开启SSL模块
切换到源码包：
`cd /usr/local/src/nginx-1.11.3`
查看nginx原有的模块
`/usr/local/nginx/sbin/nginx -V`
在configure arguments:后面显示的原有的configure参数如下：
`--prefix=/usr/local/nginx --with-http_stub_status_module`
那么我们的新配置信息就应该这样写：
`./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module`
报错：`./configure: error: SSL modules require the OpenSSL library.`
那就安装它：
`yum -y install openssl openssl-devel`
然后继续执行上边的命令，就成功了。

上面的命令运行完，再执行：
`make`
这里不要进行make install，否则就是覆盖安装
然后备份原有已安装好的nginx
`cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak`

停止nginx，先找到进程号：
`netstat -nltp|grep 80`
`kill 13589`

然后将刚刚编译好的nginx覆盖掉原有的nginx（这个时候nginx要停止状态）
`cp ./objs/nginx /usr/local/nginx/sbin/`
提示：`cp overwrite...?`
此时输入`y`，然后回车，直接回车则不执行任何操作。

然后启动nginx，在nginx的sbin目录下执行
`./nginx`

通过命令查看是否已经加入成功
`/usr/local/nginx/sbin/nginx -V`

[参考地址](https://www.cnblogs.com/ghjbk/p/6744131.html)
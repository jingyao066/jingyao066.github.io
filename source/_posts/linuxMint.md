---
title: linuxMint
tags: linux
date: 2019-03-16 12:37:30
---

一直想用deepin，奈何挣扎了一天都没装上(笔记本的问题)，耗尽了我的耐心，然后安装了linuxMint。下面记录踩坑之路。

# 安装必要指令
## gedit
`sudo apt-get install gedit`
文件编辑器，比vi、vim好用一些。

# 软件安装
## jdk
首先卸载系统自带的jdk，查看安装的软件包
sudo dpkg --list | grep -i jdk

删除jdk
sudo apt-get purge openjdk*

删除其他的包
sudo apt-get purge icedtea-* openjdk-*
cd /usr/lib/jvm （如果没有jvm这个目录，就不用管这步了）
sudo rm -rf jdk<version>

再次确认是否被删除
sudo dpkg --list | grep -i jdk

https://www.oracle.com/technetwork/java/javase/downloads/index.html
下载想要的jdk，下载完成后，解压到/opt目录
`sudo tar zxvf jdk.tar.gz -C /opt/`
注意将jdk改成自己下载的jdk名字

下面配置环境变量，需要提前安装gedit，如上。
gedit安装好后，`sudo gedit /etc/profile`
在 profile 文件的后面加上以下语句
```
# JAVA Config
export JAVA_HOME=/opt/jdk1.8.0_101
export JRE_HOME=/opt/jdk1.8.0_101/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
注意将jdk名字改成自己的。

使文件立即生效
`source /etc/profile`
在终端输入java -version，看到版本号说明成功。


## node
https://nodejs.org/zh-cn/download/
官网下载linux版本的二进制文件
`xz -d  node-v8.11.1-linux-x64.tar.xz`
上边指令执行完后，文件末尾没有.xz了，然后将文件解压到想要的目录
`sudo tar zxvf node_10.15.tar -C /home/wjy/install`

添加链接，可以全局使用命令
`ln -s /root/node-10.15/bin/node /usr/local/bin/node`
`ln -s /root/node-10.15/bin/npm /usr/local/bin/npm`

执行完在终端任意目录 输入 node -v，看到版本号说明成功。

## snap
```
sudo apt-get install snapd 
sudo apt-get install snapcraft 
```

- 列出已经安装的snap包
`sudo snap list`

- 搜索要安装的snap包
`sudo snap find <text to search>`

- 安装一个snap包
`sudo snap install <snap name>`

- 更新一个snap包，如果你后面不加包的名字的话那就是更新所有的snap包
`sudo snap refresh <snap name>`

- 把一个包还原到以前安装的版本
`sudo snap revert <snap name>`

- 删除一个snap包
`sudo snap remove <snap name>` 

- 安装网易云音乐
`sudo snap install netease-music --devmode --beta`

- 安装微信
`sudo snap install electronic-wechat`
需要重启才能在左下角管理中看到微信

## git
更新linux mint 内核及软件包
```
sudo apt-get update 
sudo apt-get upgrade
```

使用dpkg包管理器查询是否已安装git
`sudo dpkg -l git`


通过apt-get 安装软件库中最新版本的git
```
sudo apt-get install git
```

检查是否安装成功
`git --version`

### idea中配置git
使用apt-get install 方式安装的软件，
1.下载的软件存放位置 
/var/cache/apt/archives

2.安装后软件默认位置 
/usr/share

3.可执行文件位置 
/usr/bin

4.配置文件位置 
/etc

5.lib文件位置 
/usr/lib

所以在idea中配置git的可执行文件，选择位置use/bin/git
点击test，显示git版本号，说明成功，就可以从github或coding上拉取代码了。

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

cd到目录下，执行
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

## postman
https://www.getpostman.com/downloads/
下载对应版本.gz,并解压到理想位置。

创建全局变量，也就是在任何地方都可以执行postman，不用去到安装目录，执行
`sudo ln -s /home/wjy/install/Postman/Postman  /usr/bin/postman`

添加启动器应用图标，也就是可以从启动器快速启动，执行
`vim /usr/share/applications/postman.desktop`
填写如下内容：注意修改路径
```
[Desktop Entry]
Encoding=UTF-8
Name=Postman
Exec=postman
Icon=/home/wjy/install/Postman/resources/app/assets/icon.png
Terminal=false
Type=Application
Categories=Development;
```
然后就可以在启动器搜索到postman，然后可以将该应用添加到桌面。

## navicat
官网下载：https://www.navicat.com.cn/download/navicat-premium
直接下载试用版，解压到理想位置。
进入到解压好的目录,启动
`./start_navicat`
启动成功。

将navicat快捷方式添加到桌面
`vim /usr/share/applications/navicat.desktop`
填写如下内容：注意修改路径
```
[Desktop Entry]
Encoding=UTF-8
Name=Navicat Premium
Comment=The Smarter Way to manage dadabase
Exec=/bin/sh "/home/wjy/install/navicat121_premium_cs_x64/start_navicat"
Icon=/home/wjy/install/navicat121_premium_cs_x64/navicat.png
Categories=Application;Database;MySQL;navicat
Version=1.0
Type=Application
Terminal=0
```
然后就可以在启动器搜索到navicat，然后可以将该应用添加到桌面。

编辑解压目录下的start_navicat文件，修改export LANG=”en_US.UTF-8”，改为export LANG=”zh_CN.UTF-8”。
进入后发现还是部分乱码，这时需要打开navicat，点击上边选项卡的第五个，为工具，点击下拉选最后一个选项，然后将常规、编辑器、记录三个位置的字体改成能正常显示的字体。


## mysql
安装
`sudo apt-get install mysql-server mysql-common mysql-client`
`sudo /etc/init.d/mysql restart`

设置初始密码：
`sudo mysql_secure_installation`
按照引导设置密码

登录
`sudo mysql -u root -p`
必须带sudo，不然会出现`Access denied for user ‘root‘@‘localhost‘`
然后输入刚才设置的密码。

解决Access denied 问题：
参考地址：https://www.cnblogs.com/cpl9412290130/p/9583868.html

终端输入：
`sudo gedit /etc/mysql/mysql.conf.d/mysqld.cnf`
然后在这个配置文件中的[mysqld]这一块中加入skip-grant-tables这句话，位置随意，直接加在第一行即可。
作用：就是让你可以不用密码登录进去mysql。
重启mysql：`service mysql restart`

在终端上输入mysql -u root -p，遇见输入密码的提示直接回车即可,进入mysql后，分别执行下面三句话：
```
use mysql;   然后敲回车
update user set authentication_string=password("你的密码") where user="root";  然后敲回车
flush privileges;  然后敲回车
```
然后输入quit，退出mysql。

重新进入到mysqld.cnf文件中去把刚开始加的skip-grant-tables这条语句给注释掉。

再返回终端输入mysql -u root -p，应该就可以进入数据库了。

如果此时还是报出错误，那么就需要返回step3中，把注释掉的那条语句重新生效（就是删除#符号），重新进入mysql中，先选择一个数据库（use mysql;）,然后输入select user,plugin from user;
可以看到在执行了select user,plugin from user;后，错误原因是因为plugin root的字段是auth_socket，那我们改掉它为下面的mysql_native_password就行了。输入：
`update user set authentication_string=password("ln122920"),plugin='mysql_native_password' where user='root';`
然后回车执行以下，再输入select user,plugin from user;回车，我们能看到root用户的字段改成功了。
最后quit退出,再次mysql -u root -p，成功！

重启服务：
`service mysql restart`

修改密码级别：
`set global validate_password_policy=0;`
`set global validate_password_length=4;`

## beyond compare
https://www.scootersoftware.com/download.php?zz=kb_linux_install
查看官网，通过官网提示安装
```
wget https://www.scootersoftware.com/bcompare-4.2.9.23626_amd64.deb
sudo apt-get update
sudo apt-get install gdebi-core
sudo gdebi bcompare-4.2.9.23626_amd64.deb
```
卸载
`sudo apt-get remove bcompare`

# 一些问题
## 快捷键
+ 在应用程序中搜索"窗口管理器"，然后点击键盘，将：
    - "工作区"和"移动窗口至工作去"等与ctrl和alt相关的快捷键全部清除
    - 在窗口管理器-键盘，找到"显示桌面"，将快捷键改为**super+D**(保持与windows一致)
+ 在应用程序中搜索"键盘"，应用程序和快捷键选项卡，找到：
    - /usr/bin/xflock4，修改为Super+L
    - xfce4-appfinder，保留原快捷键
    - x-terminal-emulator，修改为Alt+/
+ ctrl+alt+s
这个快捷键是idea的setting的快捷键，但是在mint中被fcitx输入法占用了。
右键右下角的输入法，选择配置fcitx->全局配置->显示高级选项，找到想修改的快捷键，建议直接置空(选中后，点击esc)。

## VI模式中上下左右键和回退键出现字母
`sudo vi /etc/vim/vimrc.tiny`
上下左右字母：
这个文件里面的倒数第二句话是“set compatible”改为“set nocompatible”

回退健：
在“set nocompatible”后面(下一行)加上
set backspace=2

## ctrl+alt+f12
不只是linuxmint，Ubuntu、deepin等按这组快捷键都会黑屏，不要害怕，不是黑屏，ctrl+alt+f7可以返回。
ALT＋CTRL+F1——F6可进入6个终端，F7开始到F12有6个图形界面，f7是我们安装的图形界面，所以ctrl+alt+f7可以返回。
参考地址：https://askubuntu.com/questions/277517/what-does-ctrl-alt-f12-do

## 主文件夹的中文文件夹改为英文
因为是中文版，/home/wjy 下的用户目录都是中文的，在终端操作很不方便，需要切换输入法。而且纯命令行界面，中文文件夹会乱码，根本不知道是哪个是哪个。
打开终端，在终端中输入命令：
```
export LANG=en_US
xdg-user-dirs-gtk-update
```
跳出对话框询问是否将目录转化为英文路径，同意并关闭。
在终端中输入命令：
`export LANG=zh_CN`
关闭终端，重启系统。下次进入系统，系统会提示是否把转化好的目录改回中文，选择不再提示，并取消修改(保留旧的，旧的就是重启之前改好的英文文件夹)。这样就完成了


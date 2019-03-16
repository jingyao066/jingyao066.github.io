---
title: linuxMint
tags: 其他
date: 2019-03-16 12:37:30
---

以后办公的笔记本就用linux了。一直想用deepin，奈何挣扎了一天都没装上(笔记本的问题)，耗尽了我的耐心，然后安装了linuxMint。下面记录踩坑之路。

# 安装必要指令
## gedit
`sudo apt-get install gedit`

# 安装jdk
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
# JAVA Config
```
export JAVA_HOME=/opt/jdk1.8.0_101
export JRE_HOME=/opt/jdk1.8.0_101/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
注意将jdk名字改成自己的。

使文件立即生效
`source /etc/profile`
在终端输入java -version，看到版本号说明成功。


# node
https://nodejs.org/zh-cn/download/
官网下载linux版本的二进制文件
`xz -d  node-v8.11.1-linux-x64.tar.xz`
上边指令执行完后，文件末尾没有.xz了，然后将文件解压到想要的目录
`sudo tar zxvf node_10.15.tar -C /home/wjy/install`

添加链接，可以全局使用命令
`ln -s /root/node-10.15/bin/node /usr/local/bin/node`
`ln -s /root/node-10.15/bin/npm /usr/local/bin/npm`

执行完在终端任意目录 输入 node -v，看到版本号说明成功。

# snap
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




---
title: gitException
tags: git
date: 2018-12-25 10:14:46
---

记录一些git的异常，以及解决方式

## Please make sure you have the correct access rights and the repository exists.
1.重新设置git的名字和邮箱
```
git config --global user.name "yourname"
git config --global user.email“your@email.com"
```
注：yourname是你要设置的名字，your@email是你要设置的邮箱，建议使用github的用户名和邮箱。

2.删除.ssh文件夹
默认的路径是：
`C:\Users\wjy\.ssh`
直接删除该文件夹。因为远程ssh文件夹下包含了邮箱信息等。

3.git执行命令
`ssh-keygen -t rsa -C "your@email.com"`
填写刚才设置的邮箱

4.接下来请一路回车
然后系统会自动在.ssh文件夹下生成两个文件，id_rsa和id_rsa.pub，用记事本打开id_rsa.pub，将内容全部复制。

5.打开github-settings
单机右上角头像--settings，选择SSH-GPG keys，选择New SSH Key，title随便起，Key填写刚才复制id_rsa.pub中的内容，然后点击add ssh key。

6.git执行命令
`ssh -T git@github.com`
然后跳出一堆话，输入 yes

7.重启git bush，就可以提交了。
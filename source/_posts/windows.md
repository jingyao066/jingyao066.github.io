---
title: windows
tags: other
date: 2019-09-28 16:39:52
---

# 查看端口占用
查看全部端口：
`netstat -ano`

查看指定端口：
`netstat -ano | findstr 8080`

找到被占用端口的`PID`，就是上边结果的最后一行，并输入
`tasklist | findstr "上边的PID"`
可以看到被占用的程序

可以通过程序名结束掉它：
`taskkill /f /t /im Tencentdl.exe。`

或直接通过`PID`结束它：
`taskkill /f /t /im 上边的PID`

或通过任务管理器结束该程序，如果看不到`PID`，随意找到一选项卡右键，勾选`PID`即可，然后可以右键结束掉该程序。
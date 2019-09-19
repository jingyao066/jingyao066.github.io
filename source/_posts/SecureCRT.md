---
title: SecureCRT
tags: 其他
date: 2019-05-23 17:04:38
---

# 设置背景颜色 
永久设置背景色：
Options->Global Options->Edit Default Settings->Appearance->Current color scheme
选一个喜欢的背景色，然后点击ok，然后选择`Chanage ALL sessions(no undo)`

Options => Sessions options => Terminal => Emulation， 
在 Terminal下拉列表下选择X-term，勾选 ANSI Color。

Options>>Global Options>>Terminal>>Appearance>>ANSI Color>>Normal colors中八种颜色分别点开设置：
- 128,240,25
- 20,240,120
- 80,240,75
- 140,240,120
- 40,240,120
- 80,240,180
- 120,240,75
- 124,20,132

再设置Bold colors，这八种颜色大部分默认就行，从左到右（Hub,Sat,Lum）依次为：
- 160,0,120
- 0,240,120
- 80,240,120
- 40,240,120
- 160,240,180
- 200,240,120
- 120,240,120
- 124,20,132

# 设置全局字体和字号
Options->Global Options->Default Session->Edit Default Settings->Appearance->Fonts
字体设置为：Consolas
字号设置为：14pt

# 设置光标的颜色
options ->Session Options->Appearance->Cursor
勾选`Use color`，选择一个比较亮的颜色。
---
title: idea-Setting
tags: idea
date: 2018-12-08 20:55:11
---
记录自己idea的个性化设置。
不过最好的办法还是直接在idea中：
file->export setting

# 注释模板
1.类注释模板
`File ---> Setting ---> Editor ---> File and Code Templates ---> Includes --->File Header`
```
/**
 *@author: wjy
 *@Date: Create in ${TIME} ${DATE}
 *@description :
 */
```
idea设置页的下边，有变量的解释

2.方法注释模板
File ---> Setting ---> Editor ---> LiveTemplates
点击右边上面那个绿色的+号，选择Template Group双击，然后弹出一个窗口，起名，然后点击ok
然后在列表中，选择刚才新建的Template Group，再次点击右上角的绿色加号，选择第一个选项 live Template
![idea-1](https://user-images.githubusercontent.com/21356733/46934277-c0e30300-d089-11e8-81e6-933179e7066b.png)
注释模板中$...$中间的参数要与你刚才设置的参数名一一对应
按下 /* 加上你定义的快捷键，再按tab就能生成方法模板了~
ps:为了只按/+自定义的快捷键，我们可以在模板的顶端，注意要顶格，
加上两个**
完成后点击应用ok~

提供两种模板：
```
**
 * @author: wjy 
 * @description: 
 * @date: $date$ $time$  
 * @param: $params$  
 * @return: $returns$  
 */ 
```
```
**
 * @author: wjy 
 * @description:   
 */ 
```

# 窗口开多后自动关闭
idea 默认的窗口数量只有10个。
![tablimit](https://user-images.githubusercontent.com/21356733/47202544-13852d80-d3b0-11e8-9619-62698e96dc34.png)

# idea输入法不跟随
jetbrains官方已经在2018.2版本修复了这个问题。但是还是需要新版jdk的加持。
目前可以确定，idea--2018.3版本+jdk1.8.0_191没有问题。
win10自带的输入法也是没有问题的。
如果jdk是升级上来的，就是本机安装的事1.8.0_191以下的版本，然后通过官方升级上来的，貌似就不行。建议去官网重新下载新版jdk。
这个问题不同环境需要不同的解决方法，多试几种就是了。

先摘抄几种网上的方法

方法1：
idea界面按ctrl+shift+a，输入switch IDE Boot JDK，选择自己安装的jdk，默认是idea自带的jre，貌似是有bug。选择完之后重启

方法2：
增加一个环境变量即可。 64位增加的名称为IDEA_JDK_64，32位的为IDEA_JDK，值为本机jdk根目录。

比较靠谱的就这两种，还有很多说jdk降级，输入法降级的，不建议。

# 去掉重复代码检查

---
title: Maven详解
tags: Maven
date: 2018-12-11 11:41:35
---

## 概念：
Maven用于叙述项目间的依赖关系。
通俗讲，就是通过pom.xml文件的配置获取jar包，而不用手动去添加jar包。
当然maven还有其他功能，如：构建项目

如果需要使用pom.xml来获取jar包，那么首先该项目就必须为maven项目，maven项目可以这样去想，就是在java项目和web项目的上面包裹了一层maven，
本质上java项目还是java项目，web项目还是web项目，但是包裹了maven之后，就可以使用maven提供的一些功能了(通过pom.xml添加jar包)。

如何通过pom.xml获取想要的jar包呢？
pom.xml获取junit的jar包的编写。
![](Maven详解/1.png)
为什么通过groupId、artifactId、version三个属性就能定位一个jar包？

加入上面的pom.xml文件属于A项目，那么A项目肯定是一个maven项目，通过上面这三个属性能够找到junit对应版本的jar包，那么junit项目肯定也是一个maven项目，junit的maven项目中的pom.xml文件就会有三个标识符，比如像下图这样，然后别的maven项目就能通过这三个属性来找到junit项目的jar包了。所以，在每个创建的maven项目时都会要求写上这三个属性值的。
![](Maven详解/2.png)

## 仓库概念
通过pom.xml中的配置，就能够获取到想要的jar包，这些jar包，
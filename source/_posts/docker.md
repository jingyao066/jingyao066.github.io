---
title: docker
tags: 部署
date: 2019-03-05 10:26:24
---

# 概述
Docker是一个开源项目，诞生于2013年初，最初是dotCloud公司内部的一个业余项目。它基于Google公司推出的Go语言实现。项目后来加入了Linux基金会，遵从了Apache2.0协议，项目代码在GitHub上进行维护。

Docker项目的目标是实现轻量级的操作系统虚拟化解决方案。Docker的基础是Linux容器(LXC)等技术。在LXC的基础上Docker进行了进一步封装，让用户不用去关心容器的管理，使得操作更为简便，用户操作docker容器就像操作一个轻量级虚拟机一样简单。

下面的图片比较了 Docker 和传统虚拟化方式的不同之处，可见docker是在操作系统上实现虚拟化，直接复用本地主机的操作系统，而传统方式则是在硬件层面实现。
![](docker/1.png)

# 为什么使用docker
作为一种新兴的虚拟化方式，docker跟传统的虚拟化方式相比具备众多的优势。

* 更快速的交付和部署
Docker在整个开发周期都可以完美的辅助你实现快速交付。Docker允许开发者在装有应用和服务的本地容器中做开发，可以直接集成到可持续开发流程中。
例如：开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。 Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。 Docker 容器很轻很快！容器的启动时间是秒级的，大量地节约开发、测试、部署的时间。

* 高效的部署和扩容
Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。
Docker的兼容性和轻量特性可以很轻松的实现负载的动态管理。你可以快速扩容或方便的下线的你的应用和服务，这种速度趋近实时。

* 更高的资源利用率
Docker 对系统资源的利用率很高，一台主机上可以同时运行数千个 Docker 容器。容器除了运行其中应用外，基本不消耗额外的系统资源，使得应用的性能很高，同时系统的开销尽量小。传统虚拟机方式运行 10 个不同的应用就要起 10 个虚拟机，而Docker 只需要启动 10 个隔离的应用即可。

* 更简单的管理
使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。

举个最简单的例子：
传统的javaWeb应用是部署在tomcat中的，一个服务器可能运行多个tomcat，每个tomcat中有一个或多个web应用，那么此时docker就可以起到应用隔离和资源独立的作用。
![](docker/7.png)

- 如果不使用docker，一台服务器上多个tomcat，即A1，所有的应用共享服务器的CPU、内存等资源，这时如果一个应用出现问题，导致CPU爆满等等，其他应用也会无法运行。
- 那么这时就需要做到资源独立，一个应用一份资源，即我们所说的传统的虚拟化方式，即A2，这样需要在服务器上安装数个虚拟机，每个虚拟机运行一个应用，这样的缺点是，重启虚拟机太麻烦，迁移应用需要重装虚拟机，配置环境等等。
- A3即使用docker，此时服务器上运行多个容器，每个容器都拥有独立的CPU、内存等资源，完全满足了应用隔离的需求，重启容器只需要几秒，迁移应用也很方便，所有装了Docker的服务器，只需把镜像pull或者load进去，run，就可以了。

![](docker/8.png)
上图是docker的运行流程

# docker引擎
docker引擎是一个c/s结构的应用，主要组件见下图：
![](docker/2.png)

+ server是一个常驻进程
+ REST API实现了server和client之间的交互协议
+ CLI实现容器和镜像的管理，为用户提供统一的管理界面

# docker架构
Docker使用C/S架构，Client 通过接口与Server进程通信实现容器的构建，运行和发布。client和server可以运行在同一台集群，也可以通过跨主机实现远程通信。
![](docker/3.png)

# 核心概念
+ 镜像(Image)

docker镜像就是一个只读的模板，如：一个镜像可以包含一个完整的操作系统环境，里面仅安装了Apache或用户需要的其他应用程序。
镜像可以用来创建docker容器，一个镜像可以创建很多容器。Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

镜像（Image）就是一堆只读层（read-only layer）的统一视角，也许这个定义有些难以理解，看看下面这张图：
![](docker/4.png)
右边我们看到了多个只读层，它们重叠在一起。除了最上面一层，其它层都会有一个指针指向下一层。这些层是Docker内部的实现细节，并且能够在docker宿主机的文件系统上访问到。统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。

+ 仓库(repository)
仓库是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器混为一谈，并不严格区分。实际上，仓库注册服务器往往存放着多个仓库，每个仓库又包含了多个镜像，每个镜像有不同的标签(tag)。

仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括 时速云 、网易云 等，可以提供大陆用户更稳定快速的访问。当然，用户也可以在本地网络内创建一个私有仓库。
当用户创建了自己的镜像之后就可以使用 push 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 pull 下来就可以了。
Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。

+ 容器(container)
docker利用容器来运行应用。容器时从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。
容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。
![](docker/5.png)
一个运行态容器被定义为一个可读写的统一文件系统加上隔离的进程空间和包含其中的进程。下面这张图片展示了一个运行中的容器。
![](docker/6.png)
正是文件系统隔离技术使得Docker成为了一个非常有潜力的虚拟化技术。一个容器中的进程可能会对文件进行修改、删除、创建，这些改变都将作用于可读写层。

# docker安装部署
- Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。
通过 uname -r 命令查看你当前的内核版本
`uname -r`

- 使用 root 权限登录 Centos。确保 yum 包更新到最新。
`sudo yum update`

- 卸载旧版本(如果安装过旧版本的话)
`sudo yum remove docker  docker-common docker-selinux docker-engine`

- 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
`sudo yum install -y yum-utils device-mapper-persistent-data lvm2`

- 设置yum源
这是dockerHub上的镜像源：
`sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
我们使用阿里的镜像源：
`sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`

- 查看仓库中所有docker版本，并选择特定版本安装
`yum list docker-ce --showduplicates | sort -r`

- 安装docker
`sudo yum install docker-ce`#由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
可以指定版本安装，如：
`sudo yum install docker-ce-18.03.0.ce`

- 启动并加入开机启动
`sudo systemctl start docker`
`sudo systemctl enable docker`

- 验证安装是否成功(有client和service两部分表示docker安装启动都成功了)
`docker version`

# 删除docker
`yum remove docker docker-common docker-selinux docker-engine -y`
`/etc/systemd -name '*docker*' -exec rm -f {} ;`
`find /etc/systemd -name '*docker*' -exec rm -f {} \;`
`find /lib/systemd -name '*docker*' -exec rm -f {} \;`

# docker常用命令
## 获取镜像
`docker pull`
从仓库获取所需镜像，示例如下:
`docker pull centos:centos6`

实际上相当于 docker pull registry.hub.docker.com/centos:centos6
命令，即从注册服务器 registry.hub.docker.com 中的 centos 仓库来下载标记为 centos6 的镜像。
有时候官方仓库注册服务器下载较慢，可以从其他仓库下载。 从其它仓库下载时需要指定完整的仓库注册服务器地址。

## 查看镜像列表
`docker images`
列出了所有顶层（top-level）镜像。实际上，在这里我们没有办法区分一个镜像和一个只读层，所以我们提出了top-level镜像。
只有创建容器时使用的镜像或者是直接pull下来的镜像能被称为顶层（top-level）镜像，并且每一个顶层镜像下面都隐藏了多个镜像层。

如：
```
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos                   centos6             6a77ab6655b9        8 weeks ago         194.6 MB
ubuntu                   latest              2fa927b5cdd3        9 weeks ago         122 MB
```

在列出信息中，可以看到几个字段信息

- 来自于哪个仓库，比如 ubuntu
- 镜像的标记，比如 latest
- 它的 ID 号（唯一）
- 创建时间
- 镜像大小

## 利用dockerfile来创建镜像
`docker build`
使用 docker commit 来扩展一个镜像比较简单，但是不方便在一个团队中分享。我们可以使用docker build 来创建一个新的镜像。为此，首先需要创建一个Dockerfile，包含一些如何创建镜像的指令。新建一个目录和一个Dockerfile。
```
mkdir test_docker
cd test_docker
touch Dockerfile
```

Dockerfile 中每一条指令都创建镜像的一层，例如：
```
FROM centos:centos6
MAINTAINER sandywei <sandy@test.tech>
# move all configuration files into container

RUN yum install -y httpd
EXPOSE 80
CMD ["sh","-c","service httpd start;bash"]
```

### dockerfile基本语法：
Dockfile是一个用于编写docker镜像生成过程的文件，其有特定的语法。在一个文件夹中，如果有一个名字为Dockfile的文件，其内容满足语法要求，在这个文件夹路径下执行命令:docker build --tag name:tag .，就可以按照描述构建一个镜像了。name是镜像的名称，tag是镜像的版本或者是标签号，不写就是lastest。注意后面有一个空格和点。
Dockerfile的基本指令有十三个，分别是：FROM、MAINTAINER、RUN、CMD、EXPOSE、ENV、ADD、COPY、ENTRYPOINT、VOLUME、USER、WORKDIR、ONBUILD。下面对这些指令的用法一一说明。
#### FROM
用法：FROM <image>
说明：第一个指令必须是FROM了，其指定一个构建镜像的基础源镜像，如果本地没有就会从公共库中拉取，没有指定镜像的标签会使用默认的latest标签，可以出现多次，如果需要在一个Dockerfile中构建多个镜像。

#### MAINTAINER
用法：MAINTAINER <name> <email>
说明：描述镜像的创建者，名称和邮箱

#### RUN
用法：RUN "command" "param1" "param2"
说明：RUN命令是一个常用的命令，执行完成之后会成为一个新的镜像，这里也是指镜像的分层构建。一句RUN就是一层，也相当于一个版本。这就是之前说的缓存的原理。我们知道docker是镜像层是只读的，所以你如果第一句安装了软件，用完在后面一句删除是不可能的。所以这种情况要在一句RUN命令中完成，可以通过&符号连接多个RUN语句。RUN后面的必须是双引号不能是单引号（没引号貌似也不要紧），command是不会调用shell的，所以也不会继承相应变量，要查看输入RUN "sh" "-c" "echo" "$HOME"，而不是RUN "echo" "$HOME"。

#### CMD
用法：CMD command param1 param2
说明：CMD在Dockerfile中只能出现一次，有多个，只有最后一个会有效。其作用是在启动容器的时候提供一个默认的命令项。如果用户执行docker run的时候提供了命令项，就会覆盖掉这个命令。没提供就会使用构建时的命令。

#### EXPOSE
用法：EXPOSE <port> [<port>...]
说明：告诉Docker服务器容器对外映射的容器端口号，在docker run -p的时候生效。

#### EVN
用法：EVN <key> <value> 只能设置一个

	  EVN <key>=<value>允许一次设置多个
说明：设置容器的环境变量，可以让其后面的RUN命令使用，容器运行的时候这个变量也会保留。

#### ADD
用法：ADD <src>   <dest>
说明：复制本机文件或目录或远程文件，添加到指定的容器目录，支持GO的正则模糊匹配。路径是绝对路径，不存在会自动创建。如果源是一个目录，只会复制目录下的内容，目录本身不会复制。ADD命令会将复制的压缩文件夹自动解压，这也是与COPY命令最大的不同。

#### COPY
用法：COPY <src> <dest>
说明：COPY除了不能自动解压，也不能复制网络文件。其它功能和ADD相同。

#### ENTRYPOINT
用法：ENTRYPOINT "command" "param1" "param2"
说明：这个命令和CMD命令一样，唯一的区别是不能被docker run命令的执行命令覆盖，如果要覆盖需要带上选项--entrypoint，如果有多个选项，只有最后一个会生效。

#### VOLUME
用法：VOLUME ["path"]
说明：在主机上创建一个挂载，挂载到容器的指定路径。docker run -v命令也能完成这个操作，而且更强大。这个命令不能指定主机的需要挂载到容器的文件夹路径。但docker run -v可以，而且其还可以挂载数据容器。

#### USER
用法：USER daemon
说明：指定运行容器时的用户名或UID，后续的RUN、CMD、ENTRYPOINT也会使用指定的用户运行命令。

#### WORKDIR
用法:WORKDIR path
说明：为RUN、CMD、ENTRYPOINT指令配置工作目录。可以使用多个WORKDIR指令，后续参数如果是相对路径，则会基于之前的命令指定的路径。如：WORKDIR  /home　　WORKDIR test 。最终的路径就是/home/test。path路径也可以是环境变量，比如有环境变量HOME=/home，WORKDIR $HOME/test也就是/home/test。

#### ONBUILD
用法：ONBUILD [INSTRUCTION]
说明：配置当前所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。意思就是，这个镜像创建后，如果其它镜像以这个镜像为基础，会先执行这个镜像的ONBUILD命令。

###	Dockerfile例子
一个使用安装包安装的tomcat例子：
```
FROM centos
MAINTAINER nobody "xx@qq.com"
RUN mkdir -p /opt/jdk/
RUN mkdir -p /opt/tomcat/
ADD jdk1.7.0_79 /opt/jdk/
ADD tomcat  /opt/tomcat/
ENV CATALINA_HOME /opt/tomcat
ENV JAVA_HOME /opt/jdk
EXPOSE 8080
ENV PATH $PATH:$JAVA_HOME/bin
CMD ["/opt/tomcat/bin/catalina.sh","run"]
```

更详细的语法说明请参考 https://docs.docker.com/engine/reference/builder/

编写完成 Dockerfile 后可以使用 docker build 来生成镜像。
```
$ docker build -t test/httpd:1.0 .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM centos:centos6
 ---> 6a77ab6655b9
Step 2 : MAINTAINER sandywei <sandy@test.tech>
 ---> Running in 1b26493518a7
 ---> 8877ee5f7432
Removing intermediate container 1b26493518a7
Step 3 : RUN yum install -y httpd
 ---> Running in fe5b6f1ef888

 .....

 Step 5 : CMD sh -c service httpd start
 ---> Running in b2b94c1601c2
 ---> 5f9aa91b0c9e
Removing intermediate container b2b94c1601c2
Successfully built 5f9aa91b0c9e
```
其中 -t 标记来添加 tag，指定新的镜像的用户信息。 “.” 是 Dockerfile 所在的路径（当前目录），
也可以替换为一个具体的 Dockerfile 的路径。注意一个镜像不能超过 127 层。

用docker images 查看镜像列表
```
$ docker images
REPOSITORY                 TAG               IMAGE ID            CREATED             SIZE
test/httpd               1.0               5f9aa91b0c9e        3 minutes ago       292.4 MB
centos                   centos6             6a77ab6655b9        8 weeks ago         194.6 MB
ubuntu                   latest              2fa927b5cdd3        9 weeks ago         122 MB
```
可以发现最后一层的ID（5f9aa91b0c9e）和 image id是一样的

## 上传镜像
`docker push`
用户可以通过 docker push 命令，把自己创建的镜像上传到仓库中来共享。例如，用户在 Docker Hub 上完成注册后，可以推送自己的镜像到仓库中。

运行实例：
`$ docker push test/httpd:1.0`

## 创建容器
`docker create <image-id>`
docker create 命令为指定的镜像（image）添加了一个可读写层，构成了一个新的容器。注意，这个容器并没有运行。

docker create 命令提供了许多参数选项可以指定名字，硬件资源，网络配置等等。

如：
创建一个centos的容器，可以使用仓库＋标签的名字确定image，也可以使用image－id指定image。返回容器id
```
＃查看本地images列表
$ docker images

＃用仓库＋标签
$ docker create -it --name centos6_container centos:centos6

＃使用image－id
$ docker create -it --name centos6_container 6a77ab6655b9 bash
b3cd0b47fe3db0115037c5e9cf776914bd46944d1ac63c0b753a9df6944c7a67

#可以使用 docker ps查看一件存在的容器列表,不加参数默认只显示当前运行的容器
$ docker ps -a
```

可以使用 -v 参数将本地目录挂载到容器中。
`$ docker create -it --name centos6_container -v /src/webapp:/opt/webapp centos:centos6`
这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，如果目录不存在 Docker 会自动为你创建它。

docker -v 挂载问题：
Docker容器启动的时候，如果要挂载宿主机的一个目录，可以用-v参数指定。
譬如我要启动一个centos容器，宿主机的/test目录挂载到容器的/soft目录，可通过以下方式指定：
`docker run -it -v /test:/soft centos /bin/bash`
这样在容器启动后，容器内会自动创建/soft的目录。通过这种方式，我们可以明确一点，即-v参数中，冒号":"前面的目录是宿主机目录，后面的目录是容器内目录。
貌似简单，其实不然，下面我们来验证一下：

- 容器目录不可以为相对路径
```
[root@localhost ~]# docker run -it -v /test:soft centos /bin/bash
invalid value "/test:soft" for flag -v: soft is not an absolute path
See 'docker run --help'.
```
直接报错，提示soft不是一个绝对路径，所谓的绝对路径，必须以下斜线“/”开头。

- 宿主机目录如果不存在，则会自动生成
如果宿主机中存在/test目录，首先删除它
```
[root@localhost ~]# rm -rf /test
[root@localhost ~]# ls /
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
启动容器
```
[root@localhost ~]# docker run -it -v /test:/soft centos /bin/bash
[root@a487a3ca7997 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  soft  srv  sys  tmp  usr  var
```
查看宿主机，发现新增了一个/test目录
```
[root@localhost ~]# ls /
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  test  tmp  usr  var
```

- 宿主机的目录如果为相对路径呢？
这次，我们换个目录名test1试试
`docker run -it -v test1:/soft centos /bin/bash`
再到宿主机上查看是否新增了一个/test1目录，结果没有，是不是因为我用的是相对路径，所以生成的test1目录在当前目录下，结果发现还是没有。
那容器内的/soft目录挂载到哪里去了？通过docker inspect命令，查看容器“Mounts”那一部分，我们可以得到这个问题的答案。
```
"Mounts": [
        {
            "Name": "test1",
            "Source": "/var/lib/docker/volumes/test1/_data",
            "Destination": "/soft",
            "Driver": "local",
            "Mode": "z",
            "RW": true
        }
    ],
```
可以看出，容器内的/soft目录挂载的是宿主机上的/var/lib/docker/volumes/test1/_data目录，原来所谓的相对路径指的是/var/lib/docker/volumes/，与宿主机的当前目录无关。

- 如果只是-v指定一个目录，这个又是如何对应呢？
启动一个容器
```
[root@localhost ~]# docker run -it -v /test2 centos /bin/bash
[root@ea24067bc902 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  test2  tmp  usr  var
```
同样使用docker inspect命令查看宿主机的挂载目录
```
"Mounts": [
        {
            "Name": "96256232eb74edb139d652746f0fe426e57fbacdf73376963e3acdb411b3d73a",
            "Source": "/var/lib/docker/volumes/96256232eb74edb139d652746f0fe426e57fbacdf73376963e3acdb411b3d73a/_data",
            "Destination": "/test2",
            "Driver": "local",
            "Mode": "",
            "RW": true
        }
    ],
```
可以看出，同3中的结果类似，只不过，它不是相对路径的目录名，而是随机生成的一个目录名。

- 如果在容器内修改了目录的属主和属组，那么对应的挂载点是否会修改呢？
首先开启一个容器，查看容器内/soft目录的属性
```
[root@localhost ~]# docker run -it -v /test:/soft centos /bin/bash
[root@b5ed8216401f /]# ll -d /soft/
drwxr-xr-x 2 root root 6 Sep 24 03:48 /soft/
```
查看宿主机内/test目录的属性
```
[root@localhost ~]# ll -d /test/
drwxr-xr-x 2 root root 6 Sep 24 11:48 /test/
```

在容器内新建用户，修改/soft的属主和属组
```
[root@b5ed8216401f /]# useradd victor
[root@b5ed8216401f /]# chown -R victor.victor /soft/
[root@b5ed8216401f /]# ll -d /soft/
drwxr-xr-x 2 victor victor 6 Sep 24 03:48 /soft/
```

再来看看宿主机内/test目录的属主和属组是否会发生变化？
```
[root@localhost ~]# ll -d /test/
drwxr-xr-x 2 mycat mycat 6 Sep 24 11:48 /test/
```

竟然变为mycat了。。。
原来，这个与UID有关系，UID，即“用户标识号”，是一个整数，系统内部用它来标识用户。一般情况下它与用户名是一一对应的。
首先查看容器内victor对应的UID是多少，
```
[root@b5ed8216401f /]# cat /etc/passwd | grep victor
victor:x:1000:1000::/home/victor:/bin/bash
```
victor的UID为1000，那么宿主机内1000对应的用户是谁呢？
```
[root@localhost ~]# cat /etc/passwd |grep 1000
mycat:x:1000:1000::/home/mycat:/bin/bash
```
可以看出，宿主机内UID 1000对应的用户是mycat。

- 容器销毁了，在宿主机上新建的挂载目录是否会消失？
在这里，主要验证两种情况：一、指定了宿主机目录，即 -v /test:/soft。二、没有指定宿主机目录，即-v /soft
第一种情况：
```
[root@localhost ~]# rm -rf /test    --首先删除宿主机的/test目录
[root@localhost ~]# ls /    --可以看到，宿主机上无/test目录
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@localhost ~]# docker run -it --name=centos_test -v /test:/soft centos /bin/bash  --启动容器，为了删除方便，我用--name参数指定了容器的名字
[root@82ad7f3a779a /]# exit
exit
[root@localhost ~]# docker rm centos_test   --删除容器
centos_test
[root@localhost ~]# ls /   --发现 /test目录依旧存在
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  test  tmp  usr  var
```
可以看出，即便容器销毁了，新建的挂载目录不会消失。进一步也可验证，如果宿主机目录的属主和属组发生了变化，容器销毁后，宿主机目录的属主和属组不会恢复到挂载之前的状态。

第二种情况，通过上面的验证知道，如果没有指定宿主机的目录，则容器会在/var/lib/docker/volumes/随机配置一个目录，那么我们看看这种情况下的容器销毁是否会导致相应目录的删除
首先启动容器
```
[root@localhost ~]# docker run -it --name=centos_test -v /soft centos /bin/bash
[root@6b75579ec934 /]# exit
exit
```
通过docker inspect命令查看容器在宿主机上生成的挂载目录
```
"Mounts": [
        {
            "Name": "b53164cb1c9f1917788638692fb22ad11994cf1fbbc2461b6c390cd3e10ea301",
            "Source": "/var/lib/docker/volumes/b53164cb1c9f1917788638692fb22ad11994cf1fbbc2461b6c390cd3e10ea301/_data",
            "Destination": "/soft",
            "Driver": "local",
            "Mode": "",
            "RW": true
        }
    ],
```
对应的是/var/lib/docker/volumes/b53164cb1c9f1917788638692fb22ad11994cf1fbbc2461b6c390cd3e10ea301/_data目录
销毁容器，看目录是否存在
```
[root@localhost ~]# docker rm centos_test
centos_test
[root@localhost ~]# ll /var/lib/docker/volumes/b53164cb1c9f1917788638692fb22ad11994cf1fbbc2461b6c390cd3e10ea301
total 0
drwxr-xr-x 2 root root 6 Sep 24 14:25 _data
```

发现该目录依旧存在，即便重启了docker服务，该目录依旧存在
```
[root@localhost ~]# systemctl restart docker
[root@localhost ~]# ll /var/lib/docker/volumes/b53164cb1c9f1917788638692fb22ad11994cf1fbbc2461b6c390cd3e10ea301
total 0
drwxr-xr-x 2 root root 6 Sep 24 14:25 _data
```

- 挂载宿主机已存在目录后，在容器内对其进行操作，报“Permission denied”。
可通过两种方式解决：

1> 关闭selinux。
临时关闭：# setenforce 0
永久关闭：修改/etc/sysconfig/selinux文件，将SELINUX的值设置为disabled。

2> 以特权方式启动容器 
指定--privileged参数
如：# docker run -it --privileged=true -v /test:/soft centos /bin/bash

[docker挂载参考地址](https://www.cnblogs.com/ivictor/p/4834864.html)

## 启动容器
`docker start <container-id>`
Docker start命令为容器文件系统创建了一个进程隔离空间。注意，每一个容器只能够有一个进程隔离空间。

如：
```
#通过名字启动
$ docker start -i centos6_container

＃通过容器ID启动
$ docker start -i b3cd0b47fe3d
```

## 进入容器
`docker exec <container-id>`

在当前容器中执行新命令，如果增加 -it参数运行bash 就和登录到容器效果一样的。
`docker exec -it centos6_container bash`

## 停止容器
`docker stop <container-id>`

## 删除容器
`docker rm <container-id>`

## 运行容器
`docker run <image-id>`

docker run就是docker create和docker start两个命令的组合,支持参数也是一致的，如果指定容器
名字是，容器已经存在会报错,可以增加 --rm 参数实现容器退出时自动删除。

如：
`docker create -it --rm --name centos6_container centos:centos6`

## 查看容器列表
`docker ps`
docker ps 命令会列出所有运行中的容器。这隐藏了非运行态容器的存在，如果想要找出这些容器，增加-a参数。

## 删除镜像
`docker rmi <image-id>`

删除构成镜像的一个只读层。你只能够使用docker rmi来移除最顶层（top level layer）
（也可以说是镜像），你也可以使用-f参数来强制删除中间的只读层。

## commit容器
`docker commit <container-id>`
将容器的可读写层转换为一个只读层，这样就把一个容器转换成了不可变的镜像。

## 镜像保存
`docker save <image-id>`

创建一个镜像的压缩文件，这个文件能够在另外一个主机的Docker上使用。和export命令不同，这个命令
为每一个层都保存了它们的元数据。这个命令只能对镜像生效。

如：
```
#保存centos镜像到centos_images.tar 文件
$ docker save  -o centos_images.tar centos:centos6

＃或者直接重定向
$ docker save  -o centos_images.tar centos:centos6 > centos_images.tar
```

## 容器导出
`docker export <container-id>`

创建一个tar文件，并且移除了元数据和不必要的层，将多个层整合成了一个层，只保存了当前统一视角看到
的内容。expoxt后的容器再import到Docker中，只有一个容器当前状态的镜像；而save后的镜像则不同，
它能够看到这个镜像的历史镜像。

## inspect
`docker inspect <container-id> or <image-id>`
docker inspect命令会提取出容器或者镜像最顶层的元数据

# docker安装指定版本TAG的镜像
在docker中安装镜像，一般过程是，docker search 软件名称。
本文以tomcat为例，讲解下载指定版本TAG的tomcat。

搜索tomcat镜像：
`sudo docker search tomcat`

发现只有tomcat，而如果docker pull tomcat，下载下来的是最新版的tomcat,也就是tomcat11。假设希望下载的是tomcat:8-jre8。那么该如何做呢？
如果search出来的镜像，不是自己想要的版本，可通过在docker hub上搜索好版本TAG，地址如下：https://hub.docker.com，进入之后，在页面左上角搜索框搜索`tomcat`，
搜索出来之后，点详情，点击标签，在下面通过翻页，找到8-jre标签，复制标签名：8-jre8，下载tomcat镜像
`sudo docker pull tomcat:8-jre8`

# /var/lib/docker/overlay2 占用过大
查看磁盘使用情况
`sudo du -hs /var/lib/docker/`

查看Docker的磁盘使用情况
`docker system df`

清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)
`docker system prune`

docker system prune -a命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚.。我没用过，因为会清理 没有开启的  Docker 镜像。

## 迁移 /var/lib/docker目录
停止docker服务
`systemctl stop docker`

创建新的docker目录，执行命令df -h,找一个大的磁盘。 我在 /home目录下面建了 /home/docker/lib目录，执行的命令是：
`mkdir -p /home/docker/lib`

迁移/var/lib/docker目录下面的文件到 /home/docker/lib
`rsync -avz /var/lib/docker /home/docker/lib/`

配置 /etc/systemd/system/docker.service.d/devicemapper.conf。查看 devicemapper.conf 是否存在。如果不存在，就新建
`sudo mkdir -p /etc/systemd/system/docker.service.d/`
`sudo vi /etc/systemd/system/docker.service.d/devicemapper.conf`

然后在 devicemapper.conf 写入：（同步的时候把父文件夹一并同步过来，实际上的目录应在 /home/docker/lib/docker）
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd  --graph=/home/docker/lib/docker
```

重新加载 docker
`systemctl daemon-reload`
`systemctl restart docker`
`systemctl enable docker`

为了确认一切顺利，运行
`docker info`

命令检查Docker 的根目录.它将被更改为 /home/docker/lib/docker
```
...
Docker Root Dir: /home/docker/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
...
```

启动成功后，再确认之前的镜像还在
```
linlf@dacent:~$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
AAA/AAA               v2                  7331b8651bcc        27 hours ago        3.85GB
BBB/BBB               v1                  da4a80dd8424        28 hours ago        3.47GB
```
确定容器没问题后删除/var/lib/docker/目录中的文件。

# Docker和K8S
[参考地址](https://my.oschina.net/jamesview/blog/2994112)
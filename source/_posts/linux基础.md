---
title: linux基础
tags: linux
date: 2019-03-05 10:49:36
---

# 文件目录解析
![](linux基础/1.png)

"/"：根目录，linux文件系统入口，是最高级、最重要的目录，衍生出其他目录，还和系统开机、还原、修复有直接关系。
一般不要把应用程序直接放在根目录下，如果满了，可能导致无法开机。还需要注意某些程序的日志文件是否在根目录下。

"/bin"：存放的是在单人维护模式下还能够被操作的指令，在/bin底下的指令可以被root与一般帐号所使用，主要有：cat,chmod, chown, date, mv, mkdir, cp, bash等等常用的指令。

"/boot"：存放内核和加载内核需要的文件，即启动系统需要的文件。

"/dev"：存放linux系统下的设备文件，访问该目录下某个文件，相当于访问某个设备，常用的是挂载光驱 mount /dev/cdrom /mnt。

"/etc"：系统配置文件存放的目录，不建议在此目录下存放可执行文件。
重要的配置文件有 /etc/inittab、/etc/fstab、/etc/init.d、/etc/X11、/etc/sysconfig、/etc/xinetd.d修改配置文件之前记得备份。

"/home"：系统默认的用户家目录，新增用户账号时，用户的家目录都存放在此目录下，~表示当前用户的家目录，~edu 表示用户 edu 的家目录。建议单独分区，并设置较大的磁盘空间，方便用户存放数据

"/lib： /usr/lib： /usr/local/lib：" ：系统使用的函数库的目录，程序在执行过程中，需要调用一些额外的参数时需要函数库的协助，比较重要的目录为 /lib/modules。

"/media"：放置的就是可移除的装置。 包括软碟、光碟、DVD等等装置都暂时挂载于此。 常见的档名有：/media/floppy, /media/cdrom等等。

"/mnt"：临时文件系统的挂载点目录.以前和/media一样，但有专门/media后，专门做临时挂载

"/opt"：第三方软件的存放目录

"/root"：系统管理员(root)的家目录。 之所以放在这里，是因为如果进入单人维护模式而仅挂载根目录时，该目录就能够拥有root的家目录，所以我们会希望root的家目录与根目录放置在同一个分区中。

"/sbin"：基本的系统维护命令,只能由超级用户使用.这些命令为开机、修复、还原系统过程所需要的。常见的命令有fdisk,fsck,ifconfig,init,mkfs

"/tmp"：一般用户或正在执行的程序临时存放文件的目录,任何人都可以访问,重要数据不可放置在此目录下

"/srv"：服务启动之后需要访问的数据目录，如www服务需要访问的网页数据存放在/srv/www内。

"/proc"：是一个虚拟文件系统。放置内存中的数据，当有一个进程启动时，就有一个文件夹。
比较重要的/proc/meminfo,/proc/cpuinfo，可以通过这两文件查看内存和CPU情况，当然还有/proc/dma,/proc/interrupts,/proc/ioports,/proc/net/*等

"/sys"：和/proc相似，也是虚拟文件系统，主要记录内核相关，比如内核模块，内核检测的硬件信息。

"/lost+found"：这个目录是使用标准的ext2/ext3档案系统格式才会产生的一个目录，目的在于当档案系统发生错误时，将一些遗失的片段放置到这个目录下。 这个目录通常会在分割槽的最顶层存在，例如你加装一个硬盘于/disk中，那在这个系统下就会自动产生一个这样的目录/disk/lost+found

除了这些目录的内容之外，另外要注意的是，因为根目录与开机有关，开机过程中仅有根目录会被挂载， 其他分区则是在开机完成之后才会持续的进行挂载的行为。就是因为如此，因此根目录下与开机过程有关的目录， 就不能够与根目录放到不同的分区去。

那哪些目录不可与根目录分开呢？有底下这些：

/etc：配置文件

/bin：重要执行档

/dev：所需要的装置文件

/lib：执行档所需的函式库与核心所需的模块

/sbin：重要的系统执行文件

这五个目录千万不可与根目录分开在不同的分区。请背下来啊。

交换区：我们如果没有足够的内存，也许就不能运行某些大型的软件，解决的办法是在硬盘上划出一个区域来当作临时的内存，好像内存变大了。Windows 操作 系统把这个区域叫做虚拟内存，Linux把它叫做交换分区swap。

**/usr**：依据FHS的基本定义，/usr里面放置的数据属于可分享的与不可变动的(shareable, static)。
/usr不是user的缩写，其实usr是Unix Software Resource的缩写， 也就是Unix操作系统软件资源所放置的目录，而不是用户的数据。
 FHS建议所有软件开发者，应该将他们的数据合理的分别放置到这个目录下的次目录，而不要自行建立该软件自己独立的目录。
 因为是所有系统默认的软件(distribution发布者提供的软件)都会放置到/usr底下，因此这个目录有点类似Windows系统的C：\Windows\ + C：\Program files\这两个目录的综合体，系统刚安装完毕时，这个目录会占用最多的硬盘容量。一般来说，/usr的次目录建议有底下这些：

"/usr/X11R6/"：为X Window System重要数据所放置的目录，之所以取名为X11R6是因为最后的X版本为第11版，且该版的第6次释出之意。

"/usr/bin/"：绝大部分的用户可使用指令都放在这里。请注意到他与/bin的不同之处。(是否与开机过程有关)

"/usr/include/"：c/c++等程序语言的档头(header)与包含档(include)放置处，当我们以tarball方式 (*.tar.gz 的方式安装软件)安装某些数据时，会使用到里头的许多包含档。 

"/usr/lib/"：包含各应用软件的函式库、目标文件(object file)，以及不被一般使用者惯用的执行档或脚本(script)。 某些软件会提供一些特殊的指令来进行服务器的设定，这些指令也不会经常被系统管理员操作， 那就会被摆放到这个目录下啦。要注意的是，如果你使用的是X86_64的Linux系统， 那可能会有/usr/lib64/目录产生

"/usr/local/"：统管理员在本机自行安装自己下载的软件(非distribution默认提供者)，建议安装到此目录， 这样会比较便于管理。举例来说，你的distribution提供的软件较旧，你想安装较新的软件但又不想移除旧版， 此时你可以将新版软件安装于/usr/local/目录下，可与原先的旧版软件有分别啦。 你可以自行到/usr/local去看看，该目录下也是具有bin, etc, include, lib...的次目录

"/usr/sbin/"：非系统正常运作所需要的系统指令。最常见的就是某些网络服务器软件的服务指令(daemon)

"/usr/share/"：放置共享文件的地方，在这个目录下放置的数据几乎是不分硬件架构均可读取的数据， 因为几乎都是文本文件嘛。在此目录下常见的还有这些次目录：/usr/share/man：联机帮助文件

"/usr/share/doc"：软件杂项的文件说明

"/usr/share/zoneinfo"：与时区有关的时区文件

"/usr/src/"：一般原始码建议放置到这里，src有source的意思。至于核心原始码则建议放置到/usr/src/linux/目录下。


**/var**：如果/usr是安装时会占用较大硬盘容量的目录，那么/var就是在系统运作后才会渐渐占用硬盘容量的目录。 因为/var目录主要针对常态性变动的文件，包括缓存(cache)、登录档(log file)以及某些软件运作所产生的文件， 包括程序文件(lock file, run file)，或者例如MySQL数据库的文件等等。常见的次目录有：
"/var/cache/"：应用程序本身运作过程中会产生的一些暂存档

"/var/lib/"：程序本身执行的过程中，需要使用到的数据文件放置的目录。在此目录下各自的软件应该要有各自的目录。 举例来说，MySQL的数据库放置到/var/lib/mysql/而rpm的数据库则放到/var/lib/rpm去

"/var/lock/"：某些装置或者是文件资源一次只能被一个应用程序所使用，如果同时有两个程序使用该装置时， 就可能产生一些错误的状况，因此就得要将该装置上锁(lock)，以确保该装置只会给单一软件所使用。 举例来说，刻录机正在刻录一块光盘，你想一下，会不会有两个人同时在使用一个刻录机烧片？ 如果两个人同时刻录，那片子写入的是谁的数据？所以当第一个人在刻录时该刻录机就会被上锁， 第二个人就得要该装置被解除锁定(就是前一个人用完了)才能够继续使用

"/var/log/"：非常重要。这是登录文件放置的目录。里面比较重要的文件如/var/log/messages, /var/log/wtmp(记录登入者的信息)等。

"/var/mail/"：放置个人电子邮件信箱的目录，不过这个目录也被放置到/var/spool/mail/目录中，通常这两个目录是互为链接文件。

"/var/run/"：某些程序或者是服务启动后，会将他们的PID放置在这个目录下

"/var/spool/"：这个目录通常放置一些队列数据，所谓的“队列”就是排队等待其他程序使用的数据。 这些数据被使用后通常都会被删除。举例来说，系统收到新信会放置到/var/spool/mail/中， 但使用者收下该信件后该封信原则上就会被删除。信件如果暂时寄不出去会被放到/var/spool/mqueue/中， 等到被送出后就被删除。如果是工作排程数据(crontab)，就会被放置到/var/spool/cron/目录中。

# 系统解析
## 引导过程
    加载BIOS（通电自检），引导系统（BIOS设定）读取MBR，运行Grub，加载初始化配置文件，启动服务，运行rc.locak，生成终端或者X Window等待登录。
    BIOS：每个主板都有一个自己的BIOS，启动硬件的第一步。
    MBR：主引导记录，BIOS会默认从硬盘的第0柱面、第0磁道、第一个扇区中读取。一个扇区=512字节，446引导程序+64地盘分区表DPT+2MBR结束位，由fWindows的disk.ext或者Linux的fdisk程序产生，不依赖操作系统。
    MBR记录可以修改因此可以实现多系统共存。
    Grub：引导操作系统程序。地址记录于MBR中，其功能是根据配置文件加载kernel镜像，并运行/sbin/init，根据/etc/inittab进行初始化工作，根据runlevel确定系统运行级别和对应服务启动脚本。
   
## 引导级别
- 关机
- 单用户模式，用于系统维护，可以在忘记root密码时进入此模式修改密码。
- 无网络链接，多用户。
- 完全多用户模式，一般使用此级别。
- 保留未使用。
- 窗口模式，多用户网络链接。
- 重启。
   
# 命令管理
    Linux中，一切配置皆文件。
    man：<order_name>查看命令的帮助，如：man ls。
    种类：9
    常见说明
    可调用的系统
    函数库
    设备文件
    文件格式
    游戏说明
    杂项
    系统管理员可用的命令
    与内核有关的说明
    man 2 reboot：查看reboot在第2章里的说明。
   
## 命令格式
命令的一般格式：command [options] [arguments]
说明：
command：命令名。
options：命令的选项，一般是一个单词或字母。有的命令有选项，有的命令没有选项。选项前面一般有“-”符号。选项是对命令参数的补充，当存在参数时才可能有选项。
arguments：命令的参数，有时候选项也带参数。有的命令有参数，有的命令没有参数。
[]：方括号表示可有可无的意思。[options]表示有的命令有选项，有的命令 也可能没有选项。[arguments 表示有的命令有参数，有的命令可能没有参数。
举例：
(1) 没有参数的命令像 ls，pwd 都没有选项和参数。直接输入命令，回车即可执行命令。
(2) 有参数没有选项的命令例如删除文件 myfile.txt 的命令，myfile.txt 就是参数：rm myfile.txt。
(3) 有参数也有选项的命令通过命令“rm myfile.txt”删除文件，系统会有确认提示，询问是否确定要删除。
可以通过一个选项，在执行命令时不再需要确认提示，命令格式如下：
`rm -f myfile.txt`
此处的-f 就是选项，作用是进行强制删除，也就是没有确认提示。

## 文件和目录操作
需掌握的常用命令：cd, ls, locate, less, grep, chmod, cp, mv, mkdir, rm。
Linux不像 Windows有C盘、D盘这么一说，它只有一个根目录“/”。

### pwd
pwd 命令代表"print working directory"(打印工作目录)。当键入 pwd 时，系统 显示当前工作目录。
例如：`[root@localhost]# pwd`  /root，表明当前的工作目录是：/root。

### ls
ls命令用户显示当前工作目录下的内容，包括子目录和文件。使用方法： `[root@localhost]# ls`

### ll
ll命令和ls命令功能一样，只是显示方式不一样，ll命令会列出当前目录下子目录和文件列表的详细信息。
使用方法如下：`[root@localhost]# ll`
显示的文件列表格式如下：
`drwxr-xr-x 3 user group 102 Mar11 22：56 Filename`
文件列表显示了7个段，每一段分别的含义说明如下：
(1) 第1段，drwxr-xr-x，表示文件属性，第1个字母d表示这个是个目录，如果是“-”表示文件。 d后面有9个字符，分成三组，每3个字符为一组。我们使用括号把他括起来：d(rwx)(r-x)(r-x)，对应含意：文件类型(所有者权限)(组权限)(其他人 权限)。这三组分别说明的是：文件的所有者权限，文件所属的组权限，和“其他人”的权限。每一组中每一位字符的含意如下：
第 1 位表示是否有读权限，有读权限，显示“r”，没有读权限，显示“-”。
第 2 位表示是否有写权限，有写权限，显示“w”，没有写权限，显示“-”。
第 3 位表示是否有执行权限，有执行权限，显示“x”，没有，显示“-”。
d(rwx)(r-x)(r-x)含意说明：
第 1 位 d：表示目录。
第 1 组(rwx)：表示文件所有者对这个文件的权限是，有读，写，执行权限。
第 2 组(r-x)：表示文件所属用 户组对这个文件的权限是：有读，没有写权限，有执行权限。
第 3 组(r-x)：表示 其他人对这个文件的权限是：有读，没有写权限，有执行权限。
(2) 第2段，目录下的文件和子目录个数：3表示目录下有3个文件或 目录 (注意：每个目录都有一个指向它本身的子目录"." 和指向它上级目录的子 目录"..")
(3) 第 3 段，显示文件的所有者：user。
(4) 第 4 段，显示文件所属用户组：group。
(5) 第 5 段，文件大小：102 byte。
(6) 第 6 段，修改时间：Mar11 22：56。
(7) 第 7 段，文件名：Filename。

### cd
cd 命令来改变工作目录。
例如把当前目录更改到/usr 目录下：`[root@localhost]# cd /usr`
例如把当前目录更改到整个系统的根目录下： `[root@localhost]# cd /`
几个常用 cd 用法：
命令
功能
`cd tmp`
进入当前目录下的tmp目录
`cd ~`
回到当前用户的HOME目录下
`cd /`
回到整个系统的根目录
`cd /etc`
进入到/etc目录
`cd ..`
回到上一级目录


### locate
使用 locate 命令来搜寻文件或目录。locate 命令直接在文件索引数据库里查找， 所以搜索文件的速度很快。如果需要更新文件索引数据库，则使用 updatedb 命 令。例如，如果想搜寻所有名称中带有 finger 这个词的文件或目录，命令如下： `[root@localhost]# locate finger`
locate 有一个十分有用的选项 -r，它可以让你在搜索文件时使用正则表达式。
比如`[root@localhost]# local --regex [0-9]`
locate是Linux系统中的一个查找文件命令，若在查找文件时提示：
`locate： can not open ‘/var/lib/mlocate/mlocate.db'： No such file or directory`
原因是locate是通过生成一个文件和文件夹的索引数据库，当用户在执行loacte命令查找文件时，它会直接在索引数据库里查找，若该数据库太久没更新或不存在，则会提示以上错误。
解决方法是执行如下的命令，更新文件索引数据库。
`[root@localhost]# updatedb`

### find
这是另一个 Linux 系统中重要的文件查找命令。其一般命令格式为：`find 位置 -name 文件名称`
例如，在"/"这个根目录中查找"linux.html"文件，命令如下：
` [root@localhost]# find / -name linux.html`
你除了可以按文件名称来使用find查找文件外，也可以根据文件大小(通过 -size n 选项指定)、时间(如 -atime n 表示查找 n 天前访问过的文件)来搜索文件。
此外，find 命令同样支持在搜索文件时使用正则表达式，你只需指定 -regex 选项即可。
find 和 locate 命令的区别：locate 是在建立好的索引的基础上进行查找， 查找要快很多。find 功能强大，但它是直接在硬盘上搜寻，查找速度比较慢。

### clear
clear命令用于清除终端窗口，直接在终端输入clear，会清空当前屏幕。

### cat
cat是concatenate(连锁)的简写，意思是合并文件。该命令可以显示文件的内容(经常和more搭配使用)，或者是将多个文件合并成一个文件。
cat主要有三大功能：
(1) 一次显示整个文件。`$ cat filename`
(2) 从键盘创建一个文件。`$ cat > filename << 结束符`
只能用于创建新文件，不能编辑已有文件。例如设置结束符为EOF
`cat > 1.txt << EOF`
编辑文件完后，只需在最后一行键入“EOF”，然后回车即可。
(3) 将几个文件合并为一个文件： `$cat file1 file2 > file`
参数：
-n或--number：由1开始对所有输出的行数编号
-b或--number-nonblank：和-n相似，只不过对于空白行不编号
-s或--squeeze-blank：当遇到有连续两行以上的空白行，就代换为一行的空白行
-v或--show-nonprinting：用‘^’输出控制字符除了LFD和TAB
例：
把1.txt和2.txt合并到3.txt
`cat 1.txt 2.txt > 3.txt`
把1.txt的内容加上行号后输入4.txt这个档案里
`cat -n 1.txt > 4.txt`
把1.txt和2.txt的内容加上行号(空白行不加)之后将内容附加到3.txt里
`cat -b 1.txt 2.txt >> 3.txt`
和more一起使用，使用管道连起来。例如：`cat /etc/passwd | more`。
不过一般不这么用，使用麻烦。直接使用more命令即可，例如：`more/etc/passwd`

### more
浏览超过一页的文件。在显示满一页时暂停，此时可按空格健继续显示下一个画面。 按Q键退出显示。Enter键滚动显示下一行。例如：`[root@localhost]# more <filename>`

### less
less命令的用法与more命令类似，也可以用来浏览超过一页的文件。所不同的是less命令除了可以按空格键向下显示文件外，还可以利用上下键来卷动文件。当需要结束浏览时，只要在less命令的提示符“：”下按Q键即可。例如：
`[root@localhost]# less <filename>`

### head
你可以使用head命令来查看文件的开头部分。此项命令是：`head <filename>`
head是一个有用的命令，但是由于它只限于文件的最初几行，你看不到文件实际上有多长。按照默认设置，你只能阅读文件的前十行。你可以通过指定一个数字选项来改变显示的行数，如下面的命令所示：`[root@localhost]# head -20 <filename>`

### tail
与head命令恰恰相反的是tail命令。使用tail命令，你可以查看文件结尾的十行。这有助于查看日志文件的最后十行来阅读重要的系统消息。你还可以使用tail来观察日志文件被更新的过程。使用-f选项，tail会自动实时地把打开文件中的新消息显示到屏幕上。例如，要即时观察/var/log/messages的变化，以根用户身份在shell提示下键入以下命令：
`[root@localhost]# tail –f /var/log/messages。`

### grep
grep命令用于在文件中查找指定的字串。例如，在temp.txt文件中查找每一个"hello"，使用的命令如下：`[root@localhost]# grep hello temp.txt`
如果要显示行号，则使用如下命令：
`[root@localhost]# grep -n 要查找的内容 文件名`

### chmod
chmod命令用于改变文件或目录的访问权限。更改权限：给文件设置新的权限。例如，给test.sh文件设置权限`[root@localhost]#chmod 644 test.sh 644`
由3位数字组成：
第1位数字代表文件所有者对这个文件的权限
第2位数字代表文件所属用户组对这个文件的权限
第3位数字代表其他人对这个文件的权限。权限属性对应的数字如下：
例如：
仅有读权限，就是4。
仅有写权限就是2。
既有读，也有写权限，则把权限值加起来，4+2就是6。
644权限说明：
文件所有者对这个文件的权限是：读、写;
文件所属用户组对这个文件的权限：仅有读;
其他人对这个文件的权限：仅有读。
可通过ll命令查看权限更改后的情况。
加减权限：
在原有权限的基础之上，增加或减少权限，使用命令如下：
`chmod o+w test.sh`
o+w命令告诉系统给其他人对文件test.sh增加写权限。
`chmod go -rw test.sh //但是会报错`
组和其他人减去读取和写入权限。
下面是一个速记符号含义的列表：身份u：拥有文件的用户(User)
权限属性
对应数字
说明
r
4
文件可以被读取
w
2
文件可以被写入
x
1
文件可以被执行(如果它是程序的话)
-
0
相应的权限还没有被授予。
g：所有者所在的组群(Group)
o：其他人(不是所有者或所属用户组的其他用户，Other)
a：全部(All，包括 u、g、和 o)权限
r：读取权
w：写入权
x：执行权
+：添加权限
-：删除权限
=：使它成为唯一权限
常用的权限设置：
这里是一个某些常用设置、数值、以及它们的含义的列表：
-rw-------(600)：只有所有者才有读取和写入的权限。
-rw-r-r-- (644)：只有所有者才有读取和写入的权限，组群和其他人只有读取的权限。
-rwx-----(700)：只有所有者才有读取、写入和执行的权限。
-rwxr-xr-x(755)：所有者有读取、写入和执行的权限，组群和其他人只有读取和执行的权限。
-rwx--x--x(711)：所有者有读取、写入和执行权限，组群和其他人只有执行权限。
-rw-rw-rw-(666)：每个人都能够读取和写入文件。(请谨慎使用这些权限。)
-rwxrwxrwx(777)：每个人都能够读取、写入和执行。(再重申一次，这种权限设置可能会很危险)
下面列举了一些对目录的常见设置：
drwx----(700)：只有所有者能在目录中读取、写入。
drwx-xr-x(755)：每个人都能够读取目录，但是其中的内容却只能被告所有者更改。

### chown
chown用于改变文件的所有者。
命令格式：
`chown [-R] user[：group] file...`
user：新的档案拥有者的使用者ID
group：新的档案拥有者的使用者群体(group)
-R：对目前目录下的所有档案与子目录进行相同的拥有者变更(即以递归的方式逐个变更)
示例：`chown root 4.txt`

### cp
cp(copy)命令可以将文件或目录复制到其他目录中。在使用cp命令时，只需要指定源文件名与目标文件名或目标目录即可。命令格式：`cp <源> <目标目录>`
例如，把当前目录下的test.txt文件拷贝到/tmp目录下：`cp test.txt /tmp`
拷贝目录(命令中需要使用-r选项)：`cp -r <源目录名> <目标目录>`
例如，把当前目录下的test目录拷贝到/tmp目录下：`cp -r test /tmp`

### scp
scp是linux中功能最强大的文件传输命令，可以实现从本地到远程以及远程到本地的文件传输操作。
本地到远程的文件复制：
文件拷贝命令格式：
`scp local_file remote_username@remote_ip：remote_folder`
或
`scp local_file remote_username@remote_ip：remote_`
file local_file：本地文件。
remote_username：远程计算机的用户名。
remote_ip：远程计算机的IP 地址。
remote_folder：远程计算机的目录。
示例：`scp 1.mp3 root@192.168.1.2：/tmp/`
目录拷贝命令格式：
`scp -r local_folder remote_username@remote_ip：remote_folder`
例如将本地/home/music/目录复制到远程/tmp/目录下：
`scp -r /home/music/ root@192.168.1.2：/tmp/`
复制后远程有/tmp/music/目录。
远程到本地的复制从远程服务器复制文件到本地，只要将两个参数调换顺序即可：
拷贝文件：
例如把192.168.1.2上的/home/music/1.mp3拷贝到本地/tmp/目录下：
`scp root@192.168.1.2：/home/music/1.mp3 /tmp/`
拷贝目录：
例如把192.168.1.2上的/home/music/目录拷贝到本地/tmp/目录下：
`scp -r root@192.168.1.2：/home/music/ /tmp/`

### mv
mv命令用于移动文件和改名。mv的常见选项包括：
`-i`：互动。如果你选择的文件会覆盖目标中的现存文件，它会提示你。这是一个实用的选项，因为它像cp中的-i选项一样，会给你一个确认替换已存文件的机会。
`-f`：强制。它会超越互动模式，不提示地移动文件。除非你知道自己在干什么，这个选项很危险。在你对系统信心十足之前，请谨慎使用这个选项。
`-v`：详细。显示文件的移动进度。
命令格式：`mv <源文件> <目标目录>`
如果你想把文件从你的主目录中移到另一个现存的目录中，使用以下命令：
`[root@localhost]# mv sneakers.txt /home/newuser/`
移动后同时给文件改名：
`[root@localhost]# mv sneakers.txt /home/newuser/new_sneakers.txt`

把当前文件夹的文件a改为b：
`mv a b`

### mkdir
mkdir(make directory)命令用来建立目录。例如在系统中当前目录下建立data子目录：
`[root@localhost]# mkdir data`

### rm
rm命令用于将文件或目录删除。如果是链接文件，只是删除了该链接，原有文件保持不变。删除文件和目录的选项包括：
`-i`：互动。提示你确认删除，这个选项可以帮助你避免误删文件。
`-f`：Force，强制。代替互动模式，不提示地删除文件。除非你知道自己在干什么，使用这个选项通常不是明智之举。
`-v`：Verbose，显示操作过程的详细信息。显示文件的删除进度。
`-r`：Recursive，递归。将会删除某个目录及其中所有的文件和子目录。
例如要使用rm命令来删除文件tmp.txt，命令如下：`rm tmp.txt`
如果要删除目录，命令如下：`rm -r 目录名`
强制删除目录及以下文件：`rm -rf 目录名`

## 文件归档和压缩
需掌握的常用命令：tar，zip，unzip，gzip，gunzip。

### zip和unzip
`zip`命令用于打包和压缩文件。
压缩示例：`zip -r data.zip data`
将当前目录下的data文件夹和内部文件全部压缩成`data.zip`文件，`-r`表示递归压缩子目录下所有文件。
`zip -r data.zip *`
将当前目录下所有文件夹和文件全部压缩成data.zip文件。
`zip -r data.zip /home/data`
将/home/data这个目录下的所有文件夹和文件压缩为当前目录下的data.zip

如果现在在/home 这个目录下：
`zip -r data.zip data`

`unzip`命令用于解压扩展名为`.zip`的文件。
解压到当前目录：
`unzip data.zip`

解压到指定目录：
`unzip data.zip -d /home/wjy/Desktop/`

centos需要安装zip和unzip指令：
`yum install -y unzip zip`

### gzip/gunzip
gzip命令用于压缩".gz"文件，gunzip用于解开被gzip压缩过的文件，这些压缩文件预设最后的扩展名为".gz"。
压缩：`gzip usr.tar`
解压：`gunzip usr.tar.gz`

### tar
tar命令用于文件打包，不是压缩。
打包例子：`tar -cvf myfile.tar`
目录解包例子：`tar -xvf myfile.tar`
如果一个压缩包是.tar.gz格式，则可以先通过gunzip解压，然后再通过tar解包。也可以通过tar命令直接解压解包，只需在解包加个参数z：
`tar -xzvf myfile.tar.gz`
tar命令常用选项：
`-c`：建立新的归档文件
`-x`：从归档文件中解出文件
`-v`：处理过程中输出相关信息
`-f`：指定的文件名
`-z`：与-x 联用时调用gzip完成对gz文件的解压

## 系统管理
需掌握的常用命令：free, top, shutdown。

### free
free命令用于查看当前系统内存的使用情况，它可以显示系统中剩余和已用的物理内存、交换内存和内核缓冲区。
例如：`[root@localhost]# free`

### shutdown
该命令用于关机。语法规则如下：`shutdown [-cfFhknr(参数名称)] [-t 秒数] 时间 [警告信息]`
具体各参数功能：
`-c`：取消前一个shutdown命令。值得注意的是，当执行一个如"shutdown -h 11：10"的命令时，只要按“Ctrl+C”键就可以中断关机的命令。若是执行如"shutdown -h 11：10 & "的命令将shutdown转到后台时，则需要使用`shutdown -c`将前一个shutdown命令取消。
`-f`：重新启动时不执行fsck(注：fsck是Linux下的一个检查和修复文件系统的程序)。
`-F`：重新启动时执行fsck。
`-h`：将系统关机，在某种程度上功能与halt命令相当。
`-k`：只是送出信息给所有用户，但并不会真正关机。
`-n`：不调用init程序关机，而是由shutdown自己进行(一般关机程序是由shutdown调用init来实现关机动作)，使用此参数将加快关机速度，但是不建议用户使用此种关机方式。
`-r`：shutdown之后重新启动系统。

`-t <秒数>`：送出警告信息和关机信号之间要延迟多少秒。警告信息将提醒用户保存当前进行的工作。
`[时间]`：设置多久时间后执行shutdown命令。时间参数有`hh：mm`或+m两种模式。hh：mm格式表示在几点几分执行shutdown命令。
例如`shutdown 10：45`表示将在10：45执行shutdown。+m表示m分钟后执行shutdown。
比较特别的用法是以now表示立即执行shutdown。注意这部分参数不能省略。
`[警告信息]`：要传送给所有登入用户的信息。 
应用举例：
立即关机：
`shutdown -h now`
定时关机(11点10分关机，如果要取消按 Ctrl+C 键)：
`shutdown -h 11：10`或
`shutdown 11：10`
指定5分钟后关机，同时送出警告信息给登入用户：
`shutdown +5 "System will shutdown after 5 minutes"`
重启计算机：
`shutdown -r now`

### reboot
重启计算机。用法：
`reboot`

### date
该命令可以显示当前系统的日期和时间，也可以设置日期和时间。clock命令也可以用于显示系统的日期与时间(默认情况下，普通用户无法执行clock命令，必须用root账号登录执行)。
例如，显示日期和时间：
`date`
设置日期和时间：
`date -s "2017-04-11 11：28：30"`

### cal
该命令可显示计算机中的日历。 显示当前月日历：`# cal`
显示指定年份的日历命令：
`cal 2016`
显示指定月份的日历命令：
`cal 9 2016`

### top
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。例如：`[root@localhost]# top`
统计信息区前五行是系统整体的统计信息：
第一行是任务队列信息，同`uptime`命令的执行结果，其内容如下：
`01：06：48`：当前时间
`up 1：22`：系统运行时间，格式为时：分
`1 user`：当前登录用户数
`load average： 0.06, 0.60, 0.48`：系统负载，即任务队列的平均长度。三个数值分别为1分钟、5分钟、15分钟前到现在的平均值.
第二行为进程信息，内容如下：
`Tasks：29 total`：进程总数
`1 running`：正在运行的进程数
`28 sleeping`：睡眠的进程数
`0 stopped`：停止的进程数
`0 zombie`：僵尸进程数
第三行为CPU的信息，当有多个CPU时，这些内容可能会超过两行：
`Cpu(s)： 0.3% us`：用户空间占用CPU百分比
`1.0% sy`：内核空间占用CPU百分比
`0.0% ni`：用户进程空间内改变过优先级的进程占用CPU百分比
`98.7% id`：空闲CPU百分比
`0.0% wa`：等待输入输出的CPU时间百分比
`0.0% hi`： CPU服务于硬中断所耗费的时间总额
`0.0% si、0.0%st`：CPU服务于软中断所耗费的时间总额、Steal Time
最后两行为内存信息，内容如下：
`Mem： 191272k total`：物理内存总量
`173656k used`：使用的物理内存总量
`17616k free`：空闲内存总量
`22052k buffers`：用作内核缓存的内存量
`Swap： 192772k total`：交换区总量
`0k used`：使用的交换区总量(0kb)
`192772k free`：空闲交换区总量
`123988k cached`：缓冲的交换区总量，内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小，相应的内存再次被换出时可不必再对交换区写入。
进程信息区统计信息区域的下方显示了各个进程的详细信息：
首先来认识一下各列的含义：
`列名-含义`
PID：进程id
PPID：父进程id
RUSER：Real user name
UID：进程所有者的用户id
USER：进程所有者的用户名
GROUP：进程所有者的组名
TTY：启动进程的终端名.不是从终端启动的进程则显示为 ?
PR：优先级
NI：nice值.负值表示高优先级，正值表示低优先级
P：最后使用的CPU,仅在多CPU环境下有意义
%CPU：上次更新到现在的CPU时间占用百分比
TIME：进程使用的CPU时间总计,单位秒
TIME+：进程使用的CPU时间总计,单位1/100秒
%MEM：进程使用的物理内存百分比
VIRT：进程使用的虚拟内存总量,单位kb,VIRT=SWAP+RES
SWAP：进程使用的虚拟内存中,被换出的大小,单位kb
RES：进程使用的、未被换出的物理内存大小,单位kb,RES=CODE+DATA
CODE：可执行代码占用的物理内存大小,单位kb
DATA：可执行代码以外的部分(数据段+栈)占用的物理内存大小,单位kb
SHR：共享内存大小,单位kb
nFLT：页面错误次数
nDRT：最后一次写入到现在,被修改过的页面数
S：进程状态：
    D=不可中断的睡眠状态
    R=运行
    S=睡眠
    T=跟踪/停止
    Z=僵尸进程
COMMAND：命令名/命令行
WCHAN：若该进程在睡眠,则显示睡眠中的系统函数名
Flags：任务标志,参考 sched.h

默认情况下仅显示比较重要的
`PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND`

可以通过下面的快捷键来更改显示内容：
更改显示内容通过f键可以选择显示的内容（按f键之后会显示列的列表，按 a-z即可显示或隐藏对应的列，最后按回车键确定）
按o键可以改变列的显示顺序（按小写的a-z可以将相应的列向右移动，而大写的A-Z可以将相应的列向左移动，最后按回车键确定）
按大写的F或O键，然后按a-z可以将进程按照相应的列进行排序，而大写的R键可以将当前的排序倒转。
 
## 用户管理
要掌握的常用命令：useradd, passwd, su。

### groupadd
groupadd命令用于创建新的用户组。语法格式：`groupadd 组名`
例如：
`[root@localhost]# groupadd mygroup`

### groupdel
groupdel命令用于删除用户组。语法格式：`groupdel 组名`
例如：
`[root@localhost]# groupdel mygroup`

### groups
groups命令用于显示当前用户所在的组。 例如：
`[root@localhost]# groups`

### useradd
useradd命令用于建立用户帐号。主要参数-c：加上备注文字，备注文字保存在passwd 的备注栏中。
-d：指定用户登入时的启始目录。
-D：变更预设值。
-e：指定账号的有效期限，缺省表示永久有效。
-f：指定在密码过期后多少天即关闭该账号。
-g：指定用户所属的群组。
-G：指定用户所属的附加群组。
-m：自动建立用户的登入目录。
-M：不要自动建立用户的登入目录。
-n：取消建立以用户名称为名的群组。
-r：建立系统账号。
-s：指定用户登入后所使用的 shell。
-u：指定用户ID 号。
添加用户：`useradd newuser`
添加用户，指定相关参数，建立test用户，并把此用户加入root组：
`useradd -m -d /home/share -g root test`

### userdel
userdel命令用来删除用户帐号及其相关文件。 例如：
`[root@localhost]# userdel newuser`

### passwd
passwd(password)命令用于修改用户的密码。一般来说，设置账户密码失败有几种情况：密码太简单、密码太短、密码中的字符多数相同。
语法格式：passwd 用户名 修改密码，命令如下：
`[root@localhost]# passwd newuser`

### usermod
usermod可用来修改用户帐号的各项设定。可以用来设定用户归属哪个组。 例如使得test用户加入mail组：
`[root@localhost]# usermod -G mail test`

### su
切换到user用户：`[root@localhost]# su user`
切换到 root 用户：`[root@localhost]# su`

### who
查看当前计算机有哪些用户登录

### whoami
查看当前用户的登录名。

## 网络管理
需掌握的常用命令：ping, netstat, ftp,ssh, telnet, wget

### finger (需要安装)
finger需要安装才可以使用
`yum install finger`
finger用来查询一台主机上的登录账号的信息，通常会显示用户名、主目录、 停滞时间、登录时间、登录Shell等信息，使用权限为所有用户。
命令格式：`finger [选项] [使用者] [用户@主机]`

主要参数说明：
-s：显示用户注册名、实际姓名、终端名称、写状态、停滞时间、登录时间等信息。
-l：除了用-s选项显示的信息外，还显示用户主目录、登录 Shell、邮件状态等信息，以及用户主目录下的.plan、.project和.forward文件的内容。
-p：除了不显示.plan文件和.project文件以外，与-l选项相同。

应用实例：
su(switch user)命令可以使得当前用户切换到另一个用户。如果要退出切换后的用户登录，可以执行exit命令。
直接键入如下命令：
`finger`
`finger root`

查询远程机上的用户信息应用说明：
如果要查询远程机上的用户信息，需要在用户名后面接“@主机名”，采用[用户名@主机名]的格式，不过要查询的网络主机需要运行finger守护进程的支持。

### ftp
该命令是标准的文件传输协议的用户接口，是在 TCP/IP 网络上传输文件最简单有效的方法。

### hostname
该命令用于显示或设置系统的主机名。

### netstat
netstat命令用于显示网络连接、路由表和网络接品信息，用户可以知道目前有哪些网络连接正在运行。常用选项有：
`-a`：显示所有socket，包括正在监听的。
`-c`：每隔1秒钟就重新显示一遍网络信息，直到用户中断它。
`-l`：仅列出有在 Listen (监听) 的服务状态，格式同 “ifconfig -e”命令。
`-n`：以 IP 地址代替名称，显示网络连接信息。
`-r`：显示核心路由表，格式同“route -e”命令。
`-t`：显示 TCP 协议的连接信息。
`-u`：显示 UDP 协议的连接信息。
`-v`：显示正在进行的网络协议。

应用示例：
列出所有端口：`netstat -a`
通过管道方式查看每一页内容：`netstat -a|more`
列出所有tcp端口：`netstat -at`
列出所有udp端口：`netstat -au`
列出监听端口：`netstat -l`
列出所有监听tcp端口：`netstat -lt`
列出所有监听udp端口：`netstat -lu`
查看80是否被占用：`netstat -l|grep 80`

### ping
ping命令可用来测试计算机和网络上的其他计算机是否连通。 例如测试本机和192.168.1.2那台机器是否连通：
`[root@localhost]# ping 192.168.1.2`

### ssh
ssh命令(安全的 Shell)用于通过网络登录远程计算机，如同操作本地计算机一样。命令格式：
`ssh 用户@服务器IP地址`
例如使用root用户登录到192.168.1.2上：`ssh root@192.168.1.2`
然后根据系统提示，输入登录的用户名和密码。还有一个rsh命令和ssh命令功能类似，常用的是ssh。ssh和rsh的区别：
安全级别不同，ssh的密码等都是加密传输，而且还有密钥认证的机制，而rsh是明文传输，而且没有密钥的机制。

### telnet(不建议使用)
telnet命令用于通过网络登录远程计算机，如同操作本地计算机一样。和ssh命令功能类似。ssh和telnet区别：ssh是加密的。telnet是明码传输的，发送的数据被监听后不需要解密就能看到内容。现在不建议使用telnet。

### wget
wget命令用于Linux环境下从www上下载文件，支持 HTTP和FTP协议，支持代理服务器和断点续传功能，能够自动递归远程主机的目录，查找合乎要求的文件并下载到本地硬盘上，wget 命令可在后台运行，截获并忽略HANGUP信号，因此在用户退出登录之后，仍可继续运行。
例如：`wget http：//url/filename`

### ifconfig
可以查看网卡的IP地址。
例如：`[root@localhost]# ifconfig`
也可以临时修改IP，重启系统后，恢复原始IP。

## 进程管理
常用命令：ps, kill。

### ps
ps命令用于显示程序的状态。 例如查找所有进程：`ps -aux`
命令常用选项：
命令格式：`ifconfig IP 地址 netmask 子网掩码`
-a：显示所有用户的所有进程(包括其它用户)。
`-x`：显示没有控制终端的进程，同时显示各个命令的具体路径。
`-u`：按用户名和启动时间的顺序来显示进程。
`-aux`：显示所有包含其他使用者的进程。

STAT，状态，分别表示的含意：
D：不可中断的静止
R：正在执行中
S：静止状态
T：暂停执行
Z：不存在但暂时无法消除
W：没有足够的记忆体分页可分配<：高优先序的行程
N：低优先序的行程
L：有记忆体分页分配并锁在记忆体内(即时系统或A I/O)

应用举例：
使用管道和more连用，进行分页查看：
`[root@localhost ~]# ps -aux |more`
使用管道和grep连用，在进程中查找指定进程。
例如查找Tomcat进程：`ps -aux|grep tomcat`

注意事项：如果有两条查询结果，则说明Tomcat已经启动，如果没有，则Tomcat没有启动。如果只查询出一条结果，并且以“grep tomcat”结尾，则此进程不是Tomcat进程，而是查找进程。

把查询到的进程导出到文件：
`[root@localhost ~]# ps -aux > tmp.txt`

### kill
该命令用于终止一个程序。
一般用法格式：`kill PID`
PID是指进程ID。例如杀掉进程号为2629的进程：`kill 2629`
向进程发送信号的方式，杀进程命令格式：`kill -s signal PID`
signal是指关闭PID所指定的进程时发送的信号，PID是指进程ID。例如通过上面的进程查到Tomcat的进程ID是2629，要杀死此进程的命令：
`[root@localhost ~]# kill -s SIGINT 2629`
或者写成：
`[root@localhost ~]# kill -INT 2629`
或者写成：
`[root@localhost ~]# kill -2 2629`
使用信号数字杀掉进程：
signal也可以使用对应的数字代替，例如上面的命令可以写成：
`kill -2 2629`
常用的几个信号(signal，可通过 kill -l 命令信号有哪些，及信号与数字对应关系)：
`SIG信号-对应数字-说明`
SIGINT-2：这个就是在shell下用Ctrl+C来结束一个程序时，发送的信号，进程收到这个程序会结束。你可以用`kill -INT pid`来发这个信号。
SIGQUIT-3：这个是在shell下用Ctrl+\ 来结束程序时，发送的信号，
SIGKILL-9：这个信号之所以被称为“强杀”，就是因为无法改变进程收到这个信号后所执行的动作，进程只能退出。(前面说的两个信号，虽然默认是退出，但是应用程序自己可以通过signal系统调用来修改成其他动作，比如忽略那两个信号等动作)。
SIGTERM-15：程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出

一般情况下使用最多的是强杀：
`kill -9 2629`

## 服务管理
### service
Service命令用于启动、停止、重启服务程序。
启动服务命令：`service 服务名 start `
停止服务命令：`service 服务名 stop `
重启服务命令：`service 服务名 restart`，在Centos系统中，该指令推荐使用systemctl替代它。

例如要启动 VSFTP 服务器：
`[root@localhost ~]# service vsftpd start`
例如要停止 VSFTP 服务器：
`[root@localhost ~]# service vsftpd stop`
使用systemctl命令如下：
`systemctl start vsftpd`
`systemctl stop vsftpd`

# 常用命令实战
## Linux新建组、用户、赋权
`useradd`：获取命令帮助
`useradd -G {groupName} userName`：新建用户添加到指定组（附加组），同时还会创建一个与用户同名的主组
`grep groupName /etc/group`：查找组
`groupadd groupName`：创建组
`passwd userName`：为用户修改密码
`useradd -g {groupName} userName`：新建用户添加到组并指定该组为主组（登录组）
`usermod -a -G groupName userName`：追加用户到某个组
`usermod -g groupName userName`：修改某个用户主组为某个组
`gpasswd -d userName groupName`：将某个用户从某个组立删除
`chmod -R 777`：修改某个文件或目录读写权限7-xwr 6-xw 5-xr 3-wr 1-r 文件/目录拥有者权限-组群权限-其他人权限
`chown -R userName{：groupName} 文件/目录`：修改文件/目录的所有者（组群）

## Linux ssh多台主机免密登录
首先使用root登录，执行：
`ssh-keygen`：生成ssh公钥私钥
`ssh-copy-id -i /用户名/.ssh/id_rsa.pub root@ip或host`：将密钥发送到对方主机，输入密码后即可实现免密码登录
`ssh ip或host`：使用ssh登录对方主机
`scp -r root@ip或者host：文件或者目录路径 本地路径`：将远程目录或者文件复制到本地


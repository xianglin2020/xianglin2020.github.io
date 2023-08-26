---
title: ubuntu-linux
description: Linux 发展至今，已经是一个相当复杂和丰富的操作系统了，其大部分源代码还是 GNU 项目的。
date: 2020-09-06 22:45:23
tags: [linux]
categories: [learn]
featuredImagePreview: https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224547.jpg
---

# Linux 介绍

## GNU 项目

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212626.png)

GNU 是GNU is Not Unix 的递归缩写。

## Linus Torvalds

1991 年，Linus Torvalds完成了 Linux 内核的第一个版本。Linux 可以说是 Linus 和 Unix 的合并，也可以说是 Linux is Not Unix 的递归缩写。

1. 操作系统的核心称为“内核”，但内核并不等于操作系统。内核提供系统服务，比如文件管理、虚拟内存、设备 I/O等。
2. 单独的 Linux 内核是没办法工作的，必须要有 GNU 项目的众多应用程序来添砖加瓦。
3. Linux 内核的官网是[https://www.kernel.org](https://www.kernel.org/)。Linux 的官方称谓是 GNU/Linux。

## Linux 发行版

Linux 发展至今，已经是一个相当复杂和丰富的操作系统了，其大部分源代码还是 GNU 项目的。为了简化安装过程，以及提供一些基本的软件，产生了不少的 Linux 发行版，不同 Linux 发行版的主要区别有：

1. 安装方法不一样；
2. 安装应用程序的方法不一样，及软件包的管理方式不一样。
3. 不同的 Linux预装的应用程序不一样。

### 众多的 Linux 发行版

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212634.png)

### Debian 一组

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224549.jpg)

### 继承自 Debian 的 Ubuntu 发行版

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212642.png)

# Linux 终端和文件操作

## 命令提示符

![image-20200510160128347](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224550.png)

### 对`tomcat@VM-0-2-ubuntu:~$`的解释

* `tomcat`是当前登录用户的名字，Linux 是一个完全的多用户多进程的操作系统。
* `@`就是英语中 `at` 的意思，`@`前面是用户名，后面是所在的域。
* `VM-0-2-ubuntu`是当前主机的名称，因为这是腾讯云的虚拟主机，所以默认名称是这样。
* `:`就是分隔符，没有特殊的含义。
* `~`是当前所在目录的名称，会随着用户进入不同的目录而改变，`~`表示当前用户的家目录。
* `$/#`指示当前用户权限的字符。`$`表示普通用户，有权限的限制；`#`表示超级用户，即`root`,拥有所有的权限。

![截屏2020-05-10 16.13.49](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224551.png)

## 命令和命令参数

### 简单的命令

* `date`显示当前日期及时间

  ![截屏2020-05-10 16.22.38](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212657.png)

* `ls` 是`list`的缩写，显示当前目录下的文件

  ![image-20200510162839664](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212707.png)

### 命令参数

命令参数就是写在命令后的一些补充选项，命令和参数之间用空格隔开。

1. 短参数：最常见的参数形式就是一个短横线后接一个字母。例如：`command -p`，如果要一次性加好几个参数，可以使用空格隔开，例如：`command -p -a -T -c`，多个短参数可以合并在一起，例如 ：`command -paTc`。

   ![image-20200510165559766](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224552.png)

2. 长参数：长参数是以两个短横开始的，例如：`command --parameter`，如果有多个长参数，只能使用空格隔开，例如：`command --parameter1 -parameter2`，可以组合使用短参数和长参数，例如：`command --parameter -acT`。

3. 有时候，同一个意义的参数有短参数和长参数两种形式，效果是一样的，例如`ls -a`，`ls --all`。

   ![image-20200510170046052](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212714.png)

4. 参数的值：短参数的赋值：`command -p 10`，长参数的赋值：`command --parameter=10`。

### 如何查找命令及历史记录

* Tab 可以补全命令及路径

* `Ctrl+R`可以查找使用过的命令

* `history`命令用于列出之前使用过的所有命令

* 一些实用的快捷键

  |      命令      | 解释                                         |
  | :------------: | -------------------------------------------- |
  |   `Ctrl + L`   | 清屏，同`clear`命令                          |
  |   `Ctrl + D`   | 给终端传递 EOF                               |
  | `Shift + PgUp` | 向上滚屏                                     |
  | `Shift + PgDn` | 向下滚屏                                     |
  |   `Ctrl + A`   | 光标跳到一行命令的开头，`Home`键有相同的效果 |
  |   `Ctrl + E`   | 光标跳到一行命令的结尾，`End`键有相同的效果  |


### 文件组织，`pwd`和`which`命令

* Linux 的两种类型文件

  1. 普通的文件：是我们已经熟知的文件类型，这样的文件包括：文本类型的文件、声音文件、程序等。
  2. 特殊的文件：这类特殊的文件表示一些东西。例如，光盘驱动就是特殊的文件。甚至根目录也是文件。

* 根目录：Linux 中只有一个根目录，就是`/`。根目录就是Linux 的最顶层目录。

* 根目录的直属子目录

  1. `bin`：英语 binary 的缩写，表示二进制文件，包含了会被所有用户使用的可执行程序

  2. `boot`：英语 boot 表示启动，包含与 Linux 启动密切相关的文件

  3. `dev`：英语 device 的缩写，表示设备，包含外设。它里面的子目录，每一个对应一个外设

  4. `etc`：包含系统的配置文件

  5. `home`：用户的私人目录，Linux 中每个用户在`home`目录下都有自己的私人目录

  6. `lib`：英语 library 的缩写，表示库，包含程序所调用的库文件

  7. `media`：当一个可移动的外设插入电脑时，Linux 就可以让我们通过 media 的子目录来访问这些外设的内容

  8. `mnt`：表示挂载，一般用于临时挂载一些装置

  9. `opt`：英语 optionnal application software package 的缩写，表示可选的应用程序，用于安装多数第三方软件和插件

  10. `root`：超级用户 root 的家目录/主目录

  11. `sbin`：系统二进制文件，包含系统级的重要可执行程序

  12. `srv`：service 的缩写，包含一些网络服务启动之后所需要存取的数据

  13. `tmp`：temporary 的缩写，表示临时的，普通用户和程序存放临时文件的地方

  14. `usr`：Unix Software Resource 的缩写，表示 Unix 操作系统软件资源，安装了大部分用户要调用的程序

  15. `var`：variable 的缩写，表示动态的、可变的。通常包含程序的数据，比如一些日志文件

      ![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224553.png)

* `pwd`命令，显示当前目录的路径。Print Working Director 的缩写，表示打印当前工作目录

  ![image-20200516205610545](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212723.png)

* `which`命令，获取命令的可执行文件的位置

  ![image-20200516205807314](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212726.png)

### 目录相关的命令：`ls`、`cd`、`du`

* `ls`是`list`的缩写，用于列出文件和目录

  1. `-a`：显示所有的文件和目录，包括隐藏的，`-A`不列出`.`和`..`两个文件

     ![image-20200516210244203](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212733.png)

  2. `-l`：列出一个显示文件和目录的详细信息列表

     ![image-20200516210406392](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224555.png)

  3. `-h`：以 Ko、Mo、Go 的形式显示文件大小

     仅用`ls -l`时，列出的文件详细信息中，文件的大小是以字节为单位的，可以加一个参数`-h`，`h`是 human-readable 的缩写，表示适合人阅读的。

     ![image-20200516210935889](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212741.png)

  4. `-t`：按文件最近一次修改时间排序

     ![image-20200516211204284](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224557.png)

* `cd`命令：切换目录

  `cd`是 change directory 的缩写，表示切换目录。

  1. `cd ..`：切换到当前目录的上级目录
  2. `cd -`：切换到前一个目录
  3. `cd ~`或`cd`返回到家目录

* `du`命令：显示目录包含的文件大小

  du命令会深入遍历每个目录的子目录，把所有文件的大小都做一个统计。`du`是 disk usage 的缩写。

  1. `-h`，同`ls -lh`，以 Ko、Mo、Go 的形式显示文件大小

     ![image-20200516212014414](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212812.png)

  2. `-a`,显示文件和目录大小

  3. `-s`，只显示当前目录的总大小

     ![image-20200516212031765](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224558.png)

### 文件操作，创建和浏览文件：`cat`、`less`、`head`、`tail`、`touch`、`mkdir`

* `cat`命令：一次性显示文件的所有内容

  `cat`命令的描述是：concatenate files and print on the standard output，意思是：把文件链接起来，一并打在在标准输出。

  1. `-n`，在显示的文件内容上加上行号

     ![截屏2020-05-30 22.48.36](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224559.png)

* `less`命令：分页显示文件内容

  `less`命令用法和`cat`命令是一样的，也是在命令后直接加文件的路径。`less`命令的好处是它会先读入文件开始的若干行，然后就停在那里，而这若干行的行数取决于终端屏幕的大小。

  * 空格键：文件内容读取下一个终端屏幕的行数，相当于前进一个屏幕。
  * 回车键：文件内容读取下一行，也就是前进一行。
  * d 键：前进半页。
  * b 键：后退一页。
  * y 键：后退一行。
  * u 键：后退半页。
  * q 键：停止读取文件，终止`less`命令。
  * `/`：进入搜索模式，只要在斜杠后面输入要搜索的文字，按下回车键，就会把素有符合的结果标识出来。要在搜索所得结果中跳转，可以按 n 键（跳到下一个结果），N 键（跳到上一个结果）。

  

* `head`命令：显示文件开头

  默认情况下，`head`会显示文件的头 10 行。

  ![image-20200530230420952](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202005/233406.png)

  1. `-n`，指定显示的行数

     ![image-20200530230546793](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212831.png)

* `tail`命令：显示文件结尾

  默认情况下，`tail`命令会显示文件的尾10 行

  1. `-n`，指定显示的行数

  ![image-20200530230742256](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224602.png)

  2. `-f`，实时追踪文件的更新，默认地，`tail -f`每隔 1 秒检车一下文件是否有更新。可以使用`-s`参数指定间隔检查的秒数。

     ![image-20200530231351046](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212843.png)

* `touch`命令：创建一个空白文件

  `touch`命令设计初衷是修改文件的时间戳，就是可以修改文件的创建时间或修改时间，让电脑以为文件就是在那个时候被修改或创建的。如果`touch`后面跟着的文件名是不存在的，那么它就会新建一个。

  ![image-20200530231710427](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224603.png)

* `mkdir`命令：创建一个目录

  `mkdir`其实是`mk`和`dir`的缩合，`mk`是`make`的缩写，表示创建，`dir`是`directory`的缩写，表示目录。

  ![image-20200530232002164](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212852.png)

  1. `-p`，递归地创建目录结构

     ![image-20200530232138483](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212856.png)

### 文件的复制，移动，删除和链接：`cp`、`mv`、`rm`、`ln`

* `cp`命令，复制文件和目录

  `cp`命令是`copy`的缩写，用于文件和目录的拷贝。

  ![image-20200601222836656](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224604.png)

  1. `-r`、`-R`，拷贝目录

     ![image-20200601223101653](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212902.png)

* `mv`命令，移动文件或目录（重命名文件或目录）

  ![image-20200601223258402](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202006/223258.png)
  
* `rm`命令，删除文件和目录
  
  `rm`是`remove`的缩写，表示移除，`rm`命令可以删除一个文件、多个文件、目录。
  
  ![image-20200601224405343](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224605.png)
  
  1. `-i`，向用户确认是否删除，对于每一个要删除的文件，终端都会询问我们是否确定删除，`i`是`inform`的缩写。
  
     ![image-20200601224604689](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212915.png)
  
  2. `-f`，不会询问是否删除，强制删除，`f`是`force`的缩写。
  
  3. `-r`，递归地删除，使用`-r`参数，可以删除目录，并且递归地删除其包含的子目录和文件。
  
     ![image-20200601224852674](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224606.png)
  
* `ln`命令，创建链接
  
  `ln`是`link`的缩写，用于在文件之间创建链接。Linux 有两种链接类型：
  
  * Physical link：物理链接或硬链接
  * Symbolic link：符号链接或软链接
  
  1. 创建硬链接，一般情况下，只能创建指向文件的硬链接
  
     ![image-20200601225457231](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224607.png)
  
  2. `-s`，创建软连接
  
     ![image-20200601225746103](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212929.png)
  
### 用户和权限、用户管理

* [ ] TODO

---
layout: post
categories: 'work'
title: '一次心血来潮的C程序编译 && makefile'
date: 2018-10-09
---

## 前言
想复习下数据结构，所以看了看相关的课程后打算手写一些东西，比如链表或者说其他的常用数据结构。

## 环境

 - MacOSX
 - VSCode

本来打算在xcode上直接写纯C的程序的，但是写了几行就发现。。真鸡儿麻烦啊，而且我xcode用的也不好，各种快捷键也不熟就被劝退了。然后看到公司同事(server)很多都在用VSCode，可能是因为大家都在用mac笔记本的原因装visualstudio基本上就是扯淡，然后vscode也比较不错就采用了这个ide。

## 前置条件
* 安装插件
![插件安装](/images/makefile/makefile1.png)
其实也就是这三个**c/c+	c/c++ Clang	CodeRunner** 

* 编译环境配置
  1. vscode下：command+shift+p 调出控制中心 输入Tasks: Configure Task 来配置任务![在这里插入图片描述](https://img-blog.csdn.net/20181009171826429?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FsZXgxOTkyYXpo/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
   2. vscode调试界面选中齿轮设置按钮 选择c/c++（也就是一开始我们安装的那几个插件提供的）![在这里插入图片描述](https://img-blog.csdn.net/20181009172050449?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FsZXgxOTkyYXpo/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
基本不需要改动就是program也就是运行程序里我改成了alex也就是我的这个程序名，这个随意。

**以上就具备了在vscode下调试c/c++程序的功能了**

## 编写代码
* list.h 链表头文件 声明 结构体 以及几个 链表操作方法![在这里插入图片描述](/images/makefile/makefile2.png)
* list.c 链表普通操作方法的实现![在这里插入图片描述](/images/makefile/makefile3.png)
* main.c 入口文件 引入list.h![在这里插入图片描述](/images/makefile/makefile4.png)

## 问题暴露
要想运行一个c的程序必须经过 编译+链接+运行 这三个阶段至于每个阶段都干了什么可以搜索其他相关文文章。这里我开始只是想简单的编译跑一下，那个时候我还没有使用makefile的方式，而是命令行(配置在task中)截图看下：![在这里插入图片描述](/images/makefile/makefile5.png)
这样的方式，大家可以看到和我上面写的command有个很大的不同，上面的是make的方式也就是采用makefile，这里这个是直接命令行通过g++来进行编译文件，这个是我参考大佬们来学习的[大佬文章地址](https://blog.csdn.net/qq_37747262/article/details/81151037) 。很简单，就是你当前焦点的文件进行g++ 编译 -g选项是为了debug调试 -o是为了给生成的目标文件重命名。
在这样的前提下，我就跑的我的程序，在main.c下按键command+shift+B来编译程序![在这里插入图片描述](/images/makefile/makefile6.png)
可以看到报错了，执行了task g++ xxxx -o xxx -g 下面为什么报错，因为insert也好display_list这些标记（方法）都不存在，为什么？这就是我一开始犯错的原因了。这个main.c 以来了list，但是我这里编译命令里并没有如何编译以来的内容，所以造成了这个报错现象。

## 解决方案MakeFile
发现了上面暴露的问题之后，我就开始搜索相关的内容和文章来看如何把以来文件也进行编译。找到了几篇相关的文章
[linux+vsCode+makefile – 调试C](https://blog.csdn.net/z896435317/article/details/77948086)
[Makefile简易教程](https://www.cnblogs.com/owlman/p/5514724.html)
我这里是根据第二篇文章进行学习的，第一篇文章给了我启发。

## MakeFile
### 一个普通的makefile
```
calc: main.c getch.c getop.c stack.c
	gcc -o calc main.c getch.c getop.c stack.c 
```
它主要分成了三个部分，第一行冒号之前的calc，我们称之为目标（target），被认为是这条语句所要处理的对象，具体到这里就是我们所要编译的这个程序calc。冒号后面的部分（main.c getch.c getop.c stack.c），我们称之为依赖关系表，也就是编译calc所需要的文件，**这些文件只要有一个发生了变化，就会触发该语句的第三部分**，我们称其为命令部分，相信你也看得出这就是一条编译命令。现在我们只要将上面这两行语句写入一个名为Makefile或者makefile的文件，然后在终端中输入make命令，就会看到它按照我们的设定去编译程序了。
但是这个问题很多
ex: 没能解决当我们只修改一个文件时就要全部重新编译的问题
ex: 如果我们需要往工程中添加一个.c或.h，可能同时就要再手动为obj常量再添加第一个.o文件，如果这列表很长，代码会非常难看

### 一个最终的makefile
```
cc = gcc
prom = alex
deps = $(shell find ./ -name "*.h")
src = $(shell find ./ -name "*.c")
obj = $(src:%.c=%.o) 

$(prom): $(obj)
	$(cc) -o $(prom) $(obj)

%.o: %.c $(deps)
	$(cc) -g -c $< -o $@

clean:
	rm -rf $(obj) $(prom)
```
这里解释一下makefile里的特殊符号
* %.o:%.c，这是一个模式规则，表示所有的.o目标都依赖于与它同名的.c文件（当然还有deps中列出的头文件）
* \$<代表的是依赖关系表中的第一项（如果我们想引用的是整个关系表，那么就应该使用$^），具体到我们这里就是%.c。
* $@代表的是当前语句的目标，即%.o。这样一来，make命令就会自动将所有的.c源文件编译成同名的.o文件。不用我们一项一项去指定了。
* shell函数主要用于执行shell命令，具体到这里就是找出当前目录下所有的.c和.h文件。
* $(src:%.c=%.o)则是一个字符替换函数，它会将src所有的.c字串替换成.o，实际上就等于列出了所有.c文件要编译的结果

## 终于跑起来
按照makefile的方式编辑好后，在main.c下按键command+shift+B来编译工程![在这里插入图片描述](/images/makefile/makefile7.png)
可以看到根据我们编写的makefile逐步处理ex:我们编译alex这个程序需要mian.c list.c ... 这里就会逐个编译list.c->list.o 一次类推，最后生成我们的目标文件-o alex 以来 xxx.o xx.o。
同样因为我们在命令里加了-g所以可以在vscode里进行调试（反之我们去掉-g就不能调试了）![在这里插入图片描述](/images/makefile/makefile8.png)


---
layout: post
categories: opengl
title: '开始我的Opengl学习之路（rua）'
date: 2018-08-15
---

**这一系列的我的总结或者我的想法都是基于[LearnOpengl](https://learnopengl-cn.github.io/)来展开的，所以要学过那个教程之后还不懂可以来看下我的土话，我的想法可能会让你理解的轻松点。**

> ## 搭TM的环境

**前言**： 关于之前的博客可能N久都未更新了，也是心血来潮写了一篇关于状态机的东西，后来也就因为各种原因（其实就是自己懒）而搁置了，今天要捡起来继续弄下博客记录下最近Opengl的学习经历

**

> 请注意 > 这些都是我在结和LearnOpengl的基础上的一些记录或者新的总结所以基本的代码可以从LearnOpengl上去看去照着写，我在这里只讲一些思路思想帮助更好的消化

**

**正式开始**


----------
今天先说下 基础的东西，关于Opengl大家都要学习的一个基本步骤。建议还是从搭环境开始说起来，毕竟干什么事准备工作往往是最关键也是最麻烦的一步，也不需要问太多就先搭（答应我先不要去看各种教程或者书上说的各种Opengl思想和原理） 废话不多说Go

> ###个人环境

 - Mac平台
 - Xcode
 - GLFW GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建OpenGL上下文，定义窗口参数以及处理用户输入，这正是我们需要的

是的，你没有看错就这三就行了什么Glad什么各种其他的东西感觉反而会让你还没写没学就因为一个环境就完蛋了不想学了，这里一切从简。

> ###如何配置

* 一台Mac笔记本老铁既然你看了这文章不是程序员我给你五毛钱。讲道理去买一台mac作为程序员拥有一台mac简直就像可以双飞一样High
* Xcode很简单打开你的mac的应用商城去下载一个就好这里我写文章的时候Version 9.4.1 (9F2000)
* 至于GLFW这个提供基本功能的库怎么弄很多人说XXX又要make又要各种咋弄么，我这里推荐一个方案。**brew**

关于Brew简单的理解为包管理工具像Mac这种类linux的操作系统不像windows那样很傻瓜，软件安装起来也没那么直接。经常下你就会用的软件很乱，通过一个软件包管理工具很好的管理是个好习惯

> 如何安装Brew在Mac下？

```
curl -LsSf http://github.com/mxcl/homebrew/tarball/master | sudo tar xvz -C/usr/local --strip 1
```

安装完成以后你就可以通过这个去安装glfw这个东东了
sudo brew install glfw
![这里写图片描述](/images/opengl/opengl_start1.png)
这里我已经安装完了 你可以通过brew list 命令查看 你通过brew所安装的所有软件包

安装了glfw之后你会在你的mac目录 /usr/local/lib/  下找到
![这里写图片描述](/images/opengl/opengl_start2.png)
如图所示的库，是不是很简单 是不是不用你编译什么啦？（偷笑ing）

接下来需要做的就是打开你的xcode新建一个cpp工程名字随便啦我这里叫opengl然后把这俩lib拖进去就好如图![这里写图片描述](/images/opengl/opengl_start3.png)

OK 大功告成，最基本的环境已经具备了，接下来就得等我有时间写喽？

PS：补充下一些注意点
* 关于工程的一些搜索设置看如图
![这里写图片描述](/images/opengl/opengl_start4.png)

* 关于项目workSpace设置看如图
![这里写图片描述](/images/opengl/opengl_start5.png)
这样你的Products下编译出来的执行文件的路径才是你想要的那个路径

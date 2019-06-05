---
layout: post
categories: 'architecture'
title: '无意间看到Pure-Mvc记录下'
date: 2018-09-14
---


## Pure-MVC ##
###前言###
   在学习creator的一些框架的时候看到一些使用了MVC的内容，就在此做个思考和总结。PureMVC-TypeScript版本代码量很少非常容易看懂。
   
### 什么是MVC ###

开局一张图，内容全靠想
![这里写图片描述](/images/mvc/puremvc1.png)

 * 很明显的分层设计 Model View Controller
 * 但是这三个部分都是单例 都是单例 都是单例 重要的事说三遍
 * 要想扩展各式各样的 MVC 要用 对应的中间类
 * Model - Proxy 
 * View - Mediator - UI （可以看出来MVC都是单纯的单例Class具体UI什么的都是在Mediator来持有的）
 * Controller - Command
 
 然后介绍下 这些个中间类
 
 * Proxy: Model 保存对 Proxy 对象的引用，Proxy 负责操作数据模型，与远程服务通信存取数据。ex:HttpProxy TcpProxy ..
 
 * Mediator: View 保存对 Mediator 对象的引用 。由 Mediator 对象来操作具体的视图组件，包括：添加事件监听器，发送或接收 Notification ，直接改变视图组件的状态。这样做实现了把视图和控制它的逻辑分离开来。
 
 * Command: Controller 保存所有 Command 的映射。Command 类是无状态的，只在需要时才被创建。Command 可以获取 Proxy 对象并与之交互，发送 Notification，执行其他的 Command
 
 这里做一个思考，其实有点顾名思义的感觉，Controller就是一个控制器，和控制器交互的最简单也直白的方法就是命令Command。 

### 延伸扩展 ###
![这里写图片描述](/images/mvc/puremvc2.png)

这个图是Pure-MVC的一个全面介绍
* 核心层MVC 三个单例
* 封装对外对接者Facade，这个单词也就是表面的意思很好理解，就是壳
* 然后衍生出的所有Proxy与Mediator之间的通讯都靠Notification观察者模式实现
* Command是核心逻辑与业务的集中地

### 工程目录 ###
![这里写图片描述](/images/mvc/puremvc3.png)

这里贴一张当前我在学习的框架的目录结构

* controller：包含两个目录和一个启动命令文件 boostrap里放了Command以及Model以及View的启动文件 commands下放了所有的command
* model：包含了所有的proxy代理类
* view：包含了所有的 Mediator中间类，由于creator是组件化开发这里多了个component目录存放UI组件（也就是UI都是组件化的了）

### 实例代码 ###
![这里写图片描述](/images/mvc/puremvc4.png)

![这里写图片描述](/images/mvc/puremvc5.png)

可以看到组件的UI里在生命周期函数start中通过唯一对外的AppFacade来进行registerMediator也就是让单例View持有的Map数据表里增加一个这个mediator的引用。这个mediator就是参数里new出来的一个。

然后，在这个StartViewMediator中间类里进行了相关的具体这个UI的相关操作，onRegister在上面registerMediator中就会自动触发。

注意点：Mediator和Proxy也就是UI的持有者和数据的持有者不应该直接交互，但是这里为什么可以，因为其实实际项目避免不了想拿到某些数据，PureMvc也提供了方便的API，retrieveProxy来获取某一个数据的代理对象从而获取数据操作数据。

### 相关链接 ###
  [puremvc](http://puremvc.org/)
  [示例工程git地址](https://github.com/Ander456/skynet-cocos-creator-client)
  [一篇很好的文章](http://m.gad.qq.com/#/article/detail/287236)

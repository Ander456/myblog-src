---
layout: post
categories: 'work'
title: '关于 quick-cocos 状态机'
date: 2016-12-12
---

我也不知道第一篇博客到底要搞什么，刚好最近有朋友在研究状态机，想起自己在quick-cocos的时候研究过这里再做个详细的研究或者介绍。
	其实，讲道理状态机是什么我没法给你讲的太清楚，顾名思义就是一个管理状态的机器，你这么理解就差不多的。quick引擎是一款很轻量级的2D游戏引擎（不是广告啊兄弟们），它的源码也都很简单易懂，加上cocos引擎的luabinding可以使用lua语言快速开发各种2d游戏，而在源码里，如果你很细心，那么肯定会发现有一个叫stateMachine的lua文件 ，对就是它，是它就是它我们的状态机源码。
	对于如何快速理解这个状态机，我想我们直接举例子来说吧


状态机创建
-----

```
self.fsm = {}  			  cc.GameObject.extend(self.fsm):addComponent("components.behavior.StateMachine"):exportMethods()  
```

初始化状态机参数
-----------

	设置状态逻辑是重写setupState方法，这其中有这么几个字段参数，

	initial：状态机的初始状态 
	terminal （final）：结束状态 
	events：状态发生转变时对应的事件 
	callbacks：发生转变时的回调函数

	一般我们会设置initial，events和callbacks这三个。
	
	先看events，在events中需要分清楚“事件”和“状态”，events采用table结构，例如我们来写一个
```
events = { 
	{name = “move”, from = {“idle”, “jump”}, to = “walk”}, 
	｝
```
	这里的move就是事件名称，然后后面的from和to就很好理解了，就是起始的状态到变化后的状态。
	当然，复杂一点的可以这么写：
```
	events = { 
	{name = “move”, from = {“idle”, “jump”}, to = “walk”}, 
	{name = “attack”, from = {“idle”, “walk”}, to = “jump”}, 
	{name = “normal”, from = {“walk”, “jump”}, to = “idle”}, 
	},
```
	注意，from可以是一个或者多个，但是to也就是目标状态只能有一个，因为你想啊，你总不能不确定你到底想变成什么就要变吧


	

callbacks参数
------------------------------------

 - onbeforeEVNET： 在事件EVENT开始前被激活 
 - onleaveSTATE： 在离开旧状态STATE时被激活 
 - onenterSTATE 或 onSTATE：在进入新状态STATE时被激活 
 - onafterEVENT 或 onEVENT：在事件EVENT结束后被激活

	此外还有5种通用型的回调来捕获所有事件和状态的变化:
 - onbeforeevent： 在任何事件开始前被激活 
 - onleavestate： 在离开任何状态时被激活 
 - onenterstate：在进入任何状态时被激活 
 - onafterevent ：在任何事件结束后被激活 
 - onchangestate ：当状态发生改变的时候被激活 
	这里面的名称是不可以修改的，它是针对于任何事件和任何状态的。 

	

状态机调用
-----------
	通过self.fsm:doEvent(event)就可以了，参数event对应events参数名称。 
	fsm:isReady() ：返回状态机是否就绪 
	fsm:getState() ：返回当前状态 
	fsm:isState(state) ：判断当前状态是否是参数state状态 
	fsm:canDoEvent(eventName) ：当前状态如果能完成eventName对应的event的状态转换，则返回true 
	fsm:cannotDoEvent(eventName) ：当前状态如果不能完成eventName对应的event的状态转换，则返回true 
	fsm:isFinishedState() ：当前状态如果是最终状态，则返回true 
	fsm:doEventForce(name, …) ：强制对当前状态进行转换

	
## 代码部分 ##
```
local Player = class(“Player”, function () 
	return display.newSprite(“icon.png”) 
	end)

	function Player:ctor() 
		self:addStateMachine() 
	end

	function Player:doEvent(event) 
		self.fsm:doEvent(event) 
	end

	function Player:addStateMachine() 
	self.fsm = {} 			  cc.GameObject.extend(self.fsm):addComponent(“components.behavior.StateMachine”):exportMethods()

	self.fsm:setupState({  
    initial = "idle",  

    events = {  
        {name = "move", from = {"idle", "jump"}, to = "walk"},  
        {name = "attack", from = {"idle", "walk"}, to = "jump"},  
        {name = "normal", from = {"walk", "jump"}, to = "idle"},  
    },  

    callbacks = {  
        onenteridle = function ()  
            local scale = CCScaleBy:create(0.2, 1.2)  
            self:runAction(CCRepeat:create(transition.sequence({scale, scale:reverse()}), 2))  
        end,  

        onenterwalk = function ()  
            local move = CCMoveBy:create(0.2, ccp(100, 0))  
            self:runAction(CCRepeat:create(transition.sequence({move, move:reverse()}), 2))  
        end,  

        onenterjump = function ()  
            local jump = CCJumpBy:create(0.5, ccp(0, 0), 100, 2)  
            self:runAction(jump)  
        end,  
    },  
	})  
	end

	return Player

```

```
local Player = import(“..views.Player”)
	
	local MyScene = class(“MyScene”, function () 
	return display.newScene(“MyScene”) 
	end)
	
	function MyScene:ctor()
	
	local player = Player.new()  
	player:setPosition(display.cx, display.cy)  
	self:addChild(player)  
	
	local function menuCallback(tag)  
	    if tag == 1 then   
	        player:doEvent("normal")  
	    elseif tag == 2 then  
	        player:doEvent("move")  
	    elseif tag == 3 then  
	        player:doEvent("attack")  
	    end  
	end  
	
	local mormalItem = ui.newTTFLabelMenuItem({text = "normal", x = display.width*0.3, y = display.height*0.2, listener = menuCallback, tag = 1})  
	local moveItem =  ui.newTTFLabelMenuItem({text = "move", x = display.width*0.5, y = display.height*0.2, listener = menuCallback, tag = 2})  
	local attackItem =  ui.newTTFLabelMenuItem({text = "attack", x = display.width*0.7, y = display.height*0.2, listener = menuCallback, tag = 3})  
	local menu = ui.newMenu({mormalItem, moveItem, attackItem})  
	self:addChild(menu)  
	end
	
	return MyScene
```

	
	

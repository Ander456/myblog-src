---
layout: post
categories: opengl
title: 'Opengl-帧缓冲(一个新的缓冲对象，想一下深度和模板缓冲)'
date: 2018-08-31
---

----------

**OpenGL中的缓冲只是一个管理特定内存块的对象，没有其它更多的功能了。**

----------
首先我们明确一个原理或者道理，不论是什么缓冲，深度也好，模板也罢。再往前说的顶点数据的颜色缓冲。都是一个存储的单元，都是一个存储对象，管理一块特定的内存。没有什么特殊的功能。千万别想太多，记住了哈。

###帧缓冲
FBO 也就是帧缓冲 FrameBufferObject ，可以用来做很多事，核心上来说，它干了什么了？
**把所有的渲染都绘制到了一张图上**，这就是它干的，也是它经常被用来干的。

**如何使用**：

```
unsigned int fbo;
glGenFramebuffers(1, &fbo);
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
```
看着很眼熟吧。。。其实既然是Opengl的对象都是这样的，你现在有没有点豁然开朗的感觉。先Gen生成，然后Bind到状态机的Context上就表示我们接下来开始使用这个。回想一下，前面说VAO的时候就明白了

![这里写图片描述](/images/opengl/framebuffer1.png)

看下这张图，然后我们来深入的了解下这个FBO。

**帧缓冲条件**：

* 附加至少一个缓冲（颜色、深度或模板缓冲）。
* 至少有一个颜色附件(Attachment)。
* 所有的附件都必须是完整的（保留了内存）。
* 每个缓冲都应该有相同的样本数。

其实简单理解就是：**帧缓冲区对象只是一个容器，它可以保存其他确实有内存存储并且可以进行渲染的对象，如纹理或者渲染缓冲区。**

到目前为止，我们见过很多缓冲了：用于写入颜色值的 **颜色缓冲** 、用于写入深度信息的 **深度缓冲** 和允许我们根据一些条件丢弃特定片段的 **模板缓冲** 。**这些缓冲结合起来叫做帧缓冲** (Framebuffer)，它被储存在内存中。OpenGL允许我们定义我们自己的帧缓冲，也就是说我们能够定义我们自己的颜色缓冲，甚至是深度缓冲和模板缓冲。

----------


**附件**：

* 纹理附件 ：当把一个纹理附加到帧缓冲的时候，所有的渲染指令将会写入到这个纹理中，就想它是一个普通的颜色/深度或模板缓冲一样。使用纹理的优点是，所有渲染操作的结果将会被储存在一个纹理图像中，我们之后可以在着色器中很方便地使用它。

```
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
```
可以看到GL_COLOR_ATTACHMENT0参数表示附件类型是颜色缓冲


----------


* 渲染缓冲对象附件(rbo)：和纹理图像一样，渲染缓冲对象是一个真正的缓冲，即一系列的字节、整数、像素等。渲染缓冲对象附加的好处是，它会将数据储存为OpenGL原生的渲染格式，**它是为离屏渲染到帧缓冲优化过的**。由于渲染缓冲对象通常都是只写的，**它们会经常用于深度和模板附件**专门为绑定到FBO而设计

```
unsigned int rbo;
glGenRenderbuffers(1, &rbo);
glBindRenderbuffer(GL_RENDERBUFFER, rbo); 
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);  
glBindRenderbuffer(GL_RENDERBUFFER, 0);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```
我们可以看到GL_DEPTH_STENCIL_ATTACHMENT这个参数指定了附件类型，这里我们让rbo来存下深度和模板信息。


----------

常用帧缓冲写入纹理来实现一些特殊效果。你想象一下，当所有的场景都被渲染到了一个张图上，就是一个图片一样，这个时候我们想对某一个像素点或者什么东西做什么的时候，是不是很方便就能找到？纹理想一哈？

步骤：

**生成帧缓冲**

```
unsigned int framebuffer;
glGenFramebuffers(1, &framebuffer);
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
```

**生成纹理并绑定到为帧缓冲附件**

```
// 生成纹理
unsigned int texColorBuffer;
glGenTextures(1, &texColorBuffer);
glBindTexture(GL_TEXTURE_2D, texColorBuffer);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glBindTexture(GL_TEXTURE_2D, 0);

// 将它附加到当前绑定的帧缓冲对象
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texColorBuffer, 0);  
```

**生成rbo绑定为深度模板附件**
```
unsigned int rbo;
glGenRenderbuffers(1, &rbo);
glBindRenderbuffer(GL_RENDERBUFFER, rbo); 
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);  
glBindRenderbuffer(GL_RENDERBUFFER, 0);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

```
// 第一处理阶段(Pass)
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); // 我们现在不使用模板缓冲
glEnable(GL_DEPTH_TEST);
DrawScene();    

// 第二处理阶段
glBindFramebuffer(GL_FRAMEBUFFER, 0); // 返回默认
glClearColor(1.0f, 1.0f, 1.0f, 1.0f); 
glClear(GL_COLOR_BUFFER_BIT);

screenShader.use();  
glBindVertexArray(quadVAO);
glDisable(GL_DEPTH_TEST);
glBindTexture(GL_TEXTURE_2D, textureColorbuffer);
glDrawArrays(GL_TRIANGLES, 0, 6);  
```

![这里写图片描述](/images/opengl/framebuffer2.png)

如图就是把箱子场景绘制到了一张图上虽然你看不出来，但是确实如此。而右边就是把原来提供的默认帧缓冲渲染了一个四边形


----------
**OpenGL中的缓冲只是一个管理特定内存块的对象，没有其它更多的功能了。**
**帧缓冲多用来写入纹理，通过添加一个纹理附件把元素都渲染都这张纹理图上**

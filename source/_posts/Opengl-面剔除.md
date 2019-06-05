---
layout: post
categories: opengl
title: 'Opengl-面剔除(一种优化方式)'
date: 2018-08-30
---

> ##前言

我们已经可以正常的渲染一个物体也好加载一个模型也好，比如立方体，可以正常的渲染，然后我们还可以挪动摄像机来观察每一个面，但是，有一个问题想一想。我们挪动相机进入了立方体里面就会发现里面也渲染了。这。。问题出来了。可能很多东西我们都不希望或者根本没想着把里面也渲染的那么好。。我们看不到还浪费这个资源去计算渲染干嘛？

> ##面剔除

OpenGL能够检查所有面向(Front Facing)观察者的面，并渲染它们，而丢弃那些背向(Back Facing)的面，节省我们很多的片段着色器调用（它们的开销很大！）。但我们仍要告诉OpenGL哪些面是正向面(Front Face)，哪些面是背向面(Back Face)。OpenGL使用了一个很聪明的技巧，分析顶点数据的环绕顺序(Winding Order)。
![这里写图片描述](/images/opengl/fontface.png)

```
float vertices[] = {
    // 顺时针
    vertices[0], // 顶点1
    vertices[1], // 顶点2
    vertices[2], // 顶点3
    // 逆时针
    vertices[0], // 顶点1
    vertices[2], // 顶点3
    vertices[1]  // 顶点2  
};
```

**如何使用**：
glEnable(GL_CULL_FACE);
glCullFace(GL_FRONT);

**使用规则**：
GL_BACK：只剔除背向面。
GL_FRONT：只剔除正向面。
GL_FRONT_AND_BACK：剔除正向面和背向面。

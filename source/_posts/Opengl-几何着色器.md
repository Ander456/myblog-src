---
layout: post
categories: opengl
title: 'Opengl-几何着色器(劫持顶点的家伙)'
date: 2018-09-01
---

> ##前言

我们知道在片段着色器和顶点着色之间，有一个几何着色器，我们之前说过它但是一直没有讲，这里了解一哈。

几何着色器的输入是一个图元（如点或三角形）的一组顶点。几何着色器可以在顶点发送到下一着色器阶段之前对它们随意变换。然而，几何着色器最有趣的地方在于，它能够将（这一组）顶点变换为完全不同的图元，并且还能生成比原来更多的顶点。

```
#version 330 core
layout (points) in;
layout (line_strip, max_vertices = 2) out;

void main() {    
    gl_Position = gl_in[0].gl_Position + vec4(-0.1, 0.0, 0.0, 0.0); 
    EmitVertex();

    gl_Position = gl_in[0].gl_Position + vec4( 0.1, 0.0, 0.0, 0.0);
    EmitVertex();

    EndPrimitive();
}
```
可以看到出来，几何着色和前面我们学习的顶点片段没有什么太大的不同。
layout的类型：

* points：绘制GL_POINTS图元时。
* lines：绘制GL_LINES或GL_LINE_STRIP时
* lines_adjacency
* triangles
* triangles_adjacency

以上是能提供给glDrawArrays渲染函数的几乎所有图元了

**in还是in out还是out 一个用来流入一个用来流出**

**EmitVertex**：每次我们调用EmitVertex时，gl_Position中的向量会被添加到图元中来。

**EndPrimitive**：当EndPrimitive被调用时，所有发射出的(Emitted)顶点都会合成为指定的输出渲染图元。在一个或多个EmitVertex调用之后重复调用EndPrimitive能够生成多个图元。

不理解没关系 我们再看一下可编程渲染管线的图
![这里写图片描述](/images/opengl/pipeline.png)

1.  首先进入几何着色器的是什么？ 是图元，其实就是一个图形你理解成这样就好
2.  几何着色器流出的是什么？ 还是图元，是的你没看错，就是图元，如果你不自己定义opengl会默认的，如果你自己定义了会根据你的代码来把图元改变然后发送到下一个阶段

```
#version 330 core
layout (points) in;
layout (triangle_strip, max_vertices = 5) out;

void build_house(vec4 position)
{    
    gl_Position = position + vec4(-0.2, -0.2, 0.0, 0.0);    // 1:左下
    EmitVertex();   
    gl_Position = position + vec4( 0.2, -0.2, 0.0, 0.0);    // 2:右下
    EmitVertex();
    gl_Position = position + vec4(-0.2,  0.2, 0.0, 0.0);    // 3:左上
    EmitVertex();
    gl_Position = position + vec4( 0.2,  0.2, 0.0, 0.0);    // 4:右上
    EmitVertex();
    gl_Position = position + vec4( 0.0,  0.4, 0.0, 0.0);    // 5:顶部
    EmitVertex();
    EndPrimitive();
}

void main() {    
    build_house(gl_in[0].gl_Position);
}
```
![这里写图片描述](https://img-blog.csdn.net/20180817151021572?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FsZXgxOTkyYXpo/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以看到，我们把发送到几何着色器的一个顶点，然后做了五次便宜，组成了一个房子。。我们虽然只定义了一个顶点数据比如：

```
float points[] = {
        -0.5f,  0.5f, 1.0f, 0.0f, 0.0f, // top-left
         0.5f,  0.5f, 0.0f, 1.0f, 0.0f, // top-right
         0.5f, -0.5f, 0.0f, 0.0f, 1.0f, // bottom-right
        -0.5f, -0.5f, 1.0f, 1.0f, 0.0f  // bottom-left
    };
```
-0.5f,  0.5f, 1.0f, 0.0f, 0.0f, // top-left 这就一个顶点的数据。。但是经过我们的劫持改变把这里变成了一个房子的形状了

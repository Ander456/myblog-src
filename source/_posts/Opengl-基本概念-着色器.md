---
layout: post
categories: opengl
title: 'Opengl-基本概念-着色器（都是固定的）'
date: 2018-08-18
---

> ##固定的结构

* **顶点着色器**
```
#version 330 core
layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为0

out vec4 vertexColor; // 为片段着色器指定一个颜色输出

void main()
{
    gl_Position = vec4(aPos, 1.0); // 注意我们如何把一个vec3作为vec4的构造器的参数
    vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // 把输出变量设置为暗红色
}
```

* **片段着色器**
```
#version 330 core
out vec4 FragColor;

in vec4 vertexColor; // 从顶点着色器传来的输入变量（名称相同、类型相同）

void main()
{
    FragColor = vertexColor;
}
```

看到了吧？就这么简单 这就是你要学的shader，一个是顶点的一个是片段的。
核心思想：
1. in 输入也就是别人流程流到你这的
2. out 输出也就是你这里最后做完要给别人的
3. layout 顶点在内存的布局 不理解没关系一会看图说话
4. gl_ 的这些变量都是opengl的内置变量
5. 都有main函数这你就想明白了吧？写任何c程序main都是程序入口，所以说我们写的这些个着色器其实就是个小程序，挂上去之后人家跑你的main然后把早就定义好的东西流到下一个管线

关于Layout的理解
![这里写图片描述](/images/opengl/shader1.png)
如图所示，我们的点比如一个点P = {x,y,x} 我们都是在3维空间说的，至于2D的就是z的值为一个固定的0。接着说图，我们的点在内存中就是如此布局的一个紧挨一个，然后接着看另一张图
![这里写图片描述](/images/opengl/shader2.png)
看出变化了么？这里我们在刚刚位置元素xyz的旁边增加了rgb，是不是很熟悉就是颜色啊老铁。是的，这也就是我们后面要讲的纹理，这里先不说。但是我们可以看出来layout的作用了
layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为0 这句说的也比较明白了。 首先in表示这个东西是传进来的 然后看vec3表示一个三维向量而已 aPos 就是这个变量，那么我们就能简单的看出来有一个aPos传了进来，这个aPos表示的位置元素都排列在了顶点数据内存的位置为0的地方。然后以后我们想要丰富这个顶点，比如说我们不满足于光有位置我们还想有颜色，那么增加下顶点属性如图所做的我们就明白怎么做了
比如：layout (location = 1) in vec3 rgb; 是不是很简单？


----------


这就是着色器，很固定的回味下


----------
接着说一个着色器的重要角色Uniform

> Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式，但uniform和顶点属性有些不同。首先，uniform是全局的(Global)。全局意味着uniform变量必须在每个着色器程序对象中都是独一无二的，而且它可以被着色器程序的任意着色器在任意阶段访问。第二，无论你把uniform值设置成什么，uniform会一直保存它们的数据，直到它们被重置或更新。

  说的很清楚了，首先uniform是cpu->gpu发送数据的方式。然后是全局的并没有什么其他特殊的意义
  然而就是这个特殊的变量可以让我们的着色器丰富多彩因为这就是变数，我们可以通过传入任何想要传入的东西来实现我们想要的效果，你可以想想，我们上面都说了一个普通着色器所需要的元素都是那么的固定，然而要想做出变化多彩的东西肯定需要变数，也就是变化的东西这个也就是uniform的作用了是不是。

```
  #version 330 core
out vec4 FragColor;

uniform vec4 ourColor; // 在OpenGL程序代码中设定这个变量

void main()
{
    FragColor = ourColor;
}
```
看这段代码就能看出来uninform的定义和使用的方式了，我们可以通过代码

```
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```
来改变最后像素的颜色，是不是很牛逼？很简单？

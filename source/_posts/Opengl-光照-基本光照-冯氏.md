---
layout: post
categories: opengl
title: 'Opengl-光照-基本光照-冯氏(千万好好理解后面所有的延伸基本都是基于这个的)'
date: 2018-08-22
---


> ##前言

  前面我们基本理解了怎么模拟光，怎么设置光的颜色以及物体的颜色来非常不生动形象的模拟光在计算机中。肯定在想，怎么能真的像生活中那样物体可以反光，然后有凉的地方也有不凉的地方，光也有强弱这种？其实前辈们已经给了一套很好的经验体系--**冯氏光照模型**

> 冯氏光照

![这里写图片描述](/images/opengl/light_feng.png)

这图是我从LearnOpengl那里拿的，为什么用这张图解释呢？就是因为找了很多都感觉这张图真的很贴切。
从左到右依次是：环境光->漫反射光->镜面光->混合后的冯氏光照。是不是已经看到了？是不是最后的结合已经有了很逼真的感觉？

* 环境光照(Ambient Lighting)：即使在黑暗的情况下，世界上通常也仍然有一些光亮（月亮、远处的光），所以物体几乎永远不会是完全黑暗的。为了模拟这个，我们会使用一个环境光照常量，它永远会给物体一些颜色。
* 漫反射光照(Diffuse Lighting)：模拟光源对物体的方向性影响(Directional Impact)。它是冯氏光照模型中视觉上最显著的分量。物体的某一部分越是正对着光源，它就会越亮。
* 镜面光照(Specular Lighting)：模拟有光泽物体上面出现的亮点。镜面光照的颜色相比于物体的颜色会更倾向于光的颜色。

环境光非常好理解，就是给了一个基本颜色不至于让物体完全黑导致的失真

```
void main()
{
    float ambientStrength = 0.1;
    vec3 ambient = ambientStrength * lightColor;

    vec3 result = ambient * objectColor;
    FragColor = vec4(result, 1.0);
}
```
看代码，我们只需要乘以环境光影子就可以让物体具有一定基本颜色不至于很黑也不会很亮

**至于漫反射概念如图**
![这里写图片描述](/images/opengl/light_feng2.png)
你可以看到，光照射到物体表面然后因为物体表面不平导致光散射向四面八方。
我们这里解释下冯氏光照的漫反射从一个点出发看
![这里写图片描述](/images/opengl/light_feng3.png)
想象一下橙色的这个平台上的一点，我们从一个点出发来看漫反射，然后这里引入一个概念-**发现向量**其实就是垂直于一面点的一个向量。引用一句话就是：**漫反射光照使物体上与光线方向越接近的片段能从光源处获得更多的亮度**
  为什么这么说呢？你像一下，θ角度越小，光源也就越正方向照这个平台，这个时候平台反射的也就越多你能感受到看到的光也就越强。如果光线垂直于物体表面，这束光对物体的影响会最大化，也就是最亮了，如果想象不出来，你可以打开手机的手电筒，照一个东西，感受一下，是直照亮还是偏着亮？
```
out vec3 FragPos;  
out vec3 Normal;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    FragPos = vec3(model * vec4(aPos, 1.0));
    Normal = aNormal;
}
```
片段着色器的主要代码
```
vec3 norm = normalize(Normal);
vec3 lightDir = normalize(lightPos - FragPos);
float diff = max(dot(norm, lightDir), 0.0);
vec3 diffuse = diff * lightColor;
vec3 result = (ambient + diffuse) * objectColor;
FragColor = vec4(result, 1.0);
```
可以看出来，我们把漫反射因子+环境光因子 然后* 物体颜色就得出了有漫反射的情况下物体的样子
![这里写图片描述](/images/opengl/light_feng4.png)
可以看到有亮有暗，以为反射的原因我们通过移动摄像机能看出来不同地方的亮度

**镜面光照**
![这里写图片描述](/images/opengl/light_feng5.png)
其实上面的结果稍微少了那么点味道，为什么，因为少了那种很闪眼的感觉，生活中你盯着一个亮的物体看，就能看出来有时候会很亮。这个是怎么做到的呢？就是镜面光照

```
vec3 viewDir = normalize(viewPos - FragPos);
vec3 reflectDir = reflect(-lightDir, norm);
float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
vec3 specular = specularStrength * spec * lightColor;
```

  ![这里写图片描述](/images/opengl/light_feng6.png)
这次晃动你的摄像机是不是能看到高亮的白色光了？


----------
总结下：我们想真正的模拟现实世界的光照不是知道颜色影响就够了，错综复杂的世界基本不可能完全模拟，前人提供了一套经验模型冯氏光照。通过几方面，**环境影响+反射影响+高光效果 来提供一种简单的模拟**，然而这已经非常不错了。让人不自禁沉迷于光的世界。

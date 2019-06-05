---
layout: post
categories: opengl
title: 'Opengl-光照-基本光照-光照贴图(现在告别单调的方块弄个箱子)'
date: 2018-08-25
---

> ##前言

  前面我们跟着LearnOpengl学习的都是通过怎么定义一些顶点数据弄出一个立方体在三维世界里模拟光照模拟光源。看着是有点真实的样子了。。可是你见过哪个真实世界里都是这些个立方体的，肯定都是真切的物体啊，什么房子，树，石头，木头等等。要怎么模拟这些呢？怎么把这些放进去呢？很简单啊。基本章节就学习了纹理坐标，我们就已经给立方体贴上了纹理让它成为了一个箱子-结果可以去看我第一章最后的成果图。
  然后，我们还缺什么呢？就是把箱子上照上光，我们都说了第一节学习东西没有生气的原因就是死板不真实就是因为没有光，现在加上光不就是真实世界的东西了么？
  

> 现实世界中的物体通常并不只包含有一种材质，而是由多种材质所组成。想想一辆汽车：它的外壳非常有光泽，车窗会部分反射周围的环境，轮胎不会那么有光泽，所以它没有镜面高光，轮毂非常闪亮（如果你洗车了的话）。汽车同样会有漫反射和环境光颜色，它们在整个物体上也不会是一样的，汽车有着许多种不同的环境光/漫反射颜色。总之，这样的物体在不同的部件上都有不同的材质属性。


----------
###漫反射贴图
我们希望通过某种方式对物体的每个片段单独设置漫反射颜色。有能够让我们根据片段在物体上的位置来获取颜色值得系统吗？
说到这句话，有没有点点熟悉的感觉？我们希望通过**物体上的位置找到物体的颜色**。这不就是纹理么？我们纹理学的就是这么个道理啊，把纹理想象成一张地图，然后根据位置可以找到纹理上的每一个像素，这就是颜色啊。所以其实漫反射贴图没啥新东西，就是纹理的那个原理。

```
struct Material {
    sampler2D diffuse;
    vec3      specular;
    float     shininess;
}; 
```
Material的diffuse 变成了sampler2D 是因为纹理采样，这个不用多说吧。前面纹理的时候介绍过了。

shader中的核心代码
```
vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
```

![这里写图片描述](/images/opengl/light_uv1.png)

有了漫反射贴图，会不会想，有咩有镜面光贴图？是的，兄弟没想错，就是这样子的，有有有。我们可以看的出来漫反射贴图使用了之后，整个箱子都会在光源的照射下一大部分有高亮，不论是木头材质还是铁材质，而真正能晃眼的其实应该是铁，木头不应该。怎么做呢？简单的想我们可以将物体的镜面光材质设置为vec3(0.0)来解决这个问题，可是。。。这样都不反光了啊。怎么做。。肯定就是要用到镜面光贴图了

###镜面光贴图

```
struct Material {
    sampler2D diffuse;
    sampler2D specular;
    float     shininess;
};
[...]
vec3 ambient  = light.ambient  * vec3(texture(material.diffuse, TexCoords));
vec3 diffuse  = light.diffuse  * diff * vec3(texture(material.diffuse, TexCoords));  
vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
FragColor = vec4(ambient + diffuse + specular, 1.0);
```
![这里写图片描述](/images/opengl/light_uv2.png)
![这里写图片描述](/images/opengl/light_uv3.png)


----------


**看出来了么？其实核心思想就是，漫反射贴图结合光照颜色就是为了体现物体本身在光照下反射的大概颜色的。然后镜面光贴图的目的就是为了让不同的才体现出不同的反光程度。**

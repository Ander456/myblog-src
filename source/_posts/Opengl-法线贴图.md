---
layout: post
categories: opengl
title: 'Opengl-法线贴图(用来细化表面的表现表现的凹凸)'
date: 2018-09-04
---

![这里写图片描述](/images/opengl/normalmap1.png)

我们通过这张图可以看出来，使用了法线贴图的物体表面更有细节更逼真，其实这就是发现贴图的作用，没什么钻牛角尖的。

![这里写图片描述](/images/opengl/normalmap2.png)

其实表面没有凹凸的情况是因为我们把表面一直按照平整来做的，要想突出这个表面的凹凸就要用到法线贴图

**到这里，我们暂停想一下，前面说的几种贴图，漫反射贴图，镜面光贴图，然后再到这个法线贴图。明白了什么？其实很简单，就是我们法线虚拟的和现实的差距就会通过这种贴图的形式来着重表现某一方面的效果。**

```
#version 330 core
out vec4 FragColor;

in VS_OUT {
    vec3 FragPos;
    vec2 TexCoords;
    vec3 TangentLightPos;
    vec3 TangentViewPos;
    vec3 TangentFragPos;
} fs_in;

uniform sampler2D diffuseMap;
uniform sampler2D normalMap;

uniform vec3 lightPos;
uniform vec3 viewPos;

void main()
{
    // obtain normal from normal map in range [0,1]
    vec3 normal = texture(normalMap, fs_in.TexCoords).rgb;
    // transform normal vector to range [-1,1]
    normal = normalize(normal * 2.0 - 1.0);  // this normal is in tangent space
    
    // get diffuse color
    vec3 color = texture(diffuseMap, fs_in.TexCoords).rgb;
    // ambient
    vec3 ambient = 0.1 * color;
    // diffuse
    vec3 lightDir = normalize(fs_in.TangentLightPos - fs_in.TangentFragPos);
    float diff = max(dot(lightDir, normal), 0.0);
    vec3 diffuse = diff * color;
    // specular
    vec3 viewDir = normalize(fs_in.TangentViewPos - fs_in.TangentFragPos);
    vec3 reflectDir = reflect(-lightDir, normal);
    vec3 halfwayDir = normalize(lightDir + viewDir);
    float spec = pow(max(dot(normal, halfwayDir), 0.0), 32.0);
    
    vec3 specular = vec3(0.2) * spec;
    FragColor = vec4(ambient + diffuse + specular, 1.0);
}
```
这是一个法线贴图的片段着色器，我们可以看到几个关键的部分。

1. vec3 normal = texture(normalMap, fs_in.TexCoords).rgb; 我们从法线贴图的纹理中取样像素
2. 我们从 diffuseMap 中取样 物体纹理图片的原本像素
3. 然后我们把normal的普通的做了综合得出了最终的FragColor

这里面还有TagnetFragPos什么的不理解没关系，它们是切线空间的一些概念


----------


> ###切线空间

法线贴图中的法线向量在切线空间中，法线永远指着正z方向。切线空间是位于三角形表面之上的空间：法线相对于单个三角形的本地参考框架。它就像法线贴图向量的本地空间；它们都被定义为指向正z方向，无论最终变换到什么方向。使用一个特定的矩阵我们就能将本地/切线空寂中的法线向量转成世界或视图坐标，使它们转向到最终的贴图表面的方向。
切线空间中的T（tangent），B（Bitangent）与贴图U，V方向一致。 保存方式（T,B,N）


----------


Normal
![这里写图片描述](/images/opengl/normalmap3.png)


----------


T（tangent）
![这里写图片描述](/images/opengl/normalmap4.png)


----------


 B（Bitangent）  = N*T
![这里写图片描述](/images/opengl/normalmap5.png)


----------
**最终效果**

![这里写图片描述](/images/opengl/normalmap6.png)

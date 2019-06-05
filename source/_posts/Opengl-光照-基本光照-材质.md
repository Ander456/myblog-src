---
layout: post
categories: opengl
title: 'Opengl-光照-基本光照-材质(有了材质一个物体才算是完整了)'
date: 2018-08-23
---

> 在现实世界里，每个物体会对光产生不同的反应。比如说，钢看起来通常会比陶瓷花瓶更闪闪发光，木头箱子也不会像钢制箱子那样对光产生很强的反射。每个物体对镜面高光也有不同的反应。有些物体反射光的时候不会有太多的散射(Scatter)，因而产生一个较小的高光点，而有些物体则会散射很多，产生一个有着更大半径的高光点。如果我们想要在OpenGL中模拟多种类型的物体，我们必须为每个物体分别定义一个材质(Material)属性。

材质是什么呢？其实就是真真切切的物体本身，因为材质物体才会形形色色各式各样，有玉，有铁，有木，等等。我们掌握了材质也就能模拟各种各样的物体，而不局限于单一的color

```
#version 330 core
struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
}; 

uniform Material material;
```
前面说的  **环境光 + 漫反射光 + 镜面光 + 散射因子（来表现这个东西到底是什么反光的强度铁就比木头强**） 就构成了我们所谓的材质-Material。看到了吧？和我们刚才讲到基本冯氏光照 只差了一个变量，就是用来控制对光的敏感程度额shiness。

> 你可以看到，我们为每个冯氏光照模型的分量都定义一个颜色向量。ambient材质向量定义了在环境光照下这个物体反射得是什么颜色，通常这是和物体颜色相同的颜色。diffuse材质向量定义了在漫反射光照下物体的颜色。（和环境光照一样）漫反射颜色也要设置为我们需要的物体颜色。specular材质向量设置的是镜面光照对物体的颜色影响（或者甚至可能反射一个物体特定的镜面高光颜色）。最后，shininess影响镜面高光的散射/半径。

```
void main()
{    
    // 环境光
    vec3 ambient = lightColor * material.ambient;

    // 漫反射 
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = lightColor * (diff * material.diffuse);

    // 镜面光
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);  
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = lightColor * (spec * material.specular);  

    vec3 result = ambient + diffuse + specular;
    FragColor = vec4(result, 1.0);
}
```

简单明了，冯氏光照+材质 最后反应出了真是的效果。
![这里写图片描述](/images/opengl/light_material.png)
我们可以根据不同的shines来体现不同程度的高光，然后通过设置材质的向量属性我们可以把物体变成各种各样的

```
lightingShader.setVec3("material.ambient",  1.0f, 0.5f, 0.31f);
lightingShader.setVec3("material.diffuse",  1.0f, 0.5f, 0.31f);
lightingShader.setVec3("material.specular", 0.5f, 0.5f, 0.5f);
lightingShader.setFloat("material.shininess", 32.0f);
```

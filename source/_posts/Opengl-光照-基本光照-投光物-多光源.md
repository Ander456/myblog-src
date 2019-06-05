---
layout: post
categories: opengl
title: 'Opengl-光照-基本光照-投光物-多光源(现实世界的光可不只有太阳也并不只有一个)'
date: 2018-08-26
---

##前言
  相信大家看过各种发光的道具，手电筒？看到过吧？灯泡看到过吧？除了太阳生活中还有各种灯红酒绿的地方（说错了）等着你去看啊

##各种光源

* 平行光-太阳或者很远处的光都可以叫做平行光
![这里写图片描述](/images/opengl/light_source1.png)
平行光的光的方向是一个固定的值不变的

```
vec3 lightDir = normalize(-light.direction);
```


* 点光源-灯泡-各种灯
![这里写图片描述](/images/opengl/light_source2.png)
我们前面一直讨论的就是一个点光源

```
float distance    = length(light.position - FragPos);
float attenuation = 1.0 / (light.constant + light.linear * distance + 
                light.quadratic * (distance * distance));
[...]
ambient  *= attenuation; 
diffuse  *= attenuation;
specular *= attenuation;
```

* 聚光灯-手电
![这里写图片描述](/images/opengl/light_source3.png)

```
float theta = dot(lightDir, normalize(-light.direction));

if(theta > light.cutOff) 
{       
  // 执行光照计算
}
else  // 否则，使用环境光，让场景在聚光之外时不至于完全黑暗
  color = vec4(light.ambient * vec3(texture(material.diffuse, TexCoords)), 1.0);
```
我们设定了聚光的开阔范围后就是看聚光的开阔的范围和光的方向哪个更偏了，其实计算两个角度哪个更大，和计算cos值是一样的都可以得到对应的想要的结果。

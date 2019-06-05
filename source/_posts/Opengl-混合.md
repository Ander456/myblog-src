---
layout: post
categories: opengl
title: 'Opengl-混合(原来不理解的src dst blender总算理解了)'
date: 2018-08-29
---

> ##混合

**混合干什么**：混合(Blending)通常是实现物体透明度(Transparency)的一种技术。也就是说为了实现透明的感觉，因为生活中玻璃等等的材质都是透明的，那么怎么模拟就用混合

**混合规则**：
![这里写图片描述](/images/opengl/blend1.png)
GL_ZERO	因子等于0
GL_ONE	因子等于1
GL_SRC_COLOR	因子等于源颜色向量C¯source
GL_ONE_MINUS_SRC_COLOR	因子等于1−C¯source
GL_DST_COLOR	因子等于目标颜色向量C¯destination
GL_ONE_MINUS_DST_COLOR	因子等于1−C¯destination
GL_SRC_ALPHA	因子等于C¯source的alpha分量
GL_ONE_MINUS_SRC_ALPHA	因子等于1− C¯source的alpha分量
GL_DST_ALPHA	因子等于C¯destination的alpha分量
GL_ONE_MINUS_DST_ALPHA	因子等于1− C¯destination的alpha分量
GL_CONSTANT_COLOR	因子等于常数颜色向量C¯constant
GL_ONE_MINUS_CONSTANT_COLOR	因子等于1−C¯constant
GL_CONSTANT_ALPHA	因子等于C¯constant的alpha分量
GL_ONE_MINUS_CONSTANT_ALPHA	因子等于1− C¯constant的alpha分量

**怎么使用**：
glEnable(GL_BLEND);
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

![这里写图片描述](/images/opengl/blend2.png)


----------


**总结：混合就是根据我们传入的参数glBlendFunc来决定如何将两个颜色的透明度进行混合的，也就是如何取舍两个颜色**

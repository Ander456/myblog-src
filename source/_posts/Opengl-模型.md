---
layout: post
categories: opengl
title: 'Opengl-模型(告别箱子加载模型)'
date: 2018-08-27
---

###先放一个效果图
![这里写图片描述](/images/opengl/model1.png)


----------
###关于模型加载

> 一个非常流行的模型导入库是Assimp，它是Open Asset Import
> Library（开放的资产导入库）的缩写。Assimp能够导入很多种不同的模型文件格式（并也能够导出部分的格式），它会将所有的模型数据加载至Assimp的通用数据结构中。当Assimp加载完模型之后，我们就能够从Assimp的数据结构中提取我们所需的所有数据了。由于Assimp的数据结构保持不变，不论导入的是什么种类的文件格式，它都能够将我们从这些不同的文件格式中抽象出来，用同一种方式访问我们需要的数据。

###关于Assimp库加载出来的格式
![这里写图片描述](/images/opengl/model2.png)

* 和材质和网格(Mesh)一样，所有的场景/模型数据都包含在Scene对象中。Scene对象也包含了场景根节点的引用。
* 场景的Root node（根节点）可能包含子节点（和其它的节点一样），它会有一系列指向场景对象中mMeshes数组中储存的网格数据的索引。Scene下的* mMeshes数组储存了真正的Mesh对象，节点中的mMeshes数组保存的只是场景中网格数组的索引。
* 一个Mesh对象本身包含了渲染所需要的所有相关数据，像是顶点位置、法向量、纹理坐标、面(Face)和物体的材质。
* 一个网格包含了多个面。Face代表的是物体的渲染图元(Primitive)（三角形、方形、点）。一个面包含了组成图元的顶点的索引。由于顶点和索引是分开的，使用一个索引缓冲来渲染是非常简单的（见你好，三角形）。
* 最后，一个网格也包含了一个Material对象，它包含了一些函数能让我们获取物体的材质属性，比如说颜色和纹理贴图（比如漫反射和镜面光贴图）。

其实我们最需要关注的部分是什么？是Mesh。
想想一个网格最少需要什么数据。**一个网格应该至少需要一系列的顶点，每个顶点包含一个位置向量、一个法向量和一个纹理坐标向量。一个网格还应该包含用于索引绘制的索引以及纹理形式的材质数据（漫反射/镜面光贴图）**。
  现在暂停下，发散一下思维。让我们想一下看过的视频或者电视又或者什么东西，想象一下无数的网格构成了这个东西，尽量去脑补出这么一个东西来有助于我们理解。
  再简单的了解下，Assimp帮我们加载模型出来一个固定格式的数据，然而对于我们渲染有用的数据是什么？我们学了什么？
  1. 材质 okay 为了对光做出反应 然后呢 还有物体自身的属性呢？
  2. 顶点数据
  3. 法向量
  4. 纹理坐标
  上面的2-4不就是网格提供的么？真是太贴心了对吧。
  


----------
模型这一章节都用的是一些现成的库然后加载出来渲染

下面这是Mesh的类里面的关键代码
```
void Draw(Shader shader)
    {
        // bind appropriate textures
        unsigned int diffuseNr  = 1;
        unsigned int specularNr = 1;
        unsigned int normalNr   = 1;
        unsigned int heightNr   = 1;
        for(unsigned int i = 0; i < textures.size(); i++)
        {
            glActiveTexture(GL_TEXTURE0 + i); // active proper texture unit before binding
            // retrieve texture number (the N in diffuse_textureN)
            string number;
            string name = textures[i].type;
            if(name == "texture_diffuse")
                number = std::to_string(diffuseNr++);
            else if(name == "texture_specular")
                number = std::to_string(specularNr++); // transfer unsigned int to stream
            else if(name == "texture_normal")
                number = std::to_string(normalNr++); // transfer unsigned int to stream
            else if(name == "texture_height")
                number = std::to_string(heightNr++); // transfer unsigned int to stream
            
            // now set the sampler to the correct texture unit
            glUniform1i(glGetUniformLocation(shader.ID, (name + number).c_str()), i);
            // and finally bind the texture
            glBindTexture(GL_TEXTURE_2D, textures[i].id);
        }
        
        // draw mesh
        glBindVertexArray(VAO);
        glDrawElements(GL_TRIANGLES, indices.size(), GL_UNSIGNED_INT, 0);
        glBindVertexArray(0);
        
        // always good practice to set everything back to defaults once configured.
        glActiveTexture(GL_TEXTURE0);
    }
```
这个章节主要理解Mesh是什么就好了，在脑袋里想象网格。

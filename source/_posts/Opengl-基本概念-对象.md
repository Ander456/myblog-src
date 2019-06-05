---
layout: post
categories: opengl
title: 'Opengl-基本概念-对象（很关键啊兄弟这章）'
date: 2018-08-17
---

> ##Opengl对象

   这里再次强调下，**Opengl就是一个状态集合**，它负责管理它下属的所有状态，然后这些个状态是怎么表现的，你想象一下啊老哥 玩过游戏吧？举个列子，你用冰冻法术打人别人别冰了这是不是状态？可能不是很恰当，但是你要明白，opengl就是一个状态集合，不同的状态表现不同的特性。然后这些都是需要一个一个的状态的，**这些也就是我们的Opengl对象提供的了**。当然了说的稍微难理解一点就是**，对象是什么？还是一块内存数据，只不过用人很好理解的方式解释了下罢了**。
   接着说，Opengl是个大的状态机有一个大的Context，然后你把任意对象（想象成技能）挂到Context的时候，这个时候Opengl就会变成不同的状态来应对啦。当然了我们改变了这个Context，也会变成其他的状态（这是废话）文字可能很难理解，我找了一张图
   ![这里写图片描述](/images/opengl/context1.png)
   这次看看，Context是我们的Opengl状态机，然后下面的Object都是一些Opengl的对象，箭头表示绑定到状态机上，然后这个Context才会因为这些具备不同的状态和特性。明白了吧？还不明白？那再看看 - -
   


----------

> VBO

  vertex buffer object 看名字也知道 也是一个Opengl的对象， 而这个也就是我们最关键的东西了，这个千万记住了，**顶点缓冲对象**，干嘛的？就是用来存储我们前面说过的顶点数据的。现在可以发散一下你的脑洞，想一哈，你所见到的或者你所理解的图形在计算机上咋显示的？还不是一个一个的点或者片段么？**然而怎么组织这些点就是在这里说了算了**
  unsigned VBO;
  glGenBuffers(1, &VBO);
看上面的代码就是生成一块VBO的方式
  
> VAO

 vertex array object 看名字也知道 也是一个Opengl的对象，它是用来干嘛的？不废话，它就是用来存储上面说的VBO的，你想啊。你见到的一半都是复杂的图形或者图像。几个顶点哪够啊都是大把的，哥哥顶点单独的数据都存在VBO里了，要想把这些点结合起来肯定用个array容器也就是我们的VAO
unsigned VAO;
glGenVertexArrays(1, &VAO);
看上面的代码就是生成一块VAO的方式
 
> EBO

  索引缓冲对象, 上面都有两个对象了怎么又来了一个？其实就是一种优化，原理上讲会让大家迷惑但是稍微说一句，一个正方形，是有两个三角形组成的，那么就需要6个顶点，说到这肯定有人跳出来打我。妈的明明四个点就行干嘛用6个？这不是占地方浪费么？说的是说的是，所以也就有了这个东西EBO，用来存放索引
  同理EBO这样生成
  unsigned EBO;
  glGenBuffers(1, &EBO);


```
float vertices[] = {
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
};

unsigned int indices[] = { // 注意索引从0开始! 
    0, 1, 3, // 第一个三角形
    1, 2, 3  // 第二个三角形
};
```
看出来了吧？

其实说这么多还不如一张图，让大家更能醍醐灌顶。
![这里写图片描述](/images/opengl/context2.png)

能看出来了吧？ 这是what？ VBO用来存放一个点一个点，VAO用来存放一组点 EBO做了层优化可以用来存放索引省去一些点 然后最有用的VAO你可以看到了吧，这么理解吧，VAO就是你的情人，记住了后面一直会用到，就像你做了很多个木头模型，都放到那了，想给别人看啥的时候就拿对应的VAO就完事了。简单吧？


----------
下面是我写的部分生成绑定的代码，注释已经写的相对明白了，大家千万要记住一点就是，**bind上了谁接下来状态机就使用的是谁了**。
```
   unsigned VBO, VAO, EBO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    glGenBuffers(1, &EBO);
    // 在gl里 bind 谁那么接下来用的就是谁 一个大状态机里面不同对象记录不同状态
    // bind vao 接下来用的就是上面Gen生成的Vao
    glBindVertexArray(VAO);
    // bind vbo 接下来用的vbo就是上面Gen生成的
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    // 往vbo（一块显存）里存放数据
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
    // 同vbo设置数据这里设置数据到ebo里
//    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
//    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
    
    // 配置vao 因为vao就是一个记录状态用的 状态集合 这里配置好告诉opengl怎么使用vbo的数据 也可以理解为 给对应的顶点属性数组指定数据：
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5* sizeof(float), (GLvoid*)0);
    glEnableVertexAttribArray(0);
    
//    // 颜色属性 (配置顶点的属性 不同属性位置的不同配置)
//    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void*)(3* sizeof(float)));
//    glEnableVertexAttribArray(1);
    
    // 新增一个纹理坐标的顶点属性解释
    glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void*)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);
```

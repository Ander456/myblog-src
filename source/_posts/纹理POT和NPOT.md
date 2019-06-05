---
layout: post
categories: work
title: 'Opengl中纹理的大小POT（2的幂）和NPOT（非2的幂）'
date: 2018-12-25
---

## 前言
  在一个技术讨论群里看到了一个老哥在说关于游戏在真机上内存的问题，说几张图片占用多大内存啊有问题啊之类的，我自己做了一些探索记录下。

### 关于图片在运行的时占用内存大小
运行时大小 = 长x宽x每个像素占的大小
举例：rgba8888 表示的是通道rgba每个通道都占用8bit那么也就是一个像素占用了4bytes
故，图片大小若为1014x1024，则大小=1024x1024x4/1024/1024 = 4M
同理rgba4444的也就能算出来了

### 关于POT和NPOT
POT: 2的幂
NPOT: 非2的幂
很多人都在说图片要求2的幂大小啊，纹理要求2的幂大小之类的，我搜索了一番，发现在opengl或者说现代设备中基本上都已经支持了NPOT的形式，也就是说不必要要求纹理宽高非要是2的幂了。
解释：
 Q:为什么有这种2的幂的需求？
 A:因为一些老的设备已经opengl早期为了性能考虑一些算法是要求纹理的宽高是要求为2的幂的比如64*64这种，同时引申出一个概念叫mipmaps就是多级纹理，也就是将一张图不断缩小，并且把这一些列图都存下来，然后根据视角的远近图片的大小显示不同的图片来达到真实效果。然后这一策略使用的也是不断的把宽高/2来实现的，所以如果是非2的幂那么可能会有一些小问题，但是影响不大

 > While modern hardware no longer has the power-of-two limitation on texture dimensions, it is generally a good idea to keep using power-of-two textures unless you specifically need NPOTs. Mip-mapping with such textures can have slightly unintended consequences, compared to the power-of-two case.
 
 同时提供一个[opengl texture wiki](https://www.khronos.org/opengl/wiki/Texture) 
 Q:有了mipmaps会增加内存大小么？
 A:会的，根据提供的数据大概是33%左右

 ### cocos中纹理的使用
```
cocos2dx 3.10版本中在 CCTexture2D.cpp文件中可以看到所有的texture初始化都会进入initWithMipmaps这个方法
bool Texture2D::initWithMipmaps(MipmapInfo* mipmaps, int mipmapsNum, PixelFormat pixelFormat, int pixelsWide, int pixelsHigh)
{
    [...]
    glGenTextures(1, &_name);
    GL::bindTexture2D(_name);

    if (mipmapsNum == 1)
    {
        glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, _antialiasEnabled ? GL_LINEAR : GL_NEAREST);
    }else
    {
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, _antialiasEnabled ? GL_LINEAR_MIPMAP_NEAREST : GL_NEAREST_MIPMAP_NEAREST);
    }
    
    glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, _antialiasEnabled ? GL_LINEAR : GL_NEAREST );
    glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE );
    glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE );

    // Specify OpenGL texture image
    int width = pixelsWide;
    int height = pixelsHigh;
    
    for (int i = 0; i < mipmapsNum; ++i)
    {
        unsigned char *data = mipmaps[i].address;
        GLsizei datalen = mipmaps[i].len;

        if (info.compressed)
        {
            glCompressedTexImage2D(GL_TEXTURE_2D, i, info.internalFormat, (GLsizei)width, (GLsizei)height, 0, datalen, data);
        }
        else
        {
            glTexImage2D(GL_TEXTURE_2D, i, info.internalFormat, (GLsizei)width, (GLsizei)height, 0, info.format, info.type, data);
        }

        if (i > 0 && (width != height || ccNextPOT(width) != width ))
        {
            CCLOG("cocos2d: Texture2D. WARNING. Mipmap level %u is not squared. Texture won't render correctly. width=%d != height=%d", i, width, height);
        }

        err = glGetError();
        if (err != GL_NO_ERROR)
        {
            CCLOG("cocos2d: Texture2D: Error uploading compressed texture level: %u . glError: 0x%04X", i, err);
            return false;
        }

        width = MAX(width >> 1, 1);
        height = MAX(height >> 1, 1);
    }

    [...]
}
```
其中省略了一些代码，但是总体思路可以看到 cocos中3.10版本，并未对什么POT或者NPOT做处理，而是直接在 **glTexImage2D** 中传入了width和height而这也是任意的（arbitry）。这就是因为opengl后来支持了NPOT，我们可以不用关心这个2的幂的问题了。
Q: 使用NPOT效率比POT效率差？
A: 是的，理论上是会差点，但是相比效率的差，我们从美术角度或者程序角度再也不用去把一个like 63x45的美术图（本来美术这么设计的）改成64x48这样，因为这样在程序里纹理的内存就变大了（见我们上面的公式）同时，上面也说过mipmaps的问题，基本上2d游戏使用mipmaps的情形很少，也就忽略那个微乎其微的影响，所以我们使用NPOT来说更好
这里看个cocos对于1.x版本的时候处理NPOT的函数就知道了
```
if (Configuration::getInstance()->supportsNPOT())
{
    powW = w;
    powH = h;
}
else
{
    powW = ccNextPOT(w);
    powH = ccNextPOT(h);
}

int ccNextPOT(int x)
{
    x = x - 1;
    x = x | (x >> 1);
    x = x | (x >> 2);
    x = x | (x >> 4);
    x = x | (x >> 8);
    x = x | (x >>16);
    return x + 1;
}
```

### 几个讨论链接：
[cocos](https://forum.cocos.com/t/npot-pot-2-n/47460)
[cocochia](http://www.cocoachina.com/bbs/read.php?tid=198792)
[google](https://www.opengl.org/discussion_boards/showthread.php/167665-POT-NPOT-textures)



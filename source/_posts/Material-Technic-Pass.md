---
layout: post
categories: shader
title: 'Shader | Material Technic Pass'
date: 2018-06-06
---

### 前言
  最近了解了下Threejs相关的技术内容，同时今天也看到了一个公众号推送的关于CocosCreator材质系统相关的文章，带着一些疑惑以及原来Opengl知识的总结打算写一篇文章来说一下关于当下比较流行的渲染引擎他们Shader的一般流程或者说做法

### 大纲
  我们首先先确认一件事一个材质文件基本上应该包含哪些部分，数据结构如下：
  ```
  MaterialTemplate
    {
         Technique
         {
              Pass{}
              Pass{}
         }
         Technique
         {
              Pass{}
              Pass{}
         }
    }
  ```
  我们带着这个疑惑往下看，来了解到底什么Technique和Pass

### Shader
  说材质就不得不提一下Shader，有关于shader的Opengl文章我原来写过或者说记录过一些，这里做个简单的解释：
  Shader(着色器)
  负责将输入的Mesh（网格）以指定的方式和输入的贴图或者颜色等组合作用，然后输出。怎么说呢，Mesh也就是Geometry+Material组成的东西，也就是一个几何体+材质就形成了网格，然后Shader来把这个网格根据一些参数作用输出到屏幕上（也就是我们说的可编程渲染管线）

### 材质
  通俗的理解，材质就是物体的表现，不同的材质富于物体不同的表现，比如金属材质，木头材质，等等。原理上说材质就是我们上面提到的shader，我们把vertex以及fragment的shader编辑出不同的效果来模拟不同的材质，当然了我们还可以加入图片来作为纹理贴图来模拟更多样化的材质

### TRHEEJS中的材质
  我们先来看一段很简单的再Threejs中使用材质的例子：
  ```
    let scene = new THREE.Scene();
    let camera = new THREE.PerspectiveCamera(75, width / height, 1, 10000);
    camera.position.set(0, 0, 1000);
    let ambient_light = new THREE.AmbientLight(0xffe1dd, 1);
    let point_light = new THREE.PointLight(0xffffff, 1, 0);
    point_light.position.z = 500;
    scene.add(ambient_light);
    scene.add(point_light);
    let renderer3D = new THREE.WebGLRenderer();
    renderer3D.setSize(width, height);
    let geometry = new THREE.BoxGeometry(500, 500, 500);
    let material = new THREE.MeshNormalMaterial();
    let cube = new THREE.Mesh(geometry, material);
    cube.position.z = -500;
    scene.add(cube);
  ```
  
  ** let material = new THREE.MeshNormalMaterial(); **
  这行代码的作用就是创建一个普通的材质
  这里的代码片段也证实了我们上面说的MESH是什么，Mesh也就是由Geometry+Material组成的。然后我们把cube添加到了scene上输出结果如下
  ![threejs_demo](/images/material/threejs_demo.png)
  图中的Cube就是我们使用了材质之后的效果，是不是很帅
  当然了，前面有一个2d图片，这个是我使用PIXIJS渲染上的，这个等以后再说

### ThreeJS自定义Material
```
scene = new THREE.Scene();
uniforms = {
    fogDensity: {type: "f", value: 0.45},//雾的密度
    fogColor: {type: "v3", value: new THREE.Vector3(0, 0, 0)},//雾的颜色
    time: {type: "f", value: 1.0},
    resolution: {type: "v2", value: new THREE.Vector2()},//分辨
    uvScale: {type: "v2", value: new THREE.Vector2(1.0, 1.0)},//uv 缩放
    texture1: {type: "t", value: THREE.ImageUtils.loadTexture("lava/cloud.png")},//云
    texture2: {type: "t", value: THREE.ImageUtils.loadTexture("lava/lavatile.jpg")}//熔浆
};
uniforms.texture1.value.wrapS = uniforms.texture1.value.wrapT = THREE.RepeatWrapping;
uniforms.texture2.value.wrapS = uniforms.texture2.value.wrapT = THREE.RepeatWrapping;
var size = 0.65;
material = new THREE.ShaderMaterial({
    uniforms: uniforms,
    vertexShader: document.getElementById('vertexShader').textContent,
    fragmentShader: document.getElementById('fragmentShader').textContent

});
mesh = new THREE.Mesh(new THREE.CylinderGeometry(0.51, 1, 2), material);
```

可以看出自定义材质就是我们自己编写shader而已，给定不同的uniform然后实现不同的表现，也就验证了我上面说的。具体shader内容就不看了有兴趣的可以去看这个link [Threejs自定义岩浆材质](https://blog.csdn.net/qq_25909453/article/details/85046416)

### CocosCreator中的材质
  ```
    // Shader Material
    let renderer = renderEngine.renderer;
    let gfx = renderEngine.gfx;

    // 创建pass
    let pass = new renderer.Pass(name);
    pass.setDepth(false, false);
    pass.setCullMode(gfx.CULL_NONE);
    pass.setBlend(
        gfx.BLEND_FUNC_ADD,
        gfx.BLEND_SRC_ALPHA, gfx.BLEND_ONE_MINUS_SRC_ALPHA,
        gfx.BLEND_FUNC_ADD,
        gfx.BLEND_SRC_ALPHA, gfx.BLEND_ONE_MINUS_SRC_ALPHA
    );

    // 创建Technic
    let mainTech = new renderer.Technique(
        ['transparent'],
        [
            { name: 'texture', type: renderer.PARAM_TEXTURE_2D },
            { name: 'color', type: renderer.PARAM_COLOR4 },
            { name: 'pos', type: renderer.PARAM_FLOAT3 },
            { name: 'size', type: renderer.PARAM_FLOAT2 },
            { name: 'time', type: renderer.PARAM_FLOAT },
            { name: 'num', type: renderer.PARAM_FLOAT }
        ],
        [pass] //数组证明可以有多个
    );

    this._texture = null;
    this._color = { r: 1.0, g: 1.0, b: 1.0, a: 1.0 };
    this._pos = { x: 0.0, y: 0.0, z: 0.0 };
    this._size = { x: 0.0, y: 0.0 };
    this._time = 0.0;
    this._num = 0.0;
    this._effect = this.effect = new renderer.Effect([mainTech], { // 数组证明可以有多个
        'color': this._color,
        'pos': this._pos,
        'size': this._size,
        'time': this._time,
        'num': this._num
    }, []);
    this._mainTech = mainTech;
  ```

因为creator的材质系统刚出，并没有实例等，所以这里直接找了相关的自定义材质部分。我们可以看出实现一个材质的细节了。更是加强了我们上面已经得出的结论，实现一个材质就是编写一个shader来实现一个效果而已。实现shader需要什么？我们Opengl已经明白了。简单点就是拿到顶点数据然后转换坐标系然后片段着色器对像素做处理。
实现特效effect我们可能需要用到很多不同uniform。而上面的mainTech的第二个参数就是uniform了，然后配合不同的shader的vertex和fragment代码来实现效果。
我们来感受一下效果：
![ccc_demo](/images/material/ccc_demo.png)
![ccc_demo2](/images/material/ccc_demo2.png)

### Pass
  顾名思义通道，通过。这个是干嘛呢？我们通过上面Creator自定义Material已经看出来了，他的大体职责就是负责我们opengl渲染管线里最后一个阶段 **测试与混合** 测试，我们知道有，深度测试，模板测试等。混合就是目标与目标blender混合颜色产生不同的效果（比如玻璃）
  那我们就明白了pass是干嘛了，它就是用来负责这个的，是否开通过度测试啊，是否通过模板测试啊等等的一个控制器。
  而为什么会有多个Pass：
  有时候，我们为了实现一个绘制效果，靠单次绘制是无法实现的。比如描边效果。这就要求我们单个物体能够在进行绘制的时候，多次提交材质并绘制。就像我们Opengl学习箱子描边那个章节，或者说我们玩游戏选中物体出现高亮描边的效果都是通过这样的方式来实现的
  ---
  这里再说下TRHEEJS里的Pass，threejs的pass用于在EffectComposer里，也就是特效组合器，post-processing后期处理。
  
  ```
  THREE.SepiaShader = {
    uniforms: {
        "tDiffuse": { value: null },
        "amount":   { value: 1.0 }
    },
    vertexShader: [
        "varying vec2 vUv;",
        "void main() {",
            "vUv = uv;",
            "gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );",
        "}"
    ].join( "\n" ),
    fragmentShader: [
        "uniform float amount;",
        "uniform sampler2D tDiffuse;",
        "varying vec2 vUv;",
        "void main() {",
            "vec4 color = texture2D( tDiffuse, vUv );",
            "vec3 c = color.rgb;",
            "color.r = dot( c, vec3( 1.0 - 0.607 * amount, 0.769 * amount, 0.189 * amount ) );",
            "color.g = dot( c, vec3( 0.349 * amount, 1.0 - 0.314 * amount, 0.168 * amount ) );",
            "color.b = dot( c, vec3( 0.272 * amount, 0.534 * amount, 1.0 - 0.869 * amount ) );",
            "gl_FragColor = vec4( min( vec3( 1.0 ), color.rgb ), color.a );",
        "}"
    ].join( "\n" )
};
  ```

  ```
    var composer = new THREE.EffectComposer(renderer);
    //NOTE: this goes in your render loop
    composer.render();
    var renderPass = new THREE.RenderPass(scene, camera);
    composer.addPass(renderPass);
    var shaderPass = new THREE.ShaderPass(THREE.SepiaShader);
    composer.addPass(shaderPass);
  ```

看得出来，就是一个pipeline，上一个pass完了然后把结果给下一个接着走下一个pass通过不同的pass来组成不同的特效
这样看来，两边对于Pass的定义是不一样的。
** 标准或者说Opengl来说Pass用于控制测试以及混合通过与否的设置 **
** Threejs的Pass是只后期处理特效的处理通道，用来pipeline我们上一步的shade结果然后再处理 **
PS: 找到一个threejs实现描边的Pass[](https://threejs.org/examples/webgl_postprocessing_outline.html)

### Technique
   这个是干嘛的呢？为啥一个材质需要这个东西。。。其实我们看了上面Creator写材质的例子也能看出一二了。这个就是我们的shader参数。为啥叫Techinique这我就不知道了。。可能这些参数就是很牛逼的技术吧（开个玩笑）
   至于为什么有多个Technique这里有个解释：
   ** 多Technique是指一个材质中，应该包含不只一个实现方案。 这样当我们进行材质更替，或者进行高中低端机适配的时候。 就不会那么麻烦。 同时在数据管理上，也显得更为规范 **
   也就是同一种材质显示效果我们可能有多种技术方案实现。区别质量和性能来做多个Techinique，不过这一点我在THREEJS里并没有看到，也没有搜到相关的Technique概念。可能Threejs那边更简单一点，不再区分不同的Technique而是直接区分成不同的Material材质吧。换个效果就换个材质而已。


### 总结
Threejs是基于webl封装的一个framework，CococCreator的render是基于opengl的，所以原理上是大相径庭的。
Material = shader 或者 狭义的时候后还会 = shader + pass(深度模板测试等的设置)
Mesh = geometry + material
Technique: 在ccc的渲染框架里表示的shader参数，shader数值以及实现方案，多Technique就是为了在不同硬件上采用不同质量的参数算法实现低性能或者高性能的效果的
pass：在ccc里表示深度模板测试等的设置 在Threejs里是后期处理的通道pipeline


### 参考链接：
[create_engine](https://github.com/cocos-creator/engine)
[threejs_framework](https://threejs.org/docs/index.html#manual/introduction/Creating-a-scene)
[thinking_in_unity](https://www.cnblogs.com/geniusalex/p/5321527.html)
[ShaderHelperCreator](https://mp.weixin.qq.com/s?__biz=MzA5MjEwOTI4Ng==&mid=2247484299&idx=1&sn=22d542c42c1895b5677898c049bb07cd&chksm=90736302a704ea14e3c1f2ca5b003cfd0db5a6b1b0ca0a1b2c7f87899d1e629d7b4c2731b7da&scene=21#wechat_redirect)




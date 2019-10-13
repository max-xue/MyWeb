---
title: 使用UE4进行VR制作的一些经验分析
categories:
  - UE4
url: 745.html
id: 745
comments: true
date: 2016-08-16 12:42:10
tags:
---

UE4的演示资源还是很不错，个个精良，我这边把UE4的有代表性的演示都跑了一遍，同时也通过Rift确认效果，和里面的资源制作方式。

首先，UE4是基于物理渲染的引擎，大部分都是偏向图像真实的。使用的材质和贴图细节也更多一些。在PC上的品质要比unity好，性能应该也要比Unity费一些，所以很适合作VR产品的质量和效率标。

这里先列出自己的总结，然后再结合每个演示做一些具体分析。这些只是针对PC的高品质制作：

1）渲染风格方面，VR也可以作出基于物理渲染的效果，只要你能贴图和光照要遵守物理法则，也可以偏卡通一些的。

2）VR也可以使用PBR的金属度/镜面颜色，法线贴图，粗糙值等，但在Unity5上，你需要在金属度/镜面颜色的工作流中选择一种。UE4为了效率考虑，最好也能选择其中一种，而不是两个同时使用

3）粒子如果是要和视觉交互的，就不能用基于BillboARd生成，而是要做成基于Mesh的粒子

4）Normal Map在VR中还是可以看出凹凸感的，并不像文档中介绍的那样没有作用，但是PC上使用POM（视差映射贴图）和Tessellation（表面细分）技术可以作出比Normal Map更高的细节。

5）PBR的Specular Aliasing的问题，会被VR更加放大，需要使用可以支持Specular Anti-Aliasing的技术才可以（比如Temporal AA）。

6）VR的分辨率比较小，为了减少锯齿感，尽量避免一些使用alpha test制作的效果。

7）UI和互动的制作也很重要，这部分Unity和UE4上缺少这方面的实例。

8）调试方面，UE4自带的命令行工具对性能分析很有帮助，这个Unity自己没有那么全面的分析工具，需要自己实现和改造。

9）性能问题的优化问题，最好能保证75FPS

接下来对这些问题逐一阐述。

#### 1）渲染风格 基于物理（PBR）或者卡通风格

PBR我翻译和分享的资料里应该有很多了，这里只做一个简单介绍，PBR大概可以分为材质着色（Material Shading），光照（Lighting），后处理特效（Camera Effect）3部分，UE4和U3D都有PBR材质着色支持，但光照和后处理部分性能和效果对VR实现压力还是过大了。现阶段的PBR，主要还是靠基于微平面的BRDF，以及IBL，通过粗糙度来增加效果。动态光源和物理镜头因为VR性能的缘故尽量少用。

    Order 1886的图释，只有材质是PBR的

    ![](http://www.52vr.com/data/attachment/portal/201609/18/191458wk31gqoov7469666.jpg)

    最近展示的VR项目里，真正采用PBR的并不多，大多是卡通，或者是着色模型简单的，如果在品质要出彩的话，确实是一个突破点。

    PBR风格

    ![](http://www.52vr.com/data/attachment/portal/201609/18/191458hwewpz684tylbbnb.jpg)

    卡通风格的场景

    ![](http://www.52vr.com/data/attachment/portal/201609/18/191458q73l0v37x67xr9rb.jpg)

    ![](http://www.52vr.com/data/attachment/portal/201609/18/191458um3ouzjcdbjjuvb2.jpg)

#### 2）VR的美术制作管线

    美术管线方面，VR和普通并没有太多特别的，卡通的沿用卡通的就可以，PBR的，美术制作时有一定的PBR管线意识，除了需要绘制粗糙度的贴图外，还需要对金属和非金属的颜色有一定的了解，而从节省资源和效率考虑，在Metalic或Specular的制作流程中选择一个就可以了

    如下图所示，Metal的美术管线和Specular相比，贴图占用更小，对美术师制作起来也更直观和简单，颜色都绘制在base color上，然后只需要用Metallic的贴图标识出那些是金属和非金属，着色模型就会自动的来确定Diffuse和Specular的颜色。

    而Specular的流水线，需要美术师需要有意识的把金属的颜色绘制到Specular上。通过Diffuse和Specular的颜色，来区分金属和非金属。

    ![](http://www.52vr.com/data/attachment/portal/201609/18/191458a3vynrn176eq57ic.jpg)

#### 3）凹凸细节的制作

UE4的官网提到，因为要左右眼分别渲染的缘故，对视觉依赖的贴图效果，如Normal Map在VR上会被“平坦化”，但实际从UE4的一些演示上看，Normal Map的效果还是有的，只是没有普通PC那么明显，同时，Directx11的POM（Parallax Occlusion Mapping）和 Tessellation，可以起到比Normal Map更好的效果。这些在Crytek的一些游戏和演示中都很常见。

 下图是UE4的VR演示，ShowDown中的机器人Boss，从下面的材质看，也使用到了Normal

 ![](http://www.52vr.com/data/attachment/portal/201609/18/191458ba56b9uvrijabizj.jpg)

    ![](http://www.52vr.com/data/attachment/portal/201609/18/191458n4h4z5jw75705433.png)

下面是Crytek的POM和TS的效果对比，在几何体细节的增加上很明显。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458uns1n1kn0c8n2115.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458m74l8tf7cpr99sfp.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458kvpf0shhpfl6fh6l.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458yljknphkn759dpwj.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458clm8vnvula5248nu.jpg)

POM一般可以用来做墙面和地板等平面的凹凸，而Tessellation是则是可以利用Directx11的TS制作各种模型的表面细分。POM制作起来相对简单，类似Bitmap2Material的工具也能作出不错的凹凸效果。

#### 4）粒子特效

传统的Billboard粒子都是一个或大量的板型多边形朝向玩家视点的方向来模拟出效果，而VR的话，一个是双眼导致Billboard出现偏差，另外就是当玩家处在粒子附近晃动头盔浏览时，Billboard就会被看出破绽

Showdown中的导弹火焰的轨迹和爆破碎片，因为轨迹和破片是穿插在玩家视点的前进路线上，玩家可以在正面或者低头和回头观看，所以都是用网格来制作的。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458o4mitis8z3wfmov8.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458qpd2hdn0m1k11h0h.jpg)

5）Specular Aliasing的问题
----------------------

Specualar Aliasing是PBR的Specular的高频部分在较远处因为采样问题导致，表现就是物体边缘的高光会跟根据视角移动“晃动”现象。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458b00b91ayj79nvv99.png)

左图是关闭了Specular AA的效果，右边是打开的，Specular Aliasing在左图运动时会更加明显。所以需要SMAA或Temproal AA这类对支持（ subpixel reconstruction AA）的后处理。不过这样就增加了对GPU的消耗。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458te1u7jjzefe1jzj5.jpg)![](http://www.52vr.com/data/attachment/portal/201609/18/191458dw2gwwhk9y9tlzwe.jpg)

#### 6）Alpha Test AA

BlackSmtih这Demo里抖动相当厉害。其中原因，一个是Unity默认的FXAA只有对几何体的锯齿有效果，对应像素的走样问题并不能很好的对应。下图中远处的场景抖动就非常厉害。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458kyh418e2n0ie14xn.jpg)

一般渲染植物都是使用贴图，然后用alpha test来作出植物的边缘，由于分辨率比较小，这种锯齿感在VR里也会被放大。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458qqfqkrzs5om7ir5w.jpg)

7）UI和互动的制作
----------

UI方面，U3D和UE4自带的3DUI在表现力上和Scaleform比还是有很大差距的。如果要制作出科技感强的界面，还是要手动定制一些功能。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458f08k3a5g35385kov.jpg)

8）调试和性能分析
---------

unity的分析主要是靠Profile，调试时需要拿下头戴设备，在Editor调整后，再用设备来确认。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458ea0ikksoiqqjoki8.jpg)

UE4除了有分析工具外，还可以直接在游戏中使用命令行工具显示各种性能参数，以及各种显示特性的参数设置，更方便判断瓶颈。Unity还是需要自己在编辑器里设置，或者修改和扩展自己的设置插件。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458e44jc2sa41ta4344.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458qicrh291cnkn5t4t.jpg)

8）性能问题
------

为了不产生眩晕感和流畅，UE4建议在Rift DK2可以维持在75FPS，那么最低硬件配置也是GTX960。在我的GTX660上，UE4的VR专用演示大概是45FPS左右，其他的大概30FPS左右。这个一个和UE4本身的优化不足有关，另外也是UE4演示里还是使用了高品质材质和特效的缘故。UE4的性能和制作规格对Unity开发还是可以起到参考作用。

附录：
---

最后是UE4自带的两个VR专用的演示，Showdown和CouchKnights的一些特性

![](http://www.52vr.com/data/attachment/portal/201609/18/191458h0qqsqhm2qioo0i2.jpg)

场景的平均Draw Call为600

![](http://www.52vr.com/data/attachment/portal/201609/18/191458xro14rhonosynzao.jpg)

静态物体对象的多边形数50w~70w，骨骼动画的多边形数全屏为30w~50w左右

![](http://www.52vr.com/data/attachment/portal/201609/18/191458u64ueofcdab6wc6z.jpg)

如果是VR的话，UE4的Draw Call都要翻倍，绘制的多边形数量自然也翻倍了。实现方式上，这点和Unity是不一样的。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458cn611ky4e66kee96.jpg)

#### 静态场景

大部分的静态场景，近处的房屋和地面等等都是标准PBR材质，有Roughness和Normal Map，远景的房屋没有使用Normal Map，地面使用了bump offset来模拟POM的效果。

配合Local Cubemap（UE4里叫ReflectionCapture）做出来地面上水洼的反射效果。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458fbf68ks6a7th7zbu.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458lcs4xux424lyik0f.png)

### 动态物体

除了玩家外，场景里只有士兵和敌方Boss两种角色。面数为6w和10w面。一个角色都有7，8种材质。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458eg42i4ttafzlkvaz.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458wtic7yq0xtajvvv7.jpg)

### 光影

场景没有使用任何动态光源，都是预先烘培的光照贴图（Lightmap）和Light Probe（UE4里叫Lightmass）

![](http://www.52vr.com/data/attachment/portal/201609/18/191458k9p5dgzavib28vbj.jpg)

角色动态阴影只是一个脚底下的贴片来Fake的。

![](http://www.52vr.com/data/attachment/portal/201609/18/191458zagakawc11w2f1k1.jpg)

![](http://www.52vr.com/data/attachment/portal/201609/18/191458tn7uleaaiphha22h.jpg)

#### 粒子特效

车的爆炸特效，是在一个做了高细分（ tesselated ）的球体上加入了世界空间的噪声来实现

![](http://www.52vr.com/data/attachment/portal/201609/18/191458b4j2uf00jjyjj3fj.jpg)

导弹的轨迹也细分几何体。沿着一个样条曲线来运动

![](http://www.52vr.com/data/attachment/portal/201609/18/191458g1kofamsmezrpmro.jpg)

转自：[使用UE4进行VR制作的一些经验分析](http://www.52vr.com/article-345-1.html)
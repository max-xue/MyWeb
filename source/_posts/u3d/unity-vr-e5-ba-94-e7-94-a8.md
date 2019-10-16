---
title: Unity5中优化VR 应用的12个技巧
categories:
  - U3D
url: 752.html
id: 752
comments: true
date: 2017-03-03 09:59:01
tags:
  - u3d
  - vr
---

VR应用比非VR应用需要更强的计算，性能优化是一个很重要的任务。若目标平台是像GearVR这样的手机设备，优化就更重要了。 以下是一些应该试着了解的性能指标：

*   每只眼睛50次绘制调用。更精确地将其称为SetPass Calls。
*   场景中顶点数少于50K～100K 且面数少于50～100K 。

下面是一些简单的技巧，用于满足上述要求： **静态批处理** 场景中可能存在大量的静态几何体，例如墙体，椅子，灯光和从不移动的网格。在编辑器中将它们标记为静态对象。为烘焙光照贴图，请确保将其标记为静态贴图。不要让每个对象都会导致一次绘制调用，而是把对象标记为可被组合成一个网格的静态对象。 静态批处理有个关键要求：所有对象必须使用相同的材质。若静态墙带有木头材质，静态椅子带有铁材质，所有墙会被批量处理为一次绘制调用，椅子作为单独网格进行另外的绘制调用。 **　纹理集** 如之前所说，每个材质引发一次绘制调用。直觉可能是木门和铁椅子需要使用不同的材质，由于它们的纹理不同。然而，若使用相同的着色器，就可以用纹理集为它们创建共用的材质。纹理集就是一个包含所有小纹理的大纹理。我们可以使用一个材质加载一个纹理，而非使用多个材质加载多次。每个对象可以对应到纹理集中不同坐标的一个纹理。 你可以的绘制管线中手动生成纹理集，但是Juan Sebastian的Pro Draw Call Optimizer工具非常有用。它可以生成纹理集，并且在替换新对象时不会搞混资源。 **动态批处理** 非静态对象可以动态批处理为一个单独的绘制调用。我曾注意到该过程大量占用CPU且每帧都在计算，但这是一个很好的优化。这只对使用相同材质且顶点数少于900的对象有效。使用纹理集为所有的动态对象创建一个材质，就可以进行简单的动态批处理啦。 **　LODs（多细节层次）** LOD组是改善性能的简便方法。使用有多个LOD的资源，并用低分辨率的几何体渲染离相机远的对象。Unity可以自动随着相机临近在各个LOD间转换。 **填充率，过度绘制及裁剪** 这是个值得关注的话题。减少过度绘制，最远的对象最先绘制，随后上面依次绘制更近的对象。这个在平均分辨率为1080P的PC显示器上没什么问题，但对于有极高分辨率的VR和手机设备来说问题就比较严重。大量的过度绘制组成了大量像素从而影响填充率。纹理填充率是限制GPU性能的关键。 一些解决方案提供了遮挡剔除和视锥体剔除。视锥体剔除是指不渲染位于相机视锥体外的对象。不渲染看不到的对象！遮挡剔除是剔除被其它对象挡住的对象。比如，门后的房间可以被整体剔除。默认情况下，遮挡剔除是针对整个场景的，如果关卡设计得当甚至可以让你剔除游戏中的整个关卡。 LOD组当然也可以裁剪离场景很远的对象，进一步使填充率最小化。 **关卡设计** 若游戏涉及到玩家从一个房间移到另一个房间，简单的解决方法是一个关卡包含整个游戏。缺点在于内存的消耗。尽管每个房间中的各对象和材质都不可见，但其仍会被加载到内存中。将每个房间放置于单独的关卡中，在代码中智能的异步加载关卡可以改善性能。 **　异步加载** 在玩家即将进入下个房间之前，加载下一个关卡。不要使用Application.LoadLevel()同步加载，因为加载时会导致游戏挂起。由于头盔的跟踪是实时的，这会导致眩晕，对玩家来说体验太糟糕。 使用Application.LoadLevelAsync()来加载关卡。你可以在Oculus Mobile SDK BlockSplosion例子的StartupSample.cs中找到使用方法。 **光照烘焙** 关掉实时阴影！接受动态阴影的对象不会被批处理，这会导致严重的绘制调用。 在PC机上，使用单个实时方向光就可以实现很好的动态阴影效果。对于大多数现代的PC都可以提供逼真的逐像素阴影。然而在移动平台，你需要烘焙光照而不是实时阴影，以高分辨率烘焙光照结合软硬阴影实现类似的效果。 **阴影** 尤其是为了高性能的手机体验，对于3D对象的阴影处理要使用传统技巧。可以通过在对象下放置一个简单的带有模糊阴影纹理的2D四边形模拟半真实的阴影。

![](http://img.manew.com/data/attachment/forum/201510/27/154941knnn9nhooe6s1hhn.jpg.thumb.jpg)

VR小提示：不要尝试使用阴影缓冲。预处理光照环境并在角色下方使用老套的模糊阴影纹理处理方法。 例如，你Hold不住像《GTA V》中这种在高性能PC机上使用的实时动态阴影。

![](http://img.manew.com/data/attachment/forum/201510/27/155004bzbadxdk6bbesmxs.jpg.thumb.jpg)

用如下方法代替：这是一张2002年《GTA Vice City》的游戏截图，你可以在Playstation 2上使用阴影斑点来提供阴影效果的幻觉。

![](http://img.manew.com/data/attachment/forum/201510/27/155051g5httt8opnf4t38m.png.thumb.jpg)

**　Light Probes（光照探针）** 当使用烘培光照时，静态对象效果不错但动态对象还有不妥。对于动态对象可以使用光照探针来模拟简单的动态光照。 光照探针是烘焙好的立方贴图，存储了场景中多个点直接、间接甚至自发光的信息。当动态对象移动时，它在光照探测器附近进行插值获取近似某个点的光照。这是一种在动态对象上模拟实时光照的简便办法，而不用成本高昂的实时光照。 Unity的文档解释了光照探针要如何放置。 **　避免使用透明和多个材质的对象** 类似玻璃这种使用透明着色器的对象很消耗性能。使墙壁看起来更逼真的常见做法是，用一个带有灰尘或锈斑纹理的透明材质，加上另一个单独的基本漫射材质。多材质的alpha混合是很消耗性能的，每个材质都会增加一次绘制调用！但是请注意：多个纹理并没有问题，使用多个材质才耗费性能。使用一个材质结合着色器来实现多纹理的alpha混合，而非使用多个单独材质。 **蒙皮网格渲染器** 蒙皮网格渲染器常用于角色身上，它带有动画关节，可以使用物理（布娃娃）变或自定义动画（走，跳等）来实现逼真的网格变形。 坏消息是：蒙皮网格渲染器不支持批处理。对于每只眼睛，场景中各角色都会进行多次绘制调用。目前还没什么解决方案。 **　扩展阅读** 推荐大家观看Oculus大会上关于GearVR的开发演讲。 原文链接：[http://dshankar.svbtle.com/performance-optimization-for-vr-apps](http://dshankar.svbtle.com/performance-optimization-for-vr-apps) 作者：DARSHAN SHANKAR
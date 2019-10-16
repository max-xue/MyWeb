---
title: Unity3D Shader学习之十一—Shader性能优化
categories:
  - U3D
url: 447.html
id: 447
date: 2016-12-02 10:21:18
tags:
  - u3d
  - shader
  - 优化
---

#### 使用常识

*   编写功能必要的部分，舍弃不必要的Shader(同样适用于代码，资源优化)
*   模型数<顶点数<像素数，能在脚本能实现的不要在顶点着色器去实现，能在顶点着色器做的不要在片段着色器做。
*   需要针对平台进行优化，不要使用PC着色器直接用于移动平台

##### 计算精度

当使用Cg/HLSL来写着色器的时候，主要会用到三种基本的数据类型：float，half和fixed（以及由他们组成的向量和矩阵变量，即half3和float4x4）

*   float：高精度浮点型。一般是32位，就像是正规编程语言中的单精度浮点类型。
*   half：中等精度浮点型。一般是16位，范围是-60000到+60000以及3.3 的十进制数字的精度。
*   fixed：低精度浮点型。一般是11位，范围是-2.0到+2.0以及1/256th 精度。

尽可能地使用最低的精度，这点对于iOS和Android平台特别重要。推荐的经验法则：

*   对于颜色和单位长度的向量，使用fixed.
*   对于其他的，如果范围和精度允许的话，使用half，否则使用float。

在移动平台上，关键是在片段着色器中使用尽可能多低精度数据计算。在大多数移动设备的GPU中，在低精度 （fixed/lowp） 类型上应用swizzles是比较耗时的；同时，在fixed/lowp 和高精度类型之间进行转换也是需要付出很大代价的。 如果用GLSL ES编写的着色器，浮点精确度规定如下：

*   highp  - 32位浮点格式，适合用于顶点变换，但性能最慢。
*   mediump  - 16位浮点格式，适用于纹理UV坐标和比highp 大约快两倍
*   lowp - 10位的顶点格式，适合对颜色，照明计算和其它高性能操作，速度大约是highp 的4倍

如果是用CG编写的着色器或是一个表面着色器，指定精度如下：

*   float  - 类似于在GLSL ES 的highp ，最慢
*   half  - 类似于在GLSL ES 的mediump ，比float大约快两倍
*   fixed - 类似于在GLSL ES的lowp，速度大约是float 的4倍使用技巧

*   **fixed / lowp** 用于颜色，灯光信息和法线
*   **half / mediump** 用于纹理UV坐标
*   **float / highp**  避免在像素着色器，而是使用顶点着色器，计算顶点的位置。

##### 复杂的数学运算

*   复杂的数学函数（如pow，exp，log，cos，sin，tan等等）会大大增加GPU负担，所以一个好的经验法则是，每一个片段不超过一个这样的操作。考虑使用查找纹理作为替代品。
*   不要尝试编写自己的normalize，dot，inversesqrt 等操作。然而如果您使用内置的，会为你产生更好的代码。
*   紧记discard 操作，会使你的帧速度变慢。

##### Alpha测试

固定函数AlphaTest或者与其等功能的可编程函数clip()在不同的平台上拥有不同的性能特点：

*   一般来说，用它在大多数平台上来剔除完全透明的像素具有一点小小的优势。
*   但是，在IOS设备上的PowerVR GPU和一些Android设备中，alpha test是比较耗时的。不要尝试应用它来优化性能，因为它将使性能变得更慢。
    
    ##### 4.颜色掩码
    

在某些平台上 （大多是iOS移动设备上的GPU和Android 设备），使用 ColorMask 中删除一些通道(例如 ColorMask RGB) 也是比较昂贵的，因此，除非真有必要，否则不要使用它。

#### **性能瓶颈**

CPU

*   draw call
*   复杂的脚本或物理模拟

GPU

*   顶点处理 1.顶点数目 2.顶点计算
*   片元处理 1.片元数目 2.逐片元计算

带宽

*   纹理尺寸及压缩
*   分辨率过高的帧缓存

#### **性能优化**

*   动态批处理
*   静态批处理
*   优化几何体-移除不必要的硬边以及纹理衔接，避免边界平滑和纹理分离
*   LOD（Level of Detail）
*   遮挡剔除技术
*   控制绘制顺序
*   注意透明物体
*   减少实时光照和阴影

其他

*   纹理优化
*   代码优化

#### **分析工具**

*   Unity编辑器Game窗口的Stats
*   Unity编辑器Windows/Profiler
*   Unity编辑器Windows/Frame Debug

整理自官网及Unity圣典（翻译），入门精要（书籍），主要针对Shader开发，对于其他优化待续 官网：[Optimization for Mobiles](https://docs.unity3d.com/Manual/MobileOptimizationPracticalGuide.html)  [Optimizing graphics performance](https://docs.unity3d.com/Manual/OptimizingGraphicsPerformance.html) 圣典：[优化图形性能](http://www.ceeger.com/Manual/OptimizingGraphicsPerformance.html)  [移动优化](http://www.ceeger.com/Manual/MobileOptimisation.html)
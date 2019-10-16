---
title: Unity3D Shader学习之十四—内置函数和变量
categories:
  - U3D
url: 1592.html
id: 1592
comments: true
date: 2017-01-05 23:00:56
tags:
  - u3d
  - shader
---

#### [转自： Unity3D Shader性能排行](https://www.cnblogs.com/tim-unity/p/4728274.html)

整体上，性能由高到低：

1.  Unlit，仅为纹理，光线不产生效果
2.  VertexLit
3.  Diffuse　　漫反射
4.  Normal Mapped 法线贴图
5.  Specular 高光
6.  Normal Mapped Specular
7.  Parallax Normal Mapped
8.  Parallax Normal Mapped Specular

另外，unity3d还内置有一些简化的用作移动平台的shader/着色器。 推荐文章[ 内置shader详解（带图）](http://wenku.baidu.com/link?url=A7wxYiJAw-gwswPFvnZmqPGOnd4XvaoTkHNG_3DGq1N4mI9CagunR8zU7-Kn_mntWKaCPVyjYySEFto9xHx-T4ZYV5bNmdVHsHfBD6ZgzPy "baidu")

Shader性能影响因素：
-------------

着色器性能影响因素较多，最主要有二：

*   shader本身
*   Rendering Paths  （渲染路径？）
    

 

###  性能最优的两款 内置着色器：

*   Deffered shader
*   Vertex Lit

仅做绘制一次，性能只取决于纹理数。

### 在Forward rendering path：

性能仅取决于shader本身和场景中光源。

*   Pixel Lit   性能更差，效果更好，多次绘制，故能实现（阴影，法线，高光等）
*   Vertex Lit 性能更佳，所有灯光影响仅绘制一次

对内置Shader的通俗理解（转）：
------------------

1.Vertex-Lit： 基于：  基于顶点计算的光照模型 正方体： 【直接照射到的地方不会非常亮】【光照照射不到的平面无效果】 圆形：   【直接照射到的地方非常亮】【光照照射不到的地方有高光效果】 支持：  设备自动选择【可编程管线】和【固定管线】 参数：  【主色color】【SpecColor光照颜色】【EmissiveColor自发光颜色】【Shininess光照强度】 渲染代价： 比较小   2.Diffuse： 基于：  基于简单的光照模型 lambertian 正方体： 【直接照射到的地方会非常亮】【光照照摄不到的平面无效果】 圆形：   【直接照射到的地方非常亮】【光照照射不到的地方无效果】 支持：  设备需要支持【可编程管线】如果不支持将自动使用【Vertex-Lit】 参数：  【主色color】 渲染代价： 比较小   3.Specular 基于：  和Diffuse相同的光照模型，多了一个观察角度相关的反射高光(#pragma surface surf BlinnPhong) 正方形： 【直接照射到的地方会非常亮】【光照照摄不到的平面无效果】【观察角度和光入射角度会产生反射光】 圆形：  【直接照射到的地方非常亮】【光照照射不到的地方无效果】【观察角度和光入射角度会产生反射光】 支持：  设备需要支持【可编程管线】如果不支持将自动使用【Vertex-Lit】 参数：  【主色color】【SpecularColor反射光照颜色】【Shininess反射光照强度】 渲染代价： 比较大   4.Bumped Diffuse 基于：  和Diffuse相同的光照模型，同时使用了法线贴图normal mapping技术(UnpackNormal)【灰度图，白色表示凹起，黑色表示凹进】 正方形： 和【Diffuse】一样，【多了凹凸感】 圆形：  和【Diffuse】一样，【多了凹凸感】 支持：  如果设备不支持，将自动使用【Diffuse】 参数：  【主色color】【多了法线贴图】 渲染代价： 比较大   5.Bumped Specular 凹凸反射 【Bumped Diffuse】与【Specular】的合并 支持：  如果设备不支持，将使用【Specular】   6.Parallax Diffuse 基于：  和Bumped Diffuse一样的光照模型lambertian,也使用normal mapping技术(UnpackNormal)，同时使用HeightMap(ParallaxOffset)实 现更加逼真的凹凸感【高度图在法线贴图的alpha通道保存，全黑表示么有高度，白色表示高低】 支持：  设备无法使用，会自动使用【Bumped Diffuse】 参数：  【主色color】【多了法线贴图】【多了高度贴图】【height设置高度参数】 渲染代价： 比【Bumped Diffuse】更大   7.Parallax Specular 基于：  使用【Bumped Specular】+【高度图】 支持：  设备无法使用，会自动使用【Bumped Specular】   8.Decal  【贴花】 基于：  与Diffuse一样基于Lambert，增加第二张贴图，然后融合色彩(lerp)覆盖在主纹理之上【注：DiffusDetail的融合是rgb\*rgb】 支持：  设备需要支持【可编程管线】如果不支持将自动使用【Vertex-Lit】 参数：  【主色color】【两张贴图】   9.Diffuse Detail 【细节贴图】 基于：  与Diffuse一样基于Lambert，多了一张贴图与之融合(rgb\*rgb),一般用于地形，摄像机拉近时额外的细节会出现。 说明：         Detail 纹理是覆盖在主纹理上面的。Detail 纹理中深色的部分将会使得主纹理变深，而淡色的部分将会使主纹理变亮， Detail 纹 理通常是浅灰色。(与Decal 里面 Decal 纹理不同的是，Decal 纹理是 RGBA，通过 alpha 控制 DecalTexture 与 Main Texture 的融合，而 Detail 的纹理是 RGB，直接是两张纹理的rgb 通道分别相乘再*2，就是说，Detail 纹理中颜色数值 = 0.5 不会改变主纹理颜色，>0.5 会变亮，<0.5 加深)  

参考
--

参考文章 [shader实例（五）如何在unity中更好的运用shader](http://www.cnblogs.com/blog.sina.com.cn/s/blog_89d90b7c0102v7ab.html "shader") 参考文档 [Usage and Performance of Built-in Shaders](http://docs.unity3d.com/Manual/shader-Performance.html "unitydoc")
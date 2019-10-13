---
title: Unity3D Shader学习之十—释疑解惑
categories:
  - U3D
url: 336.html
id: 336
date: 2016-11-10 12:11:48
tags:
---

总结一下自己在学习过程中，令人困惑不解的问题及解答。 1、OpenGL/OpenGL ES、DirectX是什么？ 在GPU发展的早期，是不支持编程的。随着产品的更新换代、技术的革新，逐渐放开了控制接口，即固定功能着色器，但开始只能通过低级语言来使用有限的控制。接着 vertex programmability（顶点着色器编程）和 fragment programmability （片段着色器编程）相继出现，就诞生了GPU编程。GPU编程与硬件有很大的关系，GPU的品牌又很多，与硬件无关的图形开发库存在就可以解决这个问题，这个开发库就是OpenGL、DirectX，开发者可以绕过各种硬件，直接使用图形接口就可以开发出支持众多的显卡。但前提是各显示需要在硬件驱动程序中支持OpenGL或DirectX。 可见OpenGL/DirectX就是图形应用程序编程接口，GPU开发者藉此以开发图形应用程序。 2、3D模型是什么？材质，着色器，纹理,贴图定义与区别？UV是什么？ 3D模型是指通过软件制作出来的三维立体图形，在Unity里是通过3DMax或Maya等软件制作导入，支持读取FBX，DAE（collada一种开源的3D格式），3DS ,DFX和OBJ格式的模型 材质（Material）是描述渲染模型对象时的属性和资源，如使用什么着色器，使用什么纹理。材质应用到模型上，来改变模型的属性与外观。 材质是着色器的实例，相同的着色器可以创建不同的材质（配置和参数不同）！ 着色器（Shader）包含定义要使用的属性和资源类型的代码（顶点着色器、片段着色器），着色器需要通过材质才能发挥作用。 纹理(Textures) 使网格 (Meshes)、粒子 (Particles) 和界面变得生动！它是覆盖或环绕对象的图像或电影文件。纹理分为2D纹理，2D纹理数组，3D纹理，立方图(Cubemap)纹理。纹理是应用到一个网格表面，为其添加网格的几何形状无法表达的额外的信息。 对于材质，纹理只是用于提供某种基于像素的数据的图像。这些数据可能是对象的颜色、光泽度、透明度以及各种其他方面。 贴图美术人员称之为贴图，图形渲染中称之为纹理，Unity中没有贴图概念，纹理概念与之对应。贴图一般有法线贴图(Normal)、高光贴图（Specular），金属贴图(Metalic)等 UV纹理映射坐标（texture-mapping coordinates）存储在每个顶点上，区别于顶点的位置(x,y,z),使用(u,v,w)来表示顶点在纹理上的位置，纹理经常以2D图片的形式提供，所以就称为UV坐标。 从前面的各定义可知网格 (Meshes)包含---材质(Material)包含---着色器(Shader)包含---纹理（Textures）/贴图。 UV坐标属于顶点的属性。 3、顶点是什么？ 在图形API中，基本的绘制图元有三角形、直线、点精灵。顶点数目分别是3、2、1；一个立方体模型是由6个面组成，每个面由2个三角形构成的，因此只有8个顶点（重合一部分）。顶点包含顶点位置（POSITION）、切线（TANGENT）、法线（NORMAL）、纹理坐标（TEXCOORD0~TEXCOORD6）、颜色（COLOR）属性。 4、顶点着色器的输入及调用是怎么样的？片段着色器呢？ 顶点着色器(Vertex) 函数的输入来自应用程序中模型。调用是逐顶点调用的，模型有多少顶点就调用多少次。输入的参数内容就是顶点的位置（POSITION）、切线（TANGENT）、法线（NORMAL）、纹理坐标（TEXCOORD0~TEXCOORD6）、颜色（COLOR）属性。 片段着色器(Fragment)函数输入来自顶点着色器，逐片段调用的，每个片元(像素?)调用一次。输入的参数有裁剪空间坐标（SV_POSITION），顶点颜色（COLOR0~COLOR1)，纹理坐标(TEXCOORD0-TEXCOORD7) 5、POSITION、TEXCOORD0什么意思？ 这些在Unity Shader中叫做语义，可以理解为属性在Shader编程中内定的含义。比如下面的定义：

//应用程序(Application)传到顶点(Vertex)
struct a2v{
    float4 vertex : POSITION;//vertex表示顶点的位置
    float3 normal : NORMAL;//normal变量表示顶点的法向量
};

格式是 \[类型\] \[变量名\] : \[语义\] 6、Unity 5系统自带的着色器模板有哪些？ Unity5中有4种模板:

*   Standard Surface Shader:包含标准光照模型（物理渲染）的表面着色器模板；
*   Unlit Shader:不包含光照 ，包含雾效的顶点/片段着色器；
*   Image Effect Shader:屏幕后期处理特效模板；
*   Compute Shader:利用GPU的并行性进行与常规渲染无关的计算。

7、颜色相加与颜色相乘 相加：多种光源作用在同一材质上，比如：实际光照强度 I= 环境光(Iambient) + 漫反射光(Idiffuse) + 镜面高光(Ispecular) 相乘：颜色的非等比缩放，例如，光通過有色玻璃時，玻璃吸收某百分比的紅、藍、綠，就可以把光的紅藍綠強度分別乘以對應的百分比
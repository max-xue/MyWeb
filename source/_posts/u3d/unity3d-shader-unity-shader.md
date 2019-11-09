---
title: Unity3D Shader学习之四—Unity Shader基本概念
categories:
  - U3D
url: 312.html
id: 312
date: 2016-10-28 13:10:47
tags:
  - u3d
  - shader
---

- GPU(Graphic Process Unit)

图形处理单元，位于显卡中； 

- DirectX 

微软开发的图形编程库，支持Windows平台； 

- HLSL(High Level Shading Language)

属于DirectX,仅支持Windows平台； 

- OpenGL(Open Graphic Library)

跨平台的高级3D图形应用程序编程接口； 

- OpenGL ES(OpenGL for Embedded Systems)

以手持和嵌入式设备为目标的OpenGL编程接口。在智能手机中占据统治地位的图形API; 

- GLSL(OpenGL Shading Language)
 
 OpenGL的着色器语言，支持跨平台，但有局限性； 
 
- Cg(C for Graphic)
 
 NVIDIA开发提供。支持跨平台，语法类似于HLSL。目前已经不再支持更新，但仍有学习价值； 
 
- Unity Shader
 
 Unity 编辑器支持着色器方式，是对Cg语言的二次封装，提供一层编译器，根据发布平台不同，编译成不同的着色器(HLSL/GLSL)。 
 
- ShaderLab
 
 Unity 配置了一种强大的着色和材质语言。其语言风格类似于 CgFX 和 Direct3D Effects (.FX) 语言 – 可描述显示材质 (Material) 所需的一切信息。
  
GPU发展的历史，可以参考《GPU 编程与CG 语言之阳春白雪下里巴人》第一章绪论，

了解过去，是为了更好的了解现在，以及未来!
---
title: Unity3D Shader示例之—AR涂涂乐实现原理
categories:
  - U3D
url: 280.html
id: 280
date: 2016-11-06 12:30:52
tags:
  - u3d
  - shader
---

[Unity3D Shader示例之—AR涂涂乐项目实战](http://www.le-more.com/?p=1093) 

当传统游戏日渐泛滥的今天，通过新的技术能迅速找到突破口，AR/VR就是当前比较热门的技术。前段时间引爆全球的口袋怪物，以及通过AR打造自己IP的小熊尼奥（口袋动物园，口袋交通等），还有比较知名的游戏AR涂涂乐等。现在就来分析一下AR涂涂乐之技术实现。 

先看下面两张图：（来自于AR涂涂乐，第二张是我PS涂的） ![201510211406130](http://www.le-more.com/wp-content/uploads/2016/10/201510211406130.jpg) ![201510211406130](http://www.le-more.com/wp-content/uploads/2016/10/201510211406130.png) 

功能描述：一张卡片，上面或是人物、动物或其他物体的空白图片（如上图左），涂上颜色（如上图右）。通过AR识别上图（右），展示跳舞的小女孩（小女孩的颜色就是刚涂的颜色）。 

技术分析：

1、通过AR识别图片展示模型；

2、通过提取所涂颜色应用于模型。 

具体实现： 

一、通过AR识别图片展示模型 通过AR sdk实现单张图片识别，当前使用的是Vuforia，开发环境Unity3D。因为这个较为简单，不再详述，特别提示遇到一个看似简单但[困惑的问题](http://www.le-more.com/?p=296)。
 
 二、通过提取所涂颜色应用于模型 首先通过摄像头获得涂抹后的图片，然后提取涂抹颜色应用于模型指定区域。如何将识别的图片，将涂抹区域能正确的映射到模型上呢？就需要在制作模型将UV区域分配到涂抹图片的位置。查看下面图片： ![uv0](http://www.le-more.com/wp-content/uploads/2016/11/uv0.png) ![uv1](http://www.le-more.com/wp-content/uploads/2016/11/uv1.png)   
 
 原理看似很简单，具体究竟是如何实现呢？ 请看下节的具体的实现细节。
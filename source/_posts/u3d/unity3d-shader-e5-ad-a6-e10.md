---
title: Unity3D Shader学习之三—渲染流程
categories:
  - U3D
url: 316.html
id: 316
date: 2016-10-28 13:04:31
tags:
  - u3d
  - shader
---

Shader编程，只是图形编程的很小的一部分。如果能对GPU的工作原理有一定的了解，对于学习好Shader会有很大的益处！下面来了解下GPU的渲染流程。 下图来自《GPU 编程与CG 语言之阳春白雪下里巴人》3.1节，从图中我们可以完整的了解从应用程序到生成用于显示的Frame Buffer的过程。 ![b3a6-tmp](http://www.le-more.com/wp-content/uploads/2016/10/B3A6.tmp_.png) 下图是OpenGL 图形管线，来自《OPENGL ES 3.0编程指南(原书第2版)》1.1节，在灰色部分就是可编程部分。 ![961b-tmp](http://www.le-more.com/wp-content/uploads/2016/10/961B.tmp_.png) 再来看下面的图，更为清晰。这是来自![dc15-tmp](http://www.le-more.com/wp-content/uploads/2016/10/DC15.tmp_.png) 上图来自《Unity Shader入门精要》2.3节，这本书浅显易懂，对于基础不好的，推荐大家看这本书。然后再结合《GPU 编程与CG 语言之阳春白雪下里巴人》等入门会更快！ 对于顶点数据是什么，3D程序或游戏场景等数据在这个流程中怎样在最后呈现在屏幕下，就需要继续了解坐标变换相关内容！
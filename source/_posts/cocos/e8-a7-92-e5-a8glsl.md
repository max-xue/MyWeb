---
title: Cocos OpenGL/ES学习之一—你好，三角形（2）着色器GLSL
categories:
- Cocos
url: 486.html
id: 486
date: 2016-12-15 12:25:42
tags:
---

在上一篇介绍在cocos中如果使用自定义shader，通过OpenGL ES绘制一个三角形，下面来对其实现进行分析一下。 初始化部分加载一个顶点着色器和一个片段着色器，下面对两个着色器做一个简单的注解： 顶点着色程序：
    
    //输入变量，顶点位置
    attribute vec4 a_position;
    //输入变量，顶点颜色
    attribute vec4 a_color;
    
    //输出变量片段颜色
    varying vec4 v_fragmentColor;
    
    void main()
    {
     //顶点位置通过模型、视图、投影矩阵点乘顶点坐标转化为裁剪空间位置
     //CC_MVPMatrix 是cocos提供的实现（待确定）
     gl\_Position = CC\_MVPMatrix * a_position;
     v\_fragmentColor = a\_color;
    }
    
    片段着色器
    
    //输入变量
    varying vec4 v_fragmentColor;
     
    void main()
    {
        //给内置片段着色变量赋值
        gl\_FragColor = v\_fragmentColor;
    }

当前cocos版本（3.13）使用的OpenGL ES版本是2.0，和最新的3.0规范不同。在着色器程序的开头一般有版本的声明：

    #version 300 es

如果开头没有版本声明，默认是OpenGL ES 2.0的着色器程序，但要注意声明的版本号不是200，因为OpenGL ES 2.0区别与1.0，1.1是第一个可编程的API，它的着色器版本是1.0。 着色器程序中的变量限定符转自一篇博文说的很清楚：[原文地址](http://blog.csdn.net/renai2008/article/details/7844495)

> 1.uniform变量 uniform变量是外部application程序传递给（vertex和fragment）shader的变量。因此它是application通过函数glUniform**（）函数赋值的。在（vertex和fragment）shader程序内部，uniform变量就像是C语言里面的常量（const ），它不能被shader程序修改。（shader只能用，不能改） 如果uniform变量在vertex和fragment两者之间声明方式完全一样，则它可以在vertex和fragment共享使用。（相当于一个被vertex和fragment shader共享的全局变量）uniform变量一般用来表示：变换矩阵，材质，光照参数和颜色等信息。 以下是例子： uniform mat4 viewProjMatrix; //投影+视图矩阵 uniform mat4 viewMatrix;        //视图矩阵 uniform vec3 lightPosition;     //光源位置 2.attribute变量 attribute变量是只能在vertex shader中使用的变量。（它不能在fragment shader中声明attribute变量，也不能被fragment shader中使用）。一般用attribute变量来表示一些顶点的数据，如：顶点坐标，法线，纹理坐标，顶点颜色等。 在application中，一般用函数glBindAttribLocation（）来绑定每个attribute变量的位置，然后用函数glVertexAttribPointer（）为每个attribute变量赋值。 以下是例子： uniform mat4 u\_matViewProjection; attribute vec4 a\_position; attribute vec2 a\_texCoord0; varying vec2 v\_texCoord; void main(void) { gl\_Position = u\_matViewProjection * a\_position; v\_texCoord = a\_texCoord0; } 3.varying变量 varying变量是vertex和fragment shader之间做数据传递用的。一般vertex shader修改varying变量的值，然后fragment shader使用该varying变量的值。因此varying变量在vertex和fragment shader二者之间的声明必须是一致的。application不能使用此变量。 以下是例子： // Vertex shader uniform mat4 u\_matViewProjection; attribute vec4 a\_position; attribute vec2 a\_texCoord0; varying vec2 v\_texCoord; // Varying in vertex shader void main(void) { gl\_Position = u\_matViewProjection * a\_position; v\_texCoord = a\_texCoord0; } // Fragment shader precision mediump float; varying vec2 v\_texCoord; // Varying in fragment shader uniform sampler2D s\_baseMap; uniform sampler2D s\_lightMap; void main() { vec4 baseColor; vec4 lightColor; baseColor = texture2D(s\_baseMap, v\_texCoord); lightColor = texture2D(s\_lightMap, v\_texCoord); gl\_FragColor = baseColor * (lightColor + 0.25); }

 （1）attribute变量(属性变量)只能用于顶点着色器中，不能用于片元着色器。一般用该变量来表示一些顶点数据，如：顶点坐标、纹理坐标、颜色等。

 （2）uniforms变量(一致变量)用来将数据值从应用程其序传递到顶点着色器或者片元着色器。该变量有点类似C语言中的常量（const），即该变量的值不能被shader程序修改。一般用该变量表示变换矩阵、光照参数、纹理采样器等。

  （3）varying变量(易变变量)是从顶点着色器传递到片元着色器的数据变量。顶点着色器可以使用易变变量来传递需要插值的颜色、法向量、纹理坐标等任意值。在顶点与片元shader程序间传递数据是很容易的，一般在顶点shader中修改varying变量值，然后片元shader中使用该值，当然，该变量在顶点及片元这两段shader程序中声明必须是一致的。例如：上面代码中应用程序中由顶点着色器传入片元着色器中的vColor变量。

 （4）gl\_Position为内建变量，表示变换后点的空间位置。顶点着色器从应用程序中获得原始的顶点位置数据，这些原始的顶点数据在顶点着色器中经过平移、旋转、缩放等数学变换后，生成新的顶点位置。新的顶点位置通过在顶点着色器中写入gl\_Position传递到渲染管线的后继阶段继续处理。

参考下面两张图来理解上面的定义： ![](http://www.le-more.com/wp-content/uploads/2016/12/vetex_shader.png)![](http://www.le-more.com/wp-content/uploads/2016/12/fragment_shader.png) 两个着色器程序很简单，但在OpenGL ES中必须至少包含一个顶点着色器程序和一个片段程序，否则不会出现任何图形。 初始化着色器:
    
    auto glProgram = GLProgram::createWithByteArrays(vShaderStr, fShaderStr);
    auto state = GLProgramState::getOrCreateWithGLProgram(glProgram);
    setGLProgramState(state);

cocos通过上面代码加载，编译着色器程序，欲了解cocos对OpenGL ES封装，下一篇再介绍。 示例程序开源托管在Github: [CocosShader](https://github.com/max-xue/CocosShader)
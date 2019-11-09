---
title: OpenGL ES学习实践—Shader实现不显示图片4个角
categories:
- Cocos
id: 1041
comments: true
date: 2017-06-29 14:14:29
tags:
---

前一段时间遇到一个问题，如果在程序中不改变图片的情况下，把图片4个角去掉。在看到这个题目的时候，想到通过UV坐标可以实现，但具体的细节不甚明白。如下图 如何过滤斜线覆盖的坐标呢？ ![](http://www.le-more.com/wp-content/uploads/2017/06/opengles_samples_uv.png) 这个问题就要用数学来解决了：假设截取的长度为e,因UV坐标的范围是\[0,1\]，图中标出边界点的坐标，可通过直线公式分别解出各条直线。 因在window上测试，UV坐标的原点在左上角。求出的各条直接为： 左上：y = -x + e 右上：y = x + e - 1 左下：y = x + 1-e 右下：y = -x + 2-e 根据直线公式分得出落在斜线部分点的条件为： 左上：y +x < e 右上：x- y > 1 -e 左下：y - x > 1-e 右下：y + x > 2-e Shader实现如下(片段着色器)：
    
    #ifdef GL_ES
    precision mediump float;
    #endif
    
    varying vec4 v_fragmentColor;
    varying vec2 v_texCoord;
    
    const float edge = 0.4;
    
    void main(void)
    {
        vec4 c = texture2D(CC\_Texture0, v\_texCoord);
        
        //left-up
            //也可以使用discard，透明度测试
            //if(v\_texCoord.x + v\_texCoord.y < 0.4) discard; 
        if(v\_texCoord.x + v\_texCoord.y  < 0.4) c = vec4(0.0); 
        //right-up
        if(v\_texCoord.x - v\_texCoord.y  > 0.6) c = vec4(0.0); 
        //left-bottom
        if(v\_texCoord.y - v\_texCoord.x  > 0.6) c = vec4(0.0); 
        //right-bottom
        if(v\_texCoord.y + v\_texCoord.x  > 1.6) c = vec4(0.0); 
    
        gl_FragColor = c;
    }

C++类相对简单，直接扩展使用cocos实现的类：
    
    class SpriteCutCorner : public Effect
    {
    public:
        CREATE_FUNC(SpriteCutCorner);
    
        bool init()
        {
            initGLProgramState("Shaders/custom\_cut\_corner.fsh");
            return true;
        }
    };

效果如下： ![](http://www.le-more.com/wp-content/uploads/2017/06/opengles_samples_uv_ret.png) 实现这个功能还有其他办法，待续... 工程源码：[CocosShader](https://github.com/max-xue/MyProject_Cocos)
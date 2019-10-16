---
title: Cocos OpenGL/ES学习之一—你好，三角形（3）OpenGL ES 着色器与程序
categories:
- Cocos
url: 491.html
id: 491
date: 2016-12-28 11:53:34
tags:
---

OpenGL ES使用着色器渲染需要创建着色器对象和程序对象。着色器对象负责加载、编译着色器程序源码，程序对象负责连接着色器对象，可以连接多个着色器对象。在OpenGL ES中程序对象只能并且至少连接一个顶点着色器对象和一个片段着色器对象。 具体的步骤：

1.  创建一个顶点着色器对象和一个片段着色器对象；
2.  着色器对象连接相应的源代码；
3.  编译着色器对象；
4.  创建一个程序对象；
5.  程序对象连接着色器对象
6.  链接程序对象。

创建着色器

函数原型：
int glCreateShader (int type)
方法参数：
GL\_VERTEX\_SHADER          (顶点shader)
GL\_FRAGMENT\_SHADER        (片元shader)

加载着色器源码

函数原型：
void glShaderSource (int shader, String string)
参数含义：
shader是代表shader容器的id（由glCreateShader返回的整形数）；
string是包含源程序的字符串数组。

编译着色器

函数原型：
void glCompileShader (int shader)
参数含义：
shader是代表shader容器的id。

创建程序

函数原型：
int glCreateProgram ()
如果函数调用成功将返回一个正整数作为该着色器程序的id。

连接着色器对象

函数原型：
void glAttachShader (int program, int shader)
参数含义：
program是着色器程序容器的id；
shader是要添加的顶点或者片元shader容器的id。

链接程序对象

函数原型：
void glLinkProgram (int program)
参数含义：
program是着色器程序容器的id。

函数原型：
void glUseProgram (int program)
参数含义：
program是要使用的着色器程序的id。

上面分别列出了各步骤中用到的关键函数。 其他OpenGL ES API： 编译阶段使用glGetShaderiv获取编译情况

函数原型：
void glGetShaderiv (int shader, int pname, int\[\] params, int offset)
参数含义：
shader是一个shader的id；
pname使用GL\_COMPILE\_STATUS；
params是返回值，如果一切正常返回GL\_TRUE代，否则返回GL\_FALSE。

编译阶段使用glGetShaderInfoLog获取编译错误

函数原型：
String glGetShaderInfoLog (int shader)
参数含义：
shader是一个顶点shader或者片元shader的id。

在连接阶段使用glGetProgramiv获取连接情况

函数原型：
void glGetProgramiv (int program, int pname, int\[\] params, int offset)
参数含义：
program是一个着色器程序的id；
pname是GL\_LINK\_STATUS；
param是返回值，如果一切正常返回GL\_TRUE代，否则返回GL\_FALSE。

在连接阶段使用glGetProgramInfoLog获取连接错误

函数原型：
String glGetProgramInfoLog (int program)
参数含义：
program是一个着色器程序的id。

清理shader的glDeleteShader方法

函数原型：
void glDeleteShader (int shader)；
参数含义：
shader是要被排除的顶点shader或者片元shader的id。

在cocos引擎的renderer/CCGLProgram文件定义GLProgram封装了着色器和程序操作。 步骤1到3包含在GLProgram::compileShader函数中:

bool GLProgram::compileShader(GLuint* shader, GLenum type, const GLchar* source, const std::string& convertedDefines)
{
    GLint status;

    if (!source)
    {
        return false;
    }

    //拼接着色器源码
    //根据不同平台设置精度类型
    //设置统一变量
    const GLchar *sources\[\] = {
#if CC\_TARGET\_PLATFORM == CC\_PLATFORM\_WINRT
        (type == GL\_VERTEX\_SHADER ? "precision mediump float;\\n precision mediump int;\\n" : "precision mediump float;\\n precision mediump int;\\n"),
#elif (CC\_TARGET\_PLATFORM != CC\_PLATFORM\_WIN32 && CC\_TARGET\_PLATFORM != CC\_PLATFORM\_LINUX && CC\_TARGET\_PLATFORM != CC\_PLATFORM\_MAC)
        (type == GL\_VERTEX\_SHADER ? "precision highp float;\\n precision highp int;\\n" : "precision mediump float;\\n precision mediump int;\\n"),
#endif
        COCOS2D\_SHADER\_UNIFORMS,
        convertedDefines.c_str(),
        source};
    //创建着色器
    *shader = glCreateShader(type);
    //加载源码
    glShaderSource(\*shader, sizeof(sources)/sizeof(\*sources), sources, nullptr);
    //编译着色器
    glCompileShader(*shader);
    
    //查询状态
    glGetShaderiv(*shader, GL\_COMPILE\_STATUS, &status);

    if (! status)
    {
        GLsizei length;
        glGetShaderiv(*shader, GL\_SHADER\_SOURCE_LENGTH, &length);
        GLchar* src = (GLchar *)malloc(sizeof(GLchar) * length);

        glGetShaderSource(*shader, length, nullptr, src);
        CCLOG("cocos2d: ERROR: Failed to compile shader:\\n%s", src);

        if (type == GL\_VERTEX\_SHADER)
        {
            CCLOG("cocos2d: %s", getVertexShaderLog().c_str());
        }
        else
        {
            CCLOG("cocos2d: %s", getFragmentShaderLog().c_str());
        }
        free(src);

        return false;
    }

    return (status == GL_TRUE);
}

步骤4(创建程序对象)到5(连接程序与着色器)在下面函数中

bool GLProgram::initWithByteArrays(const GLchar* vShaderByteArray, const GLchar* fShaderByteArray, const std::string& compileTimeDefines)
{
    //创建程序对象
    _program = glCreateProgram();
    CHECK\_GL\_ERROR_DEBUG();

    // convert defines here. If we do it in "compileShader" we will do it twice.
    // a cache for the defines could be useful, but seems like overkill at this point
    std::string replacedDefines = "";
    replaceDefines(compileTimeDefines, replacedDefines);

    \_vertShader = \_fragShader = 0;

    if (vShaderByteArray)
    {
        if (!compileShader(&\_vertShader, GL\_VERTEX_SHADER, vShaderByteArray, replacedDefines))
        {
            CCLOG("cocos2d: ERROR: Failed to compile vertex shader");
            return false;
       }
    }

    // Create and compile fragment shader
    if (fShaderByteArray)
    {
        if (!compileShader(&\_fragShader, GL\_FRAGMENT_SHADER, fShaderByteArray, replacedDefines))
        {
            CCLOG("cocos2d: ERROR: Failed to compile fragment shader");
            return false;
        }
    }

    if (_vertShader)
    {   //连接程序与着色器
        glAttachShader(\_program, \_vertShader);
    }
    CHECK\_GL\_ERROR_DEBUG();

    if (_fragShader)
    {   //连接程序与着色器
        glAttachShader(\_program, \_fragShader);
    }

    _hashForUniforms.clear();

    CHECK\_GL\_ERROR_DEBUG();

    return true;
}

6链接程序：

bool GLProgram::link()
{
    CCASSERT(_program != 0, "Cannot link invalid program");

    GLint status = GL_TRUE;

    bindPredefinedVertexAttribs();

    //链接程序
    glLinkProgram(_program);

    // Calling glGetProgramiv(...GL\_LINK\_STATUS...) will force linking of the program at this moment.
    // Otherwise, they might be linked when they are used for the first time. (I guess this depends on the driver implementation)
    // So it might slow down the "booting" process on certain devices. But, on the other hand it is important to know if the shader
    // linked succesfully. Some shaders might be downloaded in runtime so, release version should have this check.
    // For more info, see Github issue #16231
    glGetProgramiv(\_program, GL\_LINK_STATUS, &status);

    if (status == GL_FALSE)
    {
        CCLOG("cocos2d: ERROR: Failed to link program: %i", _program);
        GL::deleteProgram(_program);
        _program = 0;
    }
    else
    {
        parseVertexAttribs();
        parseUniforms();

        clearShader();
    }

    return (status == GL_TRUE);
}

使用着色器

void GLProgram::use()
{
    GL::useProgram(_program);
}

删除与清理

inline void GLProgram::clearShader()
{
    if (_vertShader)
    {
        glDeleteShader(_vertShader);
    }

    if (_fragShader)
    {
        glDeleteShader(_fragShader);
    }

    \_vertShader = \_fragShader = 0;
}

cocos引擎renderer/CCGLProgramCache文件定义GLProgramCache单例类缓存管理GLProgram 对象(shaders)，同时管理众多内置的Shaders。 在链接程序中如下的函数：

bool GLProgram::link()
{
    ...
    //绑定顶点着色器的变量名称与位置
    //从程序传给shader变量，比如：顶点位置，颜色，UV坐标等
    bindPredefinedVertexAttribs();

    ...

    if (status == GL_FALSE)
    {
       ...
    }
    else
    {
        //将顶点的属性变量的名称及位置存储与类变量：_vertexAttribs
        parseVertexAttribs();
        //将顶点的属性常量的名称及位置存储与类变量：_userUniforms
        parseUniforms();

        ...
    }

    return (status == GL_TRUE);
}

这些函数将在下一节将介绍：顶点属性、顶点数组和缓冲区对象。 示例程序开源托管在Github: [CocosShader](https://github.com/max-xue/CocosShader) 这一篇对过程介绍的比较清楚，推荐阅读：[OpenGL ES 2.0 Shader相关介绍](http://blog.csdn.net/grafx/article/details/35561487?utm_source=tuicool&utm_medium=referral)
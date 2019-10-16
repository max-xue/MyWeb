---
title: OpenGL ES学习之三---你好三角形（顶点属性数组）
categories:
- Cocos
url: 993.html
id: 993
comments: true
date: 2017-06-05 09:52:15
tags:
---

加载顶点属性的方式一：顶点属性数组 结合绘制三角形的实例，介绍使用定义顶点属性及其赋值的方法。 初始化，获得程序对象

bool HelloTriangle::init()
{
	bool ret = false;
	do
	{
		CC\_BREAK\_IF(false == TestCase::init());

		char vShaderStr\[\] =
			"//着色器语言版本							  \\n"
			"#version 300 es                          \\n"
			"//布局变量块指定变量在内存布局方式		  \\n"
			"layout(location = 0) in vec4 vPosition;  \\n"
			"void main()                              \\n"
			"{                                        \\n"
			"   gl_Position = vPosition;              \\n"
			"}                                        \\n";

		char fShaderStr\[\] =
			"#version 300 es                              \\n"
			"//指定默认精度限定符为mediump                  \\n"
			"precision mediump float;                     \\n"
			"//指定输出变量                                \\n"
			"out vec4 fragColor;                          \\n"
			"void main()                                  \\n"
			"{                                            \\n"
			"   fragColor = vec4 ( 1.0, 0.0, 0.0, 1.0 );  \\n"
			"}                                            \\n";

		//创建程序对象
		programObject = GLShader::LoadProgram(vShaderStr, fShaderStr);

		//获取最大顶点属性数量
		GLint maxVertexAttribs; // n will be >= 16
		glGetIntegerv(GL\_MAX\_VERTEX_ATTRIBS, &maxVertexAttribs);

		// Bind vPosition to attribute 0   
		// 第二个方法使用layout 在着色器中指定
		glBindAttribLocation(programObject, 0, "vPosition");

		////获取属性的索引
		//GLuint index =  glGetAttribLocation(programObject,
		//	"vPosition");

		ret = true;
	} while (0);

	return ret;
}

绘制函数

void HelloTriangle::onDraw(const Mat4& transform, uint32_t flags)
{
	GLfloat vVertices\[\] = { 0.0f,  0.5f, 0.0f,
		-0.5f, -0.5f, 0.0f,
		0.5f, -0.5f, 0.0f };

	// Clear the color buffer
	glClear(GL\_COLOR\_BUFFER_BIT);

	// Use the program object
	//函数原型：
	//	void glUseProgram(int program)
	//	参数含义：
	//	program是要使用的着色器程序的id。
	glUseProgram(programObject);

	//glVertexAttribPointer( GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride,const GLvoid * pointer):加载顶点数据
	//index:顶点属性的索引值
	//size:每个顶点属性的组件数量。必须为1、2、3或者4。初始值为4。（如position是由3个（x,y,z）组成，而颜色是4个（r,g,b,a））
	//type:顶点的数据类型
	//normalized:是否归一化
	//stride:顶点属性之间的偏移量
	//pointer:指定第一个组件在数组的第一个顶点属性中的偏移量
	// Load the vertex data
	glVertexAttribPointer(0, 3, GL\_FLOAT, GL\_FALSE, 0, vVertices);
	glEnableVertexAttribArray(0);

	//(v0,v1,v2,v3,v4,v5)
	//GL_TRIANGLES:两个三角形 (v0,v1,v2) (v3,v4,v5)
	//GL\_TRIANGLE\_STRIP:三个三角形(v0,v1,v2)(v1,v2,v3)(v2,v3,v4)
	//GL\_TRIANGLE\_FAN:三个三角形(v0,v1,v2)(v0,v2,v3)(v0,v3,v4)
	//glDrawArrays (GLenum mode, GLint first, GLsizei count):绘制三角形
	//mode:GL\_POINTS、GL\_LINES、GL\_LINE\_LOOP、GL\_LINE\_STRIP、GL\_TRIANGLES、GL\_TRIANGLE\_STRIP、GL\_TRIANGLE_FAN
	//first: 0
	//count: 3
	glDrawArrays(GL_TRIANGLES, 0, 3);
}

###### ![](http://www.le-more.com/wp-content/uploads/2017/06/hello_triangle_vertex_arry.png)

###### **使用常量指定顶点属性**

初始化

bool HelloTriangle_ConstAndVertexArray::init()
{
	bool ret = false;
	do
	{
		CC\_BREAK\_IF(false == TestCase::init());

		const char vShaderStr\[\] =
			"#version 300 es							\\n"
			"layout(location = 0) in vec4 a_color;		\\n"
			"layout(location = 1) in vec4 a_position;	\\n"
			"out vec4 v_color;							\\n"
			"void main()								\\n"
			"{											\\n"
			" v\_color = a\_color;						\\n"
			" gl\_Position = a\_position;					\\n"
			"}";
		const char fShaderStr\[\] =
			"#version 300 es							\\n"
			"precision mediump float;					\\n"
			"in vec4 v_color;							\\n"
			"out vec4 o_fragColor;						\\n"
			"void main()								\\n"
			"{											\\n"
			" o\_fragColor = v\_color;					\\n"
			"}";

		//创建程序对象
		programObject = GLShader::LoadProgram(vShaderStr, fShaderStr);

		glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

		ret = true;
	} while (0);

	return ret;
}

绘制函数

void HelloTriangle\_ConstAndVertexArray::onDraw(const Mat4& transform, uint32\_t flags)
{
	auto winSize = Director::getInstance()->getWinSize();
	GLfloat color\[4\] = { 1.0f, 0.0f, 0.0f, 1.0f };
	// 3 vertices, with (x, y, z) per-vertex
	GLfloat vertexPos\[3 * 3\] =
	{
		0.0f, 0.5f, 0.0f, // v0
		-0.5f, -0.5f, 0.0f, // v1
		0.5f, -0.5f, 0.0f // v2
	};
	glViewport(0, 0, winSize.width, winSize.height);
	
	glClear(GL\_COLOR\_BUFFER_BIT);
	
	glUseProgram(programObject);

	//指定常量顶点属性
	glVertexAttrib4fv(0, color);
	glVertexAttribPointer(1, 3, GL\_FLOAT, GL\_FALSE, 0,vertexPos);

	//glEnableVertexAttribArray(0);
	glEnableVertexAttribArray(1);
	glDrawArrays(GL_TRIANGLES, 0, 3);
	//glDisableVertexAttribArray(1);

	//查询活动属性变量数量
	//GLint numActiveAttribs;
	//glGetProgramiv(programObject, GL\_ACTIVE\_ATTRIBUTES, &numActiveAttribs);
}

不知道为什么三个顶点的颜色不一样... ![](http://www.le-more.com/wp-content/uploads/2017/06/hello_triangle_vertex_const.png) 工程源码：[CocosShader](https://github.com/max-xue/MyProject_Cocos)
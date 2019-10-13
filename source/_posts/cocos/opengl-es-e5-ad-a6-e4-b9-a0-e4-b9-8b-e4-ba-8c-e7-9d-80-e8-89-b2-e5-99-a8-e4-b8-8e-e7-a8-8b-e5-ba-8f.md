---
title: OpenGL ES学习之二---着色器与程序
categories:
- Cocos
url: 990.html
id: 990
comments: true
date: 2017-06-02 12:19:30
tags:
---

主要代码来自《OpenGL.ES.3.0.Programming.Guide.2nd.Edition》 在原书代码基础上，加入到Cocos的工程。具体的使用及API介绍会在代码详细描述,原理及流程前面已经有所说明，不再赘述！ 函数声明：

//加载编译Shader源码，返回shader实例句柄
static GLuint LoadShader(GLenum type, const char *shaderSrc);
//加载程序，返回程序句柄
static GLuint LoadProgram(const char \*vertShaderSrc, const char \*fragShaderSrc);

函数定义：

GLuint GLShader::LoadShader(GLenum type, const char *shaderSrc)
{
	GLuint shader;
	GLint compiled;

	// Create the shader object
	//函数原型：
	//	int glCreateShader(int type)
	//	方法参数：
	//	GL\_VERTEX\_SHADER(顶点shader)
	//	GL\_FRAGMENT\_SHADER(片元shader)
	shader = glCreateShader(type);

	if (shader == 0)
		return 0;

	// Load the shader source
	//函数原型：
	//	void glShaderSource(int shader, String string)
	//	参数含义：
	//	shader是代表shader容器的id（由glCreateShader返回的整形数）；
	//	string是包含源程序的字符串数组。
	glShaderSource(shader, 1, &shaderSrc, NULL);

	// Compile the shader
	//函数原型：
	//	void glCompileShader(int shader)
	//	参数含义：
	//	shader是代表shader容器的id。
	glCompileShader(shader);

	// Check the compile status
	//编译阶段使用glGetShaderiv获取编译情况
	//函数原型：
	//	void glGetShaderiv(int shader, int pname, int\[\] params, int offset)
	//	参数含义：
	//	shader是一个shader的id；
	//	pname使用GL\_COMPILE\_STATUS；
	//	params是返回值，如果一切正常返回GL\_TRUE代，否则返回GL\_FALSE。
	glGetShaderiv(shader, GL\_COMPILE\_STATUS, &compiled);

	if (!compiled)
	{
		GLint infoLen = 0;

		glGetShaderiv(shader, GL\_INFO\_LOG_LENGTH, &infoLen);

		if (infoLen > 1)
		{
			char* infoLog = (char *)malloc(sizeof(char) * infoLen);

			//编译阶段使用glGetShaderInfoLog获取编译错误
			//函数原型：
			//	String glGetShaderInfoLog(int shader)
			//	参数含义：
			//	shader是一个顶点shader或者片元shader的id。
			glGetShaderInfoLog(shader, infoLen, NULL, infoLog);
			GLUtils::LogMessage("Error compiling shader:\\n%s\\n", infoLog);

			free(infoLog);
		}

		// Free up no longer needed shader resources
		//函数原型：
		//	void glDeleteShader(int shader)；
		//	参数含义：
		//	shader是要被排除的顶点shader或者片元shader的id。
		glDeleteShader(shader);
		return 0;
	}

	return shader;
}

获取程序对象

GLuint GLShader::LoadProgram(const char \*vertShaderSrc, const char \*fragShaderSrc)
{
	GLuint vertexShader;
	GLuint fragmentShader;
	GLuint programObject;
	GLint linked;

	// Load the vertex/fragment shaders
	vertexShader = LoadShader(GL\_VERTEX\_SHADER, vertShaderSrc);
	if (vertexShader == 0)
		return 0;

	fragmentShader = LoadShader(GL\_FRAGMENT\_SHADER, fragShaderSrc);
	if (fragmentShader == 0)
	{
		glDeleteShader(vertexShader);
		return 0;
	}

	// Create the program object
	//函数原型：
	//	int glCreateProgram()
	//	如果函数调用成功将返回一个正整数作为该着色器程序的id。
	programObject = glCreateProgram();

	if (programObject == 0)
		return 0;

	//连接着色器对象
	//函数原型：
	//	void glAttachShader(int program, int shader)
	//	参数含义：
	//	program是着色器程序容器的id；
	//	shader是要添加的顶点或者片元shader容器的id。
	glAttachShader(programObject, vertexShader);
	glAttachShader(programObject, fragmentShader);

	// Link the program
	//函数原型：
	//	void glLinkProgram(int program)
	//	参数含义：
	//	program是着色器程序容器的id。
	glLinkProgram(programObject);

	// Check the link status
	//函数原型：
	//	void glGetProgramiv(int program, int pname, int\[\] params, int offset)
	//	参数含义：
	//	program是一个着色器程序的id；
	//	pname是GL\_LINK\_STATUS；
	//	param是返回值，如果一切正常返回GL\_TRUE代，否则返回GL\_FALSE。
	glGetProgramiv(programObject, GL\_LINK\_STATUS, &linked);

	if (!linked)
	{
		GLint infoLen = 0;

		glGetProgramiv(programObject, GL\_INFO\_LOG_LENGTH, &infoLen);

		if (infoLen > 1)
		{
			char* infoLog = (char*)malloc(sizeof(char) * infoLen);

			//在连接阶段使用glGetProgramInfoLog获取连接错误
			//函数原型：
			//	String glGetProgramInfoLog(int program)
			//	参数含义：
			//	program是一个着色器程序的id。
			glGetProgramInfoLog(programObject, infoLen, NULL, infoLog);
			GLUtils::LogMessage("Error linking program:\\n%s\\n", infoLog);

			free(infoLog);
		}


		glDeleteProgram(programObject);
		return 0;
	}

	// Free up no longer needed shader resources
	//函数原型：
	//	void glDeleteShader(int shader)；
	//	参数含义：
	//	shader是要被排除的顶点shader或者片元shader的id。
	glDeleteShader(vertexShader);
	glDeleteShader(fragmentShader);

        //Query
        //QueryActiveUniform(programObject);

	return programObject;
}

查询统一变量

void QueryActiveUniform(GLuint programObject) {
	GLint maxUniformLen;
	GLint numUniforms;
	char *uniformName;
	GLint index;

	glGetProgramiv(programObject, GL\_ACTIVE\_UNIFORMS, &numUniforms);
	glGetProgramiv(programObject, GL\_ACTIVE\_UNIFORM\_MAX\_LENGTH,
		&maxUniformLen);
	uniformName = (char*)malloc(sizeof(char) * maxUniformLen);
	for (index = 0; index < numUniforms; index++)
	{
		GLint size;
		GLenum type;
		GLint location;

		// Get the uniform info
		glGetActiveUniform(programObject, index, maxUniformLen, NULL,
			&size, &type, uniformName);

		// Get the uniform location
		location = glGetUniformLocation(programObject, uniformName);
		switch (type)
		{
		case GL_FLOAT:
			//
			break;
		case GL\_FLOAT\_VEC2:
			//
			break;
		case GL\_FLOAT\_VEC3:
			//
			break;
		case GL\_FLOAT\_VEC4:
			//
			break;
		case GL_INT:
			//
			break;
			// ... Check for all the types ...
		default:
			// Unknown type
			break;
		}
	}
}

  工程源码：[CocosShader](https://github.com/max-xue/MyProject_Cocos)
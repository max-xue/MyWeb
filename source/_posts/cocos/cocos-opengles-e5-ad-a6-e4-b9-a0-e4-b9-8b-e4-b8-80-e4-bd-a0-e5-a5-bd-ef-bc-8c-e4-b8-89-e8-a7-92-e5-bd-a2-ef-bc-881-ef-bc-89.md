---
title: Cocos OpenGL/ES学习之一—你好，三角形（1）概览
categories:
- Cocos
url: 457.html
id: 457
date: 2016-12-14 11:27:00
tags:
---

cocos 版本为3.13.1，后续可能会根据cocos版本的升级变的不同。 本篇基于cocos引擎，使用OpenGL/ES实现一个三角形。 先看一下效果图： ![triangle](http://www.le-more.com/wp-content/uploads/2016/12/triangle-1.png) 函数声明：

//函数声明
//重载父类函数
virtual void draw(cocos2d::Renderer *renderer, const cocos2d::Mat4 &transform, uint32_t flags) override;
//自定义绘图函数
void onDraw(const cocos2d::Mat4& transform, uint32_t flags);

函数实现：

void Triangle::draw(Renderer *renderer, const Mat4 &transform, uint32_t flags)
{
 \_customCommand.init(\_globalZOrder, transform, flags);
 \_customCommand.func = CC\_CALLBACK_0(Triangle::onDraw, this, transform, flags);
 renderer->addCommand(&_customCommand);
}


void Triangle::onDraw(const Mat4& transform, uint32_t flags)
{
 //获取程序对象
 auto glProgram = getGLProgram();
 //设置为活动程序
 glProgram->use();
 //设置统一变量（通过API传送给着色器源码的只读常量）
 glProgram->setUniformsForBuiltins();

 auto winSize = Director::getInstance()->getWinSize();
 //顶点坐标
 float vertercies\[\] = { 0, 0
 ,winSize.width, 0,
 winSize.width / 2, winSize.height };
 //顶点颜色
 float color\[\] = { 1, 0, 0, 1,
 0, 1, 0, 1,
 0, 0, 1, 1 };

 //启用position和color的vertex attribute
 GL::enableVertexAttribs(GL::VERTEX\_ATTRIB\_FLAG\_POSITION | GL::VERTEX\_ATTRIB\_FLAG\_COLOR);

 //glVertexAttribPointer( GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride,const GLvoid * pointer):加载顶点数据
 //index:顶点属性的索引值
 //size:每个顶点属性的组件数量。必须为1、2、3或者4。初始值为4。（如position是由3个（x,y,z）组成，而颜色是4个（r,g,b,a））
 //type:顶点的数据类型
 //normalized:是否归一化
 //stride:顶点属性之间的偏移量
 //pointer:指定第一个组件在数组的第一个顶点属性中的偏移量
 glVertexAttribPointer(GLProgram::VERTEX\_ATTRIB\_POSITION, 2, GL\_FLOAT, GL\_FALSE, 0, vertercies);
 glVertexAttribPointer(GLProgram::VERTEX\_ATTRIB\_COLOR, 4, GL\_FLOAT, GL\_FALSE, 0, color);

 //glDrawArrays (GLenum mode, GLint first, GLsizei count):绘制三角形
 //mode:GL\_POINTS、GL\_LINES、GL\_LINE\_LOOP、GL\_LINE\_STRIP、GL\_TRIANGLES、GL\_TRIANGLE\_STRIP、GL\_TRIANGLE_FAN
 //first: 9
 //count: 3
 glDrawArrays(GL_TRIANGLES, 0, 3);
 //添加draw batches和顶点数目
 CC\_INCREMENT\_GL\_DRAWN\_BATCHES\_AND\_VERTICES(1, 3);
 //检测是否产生错误，如果有打印错误信息
 CHECK\_GL\_ERROR_DEBUG();
}

初始化：

bool Triangle::init()
{
	bool ret = false;
	do
	{
		CC\_BREAK\_IF(false == TestCase::init());

		char vShaderStr\[\] =
			"attribute vec4 a_position;					\\n"
			"attribute vec4 a_color;					\\n"
			"varying vec4 v_fragmentColor;				\\n"
			"void main()								\\n"
			"{											\\n"
			"   gl\_Position = CC\_MVPMatrix * a_position;\\n"
			"	v\_fragmentColor = a\_color;				\\n"
			"}											\\n";

		char fShaderStr\[\] =
			"varying vec4 v_fragmentColor;				\\n"
			"void main()                                \\n"
			"{                                          \\n"
			"   gl\_FragColor = v\_fragmentColor;			\\n"
			"}                                          \\n";

		// init shader
		//auto glProgram = GLProgram::createWithFilenames("Shaders/vert.vsh", "Shaders/frag.fsh");
		auto glProgram = GLProgram::createWithByteArrays(vShaderStr, fShaderStr);
		auto state = GLProgramState::getOrCreateWithGLProgram(glProgram);
		setGLProgramState(state);

		ret = true;
	} while (0);

	return ret;
}

通过上面的代码及设置，就可以使用自定义的Shader了。在下一篇中对上面的代码进行一下分析。 示例程序开源托管在Github: [CocosShader](https://github.com/max-xue/MyProject_Cocos)
---
title: C++开发之---单例类的实现
categories:
  - C++
date: 2018-08-28 18:39:00
tags:
---

单倒模式是一种常见的设计模式，在cocos2d很多地方都使用到。下面贴出来我的一种方式。 Utils.h

#ifndef \_\_UTILS\_H_
#define \_\_UTILS\_H_


#define UTILS                       Utils::Instance()

class Utils  
{
private:
    //将构造与析构声明 为私有，防止外部创建对象，因为这个对象是单根类
    Utils();
    ~Utils();
public:
    static Utils*    Instance();
    
};

#endif

Utils.cpp

#include "Utils.h"

Utils::Utils()
{
}

Utils::~Utils()
{
}

Utils* Utils::Instance()
{
    static Utils instance;
    
    return &instance;
}

原理都是类似的，使用类的静态变量或全局的静态变量保存唯一实例。最重要一点：将构造与析构声明 为私有，防止外部创建对象，保持此类只有唯一一个实例  QQ群：239759131 cocos 技术交流 欢迎您
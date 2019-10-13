---
title: quick-3.5 打包加密的个人总结
categories:
- Cocos
url: 52.html
id: 52
date: 2016-09-01 18:25:28
tags:
---

对于quick-lua打包问题，在网上有很多文章介绍。可到了quick-lua 3.5 框架有很大的变化，以前的都不适用了。 最近从网上找了一些资料，总结记录下来 ，仅供个人参考 ###加密### quick-lua 3.5 1. quick2.2.6和quick 3.3使用lua 5.1.5 2. 2dx的方案是，64位iOS使用lua 5.1.5，其他的平台都使用luajit，所以如果编译lua代码，需要两套 3. quick 3.5可以看做是2dx 3.5之上的一个插件，所以在这方面和2dx的方案是一样的 因为luajit不支持64位，iOS 64位上使用了lua。考虑到性能问题，其他所有的平台（包含iOS 32位），使用了luajit。这意味着如果想让一套lua脚本同时运行在iOS32位和64位设备上，那就不能使用lua字节码，lua和luajit生成的字节码是不兼容的。 首先，为什么要用luajit呢？我能想到的原因有两个，一是效率，二是加密。用luajit之后执行效率会有很大的提升，特别是在android下，另外呢luajit之后得到的文件是二进制的bytecode，基本无法反编译，因此对于商业产品来说，提高了被反解的成本。 那么luajit与lua到底有什么区别呢？从使用上来说没有区别，因为头文件都是一致的，所以将lua换成luajit来说，有两个步骤，一个是将头文件换成luajit的，另外一个是将动态链接库或者静态链接库也换成luajit的 加密命令 1.windows,android ios 确保不编译代码，只加密不编译，安全性未知 cocos luacompile -s src/ -d out/src/ -e -k key123 -b sign123 --disable-compile 2.未验证通过（无法实现） WIN: compile\_scripts.bat -i D:\\MyGame\\src -o D:\\MyGame\\res\\game.zip  -e xxtea\_zip -es key123 -ek sign123 MAC: ./compile\_scripts.sh -i /Developer/MyGame/src -o /Developer/MyGame/res/game.zip  -e xxtea\_zip -es key123 -ek sign123 AppDelegate.cpp 修改 stack->loadChunksFromZIP("res/game.zip"); stack->executeString("require 'main'"); 解决办法：待解决 1.修改quick-3.5框架，只使用lua不使用luajit,然后使用quick-3.3的加密工具加密 （牺牲性能，较可行） 2.更新quick-3.5 luajit库 使用2.1版本支持ios 64位（目前只是alph版本） 3.升级到3.7支持ios 64 bytecode （这个行不通，已经验证） QQ群：239759131 技术交流群欢迎您
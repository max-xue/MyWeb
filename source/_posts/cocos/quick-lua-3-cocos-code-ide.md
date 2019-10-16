---
title: quick-lua 3.5 工程 cocos code ide 1.2.0 断点调试
categories:
- Cocos
url: 54.html
id: 54
date: 2016-09-01 18:26:22
tags:
---

安装环境，配置等请参考其他资料 1.复制PrebuiltRuntimeLua.apk 在quick-3.5/templates/ 下创建lua-template-runtime/runtime/android 从quck-3.3/quick/templates/lua-template-quick/runtime/android 下复制PrebuiltRuntimeLua.apk到quick-3.5刚创建的目录 2.配置cocos code ide ![](http://img.blog.csdn.net/20150914152711332?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 3.create 创建项目 WIN: cocos new MyGame -p com.maxx.mygame -l lua -d D:\\Developer\\Workspace MAC: cocos new MyGame -p com.maxx.mygame -l lua -d /Developer 4.导入工程 cocos code ide 右键导入cocos 工程 5.选中项目-选择调试配置-设置模拟器 ![](http://img.blog.csdn.net/20150914153101025?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 这样就可以断点调试了 QQ群：239759131 cocos 技术交流 欢迎您
---
title: 'Xcode 7 编译cocos2dx 报libz.dylib,libsqlite3.dylib无法打开解决办法'
categories: 
  - macOS
url: 68.html
id: 68
date: 2016-09-01 18:34:48
tags:
---

最近升级Xcode7,以前的项目出现两个库无法打开的问题 解决办法： 1.删除两个库 2.添加->Add Other...->command + Shitf + G ->输入  /usr/lib/ 在这个文件夹找到这两个库 原帖子： *For people who need dylib, follow from this [post](http://stackoverflow.com/questions/30815806/swift-2-ios-9-libz-dylib-not-found)

1.  Choose "Add other"
2.  Once in the file selection window do "CMD"+Shift+G (Go to folder) & type /usr/lib/
3.  From /user/lib you can find the *.dylib files

解决iOS模拟器无法选择的问题 ![](http://img.blog.csdn.net/20160505113336975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) QQ群：239759131 cocos 技术交流 欢迎您
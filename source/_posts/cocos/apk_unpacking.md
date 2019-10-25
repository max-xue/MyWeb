---
title: Android 破解APK包
date: 2019-10-24 20:10:00
categories:
  - Cocos
comments: true
tags:
---

## APK解包

1. 准备工具软件  
  - apktool 解压APK
  - dex2jar class 转为Jar
  - jd-gui: 查看Jar代码

2. 解压指定APK  
  ~~~
  apktool d target.apk 
  ~~~

3. 将classes.dex打包成jar包

  ~~~
  d2j-dex2jar.bat classes.dex 
  ~~~
   
4. 使用jd-gui查看Jar   


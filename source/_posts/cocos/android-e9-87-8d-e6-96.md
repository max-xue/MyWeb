---
title: Android 重新签名打包
categories:
- Cocos
url: 1571.html
id: 1571
comments: true
date: 2018-11-12 22:23:09
tags:
---

1.安装JDK 2.cd 到\\Java\\jdk1.8.0_191\\bin 目录 

3. 生成签名文件

    keytool -genkey -alias androidauto.keystore -keyalg RSA -validity 20000 -keystore demo.keystore

4.解压apk,修改后重新zip压缩，注意是zip,然而改名为demo.apk 5.重新签名 123456签名文件的密码

    jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore demo.keystore -storepass 123456 demo.apk demo.keystore
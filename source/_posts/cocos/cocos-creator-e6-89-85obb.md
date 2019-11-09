---
title: Cocos Creator 打包OBB
categories:
  - Cocos
url: 1601.html
id: 1601
comments: true
date: 2019-09-23 21:29:52
tags:
---

1.app/build.gradle,打包APK排除资源，注释复制资源
 
    variant.mergeAssets.doLast {
    //        copy {
    //           from "${buildDir}/../../../../../res"
    //           into "${buildDir}/intermediates/assets/${variant.dirName}/res"
    ////            exclude "**/resources/gb/res/audio_down"
    //        }

2.build 生成apk

3.生成OBB

    jsb-link
       \- res
       \- src
       \- main
       \- project


script目录:C:\\CocosCreator\\resources\\cocos2d-x\\cocos\\scripting\\js-bindings\\script 

4.压缩成zip,改成obb

5.安装测试：
        
    C:\\Users\\xuegang-pc\\AppData\\Local\\Android\\Sdk\\platform-tools>adb shell
     cd storage/emulated/0/Android/obb
     mkdir com.maxx.test
     adb push d:\\main.1.com.maxx.test.obb /storage/emulated/0/Android/obb/com.maxx.test
         
 - 先安装APK 
 - 在手机U盘中创建Android/obb/com.maxx.test 目录,复制main.1.com.maxx.test.obb文件
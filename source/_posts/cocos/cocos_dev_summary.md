---
title: Cocos Creator 开发问题汇总
date: 2019-10-25 16:31:13
categories:
  - Cocos
comments: true
tags:
---

## 总结开发中遇到的问题

1. Manifest merger failed with multiple errors, see logs  
  在Android Studios 命令中执行（追踪错误详情）：  
  ./gradlew processDebugManifest --stacktrace
   
2. Android Apk ICON尺寸  
 
  |  密度   | 尺寸  |
  |  ----  | ----  |
  | mipmap-mdpi  | 48 * 48 |
  | mipmap-hdpi  | 72 * 72 |
  | mipmap-mdpi  | 48 * 48 |
  | mipmap-xhdpi  | 96 * 96 |
  | mipmap-xxhdpi  | 144 * 144 |
  | mipmap-xxxhdpi  | 192 * 192 |
   
3. Cocos Creator插件，查看运行场景中节点  
  [ccInspector_v1.1](https://github.com/tidys/CocosCreatorPlugins)
---
title: Unity开发之类库封装与调用
categories:
  - U3D
url: 1186.html
id: 1186
comments: true
date: 2018-01-04 09:38:55
tags:
  - u3d
---

软件项目随着规模的增大，难免需要分模块，分类库。同时也便于积累，将和项目无关的功能封装起来，方便以后用于其他项目。

下面记录一下Unity中封装自定义的类库。 使用VS或MonoDevelop创建类库项目： 

1.引用UnityEngine.dll类库（Mac下目录Applications/Unity.app/Contents/Frameworks/Managed/UnityEngine.dll Windows位置Program Files\\Unity\\Editor\\Data\\Managed\\UnityEngine.dll，版本不一样位置可能不同）； 

2.设置项目属性：Target framework,Mono版本的C#最高支持3.5；
 
 3.编写功能类； 
 
 4.设置Build Event,编译完成自动拷贝文件Post-build event command line: copy "$(TargetDir)$(TargetFileName)"   "..\\..\\..\\..\\..\\ProjectName\\Assets\\Plugins\\xxx\\$(TargetFileName)" ![](http://www.le-more.com/wp-content/uploads/2018/01/unity_dll_001.png) ![](http://www.le-more.com/wp-content/uploads/2018/01/unity_dll_002.png)
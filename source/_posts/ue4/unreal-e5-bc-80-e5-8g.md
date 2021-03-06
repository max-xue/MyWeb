---
title: Unreal 开发之 --- 问题汇总
categories:
  - UE4
url: 86.html
id: 86
date: 2016-09-01 18:43:51
tags:
  - ue4
  - 总结
---

总结在开发和学习中遇到的常见错误及解决方法 1\. Epic Launcher 登录不成功 ![](http://img.blog.csdn.net/20160523162855947?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 首先排除在任务管理器中重复进程，有时候启动两个进程导致始终登录不了 其实为添加Epic Game Launcher 目标位置添加 -http=wininet 这个方法不一定有效....但可以一试 ![](http://img.blog.csdn.net/20160523163208776?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 2.'class' type redefinition 类重复定义 确认只定义一次类名，却老是报定义重复，问题在于 头文件中的宏 #pragma once 因unreal 工程示例中没有#ifndef /#define/#endif 宏，故为了省事也为了保持一致，才出现这样的头文件重复引用的错误。 3.debug 游戏项目却打开了引擎 ![](http://img.blog.csdn.net/20160512153129340?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 需选择指定目标，默认是DebugGame Editer 4.GlobalShaderCache-PCD3D_SM5.bin is missing ![](http://img.blog.csdn.net/20160516170942694?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 需要在Unreal 编辑文件菜单中，刷新并构建项目 5.UE 4.11源码编译报错 'bAttributeLessDraw': undeclared identifierUE4 解决办法：第2793行末尾有一个”（中文右双引号），替换为英文双引号并保存即可 6.VS2015 Build工程失败 错误描述（类似）：

> '/IC:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\VC\\INCLUDE': command line argument number 279 does not match precompiled header

解决办法1（直到VS update 2为止, build 4.12.5之前的版本都没有问题. 安装VS update 3以后, 只能build 4.12.5以及之后的版本.）：

*   Completely uninstall visual studio
*   Use the Extra visual studio uninstall cleaner. Found here: [https://github.com/Microsoft/VisualStudioUninstaller/releases](https://github.com/Microsoft/VisualStudioUninstaller/releases)
*   Delete the remaining Visual Studio installation folder.
*   Do a restart and this machine should be ready for a clean installation
*   Download the Visual Studio 2015 installer without any updates: [https://www.microsoft.com/en-us/download/details.aspx?id=48146](https://www.microsoft.com/en-us/download/details.aspx?id=48146)
*   Now make sure you do a custom installation and deselect update 3!!!
*   Once the installation is finished you need to generate visual studio project files for your unreal projects.解决办法2： UE4 to 4.12.5 版本修复了这个问题

7.UE4编辑器 虚拟现实预览 是灰色的 VR Preview grey 问题：使用UE4开发 HTC VIVE时，使用UE4 4.12版本可以使用VR Preview，但4.13及以后的版本不可以选择 解决办法：启动Steam->库>SteamVR->右键属性->打开本地文件的选项卡->点开验证工具缓存的完整性 (如果还无效，删除SteamVR重新安装)

*   Stereo Panoramic Movie Capture export black png： 错误信息：LogD3D11RHI: Timed out while waiting for GPU to catch up. (0.5 s) 解决办法：换好的显卡就OK 参考： [Stereo Panoramic plugin in Standalone causes dark shadows](https://answers.unrealengine.com/questions/581202/stereo-panoramic-plugin-in-standalone-causses-dark.html) [](https://www.unrealengine.com/zh-CN/blog/capturing-stereoscopic-360-screenshots-videos-movies-unreal-engine-4) [从虚幻 4 中采集 360 度立体电影](https://www.unrealengine.com/zh-CN/blog/capturing-stereoscopic-360-screenshots-videos-movies-unreal-engine-4)

[4.11 preview 7 Stereo Panoramic export works!](https://forums.unrealengine.com/showthread.php?103980-4-11-preview-7-Stereo-Panoramic-export-works!)

*   SeamVR 全屏 [UE4 VR 模式下全屏解决办法](http://www.manew.com/thread-92928-1-1.html)
*   制作天空盒示例
    
    [Skybox from DDS cubemap](https://wiki.unrealengine.com/Skybox_from_DDS_cubemap)
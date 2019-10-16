---
title: iOS/macOS X使用问题汇总
categories:
  - macOS
url: 800.html
id: 800
comments: true
date: 2017-03-19 10:39:43
tags:
---

*   系统清理
    
    1.清理不用的设备备份： /Users/Max/Library/Application\ Support/MobileSync/Backup 2.删除xCode不用的存档，日志等，小心删除个人的配置信息 /Users/Max/Library/Developer/Xcode
    
*   读写NTFS外接硬盘 Tuxera NTFS 软件 自动启动 （macOS 10.12下不可用） Mounty 软件 手动启动
*   “xxx”已被 macOS 使用，不能打开 ls -l 查看到文件的属性 xattr -c *.*  对目录下所有文件清除附加属性
*   "Project Name" has conflicting provisioning settings. Code Signing Identity  修改设置统一
*   'UnityDefaultViewController should be used only if unity is set to autorotate' UnityViewControllerBaseiOS->supportedInterfaceOrientations函数注销
    
    NSAssert 行
    
*   "Image not found"  WikitudeNativeSDK 打包iOS时，报这个错误原因有： 1.如果是试用版本，不能修改Bundle Identifier(com.wikitude.unityexample) 2.将WikitudeNativeSDK.framework 加入到Embedded Binaries ![](http://www.le-more.com/wp-content/uploads/2017/03/WikitudeNativeSDK.png)
*   设置安全中显示不明来源选项
    
    sudo spctl --master-disable   显示
    
    sudo spctl --master-disable 关闭
*   显示/隐藏文件夹 文件夹：defaults write com.apple.finder AppleShowAllFiles -boolean true ; killall Finder， 隐藏文件夹：defaults write com.apple.finder AppleShowAllFiles -boolean false ; killall Finder
*   SVN macOS:Cornerstone
*   macOS 运行未知来源程序，报错：程序已经损坏 执行下面命令： sudo spctl --master-disable
*   访问局域网共享文件夹 前往服务器：smb://192.168.0.10
*   android-ndk-r10e-darwin-x86\_64.bin 解压 更改权限，然后解压 chmod a+x android-ndk-r10e-darwin-x86\_64.bin ./android-ndk-r10e-darwin-x86_64.bin
*   vim常用命令：
    
    :w 保存文件但不退出vi
    :w file 将修改另外保存到file中，不退出vi
    :w! 强制保存，不推出vi
    :wq 保存文件并退出vi
    :wq! 强制保存文件，并退出vi
    q: 不保存文件，退出vi
    :q! 不保存文件，强制退出vi
    :e! 放弃所有修改，从上次保存文件开始再编辑
    
*   Mac 生成Public SSH 1. ssh-keygen -t rsa -b 4096 #生成 2. pbcopy < ~/.ssh/id_rsa.pub #复制到剪切板
*   无法选择模拟器：Build Settings->Supported Platforms:iphonsos ->iOS
*   'Xcode 7 编译cocos2dx 报libz.dylib,libsqlite3.dylib无法打开解决办法'
    1.  Choose "Add other"
    2.  Once in the file selection window do "CMD"+Shift+G (Go to folder) & type /usr/lib/
    3.  From /user/lib you can find the *.dylib files

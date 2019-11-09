---
title: CocoaPods 使用总结
categories:
  - macOS
url: 568.html
id: 568
date: 2017-01-09 11:25:42
tags:
---

解决源问题： 国内的镜像：[https://gems.ruby-china.org/](https://gems.ruby-china.org/) 安装

sudo gem install -n /usr/local/bin cocoapods

升级更新

    sudo gem update –system

如果上一步提示无法获取，需要更新源：

    gem sources –remove https://rubygems.org/
    gem sources -a https://gems.ruby-china.com

再执行

    $ sudo gem update --system
    $ sudo gem install cocoapods -n/usr/local/bin

初始化第三方库信息

    pod setup

更新第三方库信息

    pod repo update

解析Podfile，安装

    pod install

解析Podfile，升级

    pod update

搜索

    pod search chartboost

_**一般步骤：**_ 1.初始化

    pod init

2.打开Podfile添加源 3.安装源

    pod install

一般通过这三步就OK了！ 问题:

*执行pod install后显示:Setting up CocoaPods master repo卡住不动
    
    cd ~/.cocoapods/repos
    git clone https://github.com/CocoaPods/Specs
    open .

然后把Specs目录改名为master(未验正通过)

*Setting up CocoaPods master repo卡住不动 使用：du -sh *查看目录（~/.cocoapods/repos）是否有变化，我的问题实际上是变化的只是下载的时间有点长

参考：[一遍成功安装"Cocoapods"](http://blog.csdn.net/u014455765/article/details/51814488)
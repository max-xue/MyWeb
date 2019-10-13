---
title: ubuntu 使用汇总
categories:
  - Server
url: 41.html
id: 41
date: 2016-09-01 18:09:27
tags:
---

一、系统安装：

1.  http://cn.ubuntu.com/ 下载
2.  usb 安装系统工具 http://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/
3.  移动硬盘安装，选择靠前的分区
4.  创建 主分区      20480M ext3日志文件 挂载点/
5.  创建 逻辑分区 2048M 交换分区
6.  开始安装

三.VWare Ubuntu指定IP地址

1.  编辑->虚拟网络编辑器 ![](http://www.le-more.com/wp-content/uploads/2016/09/ubuntu_network_00.jpg)
2.  虚拟机->设置->网络适配器 ![](http://www.le-more.com/wp-content/uploads/2016/09/ubuntu_network_01.jpg)
3.  Ubuntu中的设置 ![](http://www.le-more.com/wp-content/uploads/2016/09/ubuntu_network_02.jpg)

  二、其他

1.  创建目录：sudo mkdir dirname
2.  默认用户下是没有文件操作权限的，使用：sudo nautilus 命令打开一个具有文件访问权限的窗口
3.  CodeBlocks 安装：[http://www.d2school.com/codeblocks/doc/codeblocks_setup.html](http://www.d2school.com/codeblocks/doc/codeblocks_setup.html)
4.  挂载U盘：1.fdisk -l    查看所有的已经挂载的磁盘
5.  mount -t vfat -o iocharset=cp936 /dev/sdb1 /mnt/usb   挂载
6.  umount /dev/sdb1      卸载
7.  除了用使用U盘外，在虚拟机下也可以使用共享文件夹，在虚拟机设置Shared Forlders
8.  查看网络地址：ifconfig -a
9.  vim常用命令：
    
    :w 保存文件但不退出vi
    :w file 将修改另外保存到file中，不退出vi
    :w! 强制保存，不推出vi
    :wq 保存文件并退出vi
    :wq! 强制保存文件，并退出vi
    q: 不保存文件，退出vi
    :q! 不保存文件，强制退出vi
    :e! 放弃所有修改，从上次保存文件开始再编辑
    
10.  删除 文件 rm -f /etc/shadowsocks/.config.json.swp 删除文件夹 rm -rf  /etc/shadowsocks
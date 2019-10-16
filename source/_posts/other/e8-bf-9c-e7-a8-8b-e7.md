---
title: Win10 解决无法多用户远程登录问题
categories:
  - Other
date: 2016-09-01 18:48:59
tags:
---

具体怎么解决win10 多用户登录请查看此文章：http://www.setsea.net/wordpress/os/2015/08/11/1636.html 使用补丁的方法，如果安装360，可以使用右键360强力删除，然后替换dll。 奇怪问题：执行install.bat 报 OpenKeyReadonly error code 0 查到问题：注册表中无termservice，这个注册表项无故被删除了，导致远程始终失败。 解决问题：从其他电脑从注册表中导出termservice节点后，导入。重启后问题解决！
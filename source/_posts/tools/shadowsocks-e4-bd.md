---
title: Shadowsocks 使用总结
categories:
  - Tools
url: 1247.html
id: 1247
comments: true
date: 2018-05-08 11:01:41
tags:
---

1.  VPS服务器购买： [VIRMACH 官网](https://billing.virmach.com/cart.php?gid=1) （我选择最便宜的一款，支付宝支付，服务器需要人工审核，大概几个小时通过） ![](http://www.le-more.com/wp-content/uploads/2018/05/vps_VRMACH.png)
2.  重新安装系统：Ubuntu 12.04 x86
3.  安装配置服务端：[Shadowsocks-go一键安装脚本 ](https://teddysun.com/392.html) ![](http://www.le-more.com/wp-content/uploads/2018/05/vps_shadowsocks_server.png)
4.  客户端配置 从 [shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows)下载windwos客户端，配置如下： ![](http://www.le-more.com/wp-content/uploads/2018/05/vps_shadowsocks_client.png)
5.  下载安装SwitchyOmega Chrome插件,添加情景shadow_sockets 代理协议SOCKET5 代理服务器：127.0.0.1 代理端口：1080 ![](http://www.le-more.com/wp-content/uploads/2018/05/vps_shadowsocks_SwitchyOmega.png)
6.  在chrome浏览器右上角选择SwitchyOmega的场景shadow_sockets，开始新世界的旅程吧！
7.  iOS手机下载Shadowrocket,如果不想花钱就下载PP助手
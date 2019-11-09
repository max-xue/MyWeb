---
title: 服务器开发之---Redis
categories:
  - Server
url: 1436.html
id: 1436
comments: true
date: 2018-08-04 12:27:01
tags:
---

[Redis 教程](http://www.runoob.com/redis/redis-tutorial.html)

##### 1.安装（Mac)

 - 官网[http://redis.io/](http://redis.io/) 下载最新的稳定版本,这里是3.2.0

 - sudo mv 到 /usr/local/

 - sudo tar -zxf redis-3.2.0.tar 解压文件

 - 进入解压后的目录 cd redis-3.2.0

 - sudo make test 测试编译

 - sudo make install


win:https://github.com/ServiceStack/redis-windows

##### 2.启动连接

 - 启动服务：redis-server
   启动服务：redis-server ./redis.conf     #加载配置

 - 连接： **redis-cli -h 127.0.0.1 -p 6379** 
 - 设置键值对： **set myKey abc**
 - 取出键值对： **get myKey**

 - 关闭服务: redis-cli shutdown
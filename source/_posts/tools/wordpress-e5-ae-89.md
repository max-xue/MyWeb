---
title: 'Wordpress 安装配置 win10,win2003'
categories:
  - Tools
url: 98.html
id: 98
date: 2016-09-01 18:47:32
tags:
---

其中部分参考网上教程，现将过程记录一下，文中记录都是在win10下面安装的，如果在win2003需要下载对应的版本软件（Appache2.2,mysql 5.5,php5.4,phpadmin4.0） WordPress是一种使用PHP语言开发的博客平台，用户可以在支持PHP和MySQL数据库的服务器上架设属于自己的网站。也可以把 WordPress当作一个内容管理系统（CMS）来使用。 安装目录：D:\\wordpress 环境准备

1.  mysql
2.  php
3.  appach

#### 1\. MySql 环境配置

下载地址：http://dev.mysql.com/downloads/mysql/

下载版本：Windows (x86, 32-bit), ZIP Archive

环境配置：

1.  路径：D:\\wordpress\\MySql
2.  添加系统环境变量Path：D:\\wordpress\\MySql\\bin
3.  复制目录下的my-default.ini创建my.ini
4.  修改basedir = D:\\wordpress\\MySql       datadir = D:\\wordpress\\MySql\\data
5.  以管理员身份运行cmd,进入D:\\wordpress\\MySql\\bin目录，执行：mysql install 安装mysql
6.  执行mysqld --initialize-insecure --user=mysql 初始化，根目录会创建一个data目录
7.  执行net start mysql 启动服务
8.  执行mysql -u root -p 登录
9.  修改密码：SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
10.  创建数据库：CREATE DATABASE wordpress
11.  分配权限：GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost IDENTIFIED BY 'pass';
12.  SET PASSWORD FOR 'wordpress'@'localhost' = OLD_PASSWORD('密码');
13.  5.5版本配置：http://blog.csdn.net/u013628152/article/details/51229136

#### 2.PHP环境配置

下载地址：http://windows.php.net/download/

下载版本：下载线程安全版本（VC11 x86 Thread Safe）

环境配置：

1.  解压目录:D:\\wordpress\\php5.6
2.  复制php.ini-production 创建php.ini
3.  设置环境变量D:\\wordpress\\php5.6 和D:\\wordpress\\php5.6\\ext
4.  常用：extension_dir = "ext"
5.  常用：extension=php_mbstring.dll
6.  常用：extension=php_mysql.dll
7.  常用：extension=php_mysqli.dll

#### 3.Apache 环境配置

下载地址：http://www.apachelounge.com/download/

下载 版本：httpd-2.4.23-win32-VC11.zip

环境配置：

1.  解压路径：D:\\wordpress\\Apache24
2.  打开conf目录下的httpd.conf
3.  修改ServerRoot DocumentRoot Directory  ScriptAlias 目录更新为：D:\\wordpress\\Apache24
4.  #ServerName www.example.com:80 去掉#
5.  DirectoryIndex index.html 修改为DirectoryIndex index.html index.php index.htm
6.  添加 LoadModule php5\_module "D:/wordpress/php5.6/php5apache2\_4.dll"
7.  添加 AddType application/x-httpd-php .php .html .htm
8.  添加 PHPIniDir "D:/wordpress/php5.6"

#### 4.Wordpress 环境配置

下载地址：http://cn.wordpress.org

下载版本：wordpress-4.5.3-zh_CN

环境配置：

1.  解压到d:\\wordpress\\Apache24\\htdocs\\wordpress
2.  localhost\\wordpress
3.  配置数据库名，数据库用户名，密码及表名前缀（前提是创建好数据库，设置好数据库用户密码等）
4.  成功就进入5分钟安装过程

其他：

phpadmin工具

http://www.phpmyadmin.net/

使用问题汇总：

1.在线下载，升级失败

修改php.ini

extension=php_curl.dll

max\_execution\_time = 60 post\_max\_size = 8M
---
title: 服务器开发之---MariaDB(MySql)
categories:
  - Server
url: 1069.html
id: 1069
comments: true
date: 2018-07-29 10:46:09
tags:
---

### 安装

    Max-MacBook-Pro:~ Max$ brew install mariadb
    ==\> Downloading https://homebrew.bintray.com/bottles/mariadb-10.2.7_1.sierra.bot
    ######################################################################## 100.0%
    ==\> Pouring mariadb-10.2.7_1.sierra.bottle.tar.gz
    ==\> Using the sandbox
    ==\> /usr/local/Cellar/mariadb/10.2.7\_1/bin/mysql\_install_db --verbose --user=Max
    ==\> Caveats
    A "/etc/my.cnf" from another install may interfere with a Homebrew-built
    server starting up correctly.
    
    MySQL is configured to only allow connections from localhost by default
    
    To connect:
        mysql -uroot
    
    To have launchd start mariadb now and restart at login:
      brew services start mariadb
    Or, if you don't want/need a background service you can just run:
      mysql.server start
    ==\> Summary
    🍺  /usr/local/Cellar/mariadb/10.2.7_1: 623 files, 162.2MB

### 手动启动

    Max-MacBook-Pro:~ Max$ mysql.server start
    Starting MySQL
    .170729 10:12:17 mysqld_safe Logging to '/usr/local/var/mysql/Max-MacBook-Pro.local.err'.
    170729 10:12:17 mysqld_safe Starting mysqld daemon with databases from /usr/local/var/mysql
     SUCCESS!

### 连接数据库

    Max-MacBook-Pro:~ Max$ mysql -uroot
    Welcome to the MariaDB monitor.  Commands end with ; or \\g.
    Your MariaDB connection id is 9
    Server version: 10.2.7-MariaDB Homebrew
    
    Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.

### 执行命令：

    MariaDB \[(none)\]> show databases
        -\> ;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | test               |
    +--------------------+
    4 rows in set (0.01 sec)

### 退出

    MariaDB \[(none)\]> exit
    Bye

[navicat for mac 完美破解版](http://download.csdn.net/download/lpluck08/5122765) 解决远程连接

    GRANT   ALL   PRIVILEGES   ON   *.*   TO   'root'@'%'   WITH   GRANT   OPTION ;
    
    FLUSH   PRIVILEGES;

### 问题汇总:
  - Host 'xxx' is not allowed to connect to this MySQL server.（1251 client does not support） 
    修改mysql库中user表User字段为root的Host为%；
    同时修改加密方式为：mysql\_native\_password，具体步骤： 
    
    1）打开CMD切到C:\\Program Files\\MySQL\\MySQL Server 8.0\\bin 目录依实际而定
    
    2）mysql -u root -p 
    
    3)  输入密码 
    
    4）修改 host:  update host = '%' from user where user = 'root' 
    
    5)  修改加密方式： 
    
        #更改加密方式并更新一下用户的密码  
        ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root的密码' PASSWORD EXPIRE NEVER; 
        ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql\_native\_password BY 'Root的密码'; 
        #刷新权限 FLUSH PRIVILEGES; 
        
        
  待续...
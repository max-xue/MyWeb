---
title: Redmine 安装 win10系统
categories:
  - 工具
url: 90.html
id: 90
date: 2016-09-01 18:45:16
tags:
---

官网安装指南：http://www.redmine.org/projects/redmine/wiki/RedmineInstall 安装环境：win10 64位 MySQL:http://dev.mysql.com/downloads/windows/installer/ （安装时版本：5.7.13） railsinstaller:http://railsinstaller.org/en （安装时版本：3.2.0） 下载安装MySQL 安装railsinstaller （注意：不能有空格） Step 1 - Redmine application[¶](http://www.redmine.org/projects/redmine/wiki/RedmineInstall#Step-1-Redmine-application) Get the Redmine source code by either downloading a packaged release or checking out the code repository. Step 2 - Create an empty database and accompanying user Redmine database user will be named `redmine` hereafter but it can be changed to anything else. MySQL

CREATE DATABASE redmine CHARACTER SET utf8;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';

###      Step 3 - Database connection configuration

Copy `config/database.yml.example` to `config/database.yml` and edit this file in order to configure your database settings for "production" environment. Example for a MySQL database using ruby 1.8 or jruby:

production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: my_password

Step 4 - Dependencies installation Redmine uses [Bundler](http://gembundler.com/) to manage gems dependencies. You need to install Bundler first:

gem install bundler

Then you can install all the gems required by Redmine using the following command:

bundle install --without development test rmagick

gem install mysql2 -- '--with-mysql-dir="C:\\Program Files\\MySQL\\MySQL Server 5.6\\mysql-connector-c-6.1.1-win32"'

Step 5 - Session store secret generation This step generates a random key used by Rails to encode cookies storing session data thus preventing their tampering. Generating a new secret token invalidates all existing sessions after restart.

*   with Redmine 2.x:

bundle exec rake generate\_secret\_token

Step 6 - Database schema objects creation Create the database structure, by running the following command under the application root directory: Windows syntax:

set RAILS_ENV=production
bundle exec rake db:migrate

###      Step 7 - Database default data set

Windows:

set RAILS_ENV=production
set REDMINE_LANG=zh
bundle exec rake redmine:load\_default\_data

###      Step 9 - Test the installation[¶](http://www.redmine.org/projects/redmine/wiki/RedmineInstall#Step-9-Test-the-installation)

Test the installation by running WEBrick web server:

*   with Redmine 3.x:

bundle exec rails server webrick -e production

Once WEBrick has started, point your browser to [http://localhost:3000/](http://localhost:3000/). You should now see the application welcome page. 其他IP也可访问

bundle exec rails server webrick -e production -b 0.0.0.0

###      Step 10 - Logging into the application

Use default administrator account to log in:

*   login: admin
*   password: admin

You can go to Administration menu and choose Settings to modify most of the application settings. 其他参考官方文档 参考链接：http://www.51testing.com/html/29/595529-853889.html
---
title: 使用Hexo搭建网站个人总结
date: 2019-10-11 11:58:48
categories:
top: 18
tags:
- hexo
- guide
---

## 使用Hexo搭建网站个人总结

1.安装环境 
- [Git](http://nodejs.org/)  
- [Node.js](http://nodejs.org/)
(因平时开发环境用这两个环境，安装过程省略)

2.[安装Hexo](https://hexo.io/zh-cn/docs/)

    $:npm install -g hexo-cli


3.创建Web项目
创建一个空的文件夹MyWeb，右键打开Git Bash Here(成功安装Git后，右键菜单有这个选项)

    $ hexo init MyWeb
    $ cd MyWeb
    $ npm install
 

执行后目录如下：

    .
    ├── _config.yml
    ├── package.json
    ├── scaffolds
    ├── source
    |   ├── _drafts
    |   └── _posts
    └── themes

安装其他依赖(根据需要) 

    $ npm install hexo-wordcount --save
    $ npm install hexo-generator-archive --save
    $ npm install hexo-generator-category --save
    $ npm install hexo-generator-feed --save
    $ npm install hexo-generator-index --save
    $ npm install hexo-generator-json-content --save
    $ npm install hexo-generator-search --save
    $ npm install hexo-generator-sitemap --save
    $ npm install hexo-generator-tag --save
    $ npm install hexo-generator-topindex --save
    $ npm install hexo-renderer-ejs --save
    $ npm install hexo-renderer-marked --save
    $ npm install hexo-renderer-stylus --save
    $ npm install hexo-server --save
    $ npm install hexo-wordcount --save

4.[复制主题](https://github.com/yelog/hexo-theme-3-hexo)

以我使用的3-Hexo为例


    git clone https://github.com/yelog/hexo-theme-3-hexo.git themes/3-hexo


5.配置主题

修改MyWeb根目录下文件_config.yml，如下


    theme: 3-hexo


[3-hexo主题详细配置参考](https://yelog.org/2017/03/23/3-hexo-instruction/)

6.创建页面


    hexo new page --path other/0001_hexo_guide


[编写内容参考Markdown语法](https://www.runoob.com/markdown/md-link.html)

7.生成预览
 
 
     $hexo clean && hexo g && hexo s
 

打开http://localhost:4000可预览效果

 8.发布到Git
 [使用Github创建项目](https://pages.github.com/),因很久以前就创建过，过程略,[我的地址](https://github.com/max-xue/max-xue.github.io)
 
 修改MyWeb根目录下文件_config.yml，如下
  
     # Deployment
     ## Docs: https://hexo.io/docs/deployment.html
     deploy:
       type: git
       repo: https://github.com/max-xue/max-xue.github.io
       branch: master
       message: "Site updateed: {{ now('YYYY-MM-DD HH:mm:ss') }}"
 

执行

    $hexo clean && hexo g && hexo d

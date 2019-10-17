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

### 安装环境 
[Git](http://nodejs.org/)

[Node.js](http://nodejs.org/)

(因平时开发环境用这两个环境，安装过程省略)

### [安装Hexo](https://hexo.io/zh-cn/docs/)

```xml
$:npm install -g hexo-cli
```

### 创建Web项目
创建一个空的文件夹MyWeb，右键打开Git Bash Here(成功安装Git后，右键菜单有这个选项)
```xml
$ hexo init MyWeb
$ cd MyWeb
$ npm install
```

执行后目录如下：
```xml
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

### [复制主题](https://github.com/yelog/hexo-theme-3-hexo)
以我使用的3-Hexo为例

```xml
git clone https://github.com/yelog/hexo-theme-3-hexo.git themes/3-hexo
```

### 配置主题

修改MyWeb根目录下文件_config.yml，如下

```xml
theme: 3-hexo
```

[3-hexo主题详细配置参考](https://yelog.org/2017/03/23/3-hexo-instruction/)

### 创建页面

```xml
hexo new page --path other/0001_hexo_guide
```

[编写内容参考Markdown语法](https://www.runoob.com/markdown/md-link.html)

 ### 生成预览
 
 ```xml
 $hexo g
 $hexo s
 ```

打开http://localhost:4000可预览效果

 ### 发布到Git
 [使用Github创建项目](https://pages.github.com/),因很久以前就创建过，过程略,[我的地址](https://github.com/max-xue/max-xue.github.io)
 
 修改MyWeb根目录下文件_config.yml，如下
  ```xml
 # Deployment
 ## Docs: https://hexo.io/docs/deployment.html
 deploy:
   type: git
   repo: https://github.com/max-xue/max-xue.github.io
   branch: master
   message: "Site updateed: {{ now('YYYY-MM-DD HH:mm:ss') }}"
  ```

执行
  ```xml
$hexo d
  ```
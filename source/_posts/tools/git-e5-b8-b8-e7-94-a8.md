---
title: Git 常用命令及使用总结
categories:
  - Tools
url: 428.html
id: 428
date: 2016-11-25 18:22:51
tags:
---

    git init

初始化一个版本库

    git clone

签出一个版本库

    git add

添加资源到本地版本库

    git commit

提交改动到本地版本库

    git pull

从服务端拉取，如果有改动合并到本地。将服务端同步到本地

    git push

将本地修改推送到服务端，将本地同步到服务端

工作流程：

1.开始 **git clone** 签出版本

2.工作 一天开始---》同步服务端到本地（**git pull** ）---》增删改项目（git add、git delete 本地操作）---》同步本地到服务端（**git push**）---》一天结束

这些是基本的git命令，实际工作中使用TortoiseGit 要方便一些。多人时会遇到冲突及手动合并的操作。

###### **.gitignore 文件的创建**

1.使用Git Bash Here 打开命令窗口

2\. 输入 vim .gitignore

3\. :wq 保存并退出。

###### **签出指定分支**

签出远程工程到本地目录

    git clone https://github.com/EpicGames/UnrealEngine.git
    
列出所有分支

    git branch -a
    
签出远程 origin/4.13分支到本地4.13分支,并切换到这个分支

    git checkout -b 4.13 origin/4.13
    
切换到4.13分支

    git checkout 4.13

###### 撤销指定提交

    //查看log
    git log
    
    //撤销到指定版本号
    git reset --soft f8be079b545a66fe3cecff5af87b126d6e8c9874

    //强制提交到服务端
    git push origin winsdk --force

###### 删除指定分支：
    
    #. update the references in our local machine 
    git fetch -p origin
    
    #.查看本地和远程分支
    git branch -a
    
    #. 先切换到其他分支
    git checkout master
    
    #.删除远程分支
    git push origin --delete dev_xue
    
    \# 删除后，再次查看分支情况
    git branch -a
    
    \# 删除本地分支
    git branch -d dev_xue
    
###### 关闭换行（解决meta文件频繁改支）：
    git config --global core.autocrlf false

问题：Please make sure you have the correct access rights
    
    //生成public key
    ssh-keygen -t rsa -C "xue\_huashan@163.com",//xue\_huashan@163.com是你git操作的账号。
    
    Generating public/private rsa key pair.
    Enter file in which to save the key (/c/Users/xuegang-pc/.ssh/id_rsa): //回车
    Enter passphrase (empty for no passphrase): //回车
    Enter same passphrase again: //回车
    Your identification has been saved in /c/Users/xuegang-pc/.ssh
    
    //显示内容
    cat /c/Users/xuegang-pc/.ssh/id_rsa.pub
    
    //将内容复制到Web帐户中的SSH Keys
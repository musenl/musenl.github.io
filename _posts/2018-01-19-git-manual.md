---
layout: post
title: git 更新操作
categories: git
description: git 提交更新操作
keywords: git github
---

入门 git 的简明手册

##  提交更新

更新部分文件时,提交更新:

- changed file or new file,the command 'git add' means add file to local git repositories  

  ``git add file``


- check is the file status is ok 

  ``git status``


- without commit command can not update the file,this is a 'must' 

  ``git commit -a -m "something"``

- 新建本地git仓库时,添加文件:(要先pull下来tar_url的文件到新建的文件夹,然后将要提交的文件复制到新建文件夹中

   ``git push -u origin master``

   *如果先把要提交的新文件放到新建的文件夹下再进行如下一串命令,将提示出错)*


- 在随便新建的一个文件夹下

​       ``git init``

- 该条命令执行后用新文件替换旧文件

​       ``git pull tar_url``

​       ``git status``

-   git add newfile 

​      ``git add .``

​      `` git commit -a -m "xxx"``

​      ``  git remote add origin tar_url``

​      ``git push -u origin master``
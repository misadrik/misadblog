---
title: 关于git常用命令的介绍
date: 2017-07-24 14:06:57
categories: 
- 学习笔记
tags: git
---

刚刚开始学python的时候接触了git，虽然现在没有做大项目，但git的功能真的是太强大了，简单记录下常用的几个命令。
<!-- more -->
# 初始化
```git init```
切换到相应文件夹后用此命令可以初始化一个git仓库
# 检查状态
`git status`
用于检测仓库中有哪些文件初修改了，增加或删除了哪些文件等
# 将文件加入仓库
`git add filename.xxx`
用于添加文件
另外还有几个更方便的命令
`git add -A`
提交文件夹下所有文件改动到暂存区
`git add .`
提交工作区变化到暂存区，包括新文件但不包括被删除文件
`git add -u`
提交被add文件改动到暂存区，不包括新文件，但包括被删除文件
# 执行提交
`git commit -m "message"`
提交修改，message中可以加入些改动说明
# 重命名文件
`git mv oldfilename.xx newfilename.xx`


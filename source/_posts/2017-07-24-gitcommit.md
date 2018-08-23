---
title: Git Commit的书写规范
date: 2017-07-24 21:35:50
categories: 
- 学习笔记
tags: git 
---
晚上搜资料发现原来git commit的message也有书写规范，学习下
<!-- more -->
比较常用的是Angular书写规范，如下
```
<type>(<scope>): <subject>

<body>

<footer>
```
由于我现在所写的程序比较简单，所以比较重要的是type和subject
1）type分为以下七个：

- feat：新功能（feature）

- fix：修补bug

- docs：文档（documentation）

- style： 格式（不影响代码运行的变动）

- refactor：重构（即不是新增功能，也不是修改bug的代码变动）

- test：增加测试

- chore：构建过程或辅助工具的变动

2）subject是commit目的简短描述

- 以动词开头，使用第一人称现在时，比如change，而不是changed或changes

- 第一个字母小写

- 结尾不加句号（.）


而scope是说明commit影响的范围
body是详细说明
footer则用于不兼容变动等情况时
以上三种等用到时再详加说明。
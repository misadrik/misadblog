---
title: Sublime3中调用cmd终端
date: 2017-07-23 10:32:45
categories: 
- 学习笔记
tags: [sublime,cmd,terminal]
---

最近在学习python，由于sublime的控制台不能支持输入，经常是一边敲代码一边复制到cmd终端，偶然发现可以配置sublime中的termianl插件支持直接调用终端，具体方法如下：
<!-- more -->
# 安装
直接在sublime中输入`ctrl+shift+p`，然后输入`install package`或`pcip`进入安装插件列表，然后输入`terminal`安装插件
# 配置
安装完成后，在sublime中打开`Preferences - Package Settings - Termial - Settings User`，然后在窗口中根据自己的终端路径输入如下代码
```
{ 
// window下终端路径 
"terminal": "C:\\Windows\\System32\\cmd.exe", 
// window下终端参数 
"parameters": ["/START", "%CWD%"] 
} 
```
如果是Linux系统则为
```
{
  "terminal": "xterm"
}
```
OS X:
```
{
  "terminal": "iTerm.sh"
}
```
由于本人系统是windows所以其他两种并未尝试，可自行试验。
# 运行
`ctrl+shif+T`就可以运行弹出终端了

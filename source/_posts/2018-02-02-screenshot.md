---
title: 截图拼接
date: 2018-02-02 11:05:18
categories: 
- 技术
- python
tags: [python, 工具]
copyright: true
comments: true
reward: true
---

# 概述

看剧时难免遇到精彩语录需要截图的，单张图不便保存，查看，而拼接的办法主要有两种  
* 美图秀秀    
* 或网上的各种工具，需要上传图片  
感到非常不方便，简单写了个程序可以拼接特定文件夹下的图片，至于图形界面的大坑就留给以后了   
<!-- more -->

# 程序

```python
import os
from PIL import Image

pic_path = './Screenshots/'
def pic_joint(SUBTITLE_COEF = 0.15,RESOLUTION = 100,Del = 1):
    # 读入并存储待截取图片名
    images = [] # 存储待截取图片名
    for root, dirs, files in os.walk(pic_path):
        for f in files:
            images.append(f) # 得到文件夹下所有图片名

    num = len(images)
    im = Image.open(pic_path + images[0])
    unit_width, unit_height = im.size #获得一张图片的像素大小
    subtitle_height = int(unit_height * SUBTITLE_COEF) # 得到字幕区高度
    target = Image.new('RGB',(unit_width,unit_height + (num - 1) * subtitle_height)) # 创建新的图片

    up = (num - 1) * subtitle_height #上边缘
    lower = unit_height + (num - 1) * subtitle_height # 下边缘

    images.reverse()# 从最后一张开始覆盖

    for image in images:
        im = Image.open(pic_path + image)
        target.paste(im,(0,up,unit_width,lower))
        lower = lower - subtitle_height
        up = up - subtitle_height
    
    # 删除已合并图片
    if Del == 1: 
        for image in images:   
            os.remove(pic_path + image)    

    target.save('./'+ images[0],quality = RESOLUTION) #要先创建result文件夹

if __name__ == '__main__':
    pic_joint()
```

本程序能合并**同目录**下screenshots文件夹下的截图，并将合并图片生成在**当前目录**下。默认所有截图**同尺寸**，字幕占有画面高度**0.15%**，并**删除**源图片。

--->当前文件夹

​	--->pic_joint.exe(本程序打包生成的exe文件)

​	--->screenshots(等合并的戴图)

***

注，打包成exe可使用pyinstaller，执行：

```python
pyinstaller -F -w pic_joint.py
```


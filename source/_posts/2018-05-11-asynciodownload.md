---
title: asyncio+aiohttp实现异步下载
date: 2018-05-11 20:16:54
categories:
- 技术
- python
tags: [python, asynchronous, asyncio]
copyright: true
comments: true
rewards: true
---

# 概述

这个程序是对多线程多进程下载图片的优化，使用了python自带的asyncio协程程aiohttp来实现异步IO操作。相比gevent我更喜欢用这个方式，因为更明了，直接，并且不用猴子补丁这样的奇怪东西。
![180511arakiyuko](https://wx4.sinaimg.cn/mw1024/711dab24gy1fr7rlwwgyvj20nc0fk0uh.jpg)

<!-- more -->

# aiohttp

**aiohttp**是requests的异步替代版，不能对requests.get()函数返回的对象使用**await**，因为requests库是基于同步 实现的，因此在asyncio中使用aiohttp能最大限度发辉异步IO的功能



#源代码

8.92351秒

```python
import csv
import time
from threading import Thread
import multiprocessing
import asyncio
import aiohttp

folder_path = 'D:/Python/WeiboCrawler/Download_pics/'
failed_urls = './failed_pics.txt'

def break_piont_continue():#读取url
    to_down = open('weibo.csv', 'r', encoding='utf-8', newline='')
    csv_data = csv.reader(to_down)

    download_urls, downloaded_urls = [], []
    count = 0
    for data in csv_data:
        if data[9] == '':
            pass
        elif len(data[9].split(',')):
            for num in range(len(data[9].split(','))):
                # print(num)
                download_urls.append(data[9].split(
                    ',')[num].strip('[').strip(']').strip(' ').strip('\''))
                print(str(count) + ' Done!')
                count = count + 1
        else:
            print('Error')

    return download_urls

'''下载图片程序'''
async def download_pics(url):
    async with aiohttp.ClientSession() as session:         
        async with session.get(url) as response:                
            pic = await response.read()    #以Bytes方式读入非文字                     
            with open (folder_path + url.split('/')[-1], 'wb') as fout:# 写入文件
                fout.write(pic)
                print('pic: ' + url.split('/')[-1] + ' done!')  

if __name__ == '__main__':
    download_urls = break_piont_continue()
    loop = asyncio.get_event_loop() #创建loop asyncio协程要在loop中运行
    start_time = time.time()  # 用于统计时间
    tasks = [download_pics(url) for url in download_urls]
    loop.run_until_complete(asyncio.gather(*tasks))
    loop.close()
    end_time = time.time()
    print(end_time - start_time)
```

<font size = 5> **相关文章:** </font>
<p>1. {% post_link 2018-04-25-downloadpics  多线程，多进程下微博图片 %}</p>
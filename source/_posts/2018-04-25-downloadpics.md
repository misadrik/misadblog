---
title: 多线程，多进程下微博图片
date: 2018-04-25 22:06:52
categories: 
- 技术
- 爬虫
tags: [多进程,多线程,技术]
copyright: true
comments: true
reward: true
---

# 概述

爬了微博后图片一直是用链接保存的，前阵子喜欢上了优子酱，喜欢着装风格，疯狂找图片，于是爬了她的粉丝微博，下了所有的图片。由于这个实现起来没有顺序，相对简单，下一步就要考虑用多线程和多进程对微博爬虫进行改造了，不过这个需要时间，等技术更加纯熟再弄可能会更好。
<!-- more -->

# 主要内容
## 下载函数urlretrieve
```python
import urllib.request
urlretrieve(url, filename=None, reporthook=None, data=None)
```
* 参数 **finename** 指定了保存本地路径（如果参数未指定，urllib会生成一个临时文件保存数据。）
* 参数**reporthook**是一个回调函数，当连接上服务器、以及相应的数据块传输完毕时会触发该回调，我们可以利用这个回调函数来显示当前的下载进度。
* 参数**data**指 post 到服务器的数据，该方法返回一个包含两个元素的(filename, headers)元组，filename 表示保存到本地的路径，header 表示服务器的响应头

## 多线程threading

### 概念

多线程是并行的一种。计算机只有一个CPU核心，同时只能处理一个任务。这样的情况下，多线程可以理解为开辟多条道路，但道路的出口只有一个。哪条路上的车抢到了通行权这条路上的汽车就会优先通过，直到下条路上的车抢到通行权，此时其他路上的汽车都会进入等待状态。使用多线程，可以大大的增加程序的性能和效率。

### 多线程的使用

在python3 中一般使用 threading 库

```python
import threading
t = Thread(target=function_name, args=(function_parameter1, function_parameterN)) # 启动刚刚创建的线程 
t.start()
```

* 参数**function_name**用于指定执行多线程的**函数名字**（只要名字）
* 参数**args**是前述函数的参数

此函数也可通过类的继承使用

```python
import threading
import datetime
import time

class my_Thread (threading.Thread):
    def __init__(self, threadID, name, delay):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.delay = delay
    def run(self):
        print ("开始线程：" + self.name)
        print_datetime(self.name, self.delay, 5)
        print ("退出线程：" + self.name)
        
def print_datetime(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print ("%s: %s" % (threadName, datetime.datetime.now()))
        counter -= 1
        
thread1 = my_Thread(1, "Thread-1", 1)
thread2 = my_Thread(2, "Thread-2", 2)

thread1.start()
thread2.start()
thread1.join()
thread2.join()
```

**输出**

```python
# 输出
开始线程：Thread-1
开始线程：Thread-2
Thread-1: 2018-04-25 20:47:20.066293
Thread-2: 2018-04-25 20:47:21.059349
Thread-1: 2018-04-25 20:47:21.066350
Thread-1: 2018-04-25 20:47:22.066407
Thread-2: 2018-04-25 20:47:23.069464
Thread-1: 2018-04-25 20:47:23.070464
Thread-1: 2018-04-25 20:47:24.102524
退出线程：Thread-1
Thread-2: 2018-04-25 20:47:25.070579
Thread-2: 2018-04-25 20:47:27.070693
Thread-2: 2018-04-25 20:47:29.070808
退出线程：Thread-2
```

### 线程之间的同步

多线程同时对同一数据进行修改时，为保证数据正确，需要多线程同步。

多线程的优势在于可以同时运行多个任务（至少感觉起来是这样）。

> 使用 Thread 对象的 Lock 和 Rlock 可以实现简单的线程同步，这两个对象都有 acquire 方法和 release 方法，对于那些需要每次只允许一个线程操作的数据，可以将其操作放到 acquire 和 release 方法之间。如下：
>
> 考虑这样一种情况：一个列表里所有元素都是0，线程"set"从后向前把所有元素改成1，而线程"print"负责从前往后读取列表并打印。
>
> 那么，可能线程"set"开始改的时候，线程"print"便来打印列表了，输出就成了一半0一半1，这就是数据的不同步。为了避免这种情况，引入了锁的概念。
>
> 锁有两种状态——锁定和未锁定。每当一个线程比如"set"要访问共享数据时，必须先获得锁定；如果已经有别的线程比如"print"获得锁定了，那么就让线程"set"暂停，也就是同步阻塞；等到线程"print"访问完毕，释放锁以后，再让线程"set"继续。
>
> 经过这样的处理，打印列表时要么全部输出0，要么全部输出1，不会再出现一半0一半1的尴尬场面。
> ——引自菜鸟教程，Python3 多线程



```python
import threading
import datetime
import time

class my_Thread (threading.Thread):
    def __init__(self, threadID, name, delay):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.delay = delay
    def run(self):
        print ("开始线程：" + self.name)
        threadLock.acquire()
        print_datetime(self.name, self.delay, 5)
        threadLock.release()
        print ("退出线程：" + self.name)
        
        
def print_datetime(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print ("%s: %s" % (threadName, datetime.datetime.now()))
        counter -= 1

threadLock = threading.Lock()
threads = []
        
thread1 = my_Thread(1, "Thread-1", 1)
thread2 = my_Thread(2, "Thread-2", 2)

thread1.start()
thread2.start()

# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)

# 等待所有线程完成
for t in threads:
    t.join()
print ("退出主线程")
```

**输出**

```python
开始线程：Thread-1
开始线程：Thread-2
Thread-1: 2018-04-25 20:59:31.691139
Thread-1: 2018-04-25 20:59:32.691196
Thread-1: 2018-04-25 20:59:33.692254
Thread-1: 2018-04-25 20:59:34.692311
Thread-1: 2018-04-25 20:59:35.692368
退出线程：Thread-1
Thread-2: 2018-04-25 20:59:37.692482
Thread-2: 2018-04-25 20:59:39.692597
Thread-2: 2018-04-25 20:59:41.692711
Thread-2: 2018-04-25 20:59:43.692826
Thread-2: 2018-04-25 20:59:45.692940
退出线程：Thread-2
退出主线程
```

**注意对比区别**

线程中还有线程优先级队列等，没有用到不介绍。

## 多进程Multiprocessing

### Process 类

Process 类用来描述一个进程对象，用于新建子进程。创建子进程的时候，只需要传入一个执行函数和函数的参数即可完成 Process 示例的创建。

- start() 方法启动进程，
- join() 方法实现进程间的同步，等待所有进程退出。
- close() 用来阻止多余的进程涌入进程池 Pool 造成进程阻塞。

```python
import multiprocession
multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={}, *,daemon=None)
```

- **target** 是函数**名字**，需要调用的函数（只要名字）
- **args** 函数需要的参数，以 tuple 的形式传入


```python
for each_group in url_groups:  # 每组执行多线程下载
    each_process = multiprocessing.Process(target=thread_run, args=(each_group,))
    process.append(each_process)
for one_process in process:
    one_process.start()
for one_process in process:
	one_process.join() # 阻塞当前进程，直到调用join方法的那个进程执行完，再继续执行当前进程。
```

### Pool类

同时创建多个进程也可用Pool类

```python
pool = multiprocessing.Pool(processes=4) # 创建4个进程
for i in xrange(10):
    msg = "hello %d" %(i)
    pool.apply_async(func, (msg, ))
pool.close() # 关闭进程池，表示不能在往进程池中添加进程
pool.join() # 等待进程池中的所有进程执行完毕，必须在close()之后调用
```

***pool.apply_async()***是***apply()***的并行版本， ***apply()***是***apply_async()***的阻塞版本，也是Python内置的函数，两者等价。在这个函数中输出结果并不是按照代码for循环中的顺序输出的。

***pool.apply_async(func, (msg, ))***可返回值，它返回pool中所有进程的**值的对象（注意是对象，不是值本身）**。

# 成果

![yuko1](https://github.com/qy19941014/blogimage/raw/master/img/arakiyuko1.png)
![yuko2](https://github.com/qy19941014/blogimage/raw/master/img/arakiyuko2.jpg)
![yuko3](https://github.com/qy19941014/blogimage/raw/master/img/arakiyuko3.jpg)
![yuko4](https://github.com/qy19941014/blogimage/raw/master/img/arakiyuko4.jpg)
![yukop5](https://github.com/qy19941014/blogimage/raw/master/img/arakiyuko5.jpg)
![yuko6](https://github.com/qy19941014/blogimage/raw/master/img/arakiyuko6.jpg)

#源代码

用38个链接作测试，第下载一个链接后睡眠一秒，具体用时要考虑服务器的访问限制！

## 多线程

采用多线程时用时14.1781秒

```python
import csv
import urllib.request
import time
from threading import Thread
import multiprocessing
import datetime

'''断点续传程序'''
def break_piont_continue():
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
def download_pics(urls, thread_id):
    folder_path = 'D:/Python/WeiboCrawler/Download_pics/'
    fout = open('failed_pics.txt', "a", encoding='utf-8')
    print('thread: %d is running' % (thread_id))
    for url in urls:
        try:
            urllib.request.urlretrieve(
                url, folder_path + url.split('/')[-1])
            time.sleep(1)
            print('pic: ' + url.split('/')[-1] + ' done!')
        except:
            print(url, file=fout)  # 下载失败 保存链接
            print('pic: ' + url.split('/')[-1] + ' failed!')

def thread_run(download_urls):  # 多线程
    url_group_0 = []
    url_group_1 = []
    url_group_2 = []
    url_group_3 = []

    for index in range(len(download_urls)):  # 将url分为四组
        if index % 4 == 0:
            url_group_0.append(download_urls[index])
        elif index % 4 == 1:
            url_group_1.append(download_urls[index])
        elif index % 4 == 2:
            url_group_2.append(download_urls[index])
        elif index % 4 == 3:
            url_group_3.append(download_urls[index])

    url_groups = [url_group_0, url_group_1, url_group_2, url_group_3]
    thread_group = []
    for i in range(0, 4):
        t = Thread(target=download_pics, args=(url_groups[i], i))
        thread_group.append(t)
    for one_thread in thread_group:
        one_thread.start()
    for one_thread in thread_group:
        one_thread.join()

if __name__ == '__main__':
    start_time = datetime.datetime.now()  # 用于统计时间
    download_urls = break_piont_continue()
    thread_run(download_urls)
    end_time = datetime.datetime.now()
    print(end_time - start_time)
```



## 多线程，多进程

共用时 9.6415秒

```python
import csv
import urllib.request
import time
from threading import Thread
import multiprocessing
import datetime

'''断点续传程序'''
def get_urls():
    to_down = open('weibo.csv', 'r', encoding='utf-8', newline='')
    csv_data = csv.reader(to_down)

    download_urls, downloaded_urls = [], []
    count = 0
    for data in csv_data:
        if data[9] == '':
            pass
        elif len(data[9].split(',')):
            # 读入数据为urls字符串，去除无用字符转为url—list
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
def download_pics(urls, thread_id):
    folder_path = '''保存路径'''
    fout = open('failed_pics.txt', "a", encoding='utf-8') 下载失败 保存链接
    print('thread: %d is running' % (thread_id))
    for url in urls:
        try:
            urllib.request.urlretrieve(url, folder_path + url.split('/')[-1])
            time.sleep(1)
            print('pic: ' + url.split('/')[-1] + ' done!')
        except:
            print(url, file=fout)  # 下载失败 保存链接
            print('pic: ' + url.split('/')[-1] + ' failed!')
            
def thread_run(download_urls):  # 多线程
    url_group_0 = []
    url_group_1 = []
    url_group_2 = []
    url_group_3 = []

    for index in range(len(download_urls)):  # 将url分为四组
        if index % 4 == 0:
            url_group_0.append(download_urls[index])
        elif index % 4 == 1:
            url_group_1.append(download_urls[index])
        elif index % 4 == 2:
            url_group_2.append(download_urls[index])
        elif index % 4 == 3:
            url_group_3.append(download_urls[index])

    url_groups = [url_group_0, url_group_1, url_group_2, url_group_3]

    for i in range(0, 4):
        try:
            t = Thread(target=download_pics, args=(url_groups[i], i))
            t.start()
        except:
            print('wrong')

def multi_process(urls): # 多进程
    process = []
    url_group_0 = []
    url_group_1 = []
    url_group_2 = []
    url_group_3 = []

    for index in range(len(urls)):  # 将url分为四组
        if index % 4 == 0:
            url_group_0.append(urls[index])
        elif index % 4 == 1:
            url_group_1.append(urls[index])
        elif index % 4 == 2:
            url_group_2.append(urls[index])
        elif index % 4 == 3:
            url_group_3.append(urls[index])
    url_groups = [url_group_0, url_group_1, url_group_2, url_group_3]

    for each_group in url_groups:  # 每组分多个进程、线程下载
        each_process = multiprocessing.Process(target=thread_run, args=(each_group,))
        process.append(each_process)
    for one_process in process:
        one_process.start()
    for one_process in process:
        one_process.join()  # 阻塞当前进程，直到调用join方法的那个进程执行完，再继续执行当前进程。

if __name__ == '__main__':
    start_time = datetime.datetime.now()# 用于统计时间
    download_urls = get_urls()
    multi_process(download_urls)
    end_time = datetime.datetime.now()
    print(end_time - start_time)
```

## 协程

协程6.2303秒

```python
import csv
import urllib.request
import time
import multiprocessing
import datetime
import gevent
from gevent.pool import Pool
from gevent import monkey
monkey.patch_all()

'''断点续传程序'''
def break_piont_continue():
    to_down = open('weibo.csv', 'r', encoding='utf-8', newline='')
    csv_data = csv.reader(to_down)

    download_urls, downloaded_urls = [], []
    count = 0
    for data in csv_data:
        if data[9] == '':
            pass
        elif len(data[9].split(',')):
            # 读入数据为urls字符串，去除无用字符转为url—list
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
def download_pics(url):
    folder_path = 'D:/Python/WeiboCrawler/Download_pics/'
    fout = open('failed_pics.txt', "a", encoding='utf-8')
    print('download_pics')
    try:
        urllib.request.urlretrieve(
            url, folder_path + url.split('/')[-1])
        time.sleep(1) #延时时马上执行另一个协程
        print('pic: ' + url.split('/')[-1] + ' done!')
    except:
        print(url, file=fout)  # 下载失败 保存链接
        print('pic: ' + url.split('/')[-1] + ' failed!')

def Coroutine_run(urls):
    p = Pool()
    p.map(download_pics, urls)
    # g = [gevent.spawn(download_pics, url) for url in urls]#将每一个url作为协程下载
    # gevent.joinall(g)

if __name__ == '__main__':
    start_time = datetime.datetime.now()  # 用于统计时间
    #    break_piont_continue()
    download_urls = break_piont_continue()
    # multi_process(download_urls)
    Coroutine_run(download_urls)
    end_time = datetime.datetime.now()
    print(end_time - start_time)
```

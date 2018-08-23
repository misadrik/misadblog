---
title: 布署Nginx和Gunicorn时遇到的问题
date: 2017-07-19 11:13:23
categories: 
- 技术
- 其他
tags: [博客,技术,Nginx]
---
写这篇文章是自己在做个人博客时的一些经验，在这几天的选择中，先后用了Python+Django，wordpress和hexo，最后决定暂时先用hexo应付，最终争取用python+Django自己完成博客的搭建。
一开始做博客参考的是[追梦人物](http://zmrenwu.com/category/django-blog-tutorial/)的博客教程，在做到发布前可以说介绍的非常详细，我按照教程也基本都做出来了，但到了发布则相当的简略，涉及到诸多问题。为些记录下我的发布历程，方便以后再次发布。
<!-- more -->
一直到启动**Nginx服务**这里都可以按照他博客介绍的做，但由于nginx中有默认配置，所以很多人会看到欢迎页面而不是
![WelcomeNginx](https://github.com/qy19941014/blogimage/raw/master/img/WelcomeNginx.png)
```
(env) yangxg@localhost:~/sites/demo.zmrenwu.com/django-blog-tutorial$ python manage.py makemigrate
(env) yangxg@localhost:~/sites/demo.zmrenwu.com/django-blog-tutorial$ python manage.py migrate
```
如果看到欢迎页面是DefaultPages，仍可以继续往下做
接着就按照**部署代码**中的步骤完成代码**部署前的项目配置、在github的上传**等，其中生成数据库时我提示错误，然后按照提示内容操作了下
具体是
直到**配置Nginx**，
配置Nginx的具体方法是：
（因为第一次没成功，我没有在虚拟环境中进行，此文仅还原我的操作过程）
首先在Xshell中输入
```
yangxg@localhost:~/sites/demo.zmrenwu.com/django-blog-tutorial$ cd
```
退出到根目录
然后
```
yangxg@localhost:$ cd /etc/nginx/sites-available/
yangxg@localhost:/etc/nginx/sites-available/$  sudo vi default
```
如果提示输入密码就输入之前设置的密码，然后就可以直接按照教程修改default文件中的配置内容（如果要新建文件夹并新建配置文件也可以，但应该要删除default文件）
删除命令是
```
yangxg@localhost:/etc/nginx/sites-available/$  sudo rm default
```
配置修改完成后按**Esc**，然后输入**:wq**进行保存，然后我重新进入按之前的步骤进入虚拟虚拟
然后发送配置文件到sites-enabled/目录
```
yangxg@localhost:~/sites/demo.zmrenwu.com/django-blog-tutorial$ sudo ln -s /etc/nginx/sites-available/demo.zmrenwu.com /etc/nginx/sites-enabled/demo.zmrenwu.com
```
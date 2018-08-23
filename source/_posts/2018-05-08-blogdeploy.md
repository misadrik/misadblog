---
title: 博客部署 ssl证书添加 动画背景添加
date: 2018-05-08 20:09:45
categories: 
- 技术
- Hexo
tags: [hexo,next,ssl]
copyright: true
reward: true
---
# 概述
本文以引用为主，并加上在部署过程中遇到的坑，跳过了本地环境和域名解析什么的。  
本地window系统，使用putty连接搬瓦工vps（Centos6 x86)，实现了把博客从github搬到了vps上。
<!-- more -->
# 服务器端
## 安装Nginx

服务器安装nginx是参照 [LNMP一键安装包](https://lnmp.org/install.html)

这个教程相当清楚，如果照着做（服务器什么的都要对应），应该不会有太多问题。我按着教程把虚拟主机也装了，后面可能会出现用ip访问时总是显示的LNMP一键安装包界面

* 这步的时间有点长，一定要用putty连接服务器，通过网页会登录超时，退出了就不好办了。使用putty需要服务器ip地址，ssh端口号，root用户的密码
* **装虚拟主机**时若选了Let's Encrypt，这个好像需要每三个月更新证书，比较麻烦
* **安装虚拟主机时**选了文件夹可能对后面有影响

所以**不要手贱**多装东西

## 安装Nodejs和git

总体参照两篇文章  **建议整体操作前先看这两篇** 
[基于CentOS搭建Hexo博客](https://segmentfault.com/a/1190000012907499) 
[在服务器上搭建hexo博客](https://blog.csdn.net/moumaobuchiyu/article/details/70312740)

其中遇到的坑有

* vim打开文件后，按i插入修改文件，改完后ESC退出 并输入`:wq!`保存
* vim新建文件并添加内容时，似乎开头内容会丢失，所以在**第一行加个#**，然后转到第二行再输入
* 新建用户时最好还是照教程来，就叫git，否则后面会涉及一系列的更改，我就在这上面浪费了蛮多时间
* 添加ssh-key时直接使用的是github的那个，注意**不要把最后的邮箱复制上去**，路径一般是在

```
C:\Users\Administrator\.ssh
```

* 验证时要注意，**git默认验证的端口是22端口，而搬瓦工的ssh端口一般不是22端口**！所以要么改git，要么改搬瓦工的端口，改了端口后，又可能涉及putty登录的端口，我改的是搬瓦工的端口。 [搬瓦工修改SSH端口的方法](https://www.banwagong.net/203.html) 

* 验证成功后deploy，我发现ip访问仍然是LNMP，说明Nginx的默认文件夹不对，这时候就要修改nginx.conf

  >把nginx的配置文件`nginx.conf`中的**部署目录改为/usr/share/nginx/html/blog**，配置文件是`/usr/local/nginx/conf`里的`nginx.conf`；同样可以使用默认目录，nginx的默认目录为`/var/www/html`，如果使用LNMP一键安装包，则默认的部署目录为`/home/wwwroot/default`

* ```nginx
  user  misad#我自己设置成了misad,默认root
  server
  {
  	listen       80;
  	server_name www.misad.me; # 改成自己的域名
  	index index.html index.htm index.php;
  	root  /usr/share/nginx/html/blog;# 改成上一步中的文件夹，存放deploy内容的文件夹

  ##########################################################################
  }
  include vhost/*.conf; # 我后来在git上传网站成功后，频繁403和ip无法正常跳转时注释掉了这个，好像和虚拟主机有关
  }
  ```
  `:wq!` 保存退出后
  首先，`  nginx -t ` 检查配置是否有误，并按照报错提示修复错误。

  然后，执行命令 `service nginx restart` ，重启 Nginx 服务。

  最后，执行命令 `service nginx reload` ，重新载入 Nginx 服务。

  这样应该就可以通过域名和ip（记得刷新以防浏览器缓存）


## 添加SSL证书

正好一起弄完早点弄完吧，再过一阵chrome不支持了可能自己都受不了。

* 先要在阿里云上申请证书（阿里云，腾讯云的证书有效期是一年，Let's Encrypt好像是三个月虽说可以设置自动更新，不过看看蛮烦，就算了- -）  
CA证书服务—>购买证书—>选**Symantec**品牌—>免费型DV SSL

* 返回CA证书服务，填写相关资料，提交审核十分钟不到就能完成审核了，然后下载证书 for Nginx。

* 上传证书到Nginx

  ```
  sudo mkdir -p /usr/share/nginx/ssl/  # 新建目录
  ```
  上传下载的证书，我是直接**cd命令打开**到上面的目录，然后用**vim**新建相关文件，并**复制上传**（注意第一行先加个#否则可能会丢失开头数据

* 然后再打开nginx.conf文件，**在上面的server下添加**

  ```nginx
  server {
    listen       443 ssl;
    server_name  misad.me;
    # ssl          on; # 建议用在443端口后加ssl的方法，否则访问http链接将显示错误
    root /usr/share/nginx/html/blog;
    index index.html; 

    ssl_certificate   /etc/nginx/ssl/213985317020706.pem;
    ssl_certificate_key  /etc/nginx/ssl/213985317020706.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;
  }
  ```


* 再**重启Nginx**，便可以通过http和https来访问博客。但会发现两者是分开的，我们还要设置http强制跳转到https，直接添加这一段，并**删掉**了原来server中的监听80端口。

  ```nginx
  # 推荐
  server {
    listen        80;
    server_name   misad.me;
    return 301    https://$host$request_uri;
  }
  ##亦可
  server {
      listen 80;
      server_name   misad.me;
      rewrite / https://$host$request_uri permanent;
  }
  ```

  在实际应用中又遇到坑了，我**错误输入了misad.com**，重启执行了，然后改回来也不行，最后发现是**浏览器缓存**了，清空缓存就可以了！

# 动画背景添加

今天还试了下其他的动画背景，其实next里面支持了好几种但直接改True似乎不行，今天研究了一个大神<font color="#FF0000" > [夏末](https://notes.wanghao.work/) </font>博客的源代码后发现，其实只需要在网页的生成代码中加入一段js就可以了

```html
{% if theme.canvas_lines %}
<script type="text/javascript" src="/lib/three/canvas_lines.min.js"></script>
{% endif %}
```

其中的src还可以改成（**相应的if条件也要改**）

```html
"/lib/three/canvas_lines.min.js"
"/lib/three/canvas_sphere.min.js"
"/lib/three/three.min.js"
"/lib/three/three-waves.min.js"
```

其实就是**\next\source\lib\three**下的几个js文件，而**\next\source\lib**下还有**canvas_nest**和**canvas_ribbon**也是同理。

# 其他
各项测试后发现文章统计阅读统计异常，因此需要去 https://leancloud.cn 把https站点添加到安全中心。



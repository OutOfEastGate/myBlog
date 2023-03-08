---
title: 利用废旧手机搭建web服务器
date: 2022-05-26 18:24:20
tags: web
category: 教程
swiper_index: 4
description: 告别云服务器，手机也能做服务器
cover: https://th.wallhaven.cc/lg/y8/y86g17.jpg
---



### 第一步 下载ksweb

首先我们需要下载ksweb这款app，这是一位俄罗斯大神制作的，里面集成了Lighttpd、nginx、Apache的服务器，还有mysql服务器等，十分强大

链接:https://pan.baidu.com/s/1EAPZVsnWpL62PfHXHqHpEg 
提取码:c5e5

打开软件后应该是这样子的

<img src="https://img-blog.csdnimg.cn/img_convert/a9039f43eed8fe0a881b83f2f9438c13.png" width="20%" />

### 第二步 配置ksweb

然后我用的是lighttpd服务器，点击主机下面那串文字，我们可以自己选一个目录作为网站的根目录，然后将网站的资源放在该目录下

<img src="https://img-blog.csdnimg.cn/img_convert/705a75db19f4bd7f38e11e95e28c6a6e.png" width="20%" />

接下来就可以直接点击Lighttpd那里的http://localhost:8080来进行访问了，这里我用的是hexo博客，直接把hexo的public文件夹的静态资源放在网站的根目录下就可以访问了。也可以使用内网ip+8080进行访问。

<img src="https://img-blog.csdnimg.cn/img_convert/5ba6a6e79ac560bc075dc8f89b619e15.png" width="20%"/>

如果你的网站是php、wordpress、hola等需要数据库的网站，也可以参考其他教程，配置一下phpmyadmin就可以了，也是很方便。

> 需要注意的是，手机一般都自带电池优化，所以只要退出ksweb这个界面或者手机锁屏都不能访问成功，所以要设置一下取消电池优化，然后允许后台自启，这样就可以一直进行访问了。

### 第三步 进行内网穿透

在以上步骤中，我们已经搭建好了内网环境，只能在局域网内访问，要想在公网下访问，就需要进行内网穿透。

为了方便快捷起见，我使用的是花生壳这款软件，对小白很友好，但是需要支付6块钱的费用，基本就是永久使用了，白嫖党们可以看看Termux+Ngrok进行内网穿透的教程。

需要安装花生壳管理和花生壳内网两款软件，进入花生壳官网即可下载这两款软件。

进入花生壳管理，选择自定义映射，映射类型选择HTTP，内网主机要填ksweb那个界面显示的内网ip，也可以进入连接wifi的界面查看手机本机ip，然后内网端口填8080，这样就OK啦。

<img src="https://img-blog.csdnimg.cn/img_convert/d8a34c92988fb61472ea36b71cdb8141.png" width="20%" />

进入花生壳内网app，出现以下界面就表示穿透成功啦！



<img src="https://img-blog.csdnimg.cn/img_convert/23f3a2c6bc9b0d41ceccca8fa590e263.png" width="20%" />

> 同样需要注意，必须在花生壳那个界面才能保证穿透成功，离开花生壳app后映射就会失败，所以也要按照之前的步骤设置允许花生壳自启动，取消电池优化。

### Last 成功建立服务器

到这里我们就成功建立服务器啦，通过花生壳给的域名就可以通过公网进行访问啦，当然这个域名十分地丑陋，我们可以自己购买域名，通过dns解析到该域名上。

![](https://img-blog.csdnimg.cn/img_convert/e3b8d883bcc9ce0b96d8279e85c025b4.png)

如果访问失败，大概率是因为手机后台的原因，要保证ksweb和花生壳同时处于启动状态才行。

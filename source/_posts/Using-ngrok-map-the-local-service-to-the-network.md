title: 使用ngrok将本地Web服务映射到外网
date: 2015-12-26 22:35:51
tags:
- ngrok
categories: notes
---

利用ngrok可以通过外网访问本地项目，这个对我们是非常有用的，比如我们平常开发的网站还没更新到线上，但是又需要通过外网可以访问到本地应用，方便测试。<!-- more -->

该工具使用方法非常简单。

首先需要下载客户端

下载地址：[http://www.tunnel.mobi/](http://www.tunnel.mobi/)

![](/images/2015122601.jpg)

下载以上两个文件。然后放到同一个目录

![](/images/2015122602.png)

然后在该目录打开命令行窗口输入命令：
ngrok -config ngrok.cfg -subdomain dts 80
dts为配置的子域名 80是本地项目端口

本地效果：
![](/images/2015122603.png)

公网效果：
![](/images/2015122604.png)


注：最近发现[http://www.tunnel.mobi/](http://www.tunnel.mobi/) 已经访问不了 不过有另外备用的可以使用：[http://qydev.com/](http://qydev.com/)
这个微信demo开发有很大帮助。mark下
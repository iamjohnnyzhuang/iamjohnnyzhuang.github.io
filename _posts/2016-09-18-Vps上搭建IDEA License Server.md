---
layout: post
title: Vps上搭建IDEA License Server
categories: [other]
---

# 前言

几年前还在学校的时候，写Java的都知道Eclipse以及Myeclipse。当时觉得IDE怎么这么好用，毕业了用上了IDEA一开始各种唾弃eclipse。

这么一款好用的IDE其实花点钱买正版个人觉得还是蛮值得的。只是迫于生活无奈，所以还是只能先暂时用着注册机了。

IDEA提供三种注册方案：

* 输入IDEA的账号密码
* 输入激活码（常用）
* 输入注册服务器地址（强烈推荐）

其中激活码可以网上找，但是现在比较难找了。而注册服务器地址以前网上也是满天飞，但是后来各种查封。因此我就尝试自己搭建。

**如果是单机使用，可以不用VPS但是这样你每次开电脑都得开启一个程序，因此如果有VPS强烈推荐在VPS搭建好，以后有小伙伴求地址很帅气的发给他你的域名。**



---



# 配置

首先我们需要下载一下运行服务器程序。参考[这篇博客](http://blog.lanyus.com/archives/174.html/comment-page-4#comments)，进入[google网盘](https://drive.google.com/file/d/0Bx7wGDIg2K-7MTJ1TGN1V1IzTVk/view?usp=sharing)下载服务端程序。

下载完后，如果你是要本地运行，如果你是windows系统，运行IntelliJIDEALicenseServer_windows_amd64.exe即可。然后IDEA license server地址输入： http://127.0.0.1:1017 这个地址是访问你本地1017服务地址。完成验证。

如果是VPS上的验证，需要将你VPS系统对应的文件上传上去。例如我用的是ubuntu，我将文件IntelliJIDEALicenseServer_linux_386上传到我的VPS上面。

然后运行这个程序，注意运行前要先设置权限，否则会运行不了

``` bash
chmod 777 ./IntelliJIDEALicenseServer_linux_386
```



ok了，试下通过ip端口访问

http://yourdomain:1017

访问会报404，因为这个程序并没有提供外部访问。

为了能够让外部访问，我们必须**使用nginx进行代理**。

Ubuntu下安装Nginx：

``` bath
sudo apt-get install nginx
```



安装完后进入配置文件夹编辑default文件：

``` bath
cd /etc/nginx/sites-available && vi default 
```



在最后面添加: 

``` bath
server
{
#暴露给外部访问的端口
listen 8080;
#域名地址
server_name zhuangjingyang.com;

location /{
#本地服务端口
proxy_pass http://127.0.0.1:1017;
proxy_redirect off;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}
```

这段代码是让Nginx给你做代理，因此外部访问 zhuangjingyang.com:8080 的时候nginx会将请求转发到 127.0.0.1:1017。

接下来重启下Nginx让配置生效:

``` bath
nginx -s reload
```



这样就完成了我们VPS上IDEA服务器的搭建。



---



# 结束语

如上所述，配置非常的简单，使用开源的代码运行+Nginx做代理即可。你所需要的仅仅是一台VPS，而得到是一个稳定的快速的、可小范围共享的IDEA认证服务器！



亲测：IDEA 2016.2.2 以及 WebStorm 2016.2.2 均可使用。






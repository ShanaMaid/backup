---
title: 博客小绿锁添加日记与nginx反向代理
date: 2017-05-02 14:10:22
tags:
- nginx
- https
- 反向代理
categories:
- 博客开发
---

## 关于`https`小绿锁的加持
小绿锁加持计划拖了有一段时间额，趁着五一假期终于搞定了，中间也是一波三折。
<!-- more -->

### 配置dns
首先小绿锁的加持使用的是国外的`cloudflare`(以下简称cf)免费提供的`SSL`。在[cf官网](https://www.cloudflare.com/)注册一个账号，添加你的域名站点，选择免费计划。

一切完成了以后点击上方一排中的`dns`图标，你可以看到`cf`为你提供了两个`ns`
{% asset_img 1.png cf免费ns %}

用这两个`ns`去替换掉你域名的dns，博主的域名是在新网购买的，替换如下图
{% asset_img 2.png 替换dns %}

在`cf`的`dns`页面添加一条`DNS Records`，如下图
{% asset_img 5.png 添加dns记录 %}

### 使用https
在`cf`的`Crypto`页面，将`SSL`设置为`flexible`
{% asset_img 3.png SSL设置为flexible %}

注意，此时你的博客并不能通过`https`，此时用`https`访问会提示无效证书。
因为刚设置完的时`flexiable`候其实没有绿色`Active Certificate`，官方在左侧已经说明了
> It may take up to 24 hours after the site becomes active on Cloudflare for new certificates to issue.

这句话的大概意思就是说需要一段时间去激活证书，大概是24小时。

实际上其实用不了那么久，大概12小时左右博主邮箱就收到了一封激活邮件，告诉博主已经激活了。此时便可以使用`https`去访问你的网站了。
{% asset_img 4.png 激活邮件 %}

### 强制https
`cf`的`Page Rules`页面，添加路由规则。通过规则中的`Always use https`选项，可以将`http`访问的用户强制跳转到https。设置如下：

{% asset_img 6.png 配置页面规则 %}

这里有个坑的地方就是在`Active Certificate`未激活前，也就是在你没收到官方的通知邮件前，你会发现你的路由规则中是没`Always use https`这个选项的。

### 加持完毕
此时`ping`一下博客域名发现已经成功，同时无论通过`http`还是`https`访问你的博客，都带上优雅的小♂绿♀锁，只是初次访问速度上会慢上许多。
{% asset_img 7.png ping博客域名 %}

╮(￣▽￣")╭，反正免费的也就这样吧！

以上大部分参照于我朋友[@Choin Tang](https://blog.chionlab.moe/)的[在GitHub Pages上使用CloudFlare https CDN](https://blog.chionlab.moe/2016/01/28/github-pages-with-https/)

## cloudflare带来的遗留问题
博客加上小绿锁了后一段时间，收到开源项目的使用者的邮件，告知github上开源的三个项目的在线版都无法使用了。
看了下,三个开源项目的网址如下

|项目|地址|
|----|----|
|oho阅读|http://www.shanamaid.top:3001/|
|163music|http://www.shanamaid.top:3000/|
|webchat|http://www.shanamaid.top:7777/|

想了想应该是自己在cf上面忘记添加根域名的A记录了，所以在cf上补上了
{% asset_img 8.png 添加dns records %}

过了一段时间，ping了一下`www.shanamaid.top`，毫无意外，ping通了，感觉没有问题了，然后打开网页带上端口号访问的时候页面死活进不去→_→ 。

翻来覆去查了下，果然是`cf`的锅，添加的dns记录只能解析到80端口。

正好最近在看`《nginx高性能web服务器详解》`，为了解决问题那只有上`nginx`进行反向代理咯~

先在cf中追加dns records，把域名全部解析到`119.29.159.156`。
{% asset_img 9.png dns records %}

然后在`nginx`的配置文件中添加下列几行代码，然后`./nginx -s reload`重新加载一下配置文件。
```
server {
  listen  80;
  server_name oho.shanamaid.top;
  location / {
     proxy_pass http://119.29.159.156:3001;
  }
}
server {
  listen  80;
  server_name 163music.shanamaid.top;
  location / {
     proxy_pass http://119.29.159.156:3000;
  }
}

server {
  listen  80;
  server_name webchat.shanamaid.top;
  location / {
     proxy_pass http://119.29.159.156:7777;
  }
}
```
将其分别解析到`3001`、`3000`、`7777`端口，此时就可以通过域名重新访问了。

|项目|地址|
|----|----|
|oho阅读|http://oho.shanamaid.top/|
|163music|http://163music.shanamaid.top/|
|webchat|http://webchat.shanamaid.top/|



## 最后
通过cf后，开源项目的在线版访问速度变得异常慢，即使开启了gzip还是很难受，过段时间也许会考虑换个域名专门挂开源项目在线版？或者说弃用`cf`？

暂时就这样吧~~！↖(^ω^)↗

## 后记
经过cf转发后速度开源项目在线版的读取速度实在惨不忍睹，于是新注册了一个shanalab域名用作挂开源项目的。项目地址改为

|项目|地址|
|----|----|
|oho阅读|http://oho.shanalab.top/|
|163music|http://163music.shanalab.top/|
|webchat|http://webchat.shanalab.top/|

nginx转发配置
```
```
server {
  listen  80;
  server_name oho.shanalab.top;
  location / {
     proxy_pass http://119.29.159.156:3001;
  }
}
server {
  listen  80;
  server_name 163music.shanalab.top;
  location / {
     proxy_pass http://119.29.159.156:3000;
  }
}

server {
  listen  80;
  server_name webchat.shanalab.top;
  location / {
     proxy_pass http://119.29.159.156:7777;
  }
}
```
```






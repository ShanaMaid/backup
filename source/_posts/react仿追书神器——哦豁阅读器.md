---
title: react仿追书神器——哦豁阅读器
date: 2017-04-11 11:12:01
tags:
- React
- javascript
- express
categories:
- 实战开发
- React

---

## 前言
都知道追书神器从某个版本开始就不支持换源了，开始实行收费制度，虽然老版追书神器依然可以使用，但是指不定那天就挂掉了。再加上最近想熟悉一下`react`，所以本项目`哦豁阅读器`就诞生了。

Github项目地址:[https://github.com/ShanaMaid/oho-reader](https://github.com/ShanaMaid/oho-reader)

欢迎`issue`，`pr`，`star` or `follow`！我将继续开源更多有趣的项目

推荐一个之前用Vue全家桶写的 [网易云音乐PC端 web版本](https://github.com/ShanaMaid/vue-163-music)

<!-- more -->

## 哦豁阅读器(oho-reader)介绍
哦豁阅读器！API源自追书神器，免费使用！

实现追书神器核心功能，做到小说阅读的极简体验，把每一分流量都用到刀刃上！

在线版:[http://oho.shanamaid.top/](http://oho.shanamaid.top/)
服务器带宽较小，初次加载比较慢，请谅解！建议`clone`到本地进行体验！

Github项目地址:[https://github.com/ShanaMaid/oho-reader](https://github.com/ShanaMaid/oho-reader)

### Oho阅读器的优势

|     | oho阅读器 |  追书神器|
|-----|-----------|----------|
|收费 | 免费      |部分章节免费,其余收费|
|广告 |绿色无广告 | 定时刷广告|
|体积 | 4MB     | 16.2MB   |
|章节大小| 每章5kb左右   | 掺杂广告，大于5kb|

oho阅读器初次打开时候加载比较慢，一部分原因是服务器带宽较小，另一部分是因为初次需要下载`700kb`左右的文件，建议初次下载在wifi下进行。初次下载后oho阅读器会自动进行缓存，以后每次打开页面基本是秒开，消耗流量约在`1KB`不到。
{% asset_img first.png  初次打开消耗流量约在700kb左右 %}

{% asset_img after.png  后续打开消耗流量约在1kb不到 %}
同时oho器抛弃所有与小说阅读无关的信息，真正做到极简！保证每一分流量都用到小说内容的阅读上，真正做到每章内容加载所用的流量集中在小说章节内容上，视章节字数而定，一般在`5kb`左右。
{% asset_img chapter.png  每章流量消耗 %}


oho阅读器目前由于服务器配置、带宽过小原因暂不支持章节内容缓存。

### 效果Gif图
{% asset_img 1.gif  %}
{% asset_img 2.gif  %}
{% asset_img 3.gif  %}
{% asset_img 4.gif  %}
{% asset_img 5.gif  %}
{% asset_img 6.gif  %}


### 实现功能
- 小说搜索
- 小说详情
- 小说换源
- 小说阅读
- 阅读字体大小变化
- 阅读背景色变化
- 阅读设置本地缓存
- 阅读进度本地缓存
- 搜索历史本地缓存

## 使用
```
git clone https://github.com/ShanaMaid/oho-reader.git

cd oho-reader

npm install 

# 开发环境
npm run serve
访问 http://localhost:8080/

# 打包
npm run dist

# 实际环境
cd server
node app.js
访问 http://localhost:3001/
```

## 目录结构
```
|
|—— api 追书神器API说明 
|—— cfg webpack配置
|—— dist 服务端
| |—— app.js 服务端启动入口文件
| |—— assets 打包后的资源文件
| |—— static 静态资源
| |__ index.html 网页入口
|
|——src 资源文件
| |—— images 图片资源
| |—— components 组件库
| |—— method  一些自定义方法，目前是过滤器
| |—— filters 自定义过滤器
| |—— redux 
| | |—— action
| | |—— reducer
| | |__ store
| |—— router 路由管理
| |—— styles 样式文件
| |__ index.jsx 入口
|_________________________________________________

```

## 一些注意事项
项目中使用追书神器的接口，需要使用`http-proxy-middleware`进行转发，开发环境下需要在`cfg/base.js`中的`dev`中添加下列配置即可
```
proxy: {
  '/api': {
    target: 'http://api.zhuishushenqi.com/',
    pathRewrite: {'^/api' : '/'},
    changeOrigin: true
  },
  '/chapter': {
    target: 'http://chapter2.zhuishushenqi.com/',
    pathRewrite: {'^/chapter' : '/chapter'},
    changeOrigin: true
  }
}
```

实际环境中，服务器端配置
```
var express = require('express');
var proxy = require('http-proxy-middleware');

var app = express();
app.use('/static', express.static('static'));
app.use('/assets', express.static('assets'));
app.use('/api', proxy({
  target: 'http://api.zhuishushenqi.com/',
  pathRewrite: {'^/api' : '/'}, 
  changeOrigin: true
}
));

app.use('/chapter', proxy({
  target: 'http://chapter2.zhuishushenqi.com/',
  pathRewrite: {'^/chapter' : '/chapter'},
  changeOrigin: true
}
));

app.get('/*', function (req, res) {
  res.sendFile(__dirname + '/index.html');
});
app.listen(3001);
```

## 支持
BUG提交请发送邮箱: uestczeng@gmail.com

Github项目地址:[https://github.com/ShanaMaid/oho-reader](https://github.com/ShanaMaid/oho-reader)

欢迎`issue`，`pr`，`star` or `follow`！我将继续开源更多有趣的项目

你的支持将有助于项目维护以及提高用户体验，感谢各位的支持！

## 后续计划
过段时间计划把`oho-reader`迁移到`react-native`，具体时间可能要看什么时候有空了。

## 总结
最近粗略使用了一下`vue`与`react`，大致感觉就是前者是在`html`里面写`js`，后者是在`js`里面写`html`，就目前来看两者现在基本上是势均力敌、各有千秋，未来的具体走向如何谁都说不准，当然这是我的个人见解。如果你有什么好的建议或者说值得探讨的话题，可以在下方留言。

---
title: vue仿163musicPC端
date: 2017-03-21 14:33:32
tags:
- vue
- javascript
categories:
- 实战开发
- vue
---
## 前言
vue2越来越受欢迎，无奈现在在公司做的平台是以ng1.x为主，一直没有机会练手vue2，虽然写过一些小demo，但是与完整的项目相比较中间会少很多东西。于是趁在公司空闲的时候以及周末双休，自己用vue2复写了163musicPC端。相比较之掘金上大大写的很多都是纯静态页面vue2，实际开发中肯定会涉及到接口、数据渲染方面，本项目接口通过`http-proxy-middleware`， 一个http代理的中间件，进行http请求转发，实现跨域请求，直接复用网易爸爸的接口，在服务端对返回的JSON进行解构即可。
<!-- more -->

## 介绍
vue-163-music(网易云音乐web版)，用vue仿写163音乐客户端版。

原计划仿写完所有页面，碍于网易的接口API有限，实现页面也有限。

不推荐手机端访问。

页面高度为`670px`，`1366 X 768`分辨率及其以下按F11全屏浏览效果更佳

Github项目地址:[https://github.com/ShanaMaid/vue-163-music](https://github.com/ShanaMaid/vue-163-music)

欢迎`issue`，`pr`，`star` or `follow`！我将继续开源更多有趣的项目

## 在线版
[点击进入 http://www.shanamaid.top:3000/](http://www.shanamaid.top:3000/)
腾讯学生云主机，最低配，存在卡顿，建议clone到本地进行体验

## 使用
```
git clone https://github.com/ShanaMaid/vue-163-music.git

cd vue-163-music

npm install 

# 开发环境
npm run dev
访问 http://localhost:8080/

# 打包
npm run build

# 实际环境
cd server
node app.js
访问 http://localhost:3000/
```

## 效果截图
{% asset_img 1.gif  %}
{% asset_img 2.gif  %}
{% asset_img 3.gif  %}
{% asset_img 4.gif  %}
{% asset_img 5.gif  %}
{% asset_img 6.gif  %}
{% asset_img 7.gif  %}

## 工具&技能
`vue` + `vuex`+ `vue-router` + `vue-resource`

`express`

`http-proxy-middleware` 一个http代理的中间件，进行http请求转发，实现跨域请求

`store.js` 一个非常棒的处理`localStorage`的轮子，原生`localStorage`只支持存储字符串类型，轮子进行封装后可以直接存储`Array`、`Object`、`function`、`Set`等类型

`animate.css` css动画库

`vue-slider-component` 滑块组件

`postman` 接口测试工具


## 实现功能
### 发现音乐
- 个性推荐(推荐歌单中除每日歌曲推荐外，其余歌单可点击进入)

### 播放音乐
- 上一曲
- 播放
- 暂停
- 下一曲
- 进度控制
- 音量控制

### 音乐搜索
输入搜索关键词，`回车键`搜索，或者点击`放大镜`图标
- 单曲(单击或双击歌曲添加至音乐播放列表，部分音乐会存在版权问题无法播放)
- 歌手
- 专辑
- MV
- 歌单(左键点击进入歌单列表)
- 主播电台 (单期节目部分单击或双击歌曲添加至音乐播放列表，目前不存在版权问题)
- 用户

### 歌单
- 播放全部

### 播放列表
- 切歌(单击切歌)
- 删歌(鼠标悬浮在要删除的歌曲上，点击右侧小X)
- 清空播放列表
- 本地缓存播放列表

## 一些问题
通过api接口获取的mv播放量基本不准，尚未找到原因，其余类型的播放量准确


## 目录结构
```
|
|—— build 
|—— config
|—— server 服务端
| |—— app.js 服务端启动入口文件
| |—— static 打包后的资源文件
| |__ index.html 网页入口
|
|——src 资源文件
| |—— assets 组件静态资源库
| |—— components 组件库
| |—— deal  163api返回的JSON字符串解构
| |—— filters 自定义过滤器
| |—— router 路由配置
| |—— store vuex状态管理
| |—— App.vue 163SPA
| |__ main.js SPA入口
|
|__ static 静态资源目录

```

## 一些注意事项
项目中使用了网易爸爸的接口，需要使用`http-proxy-middleware`进行转发，开发环境下需要在`config/index.js`中的`dev`中添加下列配置即可
```
proxyTable: {
  '/api': {
      target: 'http://music.163.com',
      changeOrigin: true,
      headers: {
          Referer: 'http://music.163.com/'
      }
  }
}
```

实际环境中，服务器端配置
```
var express = require('express');
var proxy = require('http-proxy-middleware');

var app = express();
app.use('/static', express.static('static'));
app.use('/api', proxy({
  target: 'http://music.163.com', 
  changeOrigin: true, 
  headers: {
    Referer: 'http://music.163.com/'
  }
}
));

app.get('/', function (req, res) {
  res.sendFile(__dirname + '/index.html');
});
app.listen(3000);
```

对返回的数据解构`js`文件位于`src/components/deal/`目录下，比如对单曲搜索结果进行解构
```
single: (data) => {
  let list = []
  let count = data.result.songCount
  if (count === 0) {
    return {list, count}
  }
  for (let item of data.result.songs) {
    let singer = ''
    let {
      name,
      mp3Url,
      duration,
      id,
      album: {
        name: albumName
      }
    } = item
    for (let item of item.artists) {
      singer += item.name + ' '
    }
    list.push({name, mp3Url, duration, id, albumName, singer})
  }
  return {list, count}
}
```

`vuex`状态管理位于`src/components/store`目录下

`vue-router`路由配置管理位于`src/components/router`目录下

自定义过滤器位于`src/components/filters/`目录下


网易云音乐接口来源于[http://moonlib.com/606.html](http://moonlib.com/606.html)

Github项目地址:[https://github.com/ShanaMaid/vue-163-music](https://github.com/ShanaMaid/vue-163-music)

## 总结
本项目的实际意义在于熟悉vue生态链以及ES6语法，同时思考如何用vue构建实现一个完整的项目。

最后的感觉vue2在开发中带来的体验确实很棒，vue2能如此之火自然有它的道理。

如果觉得本项目不错的话，别忘记`star`哦！


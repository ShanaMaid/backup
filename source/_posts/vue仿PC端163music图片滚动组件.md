---
title: vue仿PC端163music图片滚动组件
date: 2017-03-03 21:07:30
tags:
- vue
- javascript
categories:
- 实战开发
- vue
---
## 介绍
这是一款模仿PC端网易云音乐的vue图片滚动插件

[Github项目地址-vue-image-scroll](https://github.com/ShanaMaid/vue-image-scroll)

[在线文档和demo](http://blog.shanamaid.top/vue-image-scroll/example/)

欢迎各位dalao指点,star or PR!

<!-- more -->
## 安装与使用
### 安装
```
npm install vue-image-scroll 
```

### 使用
```
 <template>
      <div>
       <slider v-bind="setting">
      </div>
    </template>
<script>
import slider from 'vue-image-scroll';

export default {
  components: {
    slider
  },
  data: function() {
    return {
      setting: {
        image: ['1.jpg', '2.jpg', '3.jpg']
      }
    }
  }
}
</script>

```

### 本地调试
```
git clone https://github.com/ShanaMaid/vue-image-scroll.git

npm install 

npm run dev 
```
## 说明
项目使用vue-cli开发，源文件在`src/components/Slider.vue`中，`lib`中的`index.js`为压缩后的文件


## 开发遇到的问题
打包组件的时候遇到,loader error的问题，发现`webpack2.0`开始`loaders`中`babel`以及`url`等都需要写作`babel-loader`、`url-loader`


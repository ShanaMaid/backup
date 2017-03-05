---
title: 利用Php爬虫备份自己的CSDN博客
date: 2016-10-18 14:15:46
tags:
- PHP
- hack
- Spider

categories:
- 实战开发
- PHP
---

## 自己写备份博客的原因
最近逛自己博客的时候发现左边经常有个小广告，十分恶心，看着很烦。同时自己最近也准备自己搭一个博客，所以就想着把自己CSDN博客上的文章备份下来。<!-- more-->本着偷懒至上的原则⁄(⁄ ⁄•⁄㉨⁄•⁄ ⁄)⁄ ，先去百度了一下有木有现成的博客爬虫软件。发现有个叫豆约翰的软件，当时的心情是很激动的有木有φ(゜▽゜*)♪，然而打开的一瞬间居然要收费，还是按月收费。尼玛，程序员的本性简直不能忍。So，自己写了一个github开源项目CSDNSpider，接下来讲解一下思路吧。

项目地址：[CSDNSpider](https://github.com/ShanaMaid/CSDNSpider) 

## Php爬虫那些事儿
### 爬虫需要了解的函数
* fopen，用于打开需要爬虫的url，获得一个流
* stream_get_contents，用于将fopen获得的流处理成字符串
* preg_match，用于正则表达式匹配
* curl，这里用作图片的下载
* preg_replace，用于替换符合正则表达式的内容
当然还有最最重要的 <strong>正则表达式</strong>，详情可以点击[教程](http://www.runoob.com/regexp/regexp-syntax.html)
这里函数的作用就只进行简单说明了，各位先更深入了解可以去官网查看文档。

### 整个爬虫的思路
关于爬虫首先我们要理清楚我们要做的是什么，需要构造那些函数？
整理后我们可以得到以下函数，当然因人而异，可能你会有其它不同的看法

* SpiderGo，爬虫主函数
*  getWebContent，用于获得网站的主要内容
* getPageNumber，用于获得网站文章页面的总页数
* getArticleList，用于获文章列表包括文章的链接和题目
*  getArticleContent，用于获得文章的内容
*  getImage，获取文章中图片的链接
* replaceImgUrl，替换文章中的图片连接为本地连接
* downloadImg，下载图片

还有些函数就不多做说明了，其实大致文章爬虫主题上就是这些部分，根据不同个网站的需要进行增删部分辅助性的函数

## 爬虫的一些基本常识
爬虫中所谓的翻页、查看文章详情内容等这些点击操作其实都是通过url实现的，所以爬虫的核心点在我看来就是url+正则表达式，就拿我CSDN博客来举例子
打开我的CSDN个人博客页面，url栏是这样显示的：http://blog.csdn.net/shanamaid
我们可以看到url的格式为:
```
“http://blog.csdn.net/”+用户名
```
那么我们要如何通过url实现翻页呢？
我们先翻第二页看看，url栏是这样显示的:
```
http://blog.csdn.net/ShanaMaid/article/list/2
```
So,我们得到结论，文章翻页的url格式是这样的:
```
http://blog.csdn.net/用户名/article/list/页数
```
再来，我们把这个url打开会得到什么东西呢？
```
http://blog.csdn.net/ShanaMaid/article/list/1
```
很显然，
```
http://blog.csdn.net/ShanaMaid/article/list/1
```
打开的页面和
```
http://blog.csdn.net/shanamaid
```
是一样的，也就是说，这两个链接是等价的，但是为了统一url格式，我们在写爬虫的时候肯定用的是
```
http://blog.csdn.net/ShanaMaid/article/list/1
```
打开页面而不是
```
http://blog.csdn.net/shanamaid。
```
接下来我们打开文章详情页面看看，
```
http://blog.csdn.net/shanamaid/article/details/52441330
```
分析得到文章详情页面得到:
```
http://blog.csdn.net/用户名/article/details/文章ID编号
```

我们再分析一下文章列表页面发现

```
<span class="link_title"><a href="/shanamaid/article/details/52441330">
        <font color="red">[置顶]</font>
        ReactJS学习之一篇博客教你入门ReactJS            
        </a></span>
```

发现我们需要的文章的题目和链接都在class="link_title"的span里面，所以我们可以用一个正则表达式把它提取出来
```
<span class="link_title">[\w\W]*?<\/span>
```
然后再用

```
<\/?[^>]+>
```

可以去掉标签得到题目   [置顶]ReactJS学习之一篇博客教你入门ReactJS 
用

```
[0-9]{5,}
```

可以提取出文章标号，得到52441330
即

```
//获取文章列表
function getArticleList($page,$username){
	$url = 'http://blog.csdn.net/'.$username.'/article/list/'.$page;
	$content = getWebContent($url);
	$tag = '/<span class="link_title">[\w\W]*?<\/span>/';
	$tag_name =  '/<\/?[^>]+>/';//去除html标签
	$tag_url  =   '/[0-9]{5,}/';//提取编号
	preg_match_all($tag, $content, $result);//提取出包含有文章题目和编号的内容
	for ($i=0; $i < sizeof($result[0]); $i++){
		preg_match($tag_url,$result[0][$i],$number);//提出出文章题目编号
		$result[0][$i]=preg_replace($tag_name,'',$result[0][$i]);//提取出文章题目
		$result[1][$i]=$number[0];
	}
	return $result;
}
```
同理，我们观看文章详情页面发现都在id="article_content"的div里面，同时这个块后面一行内容为

```
<!-- Baidu Button BEGIN -->
```
所以我们可以这样写正则表达式

```
<div id="article_content" class="article_content">[\w\W]*<\/div>[\w\W]*<!-- B
```
则提取文章内容的函数应该这样写

```
// 获取文章内容
function getArticleContent($number,$username){
	$url = "http://blog.csdn.net/".$username."/article/details/".$number;
	$content = getWebContent($url);
	$tag = '/<div id="article_content" class="article_content">[\w\W]*<\/div>[\w\W]*<!-- B/'; //匹配正文内容
	preg_match($tag,$content,$main);//提出正文内容
	return $main[0];
}

```
再看看总页数，观察后可以很容易写出下列函数

```

//获取文章总页数
function getPageNumber($content){
	$tag = '/共[0-9]+页/';
	$tagNumber = '/[0-9]+/';
	preg_match($tag, $content,$result);
	preg_match($tagNumber, $result[0],$sumPage);
	return $sumPage[0];
}

```

同样我们继续观察img的格式，可以得出正则表达式

```
http:\/\/img.blog.csdn.net\/.*?\/Center
```

提取图片的函数为

```
//提取-文章内容中的图片
function getImage($content){
	$tag = '/http:\/\/img.blog.csdn.net\/.*?\/Center/'; 
	preg_match_all($tag, $content, $result);//筛选出图片
	return $result[0];//图片链接数组
}
```

然后是对提取出的图片的链接进行下载

```
//下载图片
function downloadImg($url,$id,$imgName){
	if(!is_dir("Img\\".$id))
	{
	  mkdir("Img\\".$id);
	}
	for ($i=0; $i <sizeof($url) ; $i++) { 
		$curl = curl_init();
		curl_setopt($curl,CURLOPT_URL, $url[$i]);
		curl_setopt ($curl, CURLOPT_HEADER, false);
		curl_setopt ($curl, CURLOPT_RETURNTRANSFER, true);
		$result = curl_exec($curl);
		curl_close($curl);
		file_put_contents($imgName[$i],$result);
	}
}
```

由于下载了图片，为了能够离线观看文章，所以还得把文章中img的src中的链接替换为本地连接，可以用下面函数

```
//替换-文章内容中的图片链接为本地
function replaceImgUrl($content,$local_url){
	$tag = '/http:\/\/img.blog.csdn.net\/.*?\/Center/'; 
	$tag_array = array();
	for ($i=0; $i < sizeof($local_url) ; $i++) { 
		$tag_array[$i] = $tag;
	}
	$count = 1;
	$st = preg_replace($tag_array, $local_url, $content,$count);
	return $st;
}
```

整个爬虫大概就是这样了，最后再提示一遍，完整的文件在我的github上面
[CSDNSpider](https://github.com/ShanaMaid/CSDNSpider)





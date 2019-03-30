---
layout: post
title: "如何科学使用Disqus"
subtitle: "How to use Disqus in your awesome Jekyll blog"
author: "Kaguya"
header-img: "https://res.cloudinary.com/dds4xetpt/image/upload/v1553520774/posts/2019-03-23/jekyll-webpack.png"
header-mask: 0.6
tags:
  - Jekyll
  - Disqus
  - 网络应用
  - Docker
  - PHP
  - Nginx
---
> 第一篇markdown，第一篇Jekyll

# 前言
由于某些众所周知的原因，Jekyll已经不能正常加载Disqus评论系统了，而国内的替代品之一——多说，也早在两年前就关门了，而网易推出的社会化评论系统并不支持国外的SNS，以及全球通用的Gravatar，是的，Gravater这一全球通用头像系统也早在很多年前就悄无声息地被那啥了。Jekyll作为目前很受程序员欢迎的静态个人博客，想要在其中嵌入评论系统只能借助第三方，想要使用Disqus和Gravatar，怎么解决呢？于是乎想到了万能的Nginx反向代理，当然实现这些的前提是你得有一台常年驻扎在海外且稳如poi的VPS嗯。

# 准备
一台国外的VPS  
Docker + docker-compose 环境  
php-fpm  
Nginx  
Disqus API  

# 正文
核心部分应该是Disqus的API转发服务了，所幸已经有先驱用PHP写了一整套的调用，在GitHub上放了一个repo，即[Disqus PHP API][disqus-php-api]{:target="_blank"}，repo中也详细说明了如何修改前端脚本  
和页面调用来访问Disqus的获取评论列表和发表评论的API，前端的修改我们稍后再叙，先来看看API后端的部署吧。

### Docker + docker-compose 环境搭建
（略）

### Disqus PHP API的部署
我fork了上述仓库的项目并且自己写了个docker-compose的自动部署脚本，存放在了[这个repo][jimleestone-disqus-php-api]{:target="_blank"}中。

在你的VPS上clone这个repo  
```
git clone https://github.com/jimleestone/disqus-php-api.git
```
修改api目录下的config.php文件为你自己Disqus open API的相关配置，请自行准备梯子或其他工具注册一个Disqus账号并且申请他们的API KEY，如果你已经有了Disqus账号，请直接去往[API KEY申请][disqus-api-apply]{:target="_blank"}页面进行申请，其中Callback URL先随便填写，之后马上会进行修改，而具体如何配置 `config.php` 在该文件中已经有比较详细的说明了  
```
cd disqus-php-api
nano api/config.php
```
修改nginx配置文件 `default.conf` ，将 `your domain or your ip address` 替换成你VPS的IP地址或你需要绑定的域名  
```
server {
    index index.php index.html;
    server_name your domain or your ip address;
    root /var/www/html;
  ...
```
接下来就是使用 `docker-compose` 命令自动部署了  
```
docker-compose up -d
```
假设你的域名是 `example.com`，那么接下来请回到[Disqus API申请][disqus-api-apply]{:target="_blank"}页面，在Settings中配置回调地址Callback URL为 `http://example.com/login.php`，保存成功后，浏览器访问这个地址，会跳转到Disqus API的授权页面，至此API后端搭建完了  

### 博客前端配置修改

首先是在你博客中引入repo的dist目录下的css和js文件
```html
<link rel="stylesheet" href="path/to/iDisqus.min.css" />
<script src="path/to/iDisqus.min.js"></script>
```
需要将标准写法的Disqus页面标签  
```html
<div class="comment">
  <div id="disqus_thread" class="disqus-thread"></div>
</div>
```
修改为
```html
<div id="comment"></div>
```
JS调用也需要修改  
```javascript
(function() {
  var dsq = document.createElement('script');
  dsq.type = 'text/javascript';
  dsq.async = true;
  dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
  (document.getElementsByTagName('head')[0] || 
    document.getElementsByTagName('body')[0])
    .appendChild(dsq);
})();
```
改为  
```javascript
var disq = new iDisqus('comment', {
  forum: your_shortname, //你Disqus站点的shortname
  site: 'http://blog.example.com', //你博客的网址
  api: 'http://api.example.com', //刚才部署的API地址
  url: disqus_url, //当前文章的url
  identifier: disqus_identifier, //当前文章的识别id，不填即为url
  init: true
});
disq.count();
```
大功告成。

# 总结

用来用去还是Docker好~

[disqus-php-api]: https://github.com/fooleap/disqus-php-api
[docker-nginx-phpfpm]: https://github.com/mochizukikotaro/docker-nginx-phpfpm
[jimleestone-disqus-php-api]: https://github.com/jimleestone/disqus-php-api
[disqus-api-apply]: https://disqus.com/api/applications/
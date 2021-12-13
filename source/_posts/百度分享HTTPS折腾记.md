---
title: 百度分享HTTPS折腾记
date: 2017-06-26 22:48:20
tags:
- hexo
- bdshare
categories:
- 701
---
>百度分享很坑！！

### 发现问题

今天打开博客忽然发现百度分享用不了了，点击分享按钮没有任何作用，于是打开控制台查看：
![](https://qiniu.nihaoshijie.com.cn/blog/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-26%2021.56.04.png)
##### 这是什么鬼！
<!--more-->
忽然想起来，之前我也是为了解决百度分享不支持https时，参考网上的做法将百度分享的js文件，也就是那个static文件夹拷贝到本地使用。
[参考这篇教程](https://www.hrwhisper.me/baidu-share-not-support-https-solution/)


### 解决问题

但是怎么今天突然不行了呢，我敢肯定当时改完时没问题的，于是尝试找到出错的地方，进行修复：
1. 这些文件都是被压缩的，想要直接修改还需要将文件反压缩一下。
2. 找到出错文件share.js，出错原因大概是语法错误，因为不知道逻辑，所以将出错的代码行删掉了。
3. 原本天真的以为万事大吉，谁知刷新之后🈶又出现两个新的错误。

![](https://qiniu.nihaoshijie.com.cn/blog/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-26%2021.56.42.png)

##### 这又是什么鬼？

难道百度源码都是没有测试过的么？？

1. 找到出问题的文件tangram.js,代码又是被压缩的，找到出问题代码fix之。
2. 找到出问题的文件view_base.js,代码又是被压缩的，找到出问题代码fix之。
3. 刷新一看，尼玛怎么又冒出新的错误？
![](https://qiniu.nihaoshijie.com.cn/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-27%2011.55.17.png)

##### 心累。。
于是又开始想别的办法，发誓不在修改百度分享的源码了。。

### 完美解决

经过一系列的查找的求助，终于找到了完美的结局办法：
1. 百度分享的js源码其实也有部署在百度的另外一个支持https的cdn上即：[](https://ss1.baidu.com/9rA4cT8aBw9FktbgoI7O1ygwehsv/static/api/js/share.js).
2. 利用这个cdn的url同样可以得到其他js源码文件的地址：
https://ss1.baidu.com/9rA4cT8aBw9FktbgoI7O1ygwehsv/static/api/js/base/tangram.js
https://ss1.baidu.com/9rA4cT8aBw9FktbgoI7O1ygwehsv/static/api/js/base/class.js
3. 将原本的bdimg.share.baidu.com修改为ss1.baidu.com/9rA4cT8aBw9FktbgoI7O1ygwehsv，同时将http换成https即可。
4. 但这并没有结束如果只改了share.js 这个js里面引入其他js同样会是http的，所以还需要将share.js inline到script标签里面，同时修改
```javascript
jscfg:{domain:{staticUrl:"https://ss1.baidu.com/9rA4cT8aBw9FktbgoI7O1ygwehsv/"}}}
```
将用于模块化加载的js的地址也替换成新的。


#### 大功告成！
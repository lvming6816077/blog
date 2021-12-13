---
title: HTML5离线存储之Application Cache
date: 2014-08-28 16:50:58
tags:
- 离线存储
- HTML5
categories:
- 425
---
关于html5的离线存储，大致可分为：
<ul>
	<li><span style="color: #303942;">localStorage, sessionStorage</span></li>
	<li>indexedDB</li>
	<li>web sql</li>
	<li>application cache</li>
</ul>
<!--more-->
可以在chrome的debug工具/Resources下产看，下面来着重说明一下Application Cache。
<h3>访问流程</h3>
<img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/applicationcache.png" alt="" width="922" height="305" />
当我们第一次正确配置app cache后，当我们再次访问该应用时，浏览器会首先检查manifest文件是否有变动，如果有变动就会把相应的变得跟新下来，同时改变浏览器里面的app cache，如果没有变动，就会直接把app cache的资源返回，基本流程是这样的。

&nbsp;
<h3>特点</h3>
<ul style="color: #4d4e53;">
	<li>离线浏览: 用户可以在离线状态下浏览网站内容。</li>
	<li>更快的速度: 因为数据被存储在本地，所以速度会更快.</li>
	<li>减轻服务器的负载: 浏览器只会下载在服务器上发生改变的资源。</li>
</ul>
&nbsp;
<h3>如何使用</h3>
首先，我们建立一个html文件，类似这样：
```html
<!DOCTYPE html>
<html lang="en" manifest="manifest.appcache">
<head>
    <meta charset="UTF-8">
    <title>APP CACHE</title>
    <link rel="stylesheet" type="text/css" href="test.css">
</head>
<body>
    <img src="img/1.jpg">
    <img src="img/2.jpg">
<script type="text/javascript">
    window.addEventListener('load', function(e){
        console.log(window.applicationCache.status);
    })
</script>
</body>
</html>
```
可能有些代码看不懂，我们先看最简单的，第一行配置了一个manifest="manifest.appcache"（<span style="color: #ff0000;">注意是mani不是main</span>），这是使用app cache首先要配置的，然后我们在这个html文件里引入了两个img做为测试用，然后监听了load时间来查看看application的status，关于applicationCache的api，可以<a href="http://www.w3school.com.cn/html5/html_5_app_cache.asp" target="_blank">查看</a>。

然后在相同目录下新建一个manifest.appcache文件，注意关于路径要和html页面配置时一致即可。
<pre class="lang:default decode:true">CACHE MANIFEST
#version 1.3
CACHE:
    img/1.jpg
    img/2.jpg
    test.css
NETWORK:
    *</pre>
关于manifest.appcache文件，基本格式为<span style="color: #4d4e53;">三段： </span><code style="color: #4d4e53;">CACHE，</code><span style="color: #4d4e53;"> </span><code style="color: #4d4e53;">NETWORK，</code><span style="color: #4d4e53;">与 </span><code style="color: #4d4e53;">FALLBACK，其中NETWORK和FALLBACK为可选项，而第一行CACHE MANIFEST为固定格式，必须写在前面。</code>

<strong><code style="font-weight: inherit; font-style: inherit;">CACHE:（必须）</code></strong>

标识出哪些文件需要缓存，可以是相对路径也可以是绝对路径。例如：aa.css，http://www.baidu.com/aa.js.



<strong><code style="font-weight: inherit; font-style: inherit;">NETWORK:（可选）</code></strong>

这一部分是要绕过缓存直接读取的文件，可以使用通配符＊，,也就是说除了上面的cache文件，剩下的文件每次都要重新拉取。例如＊，login.php。



<strong><code style="font-weight: inherit; font-style: inherit;">FALLBACK:（可选）</code></strong>

指定了一个后备页面，当资源无法访问时，浏览器会使用该页面。该段落的每条记录都列出两个 URI—第一个表示资源，第二个表示后备页面。两个 URI 都必须使用相对路径并且与清单文件同源。可以使用通配符。例如*.html  /offline.html。

有了上面两个文件之后还要配置服务器的mime.types类型，找大盘apache的mime.types文件，添加
<pre class="lang:default decode:true ">text/cache-manifest .appcache</pre>
OK，上面文件配置完成之后，application cache就可以运行了。

查看console：

<img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/appcache2.png" alt="" width="1010" height="420" />
可以看到，一下子这么多log，但是除了4是我们console的log之外，其他的都是appcache自己打的，因为我们配置了manifest，系统会默认打出appcache的log。关于status的值：

然后，通过log，我们看到一些文件已经被缓存，我们可以查看chrome Resources来查看：

<img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/appcache3.png" alt="" width="1439" height="277" />

可以看到我们的test.html文件也已经被缓存下来了，type是master，顾名思义一个管理着，而manifest.appcache文件为manifest类型。此时我们的appcache已经完成。我们可以尝试把网线断了，或者把服务器关了，同样，我们的项目仍然可以访问，这就是离线缓存。此时console：

<img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/appcache4.png" alt="" width="1010" height="218" />

证明直接从缓存拿去文件：

<img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/appcache5.png" alt="" width="794" height="267" />

<strong>更新缓存的方式</strong>
更新manifest文件
浏览器发现manifest文件本身发生变化，便会根据新的manifest文件去获取新的资源进行缓存。

当manifest文件列表并没有变化的时候，我们通常通过修改manifest注释的方式来改变文件，从而实现更新。
通过javascript操作
浏览器提供了applicationCache供js访问，通过对于applicationCache对象的操作也能达到更新缓存的目的。
清除浏览器缓存

<div>对于第一种，我们修改一下manifest文件，把version改为1.4，然后刷新页面。</div>
<div><img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/appcache6.png" alt="" width="1009" height="292" /></div>
<div>我们可以发现，appcache更新了缓存重新从网络上拉去的cache的文件，但是，我们如果想要看到改变，必须再次刷新页面。</div>
<div></div>
<div>对于第二种方法，我们首先修改一下我们的js，添加一个监听事件：</div>
<div>
```javascript
window.applicationCache.addEventListener('updateready', function(){
        console.log('updateready!');
        window.applicationCache.swapCache();
    });
```
清除浏览器缓存再试一次，这次我们在console里调用window.applicationCache.update();，看看发生了什么：

<img class="alignnone" src="http://qiniu.nihaoshijie.com.cn/appcache7.png" alt="" width="864" height="211" />

updateready事件触发了，同样，appcache也更新了缓存，其中swapCache方法的意思是重新应用跟新后的缓存来替换原来的缓存！，到这里后基本的appcache也差不多了。
<h3>注意事项：</h3>
<ul>
	<li>站点离线存储的容量限制是5M</li>
	<li>如果manifest文件，或者内部列举的某一个文件不能正常下载，整个更新过程将视为失败，浏览器继续全部使用老的缓存</li>
	<li>引用manifest的html必须与manifest文件同源，在同一个域下</li>
	<li>FALLBACK中的资源必须和manifest文件同源</li>
	<li>当一个资源被缓存后，该浏览器直接请求这个绝对路径也会访问缓存中的资源。</li>
	<li>站点中的其他页面即使没有设置manifest属性，请求的资源如果在缓存中也从缓存中访问</li>
	<li>当manifest文件发生改变时，资源请求本身也会触发更新</li>
</ul>
完！

</div>
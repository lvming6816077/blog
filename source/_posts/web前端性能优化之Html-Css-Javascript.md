---
title: web前端性能优化之Html Css Javascript
date: 2015-05-05 16:06:08
tags:
- 移动web
- 性能优化
categories:
- 530
---
<strong><span style="color: #008080;">前言</span></strong>

html css javascript可以算是前端必须掌握的东西了，但是我们的浏览器是怎样解析这些东西的呢 我们如何处理html css javascript这些东西来让我们的网页更加合理，在我这里做了一些实验，总结起来给大家看看。

<!--more-->
<strong><span style="color: #008080;">最简单的页面</span></strong>
```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
  </head>
  <body>
    <img src="download-button.png">
  </body>
</html>
```
我们打开chrome的控制台查看timeline

<img class="alignnone" src="http://cdn.alloyteam.com/wp-content/uploads/2015/04/11.png" alt="" width="1073" height="142" />


由上图 可得结论



1 图中蓝色透明条标识浏览器从发起请求到接收到服务器返回第一个字节的时间，时间还是挺长的，而蓝色实体条则为真正的html页面下载的时间 还是很短的。


2 图中红框内的这部分时间则表示浏览器从下载完成html之后开始构建dom，当发现一个image标签时所花费的时间，由此可见dom是顺序执行的，当发现image时便立即发起请求，而紫色透明条则是image发起请求时在网络传输时所消耗的时间。


3 图中timeline蓝色竖线所处的时间为domComplete时间，红色竖线为dom的onload时间，由此可见两种事件的差异。而浏览器构建dom树所花费的时间可以算出即domComplete时间 减去 html下载完成后的时间大概80ms。


<strong><span style="color: #008080;">含有css的页面</span></strong>
<pre class="lang:default decode:true  ">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
    &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
我们打开chrome的控制台查看timeline

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2015/05/2.png"><img class="alignleft wp-image-623 size-full" src="http://www.alloyteam.com/wp-content/uploads/2015/04/2.png" alt="2" width="1072" height="196" /></a>

&nbsp;


&nbsp;

&nbsp;

1 在添加了外部引入css之后，并没有发现什么异常，但是有一点指的注意，也就是红色竖线和蓝色竖线挨得更进了，这表明domComplete时间必须等待css解析完成，也就是构建dom树必须等待css解析完成，这也就解释了下图

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2015/05/3.png"><img class="alignnone wp-image-624 size-full" src="http://www.alloyteam.com/wp-content/uploads/2015/04/3.png" alt="3" width="734" height="348" /></a>

&nbsp;

<hr />

<strong><span style="color: #008080;">含有javascript和css的页面</span></strong>
<pre class="lang:default decode:true ">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;script type="text/javascript" src="H5FullscreenPage.js"&gt;&lt;/script&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
&nbsp;

我们打开chrome的控制台查看timeline

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2015/05/4.png"><img class="alignnone size-full wp-image-625" src="http://www.nihaoshijie.com.cn/wp-content/uploads/2015/05/4.png" alt="4" width="1221" height="227" /></a>

&nbsp;

1 图上显示在引入外部的js文件之后domComplete时间又被延后了，结合上面的renderTree，由于javascript代码可能会更改css属性或者是dom结构，所以在形成renderTree之前必须等待javascript解析完成才能接着构建renderTree。

2 将javascript放在head内和body底部的区别也在于此，放在head里面，由于浏览器发现head里面有javascript标签就会暂时停止其他渲染行为，等待javascript下载并执行完成才能接着往下渲染，而这个时候由于在head里面这个时候页面是白的，如果将javascript放在页面底部，renderTree已经完成大部分，所以此时页面有内容呈现，即使遇到javascript阻塞渲染，也不会有白屏出现。

<hr />

<strong><span style="color: #008080;">内嵌javascript的页面</span></strong>

&nbsp;

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2015/05/51.png"><img class="alignnone size-full wp-image-626" src="http://www.alloyteam.com/wp-content/uploads/2015/04/51.png" alt="51" width="1222" height="213" /></a>

1 图上可以看到，由于内嵌了javascript，页面上减少了一个请求，导致html文档变大，消耗时间增多，但是domComplete时间提升的并不多。

<hr />

<span style="color: #008080;"><strong>使用async的javascript</strong></span>
<pre class="lang:default decode:true">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
      &lt;script async src="H5FullscreenPage.js" type ="text/javascript" &gt;&lt;/script &gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
&nbsp;

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2015/05/6.png"><img class="alignnone size-full wp-image-627" src="http://www.alloyteam.com/wp-content/uploads/2015/05/6.png" alt="6" width="1215" height="348" /></a>

1 可以看到domComplete时间被大大提前 javascript也没有阻塞css和body里面img元素的并行下载。

2 使用<a href="http://www.w3school.com.cn/tags/att_script_async.asp">async</a>标识的script，浏览器将异步执行这中script不会阻塞正常的dom渲染，这时html5所支持的属性，另外<a href="http://www.w3school.com.cn/tags/att_script_defer.asp">defer</a>也可以达到这种效果。

<hr />

<strong><span style="color: #008080;">head里面js和css加载的关系</span></strong>

<strong>外联js在css前面</strong>
<pre class="lang:default decode:true">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;script src="H5FullscreenPage.js" type ="text/javascript" &gt;&lt;/script &gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
      &lt;link rel="stylesheet" type="text/css" href="page-animation.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
<img class="alignnone" style="height: 442px; width: 1221px;" src="http://1.lvming6816077.sinaapp.com/testaa/xx1.png" alt="" />

1 没有阻止css的并行加载但是影响了body里面img的并行加载

<hr />

<strong>外联js在css中间</strong>
<pre class="lang:default decode:true">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
      &lt;script src="H5FullscreenPage.js" type ="text/javascript" &gt;&lt;/script &gt;
      &lt;link rel="stylesheet" type="text/css" href="page-animation.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
<img class="alignnone" style="height: 382px; width: 1208px;" src="http://1.lvming6816077.sinaapp.com/testaa/xx2.png" alt="" />

1 影响了css的并行加载和body里面img的并行加载

<hr />

<strong>外联js在css最后</strong>
<pre class="lang:default decode:true ">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
      &lt;link rel="stylesheet" type="text/css" href="page-animation.css" media="screen"&gt;
      &lt;script src="H5FullscreenPage.js" type ="text/javascript" &gt;&lt;/script &gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
&nbsp;

<img class="alignnone" style="height: 346px; width: 1228px;" src="http://1.lvming6816077.sinaapp.com/testaa/xx3.png" alt="" />

1 影响了css的并行加载和body里面img的并行加载

<hr />

<strong>内嵌js在css前面</strong>
<pre class="lang:default decode:true ">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;script type ="text/javascript" &gt;
      	var f = 1;
          f++;
      &lt;/script &gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
      &lt;link rel="stylesheet" type="text/css" href="page-animation.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
&nbsp;

<img class="alignnone" style="height: 451px; width: 1229px;" src="http://1.lvming6816077.sinaapp.com/testaa/xx4.png" alt="" />

1 没有影响css的并行加载也没有影响body里面img的并行加载

<hr />

<strong>内嵌js在css中间</strong>
<pre class="lang:default decode:true ">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
      &lt;script type ="text/javascript" &gt;
      	var f = 1;
          f++;
      &lt;/script &gt;
      &lt;link rel="stylesheet" type="text/css" href="page-animation.css" media="screen"&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
&nbsp;

<img class="alignnone" style="height: 437px; width: 1219px;" src="http://1.lvming6816077.sinaapp.com/testaa/xx5.png" alt="" />

1 影响了css的并行加载没有英雄body里面img的并行加载

<hr />

<strong>内嵌js在css最后</strong>
<pre class="lang:default decode:true ">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;test&lt;/title&gt;
      &lt;link rel="stylesheet" type="text/css" href="stylesheet.css" media="screen"&gt;
      &lt;link rel="stylesheet" type="text/css" href="page-animation.css" media="screen"&gt;
      &lt;script type ="text/javascript" &gt;
      	var f = 1;
          f++;
      &lt;/script &gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;img src="download-button.png"&gt;
  &lt;/body&gt;
&lt;/html&gt;</pre>
&nbsp;

<img class="alignnone" style="height: 350px; width: 1228px;" src="http://1.lvming6816077.sinaapp.com/testaa/xx6.png" alt="" />

1 影响了css和body里面img的并行加载。

<hr />

<strong>综上所述：</strong>

<span style="font-family: verdana,arial,helvetica,sans-serif; font-size: 14px;">当浏览器从服务器接收到了HTML文档，并把HTML在内存中转换成DOM树，在转换的过程中如果发现某个节点(node)上引用了CSS或者 IMAGE，就会再发1个request去请求CSS或image,然后继续执行下面的转换，而不需要等待request的返回，当request返回 后，只需要把返回的内容放入到DOM树中对应的位置就OK。但当引用了JS的时候，浏览器发送1个js request就会一直等待该request的返回。因为浏览器需要1个稳定的DOM树结构，而JS中很有可能有代码直接改变了DOM树结构，浏览器为了防止出现JS修改DOM树，需要重新构建DOM树的情况，所以 就会阻塞其他的下载和呈现.</span>

&nbsp;

<span style="font-size: 14px;">这里的结论：</span>

<span style="font-size: 14px;">1 在head里面尽量不要引入javascript.</span>

<span style="font-size: 14px;">2 如果要引入js 尽量将js内嵌.</span>

<span style="font-size: 14px;">3 把内嵌js放在所有css的前面.</span>

&nbsp;

<strong><span style="color: #008080;">后记</span></strong>

<span style="font-size: 14px;">1 本次的测试页面 </span><a href="http://1.lvming6816077.sinaapp.com/testaa/demo.html">http://1.lvming6816077.sinaapp.com/testaa/demo.html</a>

2 测试所用浏览器 chrome

3 参考资料：<a href="http://www.zhihu.com/question/20357435/answer/14878543">http://www.zhihu.com/question/20357435/answer/14878543</a>

<a href="http://www.haorooms.com/post/web_xnyh_jscss">http://www.haorooms.com/post/web_xnyh_jscss</a>

4 如果有哪里说的不清楚或者错误的地方，欢迎留言反馈。
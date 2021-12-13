---
title: HTTP2 Server Push 研究
date: 2016-11-27 20:40:27
tags:
- http2
categories:
- 651
photos: https://qiniu.nihaoshijie.com.cn/7542543_SPDY-to-HTTP2-Upgrade-and-Nginx_thumb.jpg
---
<h2>1，HTTP2的新特性。</h2>
关于HTTP2的新特性，读着可以参看我之前的文章，这里就不在多说了，本篇文章主要讲一下server push这个特性。

<a href="http://km.oa.com/group/19674/articles/show/276552" target="_blank" rel="noopener noreferrer">HTTPS与HTTP2的原理，搭建，性能测试对比</a>

<a href="http://km.oa.com/articles/view/290595" target="_blank" rel="noopener noreferrer">HTTP,HTTP2.0,SPDY,HTTPS你应该知道的一些事</a>
<!--more-->
<h2>2，Server Push是什么。</h2>
简单来讲就是当用户的浏览器和服务器在建立链接后，服务器主动将一些资源推送给浏览器并缓存起来，这样当浏览器接下来请求这些资源时就直接从缓存中读取，不会在从服务器上拉了，提升了速率。举一个例子就是：

假如一个页面有3个资源文件<strong>index.html</strong>,<strong>index.css</strong>,<strong>index.js</strong>,当浏览器请求index.html的时候，服务器不仅返回index.html的内容，同时将index.css和index.js的内容push给浏览器，当浏览器下次请求这2两个文件时就可以直接从缓存中读取了。
<h2></h2>
<h2>3，Server Push原理是什么。</h2>
要想了解server push原理，首先要理解一些概念。我们知道HTTP2传输的格式并不像HTTP1使用文本来传输，而是启用了二进制帧(Frames)格式来传输，和server push相关的帧主要分成这几种类型：
<ol>
 	<li>HEADERS frame(请求返回头帧):这种帧主要携带的http请求头信息，和HTTP1的header类似。</li>
 	<li>DATA frames(数据帧) :这种帧存放真正的数据content，用来传输。</li>
 	<li>PUSH_PROMISE frame(推送帧):这种帧是由server端发送给client的帧，用来表示server push的帧，这种帧是实现server push的主要帧类型。</li>
 	<li>RST_STREAM(取消推送帧):这种帧表示请求关闭帧，简单讲就是当client不想接受某些资源或者接受timeout时会向发送方发送此帧，和PUSH_PROMISE frame一起使用时表示拒绝或者关闭server push。</li>
</ol>
<span style="font-weight: bolder;">Note:</span>HTTP2.0相关的帧其实包括<a href="https://tools.ietf.org/html/rfc7540#section-11.2" target="_blank" rel="noopener noreferrer">10种帧</a>，正是因为底层数据格式的改变，才为HTTP2.0带来许多的特性，帧的引入不仅有利于压缩数据，也有利于数据的安全性和可靠传输性。

<strong>了解了相关的帧类型，下面就是具体server push的实现过程了：</strong>
<ol>
 	<li>由多路复用我们可以知道HTTP2中对于同一个域名的请求会使用一条tcp链接而用不同的stream ID来区分各自的请求。</li>
 	<li>当client使用stream 1请求index.html时，server正常处理index.html的请求，并可以得知index.html页面还将要会请求index.css和index.js。</li>
 	<li>server使用stream 1发送PUSH_PROMISE frame给client告诉client我这边可以使用stream 2来推送index.js和stream 3来推送index.css资源。</li>
 	<li>server使用stream 1正常的发送HEADERS frame和DATA frames将index.html的内容返回给client。</li>
 	<li>client接收到PUSH_PROMISE frame得知stream 2和stream 3来接收推送资源。</li>
 	<li>server拿到index.css和index.js便会发送HEADERS frame和DATA frames将资源发送给client。</li>
 	<li>client拿到push的资源后会缓存起来当请求这个资源时会从直接从从缓存中读取。</li>
</ol>
下图表示了整个流程：

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2016/11/屏幕快照-2016-11-27-19.07.58.png"><img class="alignnone size-full wp-image-656" src="http://www.nihaoshijie.com.cn/wp-content/uploads/2016/11/屏幕快照-2016-11-27-19.07.58.png" alt="%e5%b1%8f%e5%b9%95%e5%bf%ab%e7%85%a7-2016-11-27-19-07-58" width="2024" height="1448" /></a>
<h2>4，Server Push怎么用。</h2>
既然server push这么神奇，那么我们如何使用呢？怎么设置服务器push哪些文件呢？

首先并不是所有的服务器都支持server push，nginx目前还不支持这个特性，可以在nginx的官方博客上得到证实<a href="https://www.nginx.com/blog/http2-r7/" target="_blank" rel="noopener noreferrer">https://www.nginx.com/blog/http2-r7/</a>，但是Apache和nodejs都已经支持了server push这一个特性，需要说明一点的是server push这个特性是基于浏览器和服务器的，所以浏览器并没有提供相应的js api来让用户直接操作和控制push的内容，所以只能是通过header信息和server的配置来实现具体的push内容，本文主要以nodejs来说明具体如何使用server push这一特性。

<strong>准备工作</strong>：下载<a href="https://github.com/molnarg/node-http2" target="_blank" rel="noopener noreferrer">nodejs http2</a>支持，本地启动nodejs服务。

<strong>1. 首先我们使用nodejs搭建基本的server：</strong>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">var</span> http2 = <span class="hljs-built_in">require</span>(<span class="hljs-string">'http2'</span>);

<span class="hljs-keyword">var</span> url=<span class="hljs-built_in">require</span>(<span class="hljs-string">'url'</span>);
<span class="hljs-keyword">var</span> fs=<span class="hljs-built_in">require</span>(<span class="hljs-string">'fs'</span>);
<span class="hljs-keyword">var</span> mine=<span class="hljs-built_in">require</span>(<span class="hljs-string">'./mine'</span>).types;
<span class="hljs-keyword">var</span> path=<span class="hljs-built_in">require</span>(<span class="hljs-string">'path'</span>);

<span class="hljs-keyword">var</span> server = http2.createServer({
  key: fs.readFileSync(<span class="hljs-string">'./zs/localhost.key'</span>),
  cert: fs.readFileSync(<span class="hljs-string">'./zs/localhost.crt'</span>)
}, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">request, response</span>) </span>{
    <span class="hljs-keyword">var</span> pathname = url.parse(request.url).pathname;
    <span class="hljs-keyword">var</span> realPath = path.join(<span class="hljs-string">"my"</span>, pathname);    <span class="hljs-comment">//这里设置自己的文件名称;</span>

    <span class="hljs-keyword">var</span> pushArray = [];
    <span class="hljs-keyword">var</span> ext = path.extname(realPath);
    ext = ext ? ext.slice(<span class="hljs-number">1</span>) : <span class="hljs-string">'unknown'</span>;
    <span class="hljs-keyword">var</span> contentType = mine[ext] || <span class="hljs-string">"text/plain"</span>;

    <span class="hljs-keyword">if</span> (fs.existsSync(realPath)) {

        response.writeHead(<span class="hljs-number">200</span>, {
            <span class="hljs-string">'Content-Type'</span>: contentType
        });

        response.write(fs.readFileSync(realPath,<span class="hljs-string">'binary'</span>));

    } <span class="hljs-keyword">else</span> {
      response.writeHead(<span class="hljs-number">404</span>, {
          <span class="hljs-string">'Content-Type'</span>: <span class="hljs-string">'text/plain'</span>
      });

      response.write(<span class="hljs-string">"This request URL "</span> + pathname + <span class="hljs-string">" was not found on this server."</span>);
      response.end();
    }

});

server.listen(<span class="hljs-number">443</span>, <span class="hljs-function"><span class="hljs-keyword">function</span>() </span>{
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'listen on 443'</span>);
});</code></pre>
</div>
这几行代码就是简单搭建一个nodejs http2服务，打开chrome，我们可以看到所有请求都走了http2，同时也可以验证多路复用的特性。

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2016/11/h21.png"><img class="alignnone size-full wp-image-657" src="http://www.nihaoshijie.com.cn/wp-content/uploads/2016/11/h21.png" alt="h21" width="1327" height="257" /></a>

<strong>这里需要注意几点：</strong>
<ol>
 	<li>创建http2的nodejs服务必须时基于https的，因为现在主流的浏览器都要支持SSL/TLS的http2，证书和私钥可以自己通过<a href="https://www.openssl.org/" target="_blank" rel="noopener noreferrer">OPENSSL</a>生成。</li>
 	<li>node http2的相关api和正常的node httpserver相同，可以直接使用。</li>
</ol>
<span style="font-weight: bolder;">2. 设置我们的server push：</span>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">var</span> pushItem = response.push(<span class="hljs-string">'/css/bootstrap.min.css'</span>, {
       request: {
            accept: <span class="hljs-string">'*/\*'</span>
       },
      response: {
            <span class="hljs-string">'content-type'</span>: <span class="hljs-string">'text/css'</span>
     }
});
pushItem.end(fs.readFileSync(<span class="hljs-string">'/css/bootstrap.min.css'</span>,<span class="hljs-string">'binary'</span>));</code></pre>
</div>
我们设置了bootstrap.min.css来通过server push到我们的浏览器,我们可以在浏览器中查看：

<a href="http://www.nihaoshijie.com.cn/wp-content/uploads/2016/11/h22.png"><img class="alignnone size-full wp-image-658" src="http://www.nihaoshijie.com.cn/wp-content/uploads/2016/11/h22.png" alt="h22" width="1716" height="356" /></a>

可以看到，启动server push的资源timelime非常快，大大加速了css的获取时间。

<strong>这里需要注意下面几点：</strong>
<ol>
 	<li>我们调用response.push(),就是相当于server发起了PUSH_PROMISE frame来告知浏览器bootstrap.min.css将会由server push来获取。</li>
 	<li>response.push()返回的对象时一个正常的ServerResponse,end(),writeHeader()等方法都可以正常调用。</li>
 	<li>这里一旦针对某个资源调用response.push()即发起PUSH_PROMISE frame后，要做好容错机制，因为浏览器在下次请求这个资源时会且只会等待这个server push回来的资源，这里要做好超时和容错即下面的代码：</li>
 	<li>
<div class="km_insert_code">
<pre><code><span class="hljs-keyword">try</span> {
    pushItem.end(fs.readFileSync(<span class="hljs-string">'my/css/bootstrap.min.css'</span>,<span class="hljs-string">'binary'</span>));
    } <span class="hljs-keyword">catch</span>(e) {
       response.writeHead(<span class="hljs-number">404</span>, {
           <span class="hljs-string">'Content-Type'</span>: <span class="hljs-string">'text/plain'</span>
       });
       response.end(<span class="hljs-string">'request error'</span>);
}

pushItem.stream.on(<span class="hljs-string">'error'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">err</span>)</span>{
    response.end(err.message);
});

pushItem.stream.on(<span class="hljs-string">'finish'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">err</span>)</span>{
   <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'finish'</span>);
});</code></pre>
</div>
上面的代码你可能会发现许多和正常nodejs的httpserver不一样的东西，那就是stream，其实整个http2都是以stream为单位，这里的stream其实可以理解成一个请求，更多的api可以参考：<a href="https://github.com/molnarg/node-http2/wiki/Public-API" target="_blank" rel="noopener noreferrer">node-http2</a>。</li>
 	<li>最后给大家推荐一个老外写的专门服务http2的node server有兴趣的可以尝试一下。<a href="https://gitlab.com/sebdeckers/http2server" target="_blank" rel="noopener noreferrer">https://gitlab.com/sebdeckers/http2server</a></li>
</ol>
<h2></h2>
<h2>5，Server Push相关问题。</h2>
<ol>
 	<li>我们知道现在我们web的资源一般都是放在CDN上的，那么CDN的优势和server push的优势有何区别呢，到底是哪个比较快呢？这个问题笔者也一直在研究，本文的相关demo都只能算做一个演示，具体的线上实践还在进行中。</li>
 	<li>由于HTTP2的一些新特性例如多路复用，server push等等都是基于同一个域名的，所以这可能会对我们之前对于HTTP1的一些优化措施例如(资源拆分域名，合并等等)不一定适用。</li>
 	<li>server push不仅可以用作拉取静态资源，我们的cgi请求即ajax请求同样可以使用server push来发送数据。</li>
 	<li>最完美的结果是CDN域名支持HTTP2,web server域名也同时支持HTTP2。</li>
</ol>
&nbsp;

参考资料：
<ol>
 	<li>HTTP2官方标准：<a href="https://tools.ietf.org/html/rfc7540" target="_blank" rel="noopener noreferrer">https://tools.ietf.org/html/rfc7540</a></li>
 	<li>维基百科：<a href="https://en.wikipedia.org/wiki/HTTP/2_Server_Push" target="_blank" rel="noopener noreferrer">https://en.wikipedia.org/wiki/HTTP/2_Server_Push</a></li>
</ol>
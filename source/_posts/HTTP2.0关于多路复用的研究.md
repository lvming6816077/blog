---
title: HTTP2.0关于多路复用的研究
date: 2017-04-25 16:52:32
tags:
- http2
categories:
- 698

---
关于HTTP2中其他特性的研究可以参考我之前写的文章

<a href="http://km.oa.com/group/19674/articles/show/286253?kmref=kb_categories" target="_blank" rel="noopener noreferrer">关于Server Push的研究</a>
<h3><span style="font-weight: bolder;">问题一：什么是keep live？</span></h3>
HTTP持久连接（HTTP persistent connection，也称作HTTP keep-alive或HTTP connection reuse）是使用同一个TCP连接来发送和接收多个HTTP请求/应答，而不是为每一个新的请求/应答打开新的连接的方法。
<!--more-->
我们知道HTTP协议采用“请求-应答”模式，当使用普通模式，即非KeepAlive模式时，每个请求/应答客户和服务器都要新建一个连接，完成 之后立即断开连接（HTTP协议为无连接的协议）；当使用Keep-Alive模式（又称持久连接）时，Keep-Alive功能使客户端到服 务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。

&nbsp;
<h3><span style="font-weight: bolder;">问题二：keep alive和传统的区别？</span></h3>
<a href="https://qiniu.nihaoshijie.com.cn/blog/kp1.png"><img class="alignnone size-full wp-image-700" src="https://qiniu.nihaoshijie.com.cn/blog/kp1.png" alt="" width="495" height="289" /></a>

一图胜千言，由图可知引入的keep alive的链接不必在每次请求都开启一个新的链接。

&nbsp;
<h3><span style="font-weight: bolder;">问题三：keep alive如何配置？</span></h3>
<span style="font-weight: bolder;">Apache服务器：</span>

httpd.conf：
```php
KeepAlive On             // 是否打开keep alive
MaxKeepAliveRequests 300 // 每个连接最大可复用的请求数
KeepAliveTimeout 3       // 每个请求可复用的time时间s
```
<span style="font-weight: bolder;"> Nginx服务器：</span>

nginx.conf：
```php
keepalive_timeout //服务器接收在10s以内的所有connection复用超过10则关闭建立一个新的connection 0代表关闭keepalive
```
Node.js

<a href="https://nodejs.org/api/http.html#http_class_http_agent" target="_blank" rel="noopener noreferrer">https://nodejs.org/api/http.html#http_class_http_agent</a>

&nbsp;
<h3><span style="font-weight: bolder;">问题4：多路复用是什么？</span><a href="https://qiniu.nihaoshijie.com.cn/blog/kp3.png"><img class="alignnone size-full wp-image-703" src="https://qiniu.nihaoshijie.com.cn/blog/kp3.png" alt="" width="1472" height="234" /></a></h3>
1）HTTP2的请求的TCP的connection一旦建立，后续请求以stream的方式发送。

2）每个stream的基本组成单位是frame（二进制帧），每种frame又分为很多种类型例如HEADERS Frame（头部帧），DATA Frame（内容帧）等等。

3）请求头HEADERS Frame组成了resquest，返回头HEADERS Frame和DATA Frame组成了response，request和response组成了一个stream。

&nbsp;
<h3><span style="font-weight: bolder;">问题5：多路复用和keep alive区别？</span></h3>
<a href="https://qiniu.nihaoshijie.com.cn/kp2.png"><img class="alignnone size-full wp-image-701" src="https://qiniu.nihaoshijie.com.cn/kp2.png" alt="" width="779" height="765" /></a>

1）线头阻塞（Head-of-Line Blocking），HTTP1.X虽然可以采用keep alive来解决复用TCP的问题，但是还是无法解决请求阻塞问题。

2）所谓请求阻塞意思就是一条TCP的connection在同一时间只能允许一个请求经过，这样假如后续请求想要复用这个链接就必须等到前一个完成才行，正如上图左边表示的。

3）之所以有这个问题就是因为HTTP1.x需要每条请求都是可是识别，按顺序发送，否则server就无法判断该相应哪个具体的请求。

4）HTTP2采用多路复用是指，在同一个域名下，开启一个TCP的connection，每个请求以stream的方式传输，每个stream有唯一标识，connection一旦建立，后续的请求都可以复用这个connection并且可以同时发送，server端可以根据stream的唯一标识来相应对应的请求。

&nbsp;
<h3><span style="font-weight: bolder;">问题6：多路复用就不会关闭了么？</span></h3>
多路复用使用的同一个TCP的connection会关闭么，什么时候关闭，这是个问题？从标准上看到一段文字：
<div class="newpage" style="padding:10px;font-size: 13.3333px; margin-top: 0px; margin-bottom: 0px; break-before: page; color: #000000;">HTTP/2 connections are persistent.  For best performance, it is
   expected that clients will not close connections until it is
   determined that no further communication with a server is necessary
   (for example, when a user navigates away from a particular web page)
   or until the server closes the connection.</div>
意思就是说关闭的时机有2个：
1）用户离开这个页面。
2）server主动关闭connection。
但是标准总归标准，不同的服务器实现时有了自己的约定，就行keep alive一样，每种服务器都有对自己多路复用的这个connection有相关的配置：

<strong>Apache：</strong>
```php
Syntax:	http2_idle_timeout time;
Default:	
http2_idle_timeout 3m;
Context:	http, server

Syntax:	http2_recv_timeout time;
Default:	
http2_recv_timeout 30s;
Context:	http, server
```
<a href="http://nginx.org/en/docs/http/ngx_http_v2_module.html#http2_idle_timeout" target="_blank" rel="noopener noreferrer">http://nginx.org/en/docs/http/ngx_http_v2_module.html#http2_idle_timeout</a>

参考资料：
<ol style="padding-top: 0px; padding-right: 0px; padding-bottom: 0px;">
 	<li style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; padding: 0px;">HTTP2官方标准：<a style="color: #336699;" href="https://tools.ietf.org/html/rfc7540" target="_blank" rel="noopener noreferrer">https://tools.ietf.org/html/rfc7540</a></li>
 	<li style="margin-top: 0px; margin-right: 0px; margin-bottom: 0px; padding: 0px;"><a href="https://cascadingmedia.com/insites/2015/03/http-2.html" target="_blank" rel="noopener noreferrer">https://cascadingmedia.com/insites/2015/03/http-2.html</a></li>
</ol>
---
title: 移动web日志查看工具
date: 2016-01-07 21:31:07
tags: 工具
categories:
- 568
photos:
- https://qiniu.nihaoshijie.com.cn/banabanb.jpg
---
<h1><span style="color: #008000;">MLogger</span></h1>
<!--more-->

一个浮在页面上的日志查看工具
<h2><a id="user-content-功能简介" class="anchor" style="color: #4078c0;" href="https://github.com/lvming6816077/MLogger#功能简介"></a><span style="color: #008000;">功能简介</span></h2>
<a href="https://github.com/lvming6816077/MLogger" target="_blank">github地址</a>

一个浮在页面上的console面板，资源保证按需加载最大化保证性能。

1.查看移动web页面上的console信息，自己可以在代码里面任意使用console，然后在手机端呼起该log便可以查看自己所打印的信息。
2.查看移动web页面的ajax请求信息，包括返回数据，请求参数等等，类似与fiddler的抓包。
<h2><a id="user-content-功能清单" class="anchor" style="color: #4078c0;" href="https://github.com/lvming6816077/MLogger#功能清单"></a><span style="color: #008000;">功能清单</span></h2>
项目分为3个资源文件

1.inline的js 这个js需要在页面的所有js之前引入(因为会拦截console方法和ajax方法).
2.非inline的js 这个js可以放在cdn上面，在初始化的时候配置进去即可。
2.非inline的css 这个css可以放在cdn上面，在初始化的时候配置进去即可。
<h2><a id="user-content-功能截图" class="anchor" style="color: #4078c0;" href="https://github.com/lvming6816077/MLogger#功能截图"></a><span style="color: #008000;">功能截图</span></h2>
<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/20150818153137955.gif" alt="" width="319" height="567" />
<h2><a id="user-content-快速上手" class="anchor" style="color: #4078c0;" href="https://github.com/lvming6816077/MLogger#快速上手"></a><span style="color: #008000;">快速上手</span></h2>
```html
<script src="log_inline.js"></script>
<script type="text/javascript">
    var opt = {
        'logExtJs': 'log_ext.js',
        'logExtCss': 'log.css'
    };
    window.MLogger.init(opt);
    console.log(1);
    console.log({a:1,b: {x:'ccc'}});
</script>
```

欢迎接入使用！
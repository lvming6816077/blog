---
title: 移动web问题小结
date: 2014-10-11 17:02:14
tags:
- 移动web
- HTML5
categories:
- 455
---
本文主要收集一些移动web开发中常见的问题和解决办法，在日常的工作中遇到新的问题会不定时更新到文章中。<h3><span style="color: #008000;">屏蔽阴影：</span></h3>
```html
-webkit-appearance:none
```
<p>亲测，可以同时屏蔽输入框怪异的内阴影，解决iOS下无法修改按钮样式，测试还发现一个小问题就是，加了上面的属性后，iOS下默认还是带有圆角的，不过可以使用 border-radius属性修改。  </p><h3><span style="color: #008000;">Meta标签：</span></h3>
<!--more-->
```html
<meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0;" name="viewport" />
```
<p>这个想必大家都知道，当页面在手机上显示时，增加这个meta可以让页面强制让文档的宽度与设备的宽度保持1:1，并且文档最大的宽度比例是1.0，且不允许用户点击屏幕放大浏览。 <!--more--></p>
```html
<meta content="telephone=no" name="format-detection" />
<meta content="email=no" name="format-detection" />
```
<p>这两个属性分别对ios上自动识别电话和android上自动识别邮箱做了限制。  </p><h3><span style="color: #008000;"> 获取滚动条的值：</span></h3>
```html
window.scrollY  window.scrollX
```
<p>桌面浏览器中想要获取滚动条的值是通过document.scrollTop和document.scrollLeft得到的，但在iOS中你会发现这两个属性是未定义的，为什么呢？因为在iOS中没有滚动条的概念，在Android中通过这两个属性可以正常获取到滚动条的值，那么在iOS中我们该如何获取滚动条的值呢？就是上面两个属性，但是事实证明android也支持这属性，所以索性都用woindow.scroll.  </p><h3><span style="color: #008000;">禁止选择文本：</span></h3>
```html
-webkit-user-select:none
```
<p>禁止用户选择文本，ios和android都支持  </p><h3> </h3><h3><span style="color: #006400;"> css之border-box：</span></h3>
```html
element{
        width: 100%;
        padding-left: 10px;
        box-sizing:border-box;
        -webkit-box-sizing:border-box;
        border: 1px solid blue;
}
```
<p><span style="color: #222222;">那我想要一个元素100%显示，又必须有一个固定的padding-left／padding-right，还有1px的边框，怎么办？</span>这样编写代码必然导致出现横向滚动条，肿么办？要相信问题就是用来解决的。这时候伟大的css3为我们提供了box-sizing属性，对于这个属性的具体解释不做赘述（想深入了解的同学可以到w3school查看，要知道自己动手会更容易记忆）。让我们看看如何解决上面的问题：  </p><h3><span style="color: #006400;"> css3多文本换行：</span></h3>```html
p {
    overflow : hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}```
<p>Webkit支持一个名为-webkit-line-clamp的属性，参见<a href="http://developer.apple.com/safari/library/documentation/AppleApplications/Reference/SafariCSSRef/Articles/StandardCSSProperties.html#//apple_ref/doc/uid/TP30001266-UnsupportedProperties">链接</a>，也就是说这个属性并不是标准的一部分，可能是Webkit内部使用的，或者被弃用的属性。需要注意的是display需要设置成box，-webkit-line-clamp表示需要显示几行。  </p><h3><span style="color: #006400;"> Retina屏幕高清图片：</span></h3>
```html
selector {
  background-image: url(no-image-set.png);
  background: image-set(url(foo-lowres.png) 1x,url(foo-highres.png) 2x) center;
}```
<p>image-set的语法，类似于不同的文本，图像也会显示成不同的：</p><ol><li> <strong>不支持image-set</strong>：在不支持image-set的浏览器下，他会支持background-image图像，也就是说不支持image-set的浏览器下，他们解析background-image中的背景图像；</li><li> <strong>支持image-set</strong>：如果你的浏览器支持image-sete，而且是普通显屏下，此时浏览器会选择image-set中的@1x背景图像；</li><li> <strong>Retina屏幕下的image-set</strong>：如果你的浏览器支持image-set，而且是在Retina屏幕下，此时浏览器会选择image-set中的@2x背景图像。</li></ol><p>&nbsp;</p><h3><strong> </strong></h3><h3><span style="color: #006400;"> html5重力感应事件：</span></h3>
```html
if (window.DeviceMotionEvent) { 
         window.addEventListener('devicemotion',deviceMotionHandler, false);  
} 
var speed = 30;//speed
var x = y = z = lastX = lastY = lastZ = 0;
function deviceMotionHandler(eventData) {  
  var acceleration =event.accelerationIncludingGravity;
        x = acceleration.x;
        y = acceleration.y;
        z = acceleration.z;
        if(Math.abs(x-lastX) > speed || Math.abs(y-lastY) > speed || Math.abs(z-lastZ) > speed) {
            //简单的摇一摇触发代码
            alert(1);
        }
        lastX = x;
        lastY = y;
        lastZ = z;
}
```
<p>关于deviceMotionEvent是HTML5新增的事件，用来检测手机重力感应效果具体可参考<a href="http://w3c.github.io/deviceorientation/spec-source-orientation.html" target="_blank">http://w3c.github.io/deviceorientation/spec-source-orientation.html</a>  </p><h3><span style="color: #006400;">移动端touch事件：</span></h3><ul><li>touchstart //当手指接触屏幕时触发</li><li>touchmove //当已经接触屏幕的手指开始移动后触发</li><li>touchend //当手指离开屏幕时触发</li><li>touchcancel//当某种touch事件非正常结束时触发</li></ul><p>这4个事件的触发顺序为： touchstart -> touchmove ->  touchend ->touchcancel 对于某些android系统touch的bug: 比如手指在屏幕由上向下拖动页面时，理论上是会触发 一个 touchstart ，很多次 touchmove ，和最终的 touchend ，可是在android 4.0上，touchmove只被触发一次，触发时间和touchstart 差不多，而touchend直接没有被触发。这是一个非常严重的bug，在<a href="http://code.google.com/p/android/issues/detail?id=19827" target="_blank">google Issue</a>已有不少人提出 ,这个很蛋疼的bug是在模拟下拉刷新是遇到的尤其当touchmove的dom节点数量变多时比出现，当时解决办法就是用settimeout来稀释touchmove。  </p><h3><span style="color: #006400;">单击延迟：</span></h3><p>click 事件因为要等待双击确认，会有 300ms 的延迟，体验并不是很好。 开发者大多数会使用封装的 tap 事件来代替click 事件，所谓的 tap 事件由 touchstart 事件 + touchmove 判断 + touchend 事件封装组成。 <a style="color: #4183c4;" title="article5" href="https://developers.google.com/mobile/articles/fast_buttons?hl=de-DE">Creating Fast Buttons for Mobile Web Applications</a> <a style="color: #4183c4;" title="article5" href="http://stackoverflow.com/questions/12238587/eliminate-300ms-delay-on-click-events-in-mobile-safari">Eliminate 300ms delay on click events in mobile Safari</a>  </p><h3><span style="color: #006400;">IOS里面fixed的文本框焦点居中</span></h3>
```html
<!DOCTYPE html>
    <head>
    input {
       position:fixed;
       top:0;left:0;
    }
    </head>
    <body>
        <div class="header">
            <form action="">
                <label>Testfield: <input type="text" /></label>
            </form>
        </div>
    </body>
</html>
```
<p>在ios里面，当一个文本框的样式为fixed时候，如果这个文本框获得焦点，它的位置就会乱掉，由于ios里面做了自适应居中，这个fixed的文本框会跑到页面中间。类似： <img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/111030pvtt0t0nfen6tef4.png" alt="" width="242" height="364" />   <strong>解决办法有两个：</strong> 可以在文本框获得焦点的时候将fixed改为absolute，失去焦点时在改回fixed，但是这样会让屏幕有上下滑动的体验不太好。</p>
```html
.fixfixed {
    position:absolute;
}
$(document)
    .on('focus', 'input', function(e) {
        $this.addClass('fixfixed');
    })
    .on('blur', 'input', function(e) {
        $this.removeClass('fixfixed');
    });
```
<p>  还有一种就是用一个假的fixed的文本框放在页面顶部，一个absolute的文本框隐藏在页面顶部，当fixed的文本框获得焦点时候将其隐藏，然后显示absolute的文本框，当失去焦点时，在把absolute的文本框隐藏，fixed的文本框显示。</p>
```html
.fixfixed {
    position:absolute;
}
$(document)
    .on('focus', 'input', function(e) {
        $absolute..show();
        $this.hide();
    })
    .on('blur', 'input', function(e) {
         $fixed..show();
        $this.hide();
    });
```
<p>  最后一种就是顶部的input不参与滚动，只让其下面滚动。  </p><h3><span style="color: #006400;">position:sticky</span></h3><p><span style="color: #444444;">position:sticky是一个新的css3属性，它的表现类似position:relative和position:fixed的合体，在目标区域在屏幕中可见时，它的行为就像position:relative; 而当页面滚动超出目标区域时，它的表现就像position:fixed，它会固定在目标位置。</span></p>
```html
.sticky { 
    position: -webkit-sticky; 
    position:sticky; 
    top: 15px; 
}
```
<p><span style="color: #444444;"><strong>浏览器兼容性</strong>：</span> 由于这是一个全新的属性，以至于到现在都没有一个规范，W3C也刚刚开始讨论它，而现在只有webkit nightly版本和chrome 开发版(Chrome 23.0.1247.0+ Canary)才开始支持它。 另外需要注意的是，如果同时定义了left和right值，那么left生效，right会无效，同样，同时定义了top和bottom，top赢～～ <strong><span style="color: #006400;">移动端点透事件</span></strong> 简单的说，由于在移动端我们经常会使用tap(touchstart)事件来替换掉click事件，那么就会有一种场景是：</p>
```html
<div id="mengceng"></div>

<a href="www.qq.com">www.qq.com</a>
```
<p>div是绝对定位的蒙层z-index高于a，而a标签是页面中的一个链接，我们给div绑定tap事件：</p>
```html
$('#mengceng').on('tap',function(){
$('#mengceng').hide();
});
```
<p>我们点击蒙层时 div正常消失，但是当我们在a标签上点击蒙层时，发现a链接被触发，这就是所谓的点透事件。 原因： <span style="background-color: #f5f8fd; color: #000000; font-family: arial,宋体; font-size: 14px;">touchstart 早于 touchend 早于 click。亦即click的触发是有延迟的，这个时间大概在300ms左右，也就是说我们tap触发之后蒙层隐藏，此时click还没有触发，300ms之后由于蒙层隐藏，我们的click触发到了下面的a链接上。</span> 解决办法： 1 尽量都使用touch事件来替换click事件。 2 阻止a链接的click的preventDefault   <strong><span style="color: #006400;">base64编码图片替换url图片</span></strong> u在移动端，网络请求是很珍贵的资源，尤其在2g或者3g网络下，所以能不发请求的资源都尽量不要发，对于一些小图片icon之类的，可以将图片用base64编码，来减少网络请求。  </p><h4><span style="color: #006400;">手机拍照和上传图片</span></h4><p><input type="file">的accept 属性</p>
```html
<!-- 选择照片 -->
<input type=file accept="image/*">
<!-- 选择视频 -->
<input type=file accept="video/*">
```
<p>  <strong><span style="color: #006400;">动画效果时开启硬件加速</span></strong> 我们在制作动画效果时经常会想要改版元素的top或者left来让元素动起来，在pc端还好但是移动端就会有较大的卡顿感，这么我们需要使用css3的  transform: translate3d;来替换， 此效果可以让浏览器开启<a href="http://www.cnblogs.com/PeunZhang/p/3510083.html">gpu</a>加速，渲染更流畅，但是笔着实验时在ios上体验良好，但在一些低端android机型可能会出现意想不到的效果。  </p><h4><span style="color: #006400;">快速回弹滚动</span></h4><p>在iOS上如果你想让一个元素拥有像 Native 的滚动效果，你可以这样做：</p>
```html
.div {
        overflow: auto;
        -webkit-overflow-scrolling: touch;
    }
```
<p>经笔着测试，此效果在不同的ios系统表现不一致， 对于局部滚动，ios8以上，不加此效果，滚动的超级慢，ios8一下，不加此效果，滚动还算比较流畅 对于body滚动，ios8以上，不加此效果同样拥有弹性滚动效果。  </p>
<h4><span style="color: #006400;">ios和android局部滚动时隐藏原生滚动条</span></h4>
<p>android</p>
```css
::-webkit-scrollbar{
    opacity: 0;
}

<p>ios 使用一个稍微高一些div包裹住这个有滚动条的div然后设置overflow:hidden挡住之</p>
```css
.wrap{
    height: 100px;
    overflow: hidden;
}
.box{
    width: 100%;
    height: -webkit-calc(100% + 5px);
    overflow-x: auto;
    overflow-y: hidden;
    -webkit-overflow-scrolling: touch;
}
<div class="wrap">
    <div class="box"></div>
</div>
```
<h4><span style="color: #006400;">设置placeholder时候 focus时候文字没有隐藏</span></h4>

```css
input:focus::-webkit-input-placeholder{
    opacity: 0;
}
```
<h4><span style="color: #006400;">移动端不同的input对应不同的键盘展示样式</span></h4><p>ios ---- android type email <img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/QQ截图20150830183824.png" alt="" width="868" height="288" /> type url <img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/QQ截图20150830184045.png" alt="" width="646" height="441" /> type tel <img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/QQ截图20150830184138.png" alt="" width="873" height="292" /> type search <img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/QQ截图20150830184250.png" alt="" width="316" height="324" />  </p><h4><span style="color: #006400;">background-image和image的加载区别</span></h4><p><span style="color: #000000;">在网页加载的过程中，以css背景图存在的图片background-image会等到结构加载完成（网页的内容全部显示以后）才开始加载，而html中的</span><span style="color: #323e32;"><span style="color: #000000;">标签img是网页结构（内容）的一部分会在加载结构的过程中加载，换句话讲，网页会先加载</span><span style="color: #000000;">标签img的内容，再加载背景图片background-image，如果你用</span><span style="color: #000000;">引入了一个很大的图片，那么在这个图片下载完成之前，img</span><span style="color: #000000;">后的内容都不会显示。而如果用css来引入同样的图片，网页结构和内容加载完成之后，才开始加载背景图片，不会影响你浏览网页内容。</span></span> 未完待续 参考资料：<a href="http://www.nihaoshijie.com.cn/index.php/archives/455">http://www.nihaoshijie.com.cn/index.php/archives/455</a></p>
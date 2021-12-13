---
title: 移动web适配之rem
date: 2016-03-14 21:13:42
tags:
- 移动web
- rem
categories:
- 593

---
<h2><span style="color: #008000;">前言</span></h2>
提到rem，大家首先会想到的是rm，px这类的词语，大多数人眼中这些单位是用于设置字体的大小的，没错这的确是用来设置字体大小的，但是对于rem来说它可以用来做移动端的响应式适配哦。
<!--more-->
&nbsp;
<h2><span style="color: #008000;">兼容性</span></h2>
<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/caniuse.png" alt="" width="1270" height="548" /> 先看看兼容性，大部分主流浏览器都支持，可以安心的往下看了。

&nbsp;
<h2><span style="color: #008000;">rem设置字体大小</span></h2>
rem是<span style="color: #111111;">（font size of the root element），官方解释</span>

<span style="color: #111111;"><img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720160313181850.png" alt="" width="868" height="129" />，</span>

<span style="color: #111111;">意思就是根据网页的跟元素来设置字体大小，和em（font size of the element）的区别是，em是根据其父元素的字体大小来设置，而rem是根据网页的跟元素（html）来设置字体大小的，举一个简单的例子，</span>

<span style="color: #111111;">现在大部分浏览器<span style="color: #4a4a4a;">IE9+，Firefox、Chrome、Safari、Opera </span>，如果我们不修改相关的字体配置，都是默认显示font-size是16px即</span>
```css
html {    
	font-size:16px;
}
```
那么如果我们想给一个P标签设置12px的字体大小那么用rem来写就是
```css
p {    
	font-size: 0.75rem; //12÷16=0.75（rem）
}
```
基本上使用rem这个单位来设置字体大小基本上是这个套路，好处是加入用户自己修改了浏览器的默认字体大小，那么使用rem时就可以根据用的调整的大小来显示了。 但是rem不仅可以适用于字体，同样可以用于width height margin这些样式的单位。下面来具体说一下

&nbsp;
<h2><span style="color: #008000;">rem进行屏幕适配</span></h2>
在讲rem屏幕适配之前，先说一下一般做移动端适配的方法，一般可以分为： <strong>1</strong> 简单一点的页面，一般高度直接设置成固定值，宽度一般盛满整个屏幕。 <strong>2</strong> 稍复杂一些的是利用百分比设置元素的大小来进行适配，或者利用flex等css去设置一些需要定制的宽度。 <strong>3</strong> 再复杂负责一些的响应式页面，需要利用css3的media query属性来进行适配，大致思路是根据屏幕不同大小，来设置对应的css样式。 上面的一些方法，其实也可以解决屏幕适配等问题，但是既然出来的rem这个新东西，也一定能兼顾到这些方面，下面具体来说具体使用rem：

&nbsp;

<strong>rem适配</strong>

&nbsp;

先看一个简单的例子：
```css
.con {
      width: 10rem;
      height: 10rem;
      background-color: red;
 }
<div class="con">
        
</div>
```
<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/div.png" alt="" width="380" height="231" />

这是一个div，宽度和高度都用rem来设置了，在浏览器里面是这样显示的，  可以看到，在浏览器里面width和height分别是160px，正好是16px * 10,那么如果将html根元素的默认font-size修改一下呢？
```css
html {
    font-size: 17px;
}
.con {
      width: 10rem;
      height: 10rem;
      background-color: red;
 }
<div class="con">
        
</div>
```
再来看看结果：

<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/div2.png" alt="" width="386" height="222" />

这时width和height都是170px，这就说明了将rem应用与width和height时，同样适用与rem的特性，根据根元素的font-size值来改变自身的值，由此我们应该可以联想到我们可以给html设定不同的值，从而达到我们css样式中的适配效果。

&nbsp;

<strong>rem数值计算</strong>

&nbsp;

如果利用rem来设置css的值，一般要通过一层计算才行，比如如果要设置一个长宽为100px的div，那么就需要计算出100px对应的rem值是 100 / 16 =6.25rem，这在我们写css中，其实算比较繁琐的一步操作了，不过这其实都不是事。 想想我们现在的工程，哪个没有用构建的，前端构建中，完全可以利用scss来解决这个问题，例如我们可以写一个scss的function px2rem即：
```css
@function px2rem($px){
    $rem : 37.5px;
    @return ($px/$rem) + rem;
}
```
这样，当我们写具体数值的时候就可以写成：
```css
height: px2rem(90px);
width: px2rem(90px);;
```
看到这里，你可能会发现一些不理解的地方，就是上面那个rem:37.5px是怎么来的，正常情况下不是默认的16px么，这个其实就是页面的基准值，和html的font-size有关。

&nbsp;

<strong>rem基准值计算</strong>

&nbsp;

关于rem的基准值，也就是上面那个37.5px其实是根据我们所拿到的视觉稿来决定的主要有以下几点原因：

<strong>1</strong> 由于我们所写出的页面是要在不同的屏幕大小设备上运行的

<strong>2</strong> 所以我们在写样式的时候必须要先已一个确定的屏幕来作为参考，这个就由我们拿到的视觉稿来定

<strong>3</strong> 假如我们拿到的视觉稿是以iphone6的屏幕为基准设计的

<strong>4</strong> iPhone6的屏幕大小是375px，
<pre class="lang:default decode:true">rem = window.innerWidth  / 10</pre>
这样计算出来的rem基准值就是37.5（iphone6的视觉稿），这里为什么要除以10呢，其实这个值是随便定义的，假如不除以10，根据我们算出来的基准值会偏大，这样在设置html的font-size时候会偏小，我们知道浏览器的font-size如果小于12px就显示不出效果了，在这里列举一下其他手机的

iphone3gs: 320px / 10 = 32px

iphone4/5: 320px  / 10 = 32px

iphone6: 375px  / 10 =37.5px

&nbsp;

<strong>动态设置html的font-size</strong>

&nbsp;

现在关键问题来了，我们该如何通过不同的屏幕去动态设置html的font-size呢，这里一般分为两种办法

<strong>1</strong> 利用css的media query来设置即
```css
@media (min-device-width : 375px) and (max-device-width : 667px) and (-webkit-min-device-pixel-ratio : 2){
      html{
      	font-size: 37.5px;
      }
}
```
<strong>2</strong> 利用javascript来动态设置 根据我们之前算出的基准值，我们可以利用js动态算出当前屏幕所适配的font-size即：
```javasript
document.getElementsByTagName('html')[0].style.fontSize = window.innerWidth / 10 + 'px';
```
然后我们看一下之前那个demo展示的效果
```css
.con {
      width: px2rem(200px);
      height: px2rem(200px);
      background-color: red;
}
<div class="con">
        
</div>
```javascript
document.addEventListener('DOMContentLoaded', function(e) {
                document.getElementsByTagName('html')[0].style.fontSize = window.innerWidth / 10 + 'px';
}, false);
```
iPhone6下，正常显示200px

<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/div3.png" alt="" width="349" height="321" />

在iphone4下，显示169px

<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/div4.png" alt="" width="320" height="284" />

由此可见我们可以通过设置不同的html基础值来达到在不同页面适配的目的，当然在使用js来设置时，需要绑定页面的resize事件来达到变化时更新html的font-size。

&nbsp;
<h2><span style="color: #008000;">rem适配进阶</span></h2>
我们知道，一般我们获取到的视觉稿大部分是iphone6的，所以我们看到的尺寸一般是双倍大小的，在使用rem之前，我们一般会自觉的将标注/2，其实这也并无道理，但是当我们配合rem'使用时，完全可以按照视觉稿上的尺寸来设置。

1 设计给的稿子双倍的原因是iphone6这种屏幕属于高清屏，也即是设备像素比(device pixel ratio)dpr比较大，所以显示的像素较为清晰。

2 一般手机的dpr是1，iphone这种高清屏是2，可以通过js的window.devicePixelRatio获取到当前设备的dpr，所以iphone6给的视觉稿大小是（*2）750×1334了。

3 拿到了dpr之后，我们就可以在viewport meta头里，取消让浏览器自动缩放页面，而自己去设置viewport的content例如
```javascript
meta.setAttribute('content', 'initial-scale=' + 1/dpr + ', maximum-scale=' + 1/dpr + ', minimum-scale=' + 1/dpr + ', user-scalable=no');
```
4 设置完之后配合rem，修改
```css
@function px2rem($px){
    $rem : 75px;
    @return ($px/$rem) + rem;
}
```
双倍75，这样就可以完全按照视觉稿上的尺寸来了。不用在/2了，这样做的好处是：

1 解决了图片高清问题。

2 解决了border 1px问题（我们设置的1px，在iphone上，由于viewport的scale是0.5，所以就自然缩放成0.5px）
<h2><span style="color: #008000;">rem进行屏幕适配总结</span></h2>
下面这个网址是针对rem来写的一个简单的demo页面，大家可以在不同的手机上看一下效果

<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/democode.png" alt="" width="280" height="280" />

但是rem也并不是万能的，下面也有一些场景是不适于使用rem的

<strong> 1</strong> 当用作图片或者一些不能缩放的展示时，必须要使用固定的px值，因为缩放可能会导致图片压缩变形等。

<strong> 2</strong> 再设置backgroundposition或者backgroundsize时不宜使用rem。

在列举几个使用rem的线上网站：

网易新闻：<a href="http://3g.163.com/touch/news/subchannel/all?version=v_standard" target="_blank">http://3g.163.com/touch/news/subchannel/all?version=v_standard</a>

聚划算：<a href="https://jhs.m.taobao.com/m/index.htm#!all" target="_blank">https://jhs.m.taobao.com/m/index.htm#!all</a>
---
title: 移动web之滚动篇
date: 2017-04-18 11:21:55
tags:
- 移动web
- 滚动
categories:
- 690

---
<blockquote>在移动端，使用滚动来处理业务逻辑的情况有很多，例如列表的滚动加载数据，下拉刷新等等都需要利用滚动的相关知识，但是滚动事件在不同的移动端机型却又有不同的表现，下面就来一一总结一下。</blockquote>
<!--more-->
<h2 id="-1-web-">知识点1:移动web滚动问题</h2>
<ol>
 	<li><strong>滚动事件</strong>：即onscroll事件，形成原因通俗解释是：当子元素的高度超过父元素的高度，并且且父元素的高度是定值（window除外），就会形成滚动条，滚动分为两种：局部滚动和body滚动。</li>
 	<li><strong>onscroll方法</strong>： 一般情况下当我们需要监听一个滚动事件时通常会用到onscroll方法来监听滚动事件的触发。 如果在浏览器上调试这个方法在浏览器上很好用，但是如果跑在手机端就没有想象中的效果了。</li>
 	<li><strong>body滚动</strong>：在移动端如果使用body滚动，意思就是页面的高度由内容自动撑大，body自然形成滚动条，这时我们监听window.onscroll，发现onscroll并没有实时触发，只在手指触摸的屏幕上一直滑动时和滚动停止的那一刻才触发,采用了wk内核的webview除外。
<img style="magin:auto;" src="https://qiniu.nihaoshijie.com.cn/QQ20170414-0@2x.png" width="320" /> body滚动
<img src="https://qiniu.nihaoshijie.com.cn/QQ20170414-1@2x.png" width="320" />局部滚动</li>
 	<li><strong>局部滚动</strong>：在移动端如果使用局部滚动，意思就是我们的滚动在一个固定宽高的div内触发，将该div设置成overflow:scroll/auto;来形成div内部的滚动，这时我们监听div的onscroll发现触发的时机区分android和ios两种情况，具体可以看下面表格：</li>
 	<li><strong>不同机型onscroll事件触发情况：</strong>

|         | body滚动       | 局部滚动  |
| ------------- |:-------------:| -----:|
| ios      | 不能实时触发 | 不能实时触发 |
| android      | 实时触发      |   实时触发 |
| ios wkwebview内核 | 实时触发      |    实时触发 |
</li>
 	<li><strong>wkwebview内核</strong>:这里说明一下关于ios的wkwebview内核是ios从ios8开始提供的新型webview内核，和之前的uiwebview相比，性能要好，具体大家可以自行查看关于wkwebview的相关概念。</li>
 	<li><strong>body滚动和局部滚动demo</strong>：这里我需要指出的是在采用wkwebview内核的页面中scroll是可以实时触发的，如果使用的是原本的uiwebview则不能够实时触发，手q目前使用的是uiwebview而新版微信使用的是wkwebview，大家可以分别使用来尝试一下下面的demo：
<img src="https://qiniu.nihaoshijie.com.cn/1492155048.png" width="220" />局部滚动
<img src="https://qiniu.nihaoshijie.com.cn/1492155773.png" width="220" />body滚动
分别用ios手q和微信和android手q体验会有不同的结果。</li>
</ol>
<h2 id="-2-">知识点2:关于模拟滚动</h2>
<ol>
 	<li><strong>正常的滚动</strong>：我们平时使用的scroll，包括上面讲的滚动都属于正常滚动，利用浏览器自身提供的滚动条来实现滚动，底层是由浏览器内核控制。</li>
 	<li><strong>模拟滚动</strong>：最典型的例子就是iscroll了，原理一般有两种：
<strong>1). </strong>监听滚动元素的touchmove事件，当事件触发时修改元素的transform属性来实现元素的位移，让手指离开时触发touchend事件，然后采用requestanimationframe来在一个线型函数下不断的修改元素的transform来实现手指离开时的一段惯性滚动距离。
<strong>2).</strong>监听滚动元素的touchmove事件，当事件触发时修改元素的transform属性来实现元素的位移，让手指离开时触发touchend事件，然后给元素一个css的animation，并设置好duration和function来实现手指离开时的一段惯性距离。</li>
 	<li><strong>方案区别</strong>：这两种方案对比起来各有好处，第一种方案由于惯性滚动的时机时由js自己控制所以可以拿到滚动触发阶段的scrolltop值，并且滚动的回调函数onscroll在滚动的阶段都会触发。</li>
 	<li>第二种方案相比第一种要劣势一些，区别在于手指离开时，采用的时css的animation来实现惯性滚动，所以无法直接触发惯性滚动过程中的onscroll事件，只有在animation结束时才可以借助animationend来获取到事件，当然也有一种方法可以实时获取滚动事件，也是借助于requestanimationframe来不断的去读取滚动元素的transform来拿到scrolltop同时触发onscroll回调。</li>
 	<li><strong>demo体验</strong>：关于模拟滚动和正常滚动，两者在性能上差别还是比较明显的，下面时两个demo，可以扫描体验一下：
<img src="https://qiniu.nihaoshijie.com.cn/1495680143.png" width="220" />正常滚动<img src="https://qiniu.nihaoshijie.com.cn/blog/1495679733.png" width="220" />模拟滚动
<strong>衡量指标</strong>：衡量滚动是否流畅的指标fps，我这边也统计了一下正常滚动和模拟滚动的fps数据：
<img src="https://qiniu.nihaoshijie.com.cn/normaltu.jpg" />正常滚动
<img src="https://qiniu.nihaoshijie.com.cn/iscrolltu.jpg" />模拟滚动
<strong>结论</strong>： 模拟滚动的fps值波动较大，这样滚动起来会有明显的卡顿感觉，各位体验的时候如果滚动超过10屏之后就可以明显感觉到两着的区别。
在使用模拟滚动时，浏览器在js层面会消耗更多的性能去改变dom元素的位置，在dom复杂层级深的页面更为高，所以在长列表滚动时还要使用正常滚动更好。</li>
</ol>
<h2 id="-3-">知识点3:滚动和下拉刷新</h2>
<ol>
 	<li>下拉刷新的元素在页面顶部，正常浏览时不可见的。</li>
 	<li>当在页面顶部往下滚动时出现下拉刷新元素，当手指离开时收起。</li>
 	<li>以上两点时实现一个下拉刷新组件的基本步骤，结合我们上述关于滚动的描述，我们可以这样实现下拉刷新：
<img src="https://qiniu.nihaoshijie.com.cn/10AA2031-24A3-4858-BFA9-FF702231FB4C.png" width="220" />
<strong>方案1</strong>：借助iscroll的原理，整个页面使用模拟滚动，将下拉刷新元素放在顶部，当页面滚动到顶部下拉时，下拉刷新元素随着页面的滚动出现，当手指离开时收回，此方案实现起来较为简单直接借助iscoll即可，但是使用了模拟滚动之后在正常的列表滚动时性能上不如正常滚动。
<strong>方案2</strong>：页面使用正常滚动，将下拉刷新元素放置在顶部top值为负值（正常情况下不可见），当页面处于顶部时下拉，这时监听touchmove事件，修改scrollcontent的tranlateY值，同时修改下拉刷新元素的tranlateY值，将两者同时位移来将下拉刷新元素显示出来，手指离开时（touchend）收回，这种方案满足了在正常列表滚动时使用原生的滚动节省性能，只在下拉刷新时使用模拟滚动来实现效果。
<strong>方案3</strong>：方案2的改良版，唯一不同是将下拉刷新元素和scrollcontent放在一个div里，将下拉刷新元素的margintop设为负值，在下拉刷新时，只需要修改scrollcontent一个元素的tranlateY值即可实现下拉，在性能上要比方案2好。
<img src="https://qiniu.nihaoshijie.com.cn/pullrefresh3.gif" width="220" /></li>
 	<li><strong>性能问题</strong>：在采用了上述方案之后，还会有一个性能上的问题就是：当页面的列表过长，dom元素过多时，在模拟滚动，下拉刷新这段时间内，页面也会有卡顿现象，这里采取了一个</li>
 	<li><strong>优化策略</strong>：
1) 列表较长时dom数量较多时，在触发下拉刷新的时机时将页面视窗之外的dom元素<strong>隐藏</strong>或者存放在fragment里面。
2) 在刷新完成之后手指离开（touchend）时将隐藏的元素显示出来。
3) 需要注意的是，隐藏和显示视窗外的元素这个操作在下拉刷新时只会<strong>执行一次</strong>，并且只有在下拉刷新时才会执行。</li>
</ol>
<h1 id="alloypullrefresh">AlloyPullRefresh（基于上述知识点开发的组件）</h1>
<ol>
 	<li>定义下拉刷新元素样式</li>
 	<li>下拉刷新事件回调</li>
 	<li>支持zepto版本和react版本</li>
</ol>
github地址：<a href="https://github.com/AlloyTeam/AlloyPullRefresh/">https://github.com/AlloyTeam/AlloyPullRefresh/</a>
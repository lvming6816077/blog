---
title: 线条之美，玩转SVG线条动画
date: 2017-02-20 20:23:10
tags:
- svg
categories:
- 667
photos: https://qiniu.nihaoshijie.com.cn/ilu2.gif
---
通常来说web前端实现动画效果主要通过下面几种方案：
<ul>
 	<li>css动画；利用css3的样式效果可以将dom元素做出动画的效果来。</li>
 	<li>canvas动画；利用canvas提供的API，然后利用清除-渲染这样一帧一帧的做出动画效果。</li>
 	<li>svg动画；同样svg也提供了不少的API来实现动画效果，并且兼容性也不差，本文主要讲解一下如何制作svg线条动画。</li>
</ul>
<!--more-->
先来看几个效果：

<a href="https://qiniu.nihaoshijie.com.cn/blog/buluofuza.gif"><img class="alignnone size-full wp-image-670" src="https://qiniu.nihaoshijie.com.cn/blog/buluofuza.gif" alt="" width="279" height="237" /></a><a href="https://www.nihaoshijie.com.cn/mypro/svg/buluofuza.html" target="_blank">demo</a>

<a href="https://qiniu.nihaoshijie.com.cn/blog/daojishi1.gif"><img class="alignnone size-full wp-image-671" src="https://qiniu.nihaoshijie.com.cn/blog/daojishi1.gif" alt="" width="298" height="148" /></a><a href="https://www.nihaoshijie.com.cn/mypro/svg/daojishi.html" target="_blank">demo</a>

<a href="https://qiniu.nihaoshijie.com.cn/alloyteam.gif"><img class="alignnone size-full wp-image-672" src="https://qiniu.nihaoshijie.com.cn/alloyteam.gif" alt="" width="785" height="592" /></a><a href="https://www.nihaoshijie.com.cn/mypro/svg/alloyteam.html" target="_blank">demo</a>

以上这些效果都是利用SVG线条动画实现的，只用了css3和svg，没有使用一行javascript代码，这一点和canvas比起来要容易一些，下面就说明一下实现这些效果的原理。

关于SVG的基础知识，我这里就不再叙述了，大家可以直接在文档中查看相关的API，这里只说一下实现线条动画主要用到的：path （路径）
<h2 style="margin-top: 0px; margin-bottom: 0px; padding: 0px; border: 0px; font-weight: bold; font-family: 微软雅黑; font-size: 14px; color: #000000; background-color: #f9f9f9;">&lt;path&gt; 标签命令</h2>
<ul style="margin-top: 10px; margin-bottom: 0px; margin-left: 35px; padding-top: 0px; padding-right: 0px; padding-bottom: 0px; border: 0px; color: #000000; font-family: Verdana, Arial, 宋体; font-size: 12px; background-color: #f9f9f9;">
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">M = moveto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">L = lineto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">H = horizontal lineto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">V = vertical lineto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">C = curveto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">S = smooth curveto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">Q = quadratic Belzier curve</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">T = smooth quadratic Belzier curveto</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">A = elliptical Arc</li>
 	<li style="margin: 3px 0px 0px; padding: 0px; border: 0px;">Z = closepath</li>
</ul>
利用path的这些命令我们可以实现我们想要的任何线条组合，以一段简单的线条为例:
```html
<path id="path" fill="none" stroke="#000" stroke-width="1px" d="M452,293c0,0,0-61,72-44c0,0-47,117,81,57
    s5-110,10-67s-51,77.979-50,33.989"/>
```
效果：

<a href="https://qiniu.nihaoshijie.com.cn/blog/simple.png"><img class="alignnone size-full wp-image-673" src="https://qiniu.nihaoshijie.com.cn/blog/simple.png" alt="" width="255" height="157" /></a>

呵呵，看起来很简单，但是，如何让这个线条动起来呢？这里就要明白到SVG里的path的一些主要属性：
<ol style="transition: all 0.3s; margin-top: 0px; margin-bottom: 0px; margin-left: 30px; padding-top: 0px; padding-right: 0px; padding-bottom: 0px; list-style-position: initial; list-style-image: initial; color: #333333; font-family: 'lucida grande', 'lucida sans unicode', lucida, helvetica, 'Hiragino Sans GB', 'Microsoft YaHei', 'WenQuanYi Micro Hei', sans-serif;">
 	<li style="transition: all 0.3s; margin: 0px; padding: 0px; list-style-position: inside; list-style-image: initial;">stroke：标识路径的颜色；</li>
 	<li style="transition: all 0.3s; margin: 0px; padding: 0px; list-style-position: inside; list-style-image: initial;">d：标识路径命令的集合，这个属性主要决定了线条的形状。</li>
 	<li style="transition: all 0.3s; margin: 0px; padding: 0px; list-style-position: inside; list-style-image: initial;"><span style="color: #333333; font-family: 'lucida grande', 'lucida sans unicode', lucida, helvetica, 'Hiragino Sans GB', 'Microsoft YaHei', 'WenQuanYi Micro Hei', sans-serif;">stroke-width：标识路径的宽度，单位是px；</span></li>
 	<li style="transition: all 0.3s; margin: 0px; padding: 0px; list-style-position: inside; list-style-image: initial;">stroke-dasharray：它是一个&lt;length&gt;和&lt;percentage&gt;数列，数与数之间用逗号或者空白隔开，指定短划线和缺口的长度。如果提供了奇数个值，则这个值的数列重复一次，从而变成偶数个值。因此，5,3,2等同于5,3,2,5,3,2；</li>
 	<li style="transition: all 0.3s; margin: 0px; padding: 0px; list-style-position: inside; list-style-image: initial;"><span style="color: #333333; font-family: 'lucida grande', 'lucida sans unicode', lucida, helvetica, 'Hiragino Sans GB', 'Microsoft YaHei', 'WenQuanYi Micro Hei', sans-serif;">stroke-dashoffset：标识的是整个路径的偏移值；</span></li>
</ol>
<span style="color: #333333;">以一张图来解释stroke-dasharray和stroke-dashoffset更容易理解一些：</span>

<a href="https://qiniu.nihaoshijie.com.cn/blog/array.png"><img class="alignnone size-full wp-image-674" src="https://qiniu.nihaoshijie.com.cn/blog/array.png" alt="" width="484" height="242" /></a>

<span style="color: #333333;">因此，我们之前的路径就会变成这个样子：</span>
```css
#path {
        stroke-dasharray: 3px, 1px;
        stroke-dasharray: 0;
}
```
效果：

<a href="https://qiniu.nihaoshijie.com.cn/%E8%99%9A%E7%BA%BF.png"><img class="alignnone size-full wp-image-675" src="https://qiniu.nihaoshijie.com.cn/%E8%99%9A%E7%BA%BF.png" alt="" width="278" height="171" /></a>

理解了stroke-dasharray的作用之后，下面我们就可以使用css3的animation来让这个路径动起来。
```css
#path {
    animation: move 3s linear forwards;
}

@keyframes move {
      0%{
          stroke-dasharray: 0, 511px;
      }
      100%{
          stroke-dasharray: 511px, 511px;
      }
}
```
&nbsp;

效果：

<a href="https://qiniu.nihaoshijie.com.cn/blog/dong2.gif"><img class="alignnone size-full wp-image-676" src="https://qiniu.nihaoshijie.com.cn/blog/dong2.gif" alt="" width="384" height="154" /></a>

511这个值是整个路径的长度，可以用js的document.getElementById('path').getTotalLength()得到

stroke-dasharray: 0, 511; 表示实线和空隙的长度分别为 0 和 511，所以一开始整个路径都是空隙，所以是空的。
然后过渡到 stroke-dasharray: 511, 511; 因为整个线条的长度就是 511，而实线的长度也慢慢变成511，所以整个线条就出现了。

同样利用stroke-dashoffset也可以实现这个效果，原理就是最初线条分为511实线，和511空隙，但是由于设置了offset使线条偏移不可见了，当不断修改offset后，线条慢慢出现。
```css
#path {
    animation: move 3s linear forwards;
    stroke-dasharray: 511px,511px;
}

@keyframes move {
  0%{
      stroke-dashoffset: 511px;
  }
  100%{
      stroke-dashoffset: 0;
  }
}
```
效果：

<a href="https://qiniu.nihaoshijie.com.cn/blog/dong2-1.gif"><img class="alignnone size-full wp-image-677" src="https://qiniu.nihaoshijie.com.cn/blog/dong2-1.gif" alt="" width="384" height="154" /></a>

当我们掌握了上述的方法后，整个利用SVG实现线条动画的原理就已经清楚了，我们需要的就是一个SVG路径了，但是总画一些简单的线条还是不美啊，那我们如何才能得到复杂的svg路径呢？
<ol>
 	<li>找UI设计师要一个。</li>
 	<li>自己利用PS和AI做一个，只需要简单的2步。</li>
</ol>
<a href="https://qiniu.nihaoshijie.com.cn/blog/psai.png"><img class="alignnone size-full wp-image-678" src="https://qiniu.nihaoshijie.com.cn/blog/psai.png" alt="" width="119" height="31" /></a>

以部落LOGO为例：

1，得到部落LOGO的png图片。

2，右键图层，然后点击从选区生成工作路径，我们就可以得到：

<a href="https://qiniu.nihaoshijie.com.cn/blog/buluopng.png"><img class="alignnone size-full wp-image-679" src="https://qiniu.nihaoshijie.com.cn/blog/buluopng.png" alt="" width="260" height="210" /></a>

3，文件--导出--路径到AI，将路径导出在AI里面打开。

<a href="https://qiniu.nihaoshijie.com.cn/buluolujing.png"><img class="alignnone size-full wp-image-680" src="https://qiniu.nihaoshijie.com.cn/buluolujing.png" alt="" width="294" height="230" /></a>

4，在AI里面选择保存成svg格式的文件，然后用sublime打开svg文件，将path的d拷贝出来即可。

5，利用上文介绍的实现动画的方法，我们就可以轻松的得到了下面这个效果。

<a href="https://qiniu.nihaoshijie.com.cn/blog/buluo.gif"><img class="alignnone size-full wp-image-681" src="https://qiniu.nihaoshijie.com.cn/blog/buluo.gif" alt="" width="312" height="173" /></a>

线条动画进阶：

可以看到上面的动画效果和文章最初显示的动画效果还是有区别的，要想实现文章最初的动画效果，需要用到SVG的&lt;symbol&gt; 和 &lt;use&gt;来实现，读者可以在网上查一下这两个标签的用法。

```html
<symbol id="pathSymbol">
    <path  class="path" stroke="#00adef"  d="M281.221,261.806c0,2.756-2.166,4.922-4.922,4.922l0,0h-33.964c-11.715-24.119-31.503-59.855-47.156-68.026
  c-15.751,7.974-35.637,43.907-47.451,68.026h-33.865l0,0c-2.756,0-4.922-2.166-4.922-4.922l0,0l0,0c0-0.295,0-0.689,0.098-0.984
  c0,0,14.078-69.109,79.15-129.161c-2.953-2.56-5.907-5.119-8.959-7.58c-1.87-1.575-2.166-4.233-0.591-6.104
  c1.575-1.772,4.43-2.166,6.497-0.689c3.347,2.461,6.694,5.218,9.746,8.073c3.15-2.953,6.497-5.71,10.041-8.368
  c2.067-1.378,4.922-1.083,6.497,0.689c1.575,1.87,1.28,4.529-0.591,6.104c-3.052,2.56-6.104,5.218-9.155,7.876
  c65.27,59.953,79.446,129.161,79.446,129.161C281.221,261.117,281.221,261.412,281.221,261.806L281.221,261.806L281.221,261.806z"/>
    <path  class="path" stroke="#00adef"  d="M194.589,212.583h0.984l0,0c19.886,28.451,31.503,54.145,31.503,54.145h-63.99C163.086,266.728,174.703,241.034,194.589,212.583
L194.589,212.583z"/>
</symbol>
<g>
  <use xlink:href="#pathSymbol"
    id="path1"></use>
    <use xlink:href="#pathSymbol"
      id="path2"></use>
</g>
```
```css
#path1 {

    stroke-dashoffset: 7% 7%;
    stroke-dasharray: 0 35%;
    animation: animation 3s linear forwards;
  }

  @keyframes animation {
      100% {
        stroke-dasharray: 7% 7%;
        stroke-dashoffset: 7%;

      }
  }

  #path2 {

    stroke-dashoffset: 7% 7%;
    stroke-dasharray: 0 35%;
    animation: animation2 3s linear forwards;
  }

  @keyframes animation2 {
      100% {
          stroke-dasharray: 7% 7%;
          stroke-dashoffset: 14%;

      }
 }
 ```
思路就是：

1，将原来只有一条path的路径替换成两条，并且这两条的路径是完全重合的。

2，分别设置两条路径的stroke-dasharray和stroke-dashoffset的css3的animation动画，注意两条路径的动画不能完全一样要有差值。

3，设置成功之后就可以利用animation动画触发的时机和改变程度来实现多条动画效果。

效果：

<a href="https://qiniu.nihaoshijie.com.cn/blog/buluofuza.gif"><img class="alignnone size-full wp-image-670" src="https://qiniu.nihaoshijie.com.cn/blog/buluofuza.gif" alt="" width="279" height="237" /></a>

那么如何实现alloyteam的文字动画呢，其实原理也是利用了stroke-dasharray和stroke-dashoffset，这两个属性不仅可以作用在&lt;path&gt;上，同样可以作用在&lt;text&gt;上。
```html
<symbol id="text">
    <text x="30%" y="35%" class="text">QQ</text>
  </symbol>

  <g>
    <use xlink:href="#text"
      class="use-text"></use>
      <use xlink:href="#text"
        class="use-text"></use>
        <use xlink:href="#text"
          class="use-text"></use>
          <use xlink:href="#text"
            class="use-text"></use>
            <use xlink:href="#text"
              class="use-text"></use>
  </g>
```
```css
.use-text:nth-child(1) {
      stroke: #360745;
      animation: animation1 8s infinite ease-in-out forwards;

}
          
.use-text:nth-child(2) {
      stroke: #D61C59;
      animation: animation2 8s infinite ease-in-out forwards;

}
          
.use-text:nth-child(3) {
       stroke: #E7D84B;
       animation: animation3 8s infinite ease-in-out forwards;

}

.use-text:nth-child(4) {
       stroke: #EFEAC5;
       animation: animation4 8s infinite ease-in-out forwards;

}

.use-text:nth-child(5) {
      stroke: #1B8798;
      animation: animation5 8s infinite ease-in-out forwards;
}

@keyframes animation1 {
       50%{
            stroke-dasharray: 7% 28%;
            stroke-dashoffset: 7%;
       }
       70%{
             stroke-dasharray: 7% 28%;
             stroke-dashoffset: 7%;
       }
}
@keyframes animation2 {
       50%{
           stroke-dasharray: 7% 28%;
           stroke-dashoffset: 14%;
       }
       70%{
            stroke-dasharray: 7% 28%;
            stroke-dashoffset: 14%;
       }
}
@keyframes animation3 {
     50%{
         stroke-dasharray: 7% 28%;
         stroke-dashoffset: 21%;
    }
    70%{
         stroke-dasharray: 7% 28%;
         stroke-dashoffset: 21%;
    }
}
@keyframes animation4 {
       50%{
            stroke-dasharray: 7% 28%;
            stroke-dashoffset: 28%;
       }
       70%{
            stroke-dasharray: 7% 28%;
            stroke-dashoffset: 28%;
       }
}
@keyframes animation5 {
      50%{
           stroke-dasharray: 7% 28%;
           stroke-dashoffset: 35%;
      }
      70%{
           stroke-dasharray: 7% 28%;
           stroke-dashoffset: 35%;
      }
}
```
这里用了5条完全重合的路径，并且每个路径的颜色和动画效果都不一样。

效果：

<a href="https://qiniu.nihaoshijie.com.cn/qq.gif"><img class="alignnone size-full wp-image-682" src="https://qiniu.nihaoshijie.com.cn/qq.gif" alt="" width="529" height="338" /></a>
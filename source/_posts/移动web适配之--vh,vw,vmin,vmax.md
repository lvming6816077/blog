---
title: 移动web适配之--vh,vw,vmin,vmax
date: 2018-04-09 23:00:13
tags:
- 移动web适配
- vmin
categories:
- 788
---

## 前言

其实关于vh,vw,vmin,vmax这四个单位，伴随着css3的出现就已经有了，但是当时移动web的浪潮已经来临，并且rem出现的要早，所以很多人忽略这个，但是rem使用适配是要依赖js来进行处理，今天介绍的vh,vw,vmin,vmax是完全基于css的自适应方案，关于rem的适配可以参考笔者之前写的[文章](https://www.nihaoshijie.com.cn/index.php/archives/593/)。
<!--more-->
## 关于vh,vw,vmin,vmax

先来看下这些单位分别代表什么意思：

* vw : 1vw 等于视口宽度的1%
* vh : 1vh 等于视口高度的1%
* vmin : 选取 vw 和 vh 中最小的那个
* vmax : 选取 vw 和 vh 中最大的那个

那么什么是视口宽度呢？你一定不会陌生：
```html
<meta name="viewport" content="width=device-width,initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">
```
没错，浏览器利用`viewport`的meta标签中的width来定义浏览器的视口大小，然而width可以使用数字单位来定义，只是我们常用的方法是：将width设置成设备的宽度 即`device-width`，关于浏览器的视口介绍可以参考这个[文章](https://www.quirksmode.org/mobile/viewports.html)。使用`document.documentElement.clientWidth`可以获取到浏览器的视口大小，这里要注意不一样的是类似`window.innerWidth`或者`window.screen.width`这些拿到的是浏览器的物理宽度，当width!=device-width时是不等效的哦。

## vh/vw与%区别在于

| 单位  | 基于情况  |
| ------------ | ------------ |
|  % | 基于父元素  |
| vh/vw  | 基于视口  |


So，vh,vw,vmin,vmax都是基于viewport定义的width来定义单位，它是利用视口单位实现，依赖于视口大小而自动缩放，所以不同的设备视口大小不一样自然就达到的适配的效果。

```css
        .main {
            width: 60vw;
            height: 60vw;
            background-color: red;
            text-align: center;
            line-height: 60vw;
        }
```
![](https://qiniu.nihaoshijie.com.cn/1522579500_92_w551_h894.gif)

## 和rem相比优势？

关于rem的适配大部分需要2个步骤：
1. 在页面头部引入js来动态设置页面的html的font-size，一般情况下基于屏幕宽度来计算 即：`var rem = docEl.clientWidth / 10;docEl.style.fontSize = rem + 'px';`
2. 在使用css时，使用less或者scss来动态计算dom的实际rem数值
```css
@function px2rem($px) {
  $rem: 75px;
  @return ($px / $rem) + rem;
}
```
由此会有2个问题：
- 依赖页面头部的js设置，虽说基本上不会阻塞页面渲染，但是放在头部始终要进行一次dom计算，不算优雅。
- 通常使用宽度来计算html的font-size时，对于那种很宽的手机或者是ipad，会导致适配的不准确(可以参考rem的页面放在ipad上横屏观察一下)。


那么使用vmin基本上可以规避上面的问题：
- 不依赖js执行。
- 选取宽高之中较小的值进行适配。

![](https://qiniu.nihaoshijie.com.cn/1522580497_97_w1044_h720.gif)
## 浏览器兼容性？
那么vh,vw,vmin,vmax这么方便，为什么还是没有流行起来呢？让我们来看看浏览器的[兼容性](https://caniuse.com/#search=vh)：
![](https://qiniu.nihaoshijie.com.cn/1522580229_33_w2608_h1258.png)

Android4.4之前不支持是硬伤！


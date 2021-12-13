---
title: Canvas制作 撞球游戏 简单易学
date: 2014-06-12 18:54:33
tags:
- HTML5游戏
categories:
- 115
---
演示地址：<iframe width="100%" height="400" src="http://jsfiddle.net/DkNb4/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

<!--more-->
<span style="color: #362e2b;">首先基本的canvas api 要掌握</span>

核心代码：
```javascript
var canvas = document.getElementById('canvas');  
var x = 50;  
var y = 50;  
var dx = 2;  
var dy = 4;  
var WIDTH;  
var HEIGHT;  
var ctx;  
var paddlex;  
var paddleh;  
var paddlew;  
var a;  
  
var rectee = {w:60,h:10,data:[{  
    x:10,  
    y:10  
},{  
    x:80,  
    y:10  
},{  
    x:150,  
    y:10  
},{  
    x:220,  
    y:10  
},{  
    x:10,  
    y:30  
},{  
    x:80,  
    y:30  
},{  
    x:150,  
    y:30  
},{  
    x:220,  
    y:30  
}]};  
function init_paddle() {  
  paddlex = WIDTH / 2;  
  paddleh = 10;  
  paddlew = 75;  
}  
function init() {  
  ctx = canvas.getContext("2d");  
  WIDTH = 300;  
  HEIGHT = 300;  
  canvas.addEventListener("mousemove",mouseMove)  
  a = setInterval(draw, 10);  
}  
function mouseMove (e) {  
    var rect = e.currentTarget.getBoundingClientRect();  
    var gravityPoint = {  
            x: e.clientX - rect.left,  
            y: e.clientY - rect.top  
          };  
    paddlex = gravityPoint.x - 30;  
}  
function circle(x,y,r) {  
  ctx.beginPath();  
  ctx.arc(x, y, r, 0, Math.PI*2, true);  
  ctx.closePath();  
  ctx.fill();  
}  
   
function rect(x,y,w,h) {  
  ctx.beginPath();  
  ctx.rect(x,y,w,h);  
  ctx.closePath();  
  ctx.fill();  
}  
   
function clear() {  
  ctx.clearRect(0, 0, WIDTH, HEIGHT);  
}  
   
function checkCollide(){  
    for (var i = 0; i < rectee.data.length ; i++) {  
        if (x <= (rectee.data[i].x + 60) && x >= (rectee.data[i].x) && rectee.data[i].y <= y && y <= (rectee.data[i].y + 10)) {  
            rectee.data.splice(i,1);  
            dy = -dy;  
        }  
      }  
      
}  
function draw() {  
  clear();  
  circle(x, y, 10);  
  rect(paddlex, 290, paddlew, paddleh);  
  for (var i = 0; i < rectee.data.length ; i++) {  
      rect(rectee.data[i].x, rectee.data[i].y, rectee.w, rectee.h);  
  }  
  if (rectee.data.length == 0) {  
      clearInterval(a);  
      alert("success!");  
  }  
  checkCollide();  
  if ((x + dx) > WIDTH || (x + dx) < 0 ) {  
      dx = -dx;  
  }  
  if ((y + dy) < 0) {  
      dy = -dy;  
  } else if ((y + dy) >= 290) {  
      if ((x + dx) >= paddlex && (x + dx) <= (paddlex + paddlew)) {  
          dy = -dy;  
      } else {  
          clearInterval(a);  
          alert("error!");  
      }  
  }  
    
  x += dx;  
  y += dy;  
    
}  
  
init();  
init_paddle();
```
<span style="color: #362e2b;">rectee定义并且存储了一组画被撞方块的位置和大小信息。</span>
<p style="color: #362e2b;">每一项的初始值必须定义在外部，这样可以实时修改他们的信息。</p>
<p style="color: #362e2b;">首先 画一个圆</p>

```javascript
function circle(x,y,r) {  
  ctx.beginPath();  
  ctx.arc(x, y, r, 0, Math.PI*2, true);  
  ctx.closePath();  
  ctx.fill();  
}
```
<span style="color: #362e2b;">很简单，直接调用这个方法。</span>
<p style="color: #362e2b;">然后画托板</p>

```javascript
function rect(x,y,w,h) {  
  ctx.beginPath();  
  ctx.rect(x,y,w,h);  
  ctx.closePath();  
  ctx.fill();  
}
```
<span style="color: #362e2b;">注意的是这个托板只能在canvas底部移动，所以传的Y值要固定，为canvas的高度减去托板的高度，很好理解。</span>
<p style="color: #362e2b;">然后让小球，也就是刚才画的圆形，不断移动。</p>
<p style="color: #362e2b;">思路就是setInterval方法每次画出小球的位置，然后调用clearRect来清除上一次的内容。</p>
<p style="color: #362e2b;">再次期间每次改变小球的位置，x y 的值就可以了。</p>
<p style="color: #362e2b;">然后判断当x 或者 y在临界边缘时，反弹，也就是代码里的 dx 和 dy取反就OK了。</p>

```javascript
if ((x + dx) > WIDTH || (x + dx) < 0 ) {  
      dx = -dx;  
  }  
  if ((y + dy) < 0) {  
      dy = -dy;  
  } else if ((y + dy) >= 290) {  
      if ((x + dx) >= paddlex && (x + dx) <= (paddlex + paddlew)) {  
          dy = -dy;  
      } else {  
          clearInterval(a);  
          alert("error!");  
      }  
  }  
    
  x += dx;  
  y += dy;</pre>

```
<span style="color: #362e2b;">然后，绑定mousemove事件给canvas，由于canvas内部的元素不支持绑定事件，所以只能通过给canvas绑定，然后判断canvas内元素的位置来决定是否触发事件。许多canvas的事件绑定就是根据这个原理来的。</span>


```javascriptfunction mouseMove (e) {  
    var rect = e.currentTarget.getBoundingClientRect();  
    var gravityPoint = {  
            x: e.clientX - rect.left,  
            y: e.clientY - rect.top  
          };  
    paddlex = gravityPoint.x - 30;  
}
```


<span style="color: #362e2b;">其中getBoundingClientRect为常用的判断canvas元素相对于body位置坐标的方法，直接用即可！</span>
<p style="color: #362e2b;">ok接下啦就是让被撞的方块消失。</p>

```javascript
function checkCollide(){  
    for (var i = 0; i < rectee.data.length ; i++) {  
        if (x <= (rectee.data[i].x + 60) && x >= (rectee.data[i].x) && rectee.data[i].y <= y && y <= (rectee.data[i].y + 10)) {  
            rectee.data.splice(i,1);  
            dy = -dy;  
        }  
      }  
      
}
```
<span style="color: #362e2b;">同理，先遍历方块，然后通过坐标来判断，OK!</span>
<p style="color: #362e2b;">很简单吧！</p>
<p style="color: #362e2b;">完！</p>
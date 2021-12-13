---
title: Canvas制作点赞送心动画
date: 2018-06-01 23:00:13
tags:
- canvas
- 点赞
categories:
- 802
photos: https://qiniu.nihaoshijie.com.cn/h52015031913.png
---

### 先看看效果

![](https://qiniu.nihaoshijie.com.cn/1527477112_60_w305_h232.gif)
<!--more-->
![](https://qiniu.nihaoshijie.com.cn/1527476652_7_w380_h433.gif)
### 实现原理

1.动画中每颗心都有自己的透明度，位移，角度，缩放的变化动画，针对这些动画，可以分别调用canvas的相关API：

```javascript

ctx.translate(this.x, this.y);
ctx.rotate(this.angle);
ctx.scale(this.scale, this.scale);
ctx.globalAlpha = this.opacity;
```

2.在有大于1颗心的情况时，针对每颗心，在实现角度，缩放时必须要保存上一次canvas的状态，改变完成之后在恢复：

```javascript

ctx.save();
ctx.translate(this.x, this.y);
ctx.rotate(this.angle);
ctx.scale(this.scale, this.scale);
ctx.globalAlpha = this.opacity;
ctx.restore();
```

3.计算心位移路径(3次被塞尔曲线)
公式：![](https://qiniu.nihaoshijie.com.cn/images/1527478364_96_w1098_h68.png)
转换成js代码计算心的x,y方向上的位移：

```javascript

    /**
     * 获得贝塞尔曲线路径
     * 一共4个点
     */
     function getBezierLine(heart){
         var obj = heart.bezierPoint;
         var p0 = obj.p0;
         var p1 = obj.p1;
         var p2 = obj.p2;
         var p3 = obj.p3;
         var t = heart.bezierDis;
         var cx = 3 * (p1.x - p0.x),
         bx = 3 * (p2.x - p1.x) - cx,
         ax = p3.x - p0.x - cx - bx,

    cy = 3 * (p1.y - p0.y),
    by = 3 * (p2.y - p1.y) - cy,
    ay = p3.y - p0.y - cy - by,

    xt = ax * (t * t * t) + bx * (t * t) + cx * t + p0.x,
    yt = ay * (t * t * t) + by * (t * t) + cy * t + p0.y;

    heart.bezierDis += heart.speed;

    return {
    xt: xt,
    yt: yt
    }
}

```

路径如图所示：
![](https://qiniu.nihaoshijie.com.cn/1527478501_6_w166_h306.png)

根据曲线路径方法可以实现心的运动轨迹：
![](https://qiniu.nihaoshijie.com.cn/1527479519_2_w305_h232.gif)

4.针对每一帧需要动态修改，心的位移，角度，和缩放：

```javascript

/**
* 计算缩放角度的方法
*/
function getFScale(heart){
    let _scale = heart.scale;

    // 随着距离起始点的距离增加，scale不断变大
    let dis = heart.orignY - heart.y;
    _scale = (dis / heart.scaleDis);

    // 当大于设置的阈值时变成1
    if (dis >= heart.scaleDis) {
        _scale = 1;
    }

    return _scale;
}
```

5.计算心角度变化

```javascript

/**
* 计算心左右摇摆的方法
*/
function rangeAngle(heart) {
    let _angle = heart.angle;
    // 心介于[start, end]之间不断变化角度
    if(_angle >= heart.angelEnd) {
    // 角度不断变小，向左摇摆
    heart.angleLeft = false;
    } else if (_angle <= heart.angelBegin){
    // 角度不断变大，向又摇摆
    heart.angleLeft = true;
    }
    // 动态改变角度
    if (heart.angleLeft) {
    _angle = _angle + 1;
    } else {
    _angle = _angle - 1;
    }
    return _angle;
}

```



### 组件化
整个组件代码不超过200行，相较于使用css3实现更加轻便，dom少，性能更好。
组件地址：https://github.com/lvming6816077/like-heart

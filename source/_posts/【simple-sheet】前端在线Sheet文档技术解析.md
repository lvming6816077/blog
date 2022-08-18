---
title: 【simple-sheet】前端在线Sheet文档技术解析
date: 2022-08-11 17:26:17
tags:
- sheet
- 在线文档
categories:
- 1929
photos: https://qiniu.nihaoshijie.com.cn/sheet122.png
---

开源项目[simple-sheet](https://github.com/simple-sheet/simple-sheet)，各位读者感兴趣可以一起参与进来！

## 什么是在线文档
一般来说，我们在PC端使用的微软Excel即是一个很强大的本地文档软件，而所谓“在线”，就是把这些功能移植到浏览器端，当然也包括移动web，小程序，Android或者iOS应用中，和本地最大的不同是在线文档除了基本的文档编辑操作之外，还加入了“协同”功能，能够同时多人编辑，这也就更能体现出在线文档的优势。
目前来说，比较流行的在线文档包括了Google Doc，钉钉文档，飞书文档，腾讯文档以及石墨文档等等，这些文档包括了Word，Excel，PPT等常用的模块，本文主要就Excel模块即Sheet文档进行技术解析。

<!--more-->
## 协同
正如上文所述，协同是在线文档的一大特色，其主要功能场景如下：

1. 多个用户在浏览器同时访问一个网页链接，并且可以实时同步到多方的修改内容。
2. 每个用户都可以对自己的修改进行保存，并且分享给任何人编辑和查看。
3. 可以看到用户每次修改的历史记录。

为了实现这些功能场景，就需要解决以下的技术问题：

1. **实时通信问题**：目前比较流行的方案是WebSocket，借助于WebSocket的长连接特性，可以实时将本地一些较为频繁的改动同步到后端，同时广播给其他正在编辑的用户。
2. **冲突处理问题**：一个比较直接的解决方案是-编辑锁，即当超过1用户在编辑同一个文件时，另外一个用户无法编辑，但这并不是一个很好的解决方案，所以针对于Sheet文档而言，当多个用户编辑同一个文件时，可能会修改到同一个单元格的内容，样式等，这时就会产生冲突问题。

处理冲突问题，常用的算法是OT算法，全称是Operation Transformation，是在线协作系统中经常使用的操作合并算法。其核心是一种操作合并指导思想，在不同的应用场景下有不同实现，这里不过多解释，感兴趣的同学可以看下[ot.js](https://github.com/Operational-Transformation/ot.js/)。关于协同这块，由于涉及到很多WebSocket和后端的交互，我们主要讲一下纯前端的在线编辑交互这块。

## 在线编辑
对于Sheet文档的在线编辑，可以看作是一个传统的前端单页应用，其包括了较为复杂的DOM交互，鼠标事件交互，样式，滚动处理等等，一般来说会采用纯DOM或者DOM和Canvas并配合Vue，React结合数据状态管理库Redux或者Vue来实现。

以DOM+Canvas+React+Redux为例子，其主要实现思路如下：

1. 每个单元格，可称为单个对象Cell，其主要属性包括了：内容，位置，长宽，样式等等，如下代码所示：

```javascript

export type CellAttrs = {
    x: number // x坐标
    y: number // y坐标
    width: number // 宽度
    height: number // 高度度
    value?: string // 值
    ownKey: string // key
    fill?: string // 背景颜色
    borderStyle?: BorderStyle // 边框属性
    fontWeight?: string | boolean // 加粗
    textColor?: string // 字体颜色
    verticalAlign?: string // 垂直居中
    align?: string // 水平居中
    fontFamily?: string // 字体
    fontSize?: number // 字号
    fontItalic?: string | boolean // 斜体
    textDecoration?: string | boolean // 下划线
    ...
}

```
2. 对于单个Sheet文档来说，包括了若干个Cell，并将这些Cell存储在Redux中进行管理。
3. 通过事件交互，修改单元格的属性会通过Redux同步到对应绑定的DOM中。
4. 不同的操作会有自己独立的层，这些操作改动最终会落到最底层的Canvas层单元格上面。


其页面结构如下图所示：

<img width=500 src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64213595b3534e8b94095109a3f60fdf~tplv-k3u1fbpfcp-watermark.image?" />

这些层，除了Canvas之外，一般设为`pointer-events:none`并且`z-index`高于Canvas。

### Canvas层

首先，使用Canvas一般是来渲染最底层的单元格，相对于DOM来实现，主要有以下好处：

1. 对于单个Sheet来说，一般会有上百个Cell，每个Cell会有单独的属性，采用DOM会占用内存会比Canvas稍大，在体验上会卡顿一些。
2. Canvas对于不同浏览器来说，其行为和表现较为统一（字体，颜色，形状等等），会减少一定的兼容性问题。
3. 对于Sheet来说，如果需要一些导出或者转换成图片之类的功能，可以采用Canvas原生的API实现，较为方便。

借助[react-konva](https://konvajs.org/docs/react/index.html)库，可以很方便的将React组件和Canvas结合起来，实现最原子的Cell操作。

### Dom选择区域层

这层主要实现的鼠标选择交互相关的UI展示，如下所示蓝色区域：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adf3d695ebea4712be0c523aae8a2f3a~tplv-k3u1fbpfcp-watermark.image?)

其包括了选择区域（透明蓝色背景）和当前激活单元格Cell（实现蓝色框）两部分，相关技术点主要有：

1. 结合Canvas点击时获取的坐标，设置当前激活Cell的位置和样式。
2. 结合鼠标mouseup，mousemove，mousedown事件和Canvas坐标，动态绘制选择区域的位置和样式。
3. 在选择区域的同时，结合Redux里面的Cell数据，方便获取到Cell列表原数据。

### Dom编辑器层

这层主要包括的是双击单个Cell单元格时，实时渲染出编辑器的逻辑，如下图所示：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62f918c8dc24469cb86d0cac165ea533~tplv-k3u1fbpfcp-watermark.image?)

简单来说，由一个文本输入框`<textarea>`实现，相关技术点主要有：

1. 结合Canvas点击时获取的坐标，设置编辑器的位置和样式。
2. `<textarea>`输入文字时，需要实时修改其`width`和`height`，来适配不同内容的展示。
2. 除了`<textarea>`之外，还可以有`<select>`，`<datepicker>`等等复杂的编辑器。

### Dom滚动条层

这层主要目的是为了模拟出Canvas的真实宽度和高度，从而通过`overflow:auto`来模拟出滚动条及其相关交互，从而使Canvas看起来是可以滚动的，其原理如下图：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1b9890911ce44ed9fbe082129d285a3~tplv-k3u1fbpfcp-watermark.image?)

### 其他层

这些层主要包括了一些其他逻辑，主要有：

1. 插入浮动图片层。
2. 右键菜单层。
3. 工具栏层。
4. 其他...

### 合并单元格

合并单元格相对于原子的修改Cell属性操作会比较复杂一些，这里可以将合并单元格看作是一个样式加属性调整的操作，相关技术点主要有：

1. 所选区域的单元格进行合并时，保留区域右下角的Cell属性，其他单元格样式置空，包括边框，使其成为完全没有样式的Cell。
2. 修改右下角单元格大小为所选区域大小，并绑定上左上角和右下角的key值，这样当点击时就可以确定合并的区域。
3. 将上面绑定的key值复制给区域内其他没有样式的Cell，这样当点击任一Cell时，都可以确定合并的区域。

key值的属性，如下所示：

```javascript

export type CellAttrs = {
    ...
    isMerge:[first:key,last:key] // 左上角和右下角的坐标key
}

```


### 修改Cell大小

由于修改列或者行的Cell的大小会影响之后的所有Cell位置，所以这是一个很消耗性能的操作，在`mousemove`时，可以采用节流`_.throttle`函数来减少一些高频操作，或者直接在`mouseup`的那一刻去做修改操作。


## 总结

对于Sheet文档来说，最重要的技术点还是对Cell数据的操作，如何管理好Cell，使其变换出不同的效果和样式是实现好Sheet功能的关键，而React以及Redux和Canvas则是提供了`数据->界面`的工具。快开始你的在线文档之旅吧。

我准备了一个开源项目[simple-sheet](https://github.com/simple-sheet/simple-sheet)，各位读者感兴趣可以一起参与进来！

---
title: React 16升级指南
date: 2018-01-15 20:23:10
tags:
- react16 升级
categories:
- 703
---


# 先说一下升级带来的好处
### 1. 更小的资源 

react+react dom打包之后 相较于上一个版本大小减少了约30%：如下图：
<!--more-->
React 15：
![](https://qiniu.nihaoshijie.com.cn/1513841084_41_w791_h51.png)
React 16：
![](https://qiniu.nihaoshijie.com.cn/blog/1513841084_41_w791_h51.png)

### 2. 更强的渲染性能
React 16是第一个对React核心代码进行了重构命名为React Fiber，Fiber 相较于之前最大的不同是它可以支持异步渲染，异步渲染的意义在于能够将渲染任务划分为多块。浏览器的渲染引擎是单线程的，这意味着几乎所有的行为都是同步发生的。React 16 使用原生的浏览器 API 来间歇性地检查当前是否还有其他任务需要完成，从而实现了对主线程和渲染过程的管理，这块具体的逻辑比较复杂，想要深入研究的可以参照React Fiber的源码。

笔者尝试了将原来频繁setState的场景，例如拖动效果：

```javascript
onTouchMove(e) {
   let po = this.getPosition(e);
    this.setState({
        top: po.y,
        left: po.x
    });
}
```
切换到React 16之后在某些低端机型上的拖动体验要比之前好一些，关于前端的渲染性能有待进一步的数据验证。

### 3. 服务端渲染性能的提升

React 16的SSR被完全重写，新的实现非常快，接近3倍性能于React 15，现在提供一种流模式streaming，可以更快地把渲染的字节发送到客户端。关于服务端性能渲染的数据有待和直出共同端统计：升级前后内存占用量对比 (蓝线升级后 绿线升级前)减低10% 的cpu内存使用量


### 4. render方法支持返回数组

之前使用React时，会经常遇到的一个问题就是在返回多余一个组件时要使用一个div包裹起来在升级之后就可以直接返回一个数组：

```javascript
render(){
  return [
    <div key="1">1</div>,
    <span key="2">2</span>,
    <p key="3">3</p>
  ]
}
```

### 5. 更强的错误处理机制

在React 15的时候，react的错误处理机制借助unstable_handleError来处理异常，这个有局限性也不是很好用，React 16将这一功能进行了升级，新增了组件生命周期的`componentDidCatch`方法：

我们可以新建一个错误处理的Component来处理react运行时的异常：

```javascript
import React, { Component } from 'react';
class ErrorHandler extends Component {
  constructor(props) {
    super(props); 
    this.state = { error: false };
  }
  componentDidCatch(error, info) {
    this.setState({ error, info });
  }
  render() {
    if (this.state.error) {
      return null;
    }
    return this.props.children;
  }
}
module.exports = ErrorHandler;
```

如何使用这个ErrorHandler组件有两种方式：

1) 包裹在页面根组件上，这样所有子组件发生异常时都会被这个组件捕获，在捕获异常的同时我们可以给与用户更友好的错误提示：

```javascript
render() {
    return (
        <ErrorHandler>
             <Main />
         </ErrorHandler>
    );
}
```

2) 自定义错误可能发生的情况，以局部组件为例：
我们将ErrorHandler包裹在Recommend上：

```javascript
render() {
    // <Main>
     return (
         <ErrorHandler><Recommend /></ErrorHandler>
     );
}
```
在ErrorHandler捕获到错误之后会采用错误的UI显示或者直接赋值为null，来保证页面其余的UI正常的加载。

# 改造过程的采踩坑

1.React 16依赖于`es6`的`Set`个`Map`，这些在android 4.4以下的机型是没有支持的，在笔者翻看源码后得知用的地方并不多，官方推荐是采用babel-polyfill来处理，但是对于部落并没有直接采用原因是：

1) `bable-polyfill`如果都引入的话基本上会增加大概200k的代码量，其实很多的shim是用不到的，如果想要单独抽离Set和Map又比较麻烦，bable-polyfill的Set和Map需要引入collection，collection又需要引入其他的js，抽离起来比较麻烦。

2) `Set`和`Map`的支持必须要优先于React引入，而部落的React是单独打包的，所以如果要引入babel-polyfill还要修改构建单独生成一份pollyfill的js放在react之前。

解决办法：手写一份Set和Map的支持，大致原理就是利用数组进行改造，然后挂在window对象上，在打包时和React文件一起打包即可。

2. React 16不再支持采用es5写法定义的Component例如：

```javascript
var Main = React.createClass({
    render: function () {
        return null;
    }
});
```

解决办法有2个：

1)  引入create-react-class：

```javascript
var createReactClass = require('create-react-class');

var Main = createReactClass({
    render: function() {
        return null;
    }
});
```

2) 改造成es6的写法：

```javascript
class Main extends React.Component {
    constructor(props) {
        super();
 }
 ```

部落改造选择了后者，评估了一下工作量差不多有60多个组件要改，对于es6的趋势，所以基本上一次性改成es6最彻底。

3. 改成es6之后this的指向问题：

```javascript
<Rank onTap={::this.onTap} ></Rank>
```

4.如果是采用的npm方式升级的话很简单，直接在npm上升级即可，如果是采用单独文件的方式升级要在[react官网](https://github.com/facebook/react/releases)下载对应版本号进行升级即可。

## 备注：

1. 目前部落采用的React版本是16.2.0
2. 改造升级React 16工作量比较大，对于原先采用es5写的业务，需要一个一个组件的修改，还要关注`this`的问题避免报错。
3. 如果代码中使用了`require.ensure`来懒加载组件，在修改的时候可以先将ensure注释掉，这样如果懒加载组件中有问题也可以立刻发现。


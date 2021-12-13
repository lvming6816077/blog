---
title: Webpack1升级Webpack2整理总结
date: 2017-07-09 23:00:13
tags:
- webpack2
- 升级
categories:
- 702
photos: https://qiniu.nihaoshijie.com.cn/u%3D3396435274%2C4251997814%26fm%3D26%26gp%3D0.jpg
---

Webpack2已经发布半年之多了，就连webpack3都已经发布了，但是项目目前还是使用的webpack1，有点跟不上节奏，webpack2的诸多特性类似tree-shaking等等新特性还是比较令人激动的，现在整理一下从webpack1升级到webpack2的过程。
### 1.package.json的整体变化
<!--more-->
如下图：分别对相关的组件进行升级，图上所示的需要升级的组件只是针对自己项目的，如果需要升级其他组件可以根据自己的需求升级。
![](https://qiniu.nihaoshijie.com.cn/blogcaikeng1.png)

### 2.安装升级组件
每种组件的升级有两种办法：
1. 将组件从node_modules里面删掉，同时删掉package.json里的配置，然后在`npm install XXX`可以直接从官网下载最新的包。
2. 采用`npm install --save-dev xxx@版本号`的方式安装指定版本的包，同时package.json会自动更新。
个人推荐使用第二种方式，比较灵活。

### 3.修改webpack配置文件webpack.config.js

#### 1). resolve.extensions的修改：
将`extensions: ['', '.js', '.jsx']`改为`extensions: ['.js', '.jsx']`webpack2修复了webpack1必须要求第一个数组元素填空的要求。
#### 2). resolve.mudule的修改：
主要是将原本的loaders和loader的搭配写法改为rules加上use的写法:
```javascript
loaders: [{
    test: /\.jsx?$/,
    loader: 'happypack/loader?id=jsx',
    include: path.resolve(config.srcPath)
}, {
    test: /\.css$/,
    loader: ExtractTextPlugin.extract("style-loader",'happypack/loader?id=css'),
    include: path.resolve(config.srcPath)
}, {
    test: /\.less$/,
    loader: ExtractTextPlugin.extract("style-loader", 'happypack/loader?id=less'),
    include: path.resolve(config.srcPath)
}, {
    test: /\.(jpe?g|png|gif|svg)$/i,
    loader: 'url-loader?limit=1500&name=images/[name].[hash].[ext]',
    include: path.resolve(config.srcPath)
}]
```
改为：
```javascript
rules: [{
    test: /\.jsx?$/,
    use: 'happypack/loader?id=jsx',
    include: path.resolve(config.srcPath)
}, {
    test: /\.less$/,
    use: ExtractTextPlugin.extract({fallback:"style-loader", use:'happypack/loader?id=less'}),
    include: path.resolve(config.srcPath)
}, {
    test: /\.css$/,
    use: ExtractTextPlugin.extract({fallback:"style-loader", use:'happypack/loader?id=css'}),
    include: path.resolve(config.srcPath)
}, {
    test: /\.(jpe?g|png|gif|svg)$/i,
    loader: 'url-loader?limit=1500&name=images/[name].[hash].[ext]',
    include: path.resolve(config.srcPath)
}]
```
这个修改并不是必须的，webpack2同时支持原有的写法，但是为了规范起见还是采用webpack2新写法为好。

#### 3). ExtractTextPlugin写法的修改：

```javascript
ExtractTextPlugin.extract("style-loader", 'happypack/loader?id=less')
```
修改为
```
ExtractTextPlugin.extract({fallback:"style-loader", use:'happypack/loader?id=less'})
```
坑：在网上查资料时有些是将`fallback`写成`fallbackCallback`不要相信，还是安装`fallback`来。

#### 3). ExtractTextPlugin的plugins写法的修改：

```
new ExtractTextPlugin("../css/[name].min.css", {
    allChunks: true
}),
```
修改为：
```
new ExtractTextPlugin({
    filename: "../css/[name].min.css",
    allChunks: true
}),
```
没啥说的，就是将原本的string加obj的形式修改为obj形式的参数。

#### 4). CommonsChunkPlugin的plugins写法的修改：

```
new CommonsChunkPlugin("common", "common.min.js", ['index', 'barindex', 'detail']),
```
修改为：
```
new webpack.optimize.CommonsChunkPlugin({
    name: 'common', 
    filename: 'common.min.js',
    chunks: ['index', 'barindex', 'detail'] 
}),
```
坑：注意如果采用了CommonsChunkPlugin将公共模块提取出来，同时采用了require.ensure的方式来异步拉取js，则上一步的ExtractTextPlugin的allChunks必须设置成为true。
坑：路径问题：webpack2采用绝对路径，所以直接填写的相对路径都需要__dirname来转换成绝对路径。

#### 5). happypack写法的修改：
1. happypack：采用多进程来加快webpack的构建速度的组件。目前最新版本是4.0.0但是笔着并没有升级到最新版本，而是采用了更稳定的3.1.0版本，如果升级到最新版本原有的相关配置项都需要修改，例如`cache`属性将不再被支持，可以参照官方的[文档](https://www.npmjs.com/package/happypack)。
2. happypack自带有.happypack目录的缓存，记得升级完成之后需要删除这个目录下的文件，重新生成。

### 4.关于require.ensure
1）升级webpack2的另外一个重要因素就是：原本采用`require.ensure`来异步加载代码时无法捕获到组件拉取失败的回调，需要单独引入一个插件`require('require-error-handler-webpack-plugin')`来解决,但是升级了webpack2之后，原生就已经支持了，所以不需要引入这个插件了。
![](https://qiniu.nihaoshijie.com.cn/blog/caikeng2.png)
2) 在webpack2环境下使用`require.ensure`还需要注意一点就是必须引入`babel-polyfill`来支持Promise写法，因为webpack2的`require.ensure`采用的Promise来实现回调，所以必须引入这个。
3）引入babel-polyfill之后可能会使js文件增加许多，这里同样推荐使用es6-promis来只针对promise来进行polyfil，减少文件增加的大小。

### 5.关于tree-shaking
在升级完成webpack2之后，webpack构建会自动支持[tree-shaking](https://webpack.js.org/guides/tree-shaking/)特性来进行代码层级的压缩优化，但是也有一些需要了解的：
1）先要理解关于CommonJS和ES6 modules的区别。
2）虽然采用了babel之后，代码还是最终会转为浏览器兼容性较好的es5，但是我们在写代码的时候还是推荐采用ES6 modules的写法。
3）基于ES6来写，tree-shaking才能职别，才能根据import和export来查找引入的多余的代码。
4）同时需要修改babel的配置文件`["es2015",{"modules":false}]` 这样才能让webpack2先于babel来进行代码级别的tree-shaking优化。

### 参考资料：
[webpack2升级官方指南](https://webpack.js.org/guides/migrating/)
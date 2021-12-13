---
title: Vue开发B端系统常见性能优化技巧
date: 2021-04-26 17:26:17
tags:
- Vue
- B端系统
categories:
- 919

---



最近工作上一直接触的时B端的管理系统，基本上都是采用Vue Cli生成脚手架后就直接开始写业务逻辑，所以一般都会忽略一些性能优化相关的想法和工作。虽说这种系统对前端要求一般来说比较简单，大多数是一些数据校验和可视化展示，可能重点的工作量在于业务逻辑，但是作为一个前端项目来说，基本的优化还是需要做的，可以参考这篇[文章](https://juejin.cn/post/6844903801296519181)。

而本文将会介绍一些基于Vue Cli项目会被我们忽略的一些性能优化，尤其是首屏优化的技巧，如果对你有帮助，可以点赞支持一下。
<!--more-->
## UI库按需加载

对于大多是B端系统而言，都会使用一些一些UI组件库，例如Ant Design或者是Element UI，这些组件都是支持按需引入，我们在使用这些组件时，如果只用到了其中一部分组件，可以配置按需加载，在`main.js`中修改代码：
```javascript
import {
    Pagination,
    Icon,
    Tabs,
} from 'ant-design-vue'
// import 'ant-design-vue/dist/antd.css'  已经通过babel引入 这里就不全局引入了

Vue.use(Pagination)
    .use(Icon)
    .use(Tabs)
```

然后修改`babel.config.js`，如下：
```javascript
  "plugins": [
    ["import", { "libraryName": "ant-design-vue", "libraryDirectory": "es", "style": "css" }], // `style: true` 会加载 less 文件

  ]
```

这样，组件对应的js和css文件就可以实现按需加载，关于`babel.config.js`的配置，可以参考之前的一篇[文章](https://juejin.cn/post/6906362999703863304)。

## 使用webpack-bundle-analyzer分析打包后的资源包
`webpack-bundle-analyzer`是一个webpack插件，它可以直观分析打包出的文件包含哪些，大小占比如何，模块包含关系，依赖项，文件是否重复，压缩后大小如何，针对这些，我们可以进行文件分割等操作，是我们后面分析资源包大小的基础。
```javascript
// 分析包内容 
npm install webpack-bundle-analyzer --save-dev
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin; 
module.exports = { 
plugins: [ 
// 开启 BundleAnalyzerPlugin 
      new BundleAnalyzerPlugin(), 
    ], 
};   

```
配置完成之后，只需要`npm run build`即可，会自动打开http://127.0.0.1:8888/

![686316-20180808143606193-2047520260.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/446d403e2acb4c4c867230399e7c0a84~tplv-k3u1fbpfcp-watermark.image)
## 路由懒加载
对于一般比较大型的B端管理系统项目，基本上都会使用Vue Router来管理路由，这些项目涉及的页面都比较多，所以为了防止首屏资源过大，需要采用路由懒加载资源即Code Splitting，将每个页面的资源进行分离，这个只需在`router.js`里配置即可：
```javascript
// 采用箭头函数和import进行懒加载处理
component: () => import('./index.vue')
```

## 有选择的使用prefetch和preload

### prefetch

```
<link rel="prefetch" ></link>
```
这段代码告诉浏览器，这段资源将会在未来某个导航或者功能要用到，但是本资源的下载顺序权重比较低。也就是说prefetch通常用于加速下一次导航，而不是本次的。

### preload

```
<link rel="preload" ></link>
```
preload通常用于本页面要用到的关键资源，包括关键js、字体、css文件。preload将会把资源得下载顺序权重提高，使得关键数据提前下载好，优化页面打开速度。

在使用Vue Cli生成的项目里，当我们配置了路由懒加载后，默认情况下webpack在构建时会对所有的懒加载资源进行`prefetch`和`preload`，所以当你打开首页时，会看到大量的`prefetch`和`preload`请求，如下图：
![0563B5CD-D598-40d1-A6D6-CE194BD3E8F5.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e74b65731ff94c3ca31fb4481f3cdc9c~tplv-k3u1fbpfcp-watermark.image)
虽说prefetch会在浏览器空闲时，下载相应文件，但这是一个很笼统的定义，这些大量的预加载资源会占用浏览器的资源，可能会导致一些关键的api请求或者图片请求受影响，所以对于这种情况，可以选择指定一些资源进行预加载或者禁止掉这些预加载，代码如下：
```javascript
// 禁止prefetch和preload
chainWebpack: (config) => {
  config.plugins.delete('prefetch')
  config.plugins.delete('preload')
}
// 有选择的prefetch和preload
config.plugin('prefetch').tap(options => {
    options[0].fileBlacklist = options[0].fileBlacklist || []
    options[0].fileBlacklist.push(/myasyncRoute(.)+?\.js$/)
    return options
})
```
上面代码修改`vue.config.js`的`chainWebpack`来添加配置。

## 拆分node_modules

对于一般比较大型的B端管理系统项目，一般都会使用一些UI组件库，图表库等第三方库，这些第三方库一般都是通过`npm install`到`node_modules`下面，并且每个页面大多都会使用这些库，所以会在`main.js`中引入：

```javascript
Vue.use(Antd)
Vue.use(Echarts)

```
在打包时，webpack会将这些公用的模块，抽离出来，放到一个公共模块中。这样不管这个模块被多少个入口引用，都只会在最终打包结果中出现一次，以此解决代码冗余。

这些在使用Vue Cli的项目中已经自动帮我们做好了，主要利用了`optimization.splitChunks`配置，但是当我们把这些公用的模块都堆在一个模块中，这个文件可能异常巨大（一般是app.js，首屏就会加载这个文件，引用的公共第三方库越多，这个文件越大），也是不利于网络请求和页面加载的。所以我们需要把这个公共模块再按照一定规则进一步拆分成几个模块文件，减小文件体积，同时也可以利用HTTP2多路复用特性，这些则需要我们自己配置，代码如下：
```javascript
  config.optimization.splitChunks.cacheGroups.antdv = {
      name:'chunk-antdv',// 抽离的模块名
      priority: 20,// 优先级
      test: /ant-design-vue/,
      chunks: 'initial',
      reuseExistingChunk: true,
      enforce: true // 强制抽离此模块
  }
  config.optimization.splitChunks.cacheGroups.antdv = {
    name:'chunk-echarts',// 抽离的模块名
    priority: 20,// 优先级
    test: /echarts/,
    chunks: 'initial',
    reuseExistingChunk: true,
    enforce: true // 强制抽离此模块
}
```
上面代码在`vue.config.js`里，修改`configureWebpack`项来添加配置，关于`optimization.splitChunks`的配置，可以参考这个[文档](https://www.webpackjs.com/plugins/split-chunks-plugin/)，这里可以简单的解释一下上面配置的含义：

1. 每个`cacheGroups`代表需要抽离的第三方库，这里有两个分别是ant-design-vue和echarts。
2. `test`项表示要扫描的`node_modules`文件路径，这里简单采用了正则来匹配。
3. `chunks`项有三个选项：`all`, `async`, `initial`。`initial` 代表负责异步模块和非异步模块加载的公共抽离，`async`代表只负责异步模块加载的公共抽离, `all`结合前面两者特点。

## 定制资源文件加载位置

对于使用Vue Cli创建的项目，一般来说都不会刻意的修改css，JavaScript文件加载的顺序，例如默认情况下webpack会把css放在<head>标签里，JavaScript文件放在<body>标签底部，所以，有时我们会在<body>标签底部引入一些第三方的JavaScript文件，例如jquery或者高德地图的api文件等等，如下所示：
```html
  <body>
    <div id="app"></div>
    <script type="text/javascript" src="https://webapi.amap.com/maps?v=1.4.15&key=c65f481aebaed72935e11654238ab5a6"></script>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
    <!-- built files will be auto injected -->
  </body>
```
我们知道，浏览器在解析HTML时，遇到非异步的资源，尤其是JavaScript文件资源时，会下载并解析，那么越先加载就越先解析，
而默认情况下，项目构建之后，业务的核心JavaScript文件会被放在这些第三方的JavaScript文件后面加载，会对首屏的展示有一定的影响。

解决这个问题，需要修改`html-webpack-plugin`配置，然后手动在index.html里，自定义文件的加载位置，代码如下：

```javascript
  config.plugin('html')
  .tap(args => {
    args[0].inject = false
    return args
  })
```
```html
<% for (var css in htmlWebpackPlugin.files.css) { %>
  <link href="<%=htmlWebpackPlugin.files.css[css] %>" rel="stylesheet">
<% } %>
...
<% for (var chunk in htmlWebpackPlugin.files.chunks) { %>
<script type="text/javascript" src="<%=htmlWebpackPlugin.files.chunks[chunk].entry %>"></script>
<% } %>
```

这样，就可以将一些重要的首屏资源加载优先级提升，在一定情况下，优化首屏的加载速度。

## PrerenderSpaPlugin首屏预渲染

对于大多数的B端系统而言，大部分都是一个已Vue开发的单页应用SPA，所以首屏的内容基本上都是依赖JavaScript来进行渲染的，这样不仅依赖于JavaScript文件资源的加载，而且不利于SEO（可能大部分B端系统无需SEO，但不排除一些欢迎页或者品牌首页类似的页面有SEO的需求）。

PrerenderSpaPlugin是一个可以将一些页面预先打包成静态资源页面的webpack插件，并且可以结合`vue-router`，来根据路由，配置指定的页面，它的工作流程如下图：

![20181128215005309.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99f9e23383d54287958c36c9e5b1e949~tplv-k3u1fbpfcp-watermark.image)

基于此，我们可以修改`vue.config.js`，添加PrerenderSpaPlugin支持，代码如下：

```javascript
npm install prerender-spa-plugin --save-dev
const PrerenderSpaPlugin = require('prerender-spa-plugin')
...
new PrerenderSpaPlugin({

  staticDir: resolve('dist'),
  // 对应自己的路由文件，比如a有参数，就需要写成 /a/param1。
  routes: ['/login', '/about'],
}),
```
上面代码中，给`/login`和`/about`两个路由配置了预加载，在项目dist之后，就可以看到两个独立的html文件，在通过一些nginx配置，将对应的路径直接转发到对应的静态页面即可。更多关于PrerenderSpaPlugin可以参考[配置](https://github.com/chrisvfritz/prerender-spa-plugin)。

## 图片压缩

默认情况下，Vue Cli创建的项目不会对项目中引入的一些图片资源进行二次压缩，这时如果引入一些较大的图片，并且比较多的情况下，会导致dist之后的包变得很大，这时可以采用`image-webpack-loader`来开启配置图片压缩，添加`image-webpack-loader`，代码如下：
```javascript
npm install image-webpack-loader --save-dev
...
config.module
    .rule('images')
    .use('image-webpack-loader')
    .loader('image-webpack-loader')
    .options({ bypassOnDebug: true })
    .end()
```

上面代码修改`vue.config.js`的`chainWebpack`来添加配置，`image-webpack-loader`始于[imagemin](https://github.com/imagemin/imagemin)开发的webpack的loader，可以支持无损和有损压缩。

## 结语

对于大多数的B端系统而言，可能更注重复杂的业务逻辑实现，但可能也会忽略一些基本的性能优化，对于整个系统而言，给到良好的用户体验才能让系统变得更加健壮，同时也是一名前端工程师的成就所在。



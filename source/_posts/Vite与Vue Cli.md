---
title: Vite与Vue Cli
date: 2021-11-22 17:26:17
tags:
- Vite
- Vue Cli
categories:
- 1201

---


Vite和Vue Cli可以是师出同门，都属于Vue整个团队的产物，他们的功能也非常相似，都是一个提供基本项目脚手架和开发服务器的构建工具。那么在这里就有几个问题需要讨论：

- Vite和Vue Cli的主要区别。
- Vite和Vue Cli哪个性能更好。
- 实际项目中如何选择。

<!--more-->

## Vite和Vue Cli的主要区别

Vite在开发环境下基于浏览器原生ES6 Modules提供功能支持，在生产环境下基于RollUp打包，Vue Cli不区分环境，都是基于Webpack。可以说在生产环境下，两者都是基于源码文件，RollUp和Webpack都是将代码进行处理，并提供出浏览器页面所需要的HTML，JavaScript，CSS，图片等静态文件。但是对于开发环境的处理，两者却有不同：

- Vue Cli在开发环境下也是基于对源码文件的转换，即利用Webpack对代码打包，结合webpack-dev-server提供静态资源服务。

- Vite在开发环境下基于浏览器原生ES6 Modules，无需对代码进行打包直接让浏览器使用。

Vite正是因为利用浏览器原生功能，而省略掉耗时的打包流程，才使得开发环境下体验非常快。而对于生产环境，他们各自所依赖的Webpack和RollUp这两个工具其实也各有优劣：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9eddb1346e6b4390a0c723d5f9531ba9~tplv-k3u1fbpfcp-watermark.image?)
- Webpack有着生态更加丰富的loader，可以处理各种各样的资源依赖，以及代码拆分(Code Splitting)和按需合并。RollUp的插件生态较Webpack弱一些，但是也可以满足基本的日常开发需要，且不能支持Code Splitting和热更新HMR。

- RollUp对ES6 Modules的代码依赖方式天然支持，而对于类似CommonJS或者UMD方式的依赖却无法可靠的处理，Webpack借助于自己的__webpack_require__函数和Babel，对于各种类型代码都支持的比较好。

- RollUp会静态分析代码中的import，并将排除任何未实际使用的代码即Tree Shaking支持很好，Webpack则从Webpack 2版本开始支持Tree Shaking，且要求使用原生的import和export语法并且不能被Babel转换过的代码。

- RollUp编译的代码可读性更好（虽然基本不会去阅读这些代码），没有过多的冗余代码，而Webpack则会插入很多__webpack_require__函数影响代码的可读性。



由于Webpack的支持性比较多，处理的场景更为广泛，而RollUp对源码的处理更加简洁，所以业界一般认为对于项目业务使用Webpack，对于类库使用RollUp，而之所以Vite使用RollUp，可能原因是整体上对浏览器ES6 Modules的使用，为了更加统一，并且摆脱Vue Cli那样对Webpack过于依赖吧。

## Vite和Vue Cli哪个性能更好

这个不用多说，必然是Vite的更快，在开发环境下体验更好。

Vue Cli的Webpack的工作方式是，它通过解析应用程序中的每一个JavaScript模块里面import或者require，借助各种loader将整个应用程序构建成一个基于JavaScript的捆绑包，并转换文件（例如Sass、.vue等）。这都是在webpack-dev-server服务器端提前完成的，文件越多，依赖越复杂，则消耗时间更多。

Vite不捆绑应用服务器端。相反，它依赖于浏览器对ES6 Modules的原生支持，浏览器直接通过HTTP请求JavaScript模块，并且在运行时处理，而对于例如Sass、.vue文件等则单独采用插件处理，并提供静态服务。这样耗时的大头JavaScript模块处理就被单独剥离了出来，利用浏览器高效处理，并且对于文件的多少，影响并不大，这样消耗的时间就更少。我们可以沿用之前章节中的图来总结这种模式区别，如图所示。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e1386bc85cf49139aa26cb8c5a12960~tplv-k3u1fbpfcp-watermark.image?)
所以总结下来，在开发模式下，Vite显然要比Vue Cli性能强，生产模式下相差不大。
## 实际项目中如何选择
关于实际项目中如何选择Vite和Vue Cli，我们先来总结一下他们各自的优缺点，如下表格所示。

Vue Cli:

Vue Cli优点          | Vue Cli缺点 |
| ------------------ | --------- |
| 生态好，应用实际项目多        | 开发环境慢，体验差 |
| 可以和Vue2.x，Vue3.x结合 | 只支持Vue    |
| 直接解析各种类型代码依赖       | 产物冗余代码多   |
| 构建配置项丰富，插件全

Vite:

Vite优点       | Vite缺点             |
| ------------ | ------------------ |
| 开发环境速度快，体验好  | 只针对ES6浏览器          |
| 支持Vue，React等 | 脚手架不包括Vuex，Router等 |
| 产物简洁清晰

Vite在开发环境下体验强，速度快是其核心优势，但是与Vue Cli/Webpack不同，Vite无法创建针对旧版浏览器，这对于一些用户来说是一个抉择点。另外，Vue Cli作为老牌构建工具，使用者众多，更加经得住实历史的考验，并且得益于使用者众多，所以在生态环境和插件数量方面更好。

Vite是一个新兴的产物，Vue团队更想把Vite做成一个通用的构建工具而不只限制于Vue，所以后面也会主推Vite，所以回到问题上来，Vue Cli和Vite到底怎么选笔者认为还是要根据自己实际的业务场景来，这里总结几条原则：


- 当前已正在运行的Vue Cli项目，不建议切换Vite，维稳！

- 企业大型项目，构建功能复杂需求多，建议Vue Cli，稳定坑少！

- 小型项目，快应用，Vue3项目，建议Vite，体验好！

 

以上原则仅供参考。






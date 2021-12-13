---
title: Vue3源码解析--目录结构
date: 2021-12-26 17:26:17
tags:
- Vue3
- 源码解析
categories:
- 913

---

## 下载并启动Vue 3源码
Vue 3的源码地址可以在Github上下载，首先安装Git，然后使用如下命令：
```
git clone https://github.com/vuejs/vue-next.git
```
<!--more-->
完成后，Vue 3的源码被下载到vue-next文件夹下，打开CMD命令行工具，由于Vue 3源码采用Yarn进行构建，所以需要提前安装Yarn，如下命令安装：
```
npm i yarn -g
```
然后进入vue-next目录，执行如下命令：
```
yarn --ignore-scripts
```
执行完该命令后，相关依赖就已经安装，执行npm run dev命令，即可开启Vue 3的源码调试模式，查看vue-next\packages\vue\dist目录下的vue.global.js文件，即是开发模式下构建出的Vue 3源码文件，源码完整目录结构，如图所示。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e59a1281bfa547c1824c35080867915c~tplv-k3u1fbpfcp-watermark.image?)

其中，Vue 3和核心源码都在packages里面，并且是基于RollUp构建，其中每个目录代表的含义，如下所示：
```
├── packages              
│   ├── compiler-core    // 核心编译器（平台无关）
│   ├── compiler-dom     // dom编译器
│   ├── compiler-sfc     // vue单文件编译器
│   ├── compiler-ssr     // 服务端渲染编译
│   ├── global.d.ts      // typescript声明文件
│   ├── reactivity       // 响应式模块，可以与任何框架配合使用
│   ├── runtime-core     // 运行时核心实例相关代码（平台无关）
│   ├── runtime-dom      // 运行时dom 关api，属性，事件处理
│   ├── runtime-test     // 运行时测试相关代码
│   ├── server-renderer   // 服务端渲染
│   ├── sfc-playground    // 单文件组件在线调试器
│   ├── shared             // 内部工具库,不对外暴露API
│   ├── size-check          // 简单应用，用来测试代码体积
│   ├── template-explorer  // 用于调试编译器输出的开发工具
│   └── vue                 // 面向公众的完整版本, 包含运行时和编译器
```
## 目录模块

通过上面源码结构，可以看到有下面几个模块比较特别：
* compiler-core
* compiler-dom
* runtime-core
* runtime-dom

可以看到core, dom 分别出现了两次，那么compiler和runtime它们之间又有什么区别呢？

* compile：我们可以理解为程序编绎时，是指我们写好的源代码在被编译成为目标文件这段时间，可以通俗的看成是我们写好的源代码在被构建工具转换成为最终可执行的文件这段时间，在这里可以理解为我们将.vue文件编绎成浏览器能识别的.js文件的一些工作。
* runtime：可以理解为程序运行时，即是程序被编译了之后，在浏览器打开程序并运行它直到程序关闭的这段时间的系列处理。


在package目录下，除了上面列举的4个目录外，还有reactivity目录比较重要，他是响应式模块的源码，由于Vue 3整体源码采用的Monorepo规范，所以其下面每个子模块都可以独立编译和打包，从而独立对外提供服务，在使用时采用require('@vue/reactivity')引入，进入reactivity目录下可以看到有对应的package.json文件。

其他目录：compiler-sfc是一个单文件组件编译工具，server-renderer目录是服务端渲染模块的源码，sfc-playground是一个在线Vue单文件组件调试工具，shared目录则包括了常用的工具库方法的源码，size-check是一个工具，可以用来测试代码体积，template-explorer用于调试编译器输出的开发工具，最后的vue目录则是Vue.js的完整源码产生目录。
## 构建版本
比较常见的是构建出来的Vue会有vue.global.js或者vue.runtime.global.js两种版本，他们分别表示：

* vue.global.js：是包含编译器和运行时的“完整”构建版本，因此它支持动态编译模板。
* vue.runtime.global.js：只包含运行时，并且需要在构建步骤期间预编译模板。

其中，如果需要在客户端上编译模板 (即：将字符串传递给template 选项，或者使用元素的DOM内HTML 作为模板挂载到元素)，将需要编译器，因此需要完整的构建版本，如下代码所示：
```
// 需要编译器
Vue.createApp({
  template: '<div>{{ hi }}</div>'
})

// 不需要
Vue.createApp({
  render() {
    return Vue.h('div', {}, this.hi)
  }
})
```
当使用Webpack的vue-loader时，*.vue 文件中的模板会在构建时预编译为JavaScript，在最终的捆绑包中并不需要编译器，因此可以只使用运行时构建版本。所以，如果直接在浏览器打开Vue的页面，可以直接采用`<script>`引入完整版本（正如之前章节中的示例代码一样），如果采用构建工具例如Webpack进行构建，则可以使用import引入运行时版本，和构建相关的脚本源码都在vue-next/scripts下面。例如，当我们需要构建出完整版时，可以在vue-next根目录执行，或者修改package.json的dev命令：
```
node scripts/dev.js -f global-runtime
```
当需要构建运行时版本，执行：
```
node scripts/dev.js -f global
```
具体的-f参数以及其他参数配置含义可以在vue-next/rollup.config.js里面找到。
执行完成后在vue-next\packages\vue\dist目录下可以得到对应的文件。所有Vue 3可构建出来的版本如下所示：
```
// cjs（用于服务端渲染）
vue.cjs.js
vue.cjs.prod.js（生产版，代码进行了压缩）

// global（用于浏览器<script src="" />标签导入，导入之后会增加一个全局的Vue对象）
vue.global.js
vue.global.prod.js（生产版，代码进行了压缩）
vue.runtime.global.js
vue.runtime.global.prod.js（生产版，代码进行了压缩）

// browser（用于支持ES6 Modules浏览器<script type="module" src=""/>标签导入）
vue.esm-browser.js
vue.esm-browser.prod.js（生产版，代码进行了压缩）
vue.runtime.esm-browser.js
vue.runtime.esm-browser.prod.js（生产版，代码进行了压缩）

// bundler（这两个版本没有打包所有的代码，只会打包使用的代码，需要配合打包工具来使用，会让Vue体积更小）
vue.esm-bundler.js
bue.runtime.esm-bundler.js
```
不同构建版本的Vue源文件需要在不同的平台和环境中使用，便于开发者可以选择合适的场景来使用。





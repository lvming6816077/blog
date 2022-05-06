---
title: B端系统改造Vite最佳实践
date: 2022-03-16 17:26:17
tags:
- vue3
- Vite
categories:
- 1903

---

## 目标和环境

最近，迫于原本的vue-cli构建速度越来越慢，每次修改代码要等很久才能看到效果，已经严重了影响了开发效率，遂决定优化项目的构建机制，也体验一把不到1s的飞速快感。

项目采用vue2 + antdv1 + vue-cli的B端项目，目标是改造成dev环境下同时支持vite和webpack，生产环境下支持webpack，这里就需要修改代码和配置时做到既能用webpack也能用vite，并且要尽量减少对src目录下的代码修改，保证生产环境不出问题。
<!--more-->
改造前后对比：

| 构建工具        | dev启动耗时   |  热更新 HMR  |
| --------   | -----:  | :----:  |
| webpack      | 51s   |   5s     |
| vite        |   17s(无预构建)<br>600ms(已有预构建)   |   500ms   |

在dev环境下，vite的效率提升非常明显，生产环境下，两者差距不大，这里不做对比。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/319317204442480298816c5ed4a4a1f6~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7f5786f398b4cc18be7d68dec58de07~tplv-k3u1fbpfcp-watermark.image?)

## 改造步骤

在项目根目录下安装vite相关的模块，包括后面会用到的插件，如下所示：

```
    "vite": "^2.8.4",
    "vite-plugin-antdv1-momentjs-resolver": "^1.0.6",
    "vite-plugin-externals": "^0.4.0",
    "vite-plugin-filter-replace": "^0.1.9",
    "vite-plugin-importer": "^0.2.5",
    "vite-plugin-replace": "^0.1.1",
    "vite-plugin-restart": "^0.1.1",
    "vite-plugin-vue2": "^1.9.3",
    "@originjs/vite-plugin-commonjs": "^1.0.3",
    "@vue/babel-helper-vue-jsx-merge-props": "^1.2.1",
    "@vue/babel-preset-jsx": "^1.2.4",
```

创建vite需要的`vite.config.js`文件和`index.html`文件，并添加对应的配置。


## 具体配置和踩坑点

### require is not defined
Vite开发模式下，完全安装ESM进行构建，无法识别原本在webpack中使用的require语法，针对此问题，有以下方案：

1. 采用import替换
```
<img :src="require('./a.png')" />

// 替换为

const a = import('./a.png')

<img :src="a" />
```

2. 采用`@originjs/vite-plugin-commonjs`插件

```
import { viteCommonjs, esbuildCommonjs } from '@originjs/vite-plugin-commonjs';
...
plugins: [
    viteCommonjs({
        transformMixedEsModules: true, // 如果项目中混用require和import，配置true
    }),
]
...
```

注意，vite在生产环境下通过`commonjsOptions`会默认转换require，此插件只会在dev下生效，建议统一改为`import xxx`方案，这种方案在vue cli的webpack中也可兼容。


### Unknown theme type: undefined, name: undefined(and-design-vue)
报错如下图：
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71820697768a48e5886a72db80668bf2~tplv-k3u1fbpfcp-watermark.image?)

此问题由于and-design-vue版本过低，部分代码不支持原生ESM，有以下解决方案。

1. 升级到ant-design-vue-1.7.8稳定版。
2. 配置alias
```
resolve: {
    // 配置路径别名
    alias: [

        {find: /ant-design-vue$/,replacement: 'ant-design-vue/dist/antd.min'},
    ],
},
```
建议采用第一种，并且全量引入and-design-vue，这种方案在vue cli的webpack中也可兼容。

### 无法识别.vue文件以及含有jsx语法的.vue文件

报错如下图：
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38abf65a43c64c358881059035f27c6e~tplv-k3u1fbpfcp-watermark.image?)

添加如下配置即可解决：
```
import { createVuePlugin } from 'vite-plugin-vue2'

...
plugins: [
    createVuePlugin({
        jsx: true
    }),
]
...
resolve: {
    extensions: ['.js', '.ts', '.jsx', '.tsx', '.json', '.vue'],
},
```
在对应使用jsx语法的vue文件中，修改：
```
<script>

// 替换为

<script lang="jsx">
```

注意，为了在vue cli的webpack中也可兼容，需要安装了`@vue/babel-preset-jsx`和`@vue/babel-helper-vue-jsx-merge-props`

### 'isMoment' is not exported by 'node_modules..

此问题出现在使用ant-design-vue 的<a-date-picker> 组件时会报错，原因是 antd 底层引用 moment 是这样写的：
```
import * as moment from "moment";
```
解决方法是使用 vite 的插件 vite-plugin-antdv1-momentjs-resolver，配置如下：
```
import AntdMomentResolver from "vite-plugin-antdv1-momentjs-resolver";

plugins: [
    AntdMomentresolver()
],
```

注意如果使用的cnpm安装的ant-design-vue，则需要传入reg参数：`/_ant-design-vue@1.7.8@ant-design-vue\\[\w-\\\/]*\.js$/`

### process is not defined

在vue cli中，我们会使用process.env来获取环境变量，但是在vite中是无法识别的，我们可以通过vite替换插件，在构建时替换掉，如下配置：
```
import { replaceCodePlugin } from "vite-plugin-replace";
...
replaceCodePlugin({
    replacements: [
        {
            from: /process.env/g,
            to: "import.meta.env",
        },
    ],
}),
```
这样这种方案在vue cli的webpack中也可兼容。

### .env变量

在vue cli的webpack中，会使用到一些变量，例如：
```
VUE_APP_BUILD_ENV='test'
VUE_APP_BASE_GRAIN_API='/dg-grain-api'
VUE_APP_BASE_GANG_API='/dg-steel-api'
VUE_APP_BASE_USER_API='/dg-user-api'
```
在vite中，只能使用VITE开头的变量，为了兼容两种方案，可以采用`envPrefix`配置，如下所示：

```
defineConfig({
    envPrefix: 'VUE_APP_',
})
```

### /deep/使用[plugin:vite:css] expected selector.

报错如下图：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c82248a6cfe54f2197b96720fd8d612a~tplv-k3u1fbpfcp-watermark.image?)

在样式中使用了`<style lang="scss">`和`/deep/`深度选择器时，在vite中会报错，有以下解决方案：

1. 将`<style lang="scss">`替换为`<style lang="less">`。
2. 将`/deep/`替换为`::v-deep`。

如果项目中没有使用到less，采用后者更保险，如果替换`::v-deep`警告，可以采用：
```
:deep(.child) {
    color: red;
}
```

注意，vite默认没有`sass`解析器，需要自行安装`npm install sass -D`

### stylus和less全局变量

Inline JavaScript is not enabled. Is it set in your options?

报错如下图：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bed1fe26afb48cd9fe3939aff94598b~tplv-k3u1fbpfcp-watermark.image?)

此问题是在项目中，使用到了stylus或者less全局变量，有以下解决方案：

```
preprocessorOptions: {
    less: {
        modifyVars: {
            //在此处设置ant-design-vue主题色等等
            'primary-color': '#0053db',
            'success-color': '#52c41a',
        },
        // 支持内联 JavaScript
        javascriptEnabled: true,
    },
    stylus: {
        imports: [path.resolve(__dirname, 'src/assets/styl/fun.styl')],// 针对后缀名.stylus注入全局变量
    },
    styl: {
        imports: [path.resolve(__dirname, 'src/assets/styl/fun.styl')],// 针对后缀名.styl注入全局变量
    },
}
```


### 不支持ESM的第三方库

在项目中，或多或少都会使用到一些第三方依赖包，由于依赖依赖包没有按照 ESM 编写，导致启动或者打包时报错，这种问题是最常见的，在开发环境下可以采用`@originjs/vite-plugin-commonjs`转换，但是对于一些变态的写法，也会出现转换失败的情况，例如`videojs-contrib-hls.js`，报错如下图：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbf35e1abe354eb992d1911fee0a4839~tplv-k3u1fbpfcp-watermark.image?)

针对此问题，可以采用Externals方式，如下配置：

```
import { viteExternalsPlugin } from 'vite-plugin-externals'
...
plugins: [

    viteExternalsPlugin({
        'video.js': 'videojs',
    }),
]

```

在index.html中引入，如下所示：
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/video.js/6.0.0/video.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/videojs-contrib-hls/5.15.0/videojs-contrib-hls.js"></script>
```

## 总结

vite在dev下有明显的性能提升，体验是飞速的，但是在浏览器请求页面时，会同时加载大量js文件，这导致在速度上也有一定的牺牲，而webpack在构建上确实会慢很多，并且项目的文件越多越慢，但是一但构建好，在浏览器端体验是要比vite快的。另外vite即便是在生产环境下采用rollup对文件进行打包，但是对于IE的支持还是不好，并且生产打包的速度和webpack基本上一致，总之，如果项目已经很稳定并且很庞大，不建议换vite。

相关配置源码：https://github.com/lvming6816077/viteconfig




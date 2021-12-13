---
title: React Native IOS集成与原理简析
date: 2015-12-11 21:31:07
tags:
- React Native
categories:
- 560
photos:
- https://qiniu.nihaoshijie.com.cn/xx.png
---
自从facebook开源了react native之后，许多关于react native实现机制和原理的文章层出不穷，但是大部分主要是从native的角度去写，本文以一个web开发者ios初学者的角度去看react native在ios平台上实现的原理，如有不对之处还望指点。
<h3 style="font-weight: bolder;"><span style="font-weight: bolder;">一，整体流程</span></h3>
<!--more-->

首先，明确一点的是React Native是基于react的，react.js提供了一套组件化机制来让我们写基于dom的react组件，而React Native是基于react组件的基础上提供了一个native的react组件，可以看做js-&gt;react.js-&gt;react native-&gt;native这个过程。

然后是生成native的大致流程，先看一张大致的流程图
<div><img alt="" /><img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/oc1.png" alt="" width="529" height="678" /></div>
<div></div>
<div>1 首先，我们在写react native时，默认的入口就是index.ios.js,无论我们怎么分文件夹，项目根目录下的js文件都会被打包一份jsbundle文件，只是index.ios.js调用的是主程序的入口即AppRegistry注册的入口。</div>
<div>2 前面说到了打包，对于前端构建来说，就是利用node处理文件合并压缩等等，跟前端使用的一些打包工具例如grunt webpack不同的是，react native自己实现了一个打包方式<a style="color: #336699;" href="https://github.com/facebook/react-native/tree/master/packager" target="_blank">packger</a>，但是实现的效果都是一样的，另外网上已经有了webpack打包react native的<a style="color: #336699;" href="https://github.com/mjohnston/react-native-webpack-server" target="_blank">库</a>。</div>
<div>3 打包之后，我们的js代码包括react native的js源码都被打包压缩成了一个.jsbundle文件，我们在index.ios.js里面可以写es6的语法，这些都会在打包的时候去编译解析，生成这个jsbundle文件里面的代码是基于commonJS规范的，便于javascriptcore解析。</div>
<div>4 <a style="color: #336699;" href="http://nshipster.cn/javascriptcore/" target="_blank">javascriptcore</a>是oc的一个能够解析js的库，这个在ios7以上才能支持，不过react native对于ios7以下也有兼容，就是利用UIWebView去实现通信，这个机制可以参考jsapi的实现机制。下图是两个javascriptexcutor的封装实现。</div>
<div><img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/oc2.png" alt="" width="203" height="110" /></div>
<div>5 能够解析javascript之后，就可以按照约定渲染出相应的native 的uikit组件了。</div>
<div></div>
<h3 style="font-weight: bolder;"><span style="font-weight: bolder;">二，通信机制</span></h3>
&nbsp;
<div>那么，究竟javascript和native是怎么通信和交互的呢？</div>
<div>我们可以简单写一个native modules来举例，在oc定义一个person类，提供一个person的fetchName方法供js调用，js端使用person.fetchName()获得结果。</div>
<div>oc代码：Person.m</div>
<div>
```c
@implementation Person
RCT_EXPORT_MODULE();


RCT_EXPORT_METHOD(fetchName:(RCTResponseSenderBlock)callback) {
    
    <span class="built_in" style="font-weight: bold;">NSString</span> *name = @<span class="string" style="color: #880000;">"hello"</span>;
    
    callback(@[name]);

}
@end
```

js代码：

```javascript
var rutil = React.NativeModules.Person;
rutil.fetchName(function(name){
      console.log(name);
});
```
当我在js代码执行fetchName方法时，过程大概是这样的

<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/oc3.png" alt="" width="1205" height="703" />

1 首先native这边会主要有两个线程来处理这些操作,主线程用来负责uikit的渲染和oc逻辑方法的执行，javascript线程用来负责执行js代码。

2 图上的modulesConfig也就是模块配置表，包含了moduleID和methodID键值对的集合，这份配置表在会在程序初始化过程中，由oc遍历所有标识为接口的方法即(RCT_EXPORT)，由这些方法生成对应的moduleID和methodID并打包到modules中，形成一个config dictionary，最终转换成json格式注入(injectJSONConfiguration)到js的执行上下文中。

3 图中执行和解析js代码的是RCTJavascriptExecutor这时一个javascriptCore的实现，他是用来执行我们的js代码。

4 当js代码需要调用fetchName方法时，就可以通过配置表找到对应的oc方法，这时请求并没有立即去执行，而是放入到native call queue队列当中，当js代码执行完之后，oc这边会去遍历这个native call queue并找个它对应person.m里面定义的方法并执行，执行完成之后将执行结果和callback放入native call queue，接着js会去native call queue按照当时放进去的的id找到结果和callback，然后在通过enqueueJSCall执行js的代码callback来完成回调。

5 oc端有专门的对参数进行转换的操作，来完成oc和javascript之间的数据类型转换。

6 关于js模块配置表调用oc方法其实是利用oc的<a style="color: #336699;" href="http://www.cnblogs.com/dyingbleed/archive/2013/05/05/3029547.html" target="_blank">反射机制</a>来实现的，简单来说就是，oc提供了方法名称的字符串，在调用时，直接传一个字符串就能运行这个字符串代表的方法。

&nbsp;
<h3 style="line-height: 28.8px;"><span style="font-weight: bolder;">三，react natvie集成到ios时踩过的坑</span></h3>
1 将一个现有的ios项目添加react native支持时，有以下几个关键步骤：

1）在项目里新建一个group，然后在node_modules下面找到React和Libraries两个文件夹，将这两个文件夹下的.xcodeproj文件引入到我们创建的group中。

2）找到项目的build settings配置，在Header Search Paths下面新增一个地址，定位到node_modules/react-native/React目录下，选择递归(recursive)。

3）找到项目的build phases配置，找到Link Binary With Libraries,将步骤1里面的.a文件全部引入即可完成配置。

2 在引入reactRootView时，官网的代码里是少了一个参数，即initialProperties参数，我们要在自己的代码里添加这个参数，传nil即可。

react native内置的api毕竟还不能满足所有的需求，有时也会自己写一些接口来供js调用，下面列举一些需要注意的点：

3 由于oc是多线程(<a style="color: #336699;" href="https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/" target="_blank">GCD</a>)的,上文也说到在执行javascript时是在javascript的线程中进行的，所以在写接口时，如果需要调用非javascript线程的逻辑，就需要在主线程进行，即在业务代码前获取到主线程即：

&nbsp;
```c
dispatch_async(dispatch_get_main_queue(),^ {
        [nav pushViewController:cg animated:YES];
    });
```
&nbsp;

4 在自定义接口时，我们在oc端定义的方法时，如果有参数，那么js端一定要传这个参数，如果没有定义参数，js端也不能传参数，这可能和js以往的语法不太一样，即使定义的一个方法，我传不传都不会报错，但是在oc是不行的，必须严格按照定义方法时的格式来传递。

</div>
<h3 style="font-weight: bolder;"><span style="font-weight: bolder;">四，总结</span></h3>
&nbsp;
<div>1 react native实现的机制确实比较新颖，在设计模式和实现方法上有很多可以学习的地方。</div>
<div>2 react native在渲染过程中，利用javascriptcore去执行js代码，并渲染出native组件，这个过程毕竟还是有很大的工作量的，对性能消耗也不小，设想一下，加入js解析如果能提前做好放在内存中，渲染时直接运行native的代码或许对性能和速度都有一定提升。</div>
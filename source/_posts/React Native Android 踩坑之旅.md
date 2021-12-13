---
title: React Native Android 踩坑之旅
date: 2015-10-22 21:31:07
tags:
- React Native
- android
categories:
- 550
photos:
- https://qiniu.nihaoshijie.com.cn/xx.png
---
<span style="color: #008000;"><strong>前言</strong></span>

Facebook 在2015.9.15发布了 React Native for Android，把 JavaScript 开发技术扩展到了移动Android平台。基于React的React Native 让前端开发者使用 JavaScript 和 React 编写应用，利用相同的核心代码就可以创建 基于Web，iOS 和 Android 平台的原生应用。在React Native for Android出来之后，本人花了些时间从环境搭建到做出几个demo，从体验来看都挺流畅，具体将此间遇到和问题和具体的新的体会，向大家分享一下。
<!--more-->
&nbsp;

<span style="color: #008000;"><strong>原理简介</strong></span>

<strong>1 </strong>React Native for Android 和 for IOS的基本原理是一致的，通过android的JavaScriptCore来异步解析js代码(jsbundle文件)，然后根据引入的支持和配置，渲染成原生native组件。

<strong>2 </strong>复用React系统，也减少了一定学习和开发成本，更重要的是利用了React里面的分层和diff机制。js层传给Native层的是一个diff后的json，然后由Native将这个数据映射成真正的布局视图。

&nbsp;

&nbsp;

<span style="color: #008000;"><strong>环境搭建</strong></span>

环境搭建的话，在网上也找到过很多教程，但是还是推荐还是去看官方文档<a href="https://facebook.github.io/react-native/docs/android-setup.html#content" data-cke-saved-href="https://facebook.github.io/react-native/docs/android-setup.html#content">https://facebook.github.io/react-native/docs/android-setup.html#content</a>，在搭建的过程中可能会遇到一些问题。

&nbsp;

<strong>1 </strong>在React Native for Android刚出来的时候，官方称是不支持在windows 系统上安装的，只有在mac上才可以使用<img src="http://7jpp2v.com1.z0.glb.clouddn.com/1.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/1.png" />，但是最新版的React Native for Android已经支持在windows上使用，<img src="http://7jpp2v.com1.z0.glb.clouddn.com/2.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/2.png" />更新React Native的方法：下载最新版的react-native-cli即npm install -g react-native-cli，并且保证node是最新版的即&gt;4.0。

&nbsp;

<strong>2 </strong>在执行react-native init AwesomeProject 命令时，由于这个命令会去下载一些node module所以要根据自己的实际情况设置npm的代理和镜像，本人就曾经因为这个问题搞了很久才成功，可以安装nrm(npm install -g nrm)来便捷设置npm的代理和镜像，其次是执行这个命令必须现在机器上装有git，并且设置好git的环境变量，另外这个命令需要等待一些时间，不要提前取消。

&nbsp;

<strong>3 </strong>在调用react-native run-android的命令时，有时会报出找不到android-sdk环境变量的错误（自己确实已经正确设置环境变量）提示例如<img src="http://7jpp2v.com1.z0.glb.clouddn.com/33.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/33.png" />

的错误时，可以单独在项目根目录下，也就是AwesomeProject/新建一个local.propertites文件，添加sdk.dir=你的android的sdk目录，然后在运行react-native run-android。

&nbsp;

<strong>4 </strong>在调用react-native run-android命令时，其实这个命令就是执行的两部分操作1是构建你的android项目并生成apk，另外一个是打开react-native的package管理工具同时编译你的js文件，其实可以在项目根目录的package.json下找到

<img src="http://7jpp2v.com1.z0.glb.clouddn.com/44.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/44.png" />

其实是执行了另外一条命令node node_modules\react-native\packager\packager.js来打开package的管理工具，有些可能没打开一个新的命令行窗口，自己手动执行这条node命令也是可以的。在这条命令执行完之后，node就会开启一个服务，同时把js文件编译成jsbundle文件，我们可以通过<a href="http://localhost:8081/index.android.bundle?platform=android" data-cke-saved-href="http://localhost:8081/index.android.bundle?platform=android">http://localhost:8081/index.android.bundle?platform=android</a>来访问到这个文件，可以简单将这个文件理解成一个html，android就是通过解析这个html来达到渲染的目的，将该文件部署到CDN可供android app从网络获取，即可实现不用发版本让app的UI随时更新，并且可获得接近native的体验，这也是react-native最吸引开发者的亮点之一。

&nbsp;

<strong>5 </strong>用react native命令生成的android项目是基于gradle构建和部署的(不清楚gradle的可以google)，这个以前一些搞用eclipse来android开发的可能不太一样，gradle是用在google主推的一款android开发IDE，android statio里面默认的项目构建方式，所以我们的项目里会看到一些build.gradle的文件，这些都是配置文件。

&nbsp;

<strong>6</strong> 我们在根据教程搭环境时会碰到需要安装android模拟器的步骤，这个步骤会提示你安装一个HAXM的东西<img src="http://7jpp2v.com1.z0.glb.clouddn.com/55.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/55.png" />可以看到这个安装不是必须的，其实这个是一个android模拟器的加速程序，按了这个你的模拟器可能会跑的更快，但是在安装这个程序时，会遇到<img src="http://7jpp2v.com1.z0.glb.clouddn.com/66.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/66.png" />的错误是由于CPU的虚拟化未开启，需要重新开机在bios上设置一下，具体怎么设置，可以自行google。

&nbsp;

<strong>7 </strong>react-native android在本地调试开发时，你只需要修改js文件，然后刷新你的项目，所以在创建android模拟器时要记得选择带有android键盘的模拟器，这样才能在模拟器上刷新你的更改。

&nbsp;

<strong>与现有的android项目集成</strong>

<strong>1 </strong>想要在现有的android项目里添加react native支持，你必须要先创建一个基于gradle的android项目，推荐使用android studio来创建项目，要记得创建的项目要高于Android 4.1 (API 16)的android项目。

&nbsp;

<strong>2 </strong>用android studio创建一个项目 并且能跑起来，这段教程可以直接去网上少，一般配置无误的情况下，很容易跑出一个android helloword来，你只需要保持之前的node package服务开启，程序依然会去寻找<a href="http://localhost:8081/index.android.bundle?platform=android" data-cke-saved-href="http://localhost:8081/index.android.bundle?platform=android">http://localhost:8081/index.android.bundle?platform=android</a>这个文件的。只是你的android模拟器是通过android studio来管理了。
<strong>3 </strong>在按照<a href="http://facebook.github.io/react-native/docs/embedded-app-android.html#content" data-cke-saved-href="http://facebook.github.io/react-native/docs/embedded-app-android.html#content">http://facebook.github.io/react-native/docs/embedded-app-android.html#content</a>配置你的结合项目时，还要注意在AndroidManifest.xml文件里面添加&lt;activity android:name="com.facebook.react.devsupport.DevSettingsActivity" /&gt; 这样才能开启调试模式。

&nbsp;

<strong>4 </strong>对于android项目来说，react native的支持也是就在Activity里面创建了一个ReactRootView<img src="http://7jpp2v.com1.z0.glb.clouddn.com/77.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/77.png" />，对这不是webview，然后将Activity的其他事件生命周期等等都交给react manager来管理，所以对于react native的android页面，就可以简单理解成一个activety里面套一个reactrootview这个view去加载并jsbundle文件，渲染出原生native的ui组件。

&nbsp;

<span style="color: #008000;"><strong>远程加载jsbundle文件</strong></span>

<strong>1 </strong>目前在react android的官方文档里面，还没有找到如何远程加载jsbundle文件的地方，只能是事先把jsbundle文件放在assets目录下面，一起打包成apk，也就是release apk文件，可以参考<a href="https://facebook.github.io/react-native/docs/signed-apk-android.html" data-cke-saved-href="https://facebook.github.io/react-native/docs/signed-apk-android.html">https://facebook.github.io/react-native/docs/signed-apk-android.html</a>。

&nbsp;

<span style="color: #008000;"><strong>样式和布局</strong></span>

<strong>1 </strong>react native的代码和react基本一样，组件的生命周期，jsx语法都支持，只是在使用jsx时要经常调用官方提供的组件。

<strong>2 </strong>react native里面的样式大部分是可以利用css语法来写的，只有文档里面有的属性才能用，不是所有的css都可以在react native里面用的，采用obj的形式将css属性横杠后面的第一个字母大写即可。

<img src="http://7jpp2v.com1.z0.glb.clouddn.com/88.png" alt="" data-cke-saved-src="http://7jpp2v.com1.z0.glb.clouddn.com/88.png" />

<strong>3 </strong>react的宽高度不支持百分比，设置宽高度时不需要带单位，在react native里面默认使用pt为单位，注意在给image设置大小时要根据<a href="https://facebook.github.io/react-native/docs/pixelratio.html#content" data-cke-saved-href="https://facebook.github.io/react-native/docs/pixelratio.html#content">PixelRatio</a>设置合适的值。

<strong>4 </strong>使用dimensions.get("window")可以获取到当前viewport的大小，这个值可能会根据屏幕横竖来动态改变。

<strong>5 </strong>react native里面没有float的用法，是根据flex来布局，alignItems和justifyContent分别决定子元素的布局，而flexDirection决定子元素的排列方式垂直还是水平，flex:number决定子元素所占的比例，alignSelf决定元素本事的布局，子view会默认根据父view来absolute，这里有个技巧，如果想让子view实现100%的效果可以设置left：0 ,right :0,同理height可以用top:0,bottom:0。

<strong>6 </strong>使用text的numberOfLines可以实现文本截取省略号，即css的text-overflow属性。

<strong>7 </strong>默认情况下如果元素超过了父元素，是不可以滚动的，必须在外部套一个&lt;ScrollView&gt;才可以。

<strong>8 </strong>react native里面没有z-index的概念，是根据jsx语法里面定义组件的顺序来实现的，后写的组件会覆盖在先写的组件上。

&nbsp;

<span style="color: #008000;"><strong>总结</strong></span>

<strong>1 </strong>react native android和ios相比，由于出现的还比较晚一些功能还没有非常完善，所以一些文档里面没有写的东西还需要自己摸索。

<strong>2 </strong>react native android在性能上要比web来的好很多，毕竟渲染出来的是原生的组件，尤其是在一些低端android机型上，但是跟真正的native相比还是要逊色一些，但是react native的优势在于一套代码可以跨平台复用，而且可以通过更新远端JS，直接更新app，并且对于前端工程师来说用js的语法写native的组件也并没有很难。

<strong>3 </strong>本人用react native android做出的demo，大家可以体验一下。
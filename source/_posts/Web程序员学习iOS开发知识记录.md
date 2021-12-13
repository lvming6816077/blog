---
title: Web程序员学习iOS开发知识记录
date: 2016-03-01 19:25:22
tags:
- ios
categories:
- 578
---
<h2><span style="color: #008000;"><strong>前言</strong></span></h2>
本文将已一个web开发者的角度来记录一些日常学习ios开发中踩过的坑和web前端与ios开发的知识对比，如果有ios大神看出什么不对的地方，还请指出哈。

<h2><span style="color: #008000;">正文</span></h2>
<strong>1</strong>. ios开发要装<span style="color: #3366ff;">xcode</span>，而且没有什么其他更好的ide，这样做前端的我感觉很不习惯，每次修改完代码后都要点击运行，这比刷新浏览器慢多了。。
<!--more-->
&nbsp;

<strong>2</strong>. ios开发中，页面顶部的导航叫做navigationBar，这是ios里面自带的一种控件，不像web需要自己写div css来实现一个header栏，navigationBar的一本和navigationController配合使用，关于navigationController其实他并不是一个真正的页面，他是一个虚拟的用来管理页面跳转的组件，并不像一般的viewcontroller有这可以看到的对应的页面，跟navigationController类似的还有tabBarController

&nbsp;

<strong>3</strong>. ios开发中每一个页面叫做一个viewController相当于web里面的html页面，其中<span style="color: #3366ff;">viewController</span>里面有<span style="color: #3366ff;">viewDidLoad</span>，<span style="color: #3366ff;">viewDidAppear</span>等方法，这些方法是viewController的生命周期方法，在这些方法里面可以写页面相关的逻辑，这有点类似与web里面window的onload方法或者domcument.ready方法，viewController里有一个view属性，这是一个大的父view，可以看做html的body标签，在这个属性里面可以添加一些子自定义的view。

&nbsp;

<strong>4</strong>. ios开发中，最基本的ui元素是uiview，可以把它当作div，并且在ios中 布局都是绝对定位的absolute（和web里面的canvas布局类似），每一个ios的ui元素有一个属性叫做frame，这个属性包括x,y,width,heigt这四个值就可以确定一个元素在界面中的位置。

&nbsp;

<strong>5</strong>.ios开发中，使用oc进行变成，可以将一些需要的依赖通过import的方式引入，import ""双引号一般表示引入自己写的一些文件，import &lt;&gt;尖括号标识引入系统的库例如uikit啥的，这个和nodejs里的require有些类似，但是又有一些区分，nodejs中require引入的依赖文件必须已module.export结尾，oc里面的import不必需要。

&nbsp;

<strong>6</strong>.关于ios中的<span style="color: #3366ff;">navigationbar</span>，下面这张图表示的比较清晰<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/navigationbar.png" alt="" width="640" height="174" />

&nbsp;

<strong>7</strong>. ios开发中，可以利用oc的宏定义一些全局的变量或者全局的方法，所谓宏就是一种特殊的语法，只要引入后，在oc的代码里都可以使用，可以理解成在window下挂一些变量和方法，使用起来比较方便，另外，由于定于宏的代码都放在一个 统一的文件里面，所以oc端想使用这个宏，必须引入他例如import "Define.h",这个跟js相比还是比较麻烦的，不过也有解决办法，在xcode里面可以一个projectName-<span style="color: #222426;">Prefix.pch以pch结尾的文件。</span>

&nbsp;

<span style="color: #222426;"><strong>8</strong>.pch全称是“<span style="color: #3366ff;">precompiled header</span>”，也就是预编译头文件，该文件里存放的工程中一些不常被修改的代码，比如常用的框架头文件，这样做的目的提高编译器编译速度。<span style="color: #000000;">假如pch中某个文件修改了，那么pch整个文件里包含的的其他文件也会重新编译一次，这样就会消耗大量时间，所以它里面添加的文件最好是是很少变动或不变动的头文件或者是预编译的代码片段：</span></span>
```c
#ifndef __IPHONE_4_0  
#warning "This project uses features only available in iOS SDK 4.0 and later."  
#endif  
  
#ifdef __OBJC__  
    #import &lt;UIKit/UIKit.h&gt;  
    #import &lt;Foundation/Foundation.h&gt;  
#endif
```
<strong>9</strong>. ios里面如果想显示一张图片并不像web那样简单，在web里，我们只用给img标签配置一个src即可，但是在ios客户端开发中，图片其实是一个文件，包括本地和网络图片，如果要想显示，我自己写逻辑将图片下载下来，然后将结果传给UIImage，但是这只是一个图片对象，还要将UIImage配置给UIImageView才能真正看到图片，当然ios已经有许多开源的图片下载库来简化我们这些操作例如AFNetworking等。

&nbsp;

<strong>10</strong>. ios里关于图片有多个展示的类型，也就是说图片较大怎么居中截取，图片较小怎么拉伸都有相关的配置，这一点比web要丰富一些，web要想实现这些必须自己写逻辑代码实现，例如UIImageview的<a href="http://www.jianshu.com/p/338af5eb5c60" target="_blank">contentmode</a>属性

其中主要分为:<span style="color: #3366ff;">ScaleToFill</span>,<span style="color: #3366ff;">ScaleAspectFit</span>,<span style="color: #3366ff;">ScaleAspectFill</span>三种。

ScaleToFill为将图片按照整个区域进行拉伸(会破坏图片的比例)
ScaleAspectFit将图片等比例拉伸，可能不会填充满整个区域
ScaleAspectFill将图片等比例拉伸，会填充整个区域，但是会有一部分过大而超出整个区域。
至于Top,Left,Right等等就是将图片在view中的位置进行调整。

&nbsp;

<strong>11</strong>. ios里实现页面跳转主要分为模态modal和入栈态push，具体思路就是找到将要<span style="color: #3366ff;">跳转</span>的页面的viewcontroller，new出来之后，调用

<strong><span class="s1">- (void)pushViewController:(<a style="color: #000000;" href="file:///Applications/Xcode.app/Contents/Developer/Documentation/DocSets/com.apple.adc.documentation.AppleiOS8.1.iOSLibrary.docset/Contents/Resources/Documents/documentation/UIKit/Reference/UIViewController_Class/index.html#//apple_ref/doc/c_ref/UIViewController"><span class="s2">UIViewController</span></a> *)viewController animated:(BOOL)animated</span></strong>

<strong><span class="s1">- (void)presentViewController:(UIViewController *)viewControllerToPresent animated:(BOOL)flag completion:(void (^)(void))completion</span></strong>

来实现页面跳转，前者一般配合navigationController使用，后者则在一般的viewcongroller里面使用即可。

&nbsp;

<strong>12</strong>. ios里面可以将一个view隐藏，调用view.hidden = true即可，但是跟web不同的是，隐藏的view同样占用这位置，同时，挨着被隐藏的view的兄弟view也并不会顶上来（占用隐藏view的位置）这一点和ios的绝对定位有关，所以如果用web的思想去隐藏和显示一个view在ios上会很麻烦，这里笔着曾经一度不能接受这样的转变，不能适应过来。。

&nbsp;

<strong>13</strong>. ios里也有一个类似于web相对定位的东西叫做<a href="http://blog.csdn.net/sxfcct/article/details/8776928" target="_blank">autolayout</a>，这中开发方式完全抛弃了使用frame去定义一个view位置的方式，而且这个方式配合xib（<span style="color: #000000;">Interface Builder</span>），来构建页面的布局跟web很相似，所见即所得，我们在xcode里面直接将我们需要的组件拖入进来即可，同时可以定义view直接的约束，类似与web的margin padding 之类的限制条件，就可以很方便的布局和适配，具体可以参考<a href="http://www.cnblogs.com/WhoJun/p/3498408.html?utm_source=tuicool&amp;utm_medium=referral" target="_blank">这个链接</a>。

<strong>14</strong>.<img class="alignnone" src="http://7jpp2v.com1.z0.glb.clouddn.com/NavigationViews_2x.png" alt="" width="977" height="986" />

上面这个图完美的表现了 底部tabbar，顶部navigationbar，viewcontroller，view的层次图，所以对于ios来说，很多组件系统都是提供的，并不像web很多组件要自己用div和css来实现。

&nbsp;

<strong>15</strong>. 在ios中，如果使用的TabBarController（一种底部bar的viewController）这是一种特殊的controller，通过配置TabBarController的viewControllers属性就可以往这个TabBarController里面添加需要展示的页面，TabBarController会自己控制当点击哪一个tab时显示哪个页面，这点比web要智能很多，web要自己实现这种控件，而ios里面都已经实现了了。

&nbsp;

<strong>16</strong>. 在含有TabBarController的页面中，当从这个页面跳转出去时，<span style="color: #3366ff;">必须手动把 底部tabba给隐藏掉</span>，这点一开始也是很不理解，不过理解下来，其实用了TabBarController时，整个页面都在这个页面里的那个区域内跳转，所以底部的bar是常驻的，因此要隐藏底部。

&nbsp;

<strong>17.</strong> 关于ios里面的<span style="color: #3366ff;">重写</span>和<span style="color: #3366ff;">重载</span>，重写就是子类继承父类时，子类写一个方法和父类的方法名，参数，返回一样这样就是重写，重载就是在子类写两个相同的方法叫做重载，在viewController里会经常看到，viewDidLoad，viewDidAppear这类方法中经常看到[super viewDidLoad]这样调用父类的方法，原因是调用子类的方法时候 如果子类没有这个方法就会去父类找 所以如果不重写父类方法，不会影响到父类的调用，但是如果重写了方法，就不会去父类找 这个时候如果不再子类里面调用一下[super viewDidLoad]就会调用不到父类的默认方法了。

&nbsp;

<strong>18.</strong>viewConfroller的<span style="color: #3366ff;">生命周期</span>
<p style="color: #555555;"><strong>init</strong>－初始化程序</p>
<p style="color: #555555;"><strong>loadView- </strong>加载视图的view（通常这一步不需要去干涉）</p>
<p style="color: #555555;"><strong>viewDidLoad</strong>－加载视图（各种初始数据的载入，初始设定等很多内容，都会在这个方法中实现）</p>
<p style="color: #555555;"><strong>viewWillAppear</strong>－UIViewController对象的视图即将加入窗口时调用；（当APP有多个视图时，在视图间切换时，并不会再次载入viewDidLoad方法，所以如果在调入视图时，需要对数据做更新，就只能在这个方法内实现了）</p>
<p style="color: #555555;"><strong>viewDidApper</strong>－UIViewController对象的视图已经加入到窗口时调用；</p>
<p style="color: #555555;"><strong>viewWillDisappear</strong>－UIViewController对象的视图即将消失、被覆盖或是隐藏时调用；</p>
<p style="color: #555555;"><strong>viewDidDisappear</strong>－UIViewController对象的视图已经消失、被覆盖或是隐藏时调用；</p>
<p style="color: #555555;"><strong>viewVillUnload</strong>－当内存过低时，需要释放一些不需要使用的视图时，即将释放时调用；</p>
<p style="color: #555555;"><strong>viewDidUnload</strong>－当内存过低，释放一些不需要的视图时调用。</p>
<p style="color: #555555;"><strong>19.</strong>关于ios里的<span style="color: #222222;"><span style="color: #3366ff;">edgesForExtendedLayout</span>，<span style="color: #3366ff;">automaticallyAdjustsScrollViewInsets</span>，<span style="color: #3366ff;">translucent</span>，和navigationBar之间的关系其中：</span></p>
<p style="color: #555555;"><span style="color: #222222;">1 <span style="color: #3366ff;">translucent是navigationBar的属性优先级最高</span></span></p>
<p style="color: #555555;"><span style="color: #222222;">当translucent=true（默认）时，即在含有navigationBar的页面中，frame的0是从页面的statusBar开始算的。</span></p>
<p style="color: #555555;">当<span style="color: #222222;">translucent=false时，即在含有navigationBar的页面中，frame的0是从navigationBar下面开始算的。</span></p>
<p style="color: #555555;">2 <span style="color: #3366ff;">edgesForExtendedLayout是viewController的属性优先级其次</span></p>
<p style="color: #555555;">当<span style="color: #222222;">edgesForExtendedLayout=RectEdgeNone时，即在含有navigationBar的页面中，frame的0是从navigationBar下面开始算的。</span></p>
<p style="color: #555555;">当<span style="color: #222222;">edgesForExtendedLayout=RectEdgeAll（默认）时，即在含有navigationBar的页面中，frame的0是从statusBar开始算的。</span></p>
<p style="color: #555555;">3 <span style="color: #3366ff;">automaticallyAdjustsScrollViewInsets是viewController的属性，只对UIScrollView，UITableView有效</span></p>
<p style="color: #555555;">当automaticallyAdjustsScrollViewInsets<span style="color: #222222;">=true时，即在含有navigationBar的页面中，UIScrollView是从navigationBar底部开始滚动的，frame的0根据前两个属性影响。</span></p>
<p style="color: #555555;">当automaticallyAdjustsScrollViewInsets<span style="color: #222222;">=false时，即在含有navigationBar的页面中，UIScrollView是从本身的位置开始滚动的，frame的0根据前两个属性影响。</span></p>
<p style="color: #555555;">再设置<span style="color: #222222;">edgesForExtendedLayout和translucent时会影响viewController的view的高度，在viewDidLoad和viewWillAppear会有所体现。</span></p>
<p style="color: #555555;"></p>
<p style="color: #555555;"><span style="font-weight: bolder; line-height: 28.7999992370605px;">20.</span><span>关于<span style="color: #008080;">alloc init</span> 和 <span style="color: #008080;">new</span> 的区别，在oc中，创建一个对象经常是alloc init调用两个方法，其实oc也支持用new来创建对象的，这两个方法其实没什么区别，使用alloc init思路更清晰一些，先分配内存，然后返回实例对象，可以参考<a style="color: #336699;" href="http://stackoverflow.com/questions/719877/use-of-alloc-init-instead-of-new" target="_blank">stackoverflow</a>。</span></p>
<p style="color: #555555;"></p>
<p style="color: #555555;"><span style="font-weight: bold;">21.</span>appDelegate生命周期方法</p>

```c
//当应用程序将要进入非活动状态执行，在此期间，应用程序不接受消息或事件，比如来电  
- (void)applicationWillResignActive:(UIApplication *)application  
{  
    NSLog(@"应用程序将要进入非活动状态，即将进入后台");  
}  
  
//应用程序已经进入后台运行  
- (void)applicationDidEnterBackground:(UIApplication *)application  
{  
    NSLog(@"如果应用程序支持后台运行，则应用程序已经进入后台运行");  
}  
  
//应用程序将要进入活动状态执行  
- (void)applicationWillEnterForeground:(UIApplication *)application  
{  
    NSLog(@"应用程序将要进入活动状态，即将进入前台运行");  
}  
  
//应用程序已经进入活动状态  
- (void)applicationDidBecomeActive:(UIApplication *)application  
{  
    NSLog(@"应用程序已进入前台，处于活动状态");  
}  
  
//应用程序将要退出，通常用于保存书架喝一些推出前的清理工作，  
- (void)applicationWillTerminate:(UIApplication *)application  
{  
    NSLog(@"应用程序将要退出，通常用于保存书架喝一些推出前的清理工作");  
}  
  
//当设备为应用程序分配了太多的内存，操作系统会终止应用程序的运行，在终止前会执行这个方法  
//通常可以在这里进行内存清理工作，防止程序被终止  
-(void)applicationDidReceiveMemoryWarning:(UIApplication *)application  
{  
    NSLog(@"系统内存不足，需要进行清理工作");  
}  
  
//当系统时间发生改变时执行  
-(void)applicationSignificantTimeChange:(UIApplication *)application  
{  
    NSLog(@"当系统时间发生改变时执行");  
}  
  
//当程序载入后执行  
-(void)applicationDidFinishLaunching:(UIApplication *)application  
{  
    NSLog(@"当程序载入后执行");  
}
```
&nbsp;
<p style="color: #555555;">附上最近做的一个ios的demo：<a href="https://github.com/lvming6816077/wodu" target="_blank">https://github.com/lvming6816077/wodu</a></p>
http://www.cocoachina.com/bbs/read.php?tid=280826

未完。。
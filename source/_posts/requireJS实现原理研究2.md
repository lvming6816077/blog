---
title: requireJS实现原理研究2
date: 2014-08-12 18:31:29
tags:
- requirejs
categories:
- 390
---
<em><strong>上文说到了main.js被引入进来，然后我们接着往下看。</strong></em>
<h3>接着写</h3>
main.js被引入进来后，当然需要运行了，看一下main.js
<!--more-->
```javascript
require.config({
　　　　paths: {
　　　　　　"a": "a",
　　　　　　"b": "b"
　　　　}
});
require(['a'], function (a){
        console.log('main');
        //console.log(a);
});
```
第一行是关于config的配置，于是去源码里寻找config方法，发现：
```javascript
req.config = function (config) {
    return req(config);
};
```
原来还是调用的require方法，继续看debug的require方法，看这段代码：
```javascript
// Determine if have config object in the call.
if (!isArray(deps) &amp;&amp; typeof deps !== 'string') {
    // deps is a config object
    config = deps;
    if (isArray(callback)) {
        // Adjust args if there are dependencies
        deps = callback;
        callback = errback;
        errback = optional;
    } else {
        deps = [];
    }
}
```
由于我们传入的deps不是数组而是一个对象，所以进过这段代码后，config被全局变量config保存了起来，deps又被设置成空，这又回到了以前的那一布，由于我们传入的defs是空，这个function里的大多数逻辑都没有走，所以这个方法就结束了，程序会继续往下运行。

这时程序又回到main.js里面了，执行完config方法后，又去执行require方法，这时由于传入的deps和callback都不为空，这样，真正的调用require方法了！我们接着debug。
```javascript
	console.log(deps, callback)；
	["a"] function (a){
	        console.log('main');
	        //console.log(a);
	}
```
经过require方法，最后就会到loadRequire方法，根据上面main.js引入的原理，由于我们的deps里面有a，所以也会依次经过nextTick方法，module的init方法，enable方法，fetch方法，最后经过load方法来吧a.js引入进来，由于我们是真正调用的require方法，callback已经被传入进去，所以，这个main.js调用的require方法就为自己生产了一个含有callback的module，这个callback被存储在factory（之前都不不含有callback的也可以说没有factory），通过代码：
```javascript
function getModule(depMap) {
    var id = depMap.id,
        mod = getOwn(registry, id);

    if (!mod) {
        mod = registry[id] = new context.Module(depMap);
    }
    return mod;
}
```
打印出这么main.js的module：
```javascript
Module {events: Object, map: Object, shim: false, depExports: Array[0], depMaps: Array[0]…}
defineEmitComplete: true
defineEmitted: true
defined: true
defining: false
depCount: 0
depExports: Array[1]
0: Object
text: 1
__proto__: Object
length: 1
__proto__: Array[0]
depMaps: Array[1]
depMatched: Array[1]
enabled: true
enabling: false
errback: undefined
events: Object
exports: undefined
factory: function (a){
arguments: null
caller: null
length: 1
name: ""
prototype: Object
__proto__: function Empty() {}
function scope
ignore: undefined
inited: true
map: Object
id: "_@r6"
isDefine: false
name: "_@r6"
originalName: null
parentMap: undefined
prefix: undefined
unnormalized: false
url: "./_@r6.js"
__proto__: Object
pluginMaps: Object
shim: false
skipMap: undefined
__proto__: Object
```
这个时候我们知道a,js已经被引入进来了，然后便会执行a.js里面的代码，和main.js不同的是a用的define方法来加载模块，那么这和用require方法有什么不同呢？看一下define方法：
```javascript
define = function (name, deps, callback) {
    var node, context;

    //Allow for anonymous modules
    if (typeof name !== 'string') {
        //Adjust args appropriately
        callback = deps;
        deps = name;
        name = null;
    }

    //This module may not have dependencies
    if (!isArray(deps)) {
        callback = deps;
        deps = null;
    }

    if (!deps &amp;&amp; isFunction(callback)) {
        deps = [];
       
        if (callback.length) {
            callback
                .toString()
                .replace(commentRegExp, '')
                .replace(cjsRequireRegExp, function (match, dep) {
                    deps.push(dep);
                });

            
            deps = (callback.length === 1 ? ['require'] : ['require', 'exports', 'module']).concat(deps);
        }
    }

    
    if (useInteractive) {
        node = currentlyAddingScript || getInteractiveScript();
        if (node) {
            if (!name) {
                name = node.getAttribute('data-requiremodule');
            }
            context = contexts[node.getAttribute('data-requirecontext')];
        }
    }

   
    (context ? context.defQueue : globalDefQueue).push([name, deps, callback]);
};
```
OK，又是这么多代码，但是我看到了最后一步比较重要的(context ? context.defQueue : globalDefQueue).push([name, deps, callback]);这段代码往 context.defQueue塞了一个对象，还记得之前提到过的intakeDefines方法么，当程序运行到这之后，script的onload事件就会触发，然后就会走到intakeDefines方法里面，在intakeDefines方法里面便会执行callGetModule方法，得到a.js对应的module，然后，a对应的module也打印出来看：
```javascript
Module {events: Object, map: Object, shim: false, depExports: Array[0], depMaps: Array[0]…}
defineEmitComplete: true
defineEmitted: true
defined: true
defining: false
depCount: 0
depExports: Array[1]
0: undefined
length: 1
__proto__: Array[0]
depMaps: Array[1]
0: Object
id: "b"
isDefine: true
name: "b"
originalName: "b"
parentMap: Object
prefix: undefined
unnormalized: false
url: "./b.js"
__proto__: Object
length: 1
__proto__: Array[0]
depMatched: Array[1]
enabled: true
enabling: false
errback: undefined
events: Object
exports: Object
factory: function (b){
fetched: true
ignore: undefined
inited: true
map: Object
id: "a"
isDefine: true
name: "a"
originalName: "a"
parentMap: undefined
prefix: undefined
unnormalized: false
url: "./a.js"
__proto__: Object
pluginMaps: Object
shim: false
__proto__: Object
```
map的id证明了这个module是属于a的，depMaps证明这个a依赖b，这个module对像里depExports存的就是这个a依赖b的callback返回的结果，factory存的就是自己的callback，<span style="color: #303942;">那么这个b的callback是在什么时候运行的，又是在什么时候塞到depExports里呢？于是在源码里找到：</span>
```javascript
exports = context.execCb(id, factory, depExports, exports);
```
执行的就是：
```javascript
execCb: function (name, callback, args, exports) {
    return callback.apply(exports, args);
},
```
最后，终于找到callback被执行的地方了，随着callback被执行把得到的就过塞入到module，我们的module也基本完成了，但是这还不算完，虽然module里面的东西都有了，但是还并没有应用到我们自己写的代码里，也就是我们自身的callback还没有执行，于是发现了这段代码：
```javascript
on: function (name, cb) {
    var cbs = this.events[name];
    if (!cbs) {
        cbs = this.events[name] = [];
    }

    cbs.push(cb);
},

emit: function (name, evt) {
    //console.log(this.events[name]);
    
    each(this.events[name], function (cb) {
        console.log(cb);
        cb(evt);
    });
    if (name === 'error') {
        //Now that the error handler was triggered, remove
        //the listeners, since this broken Module instance
        //can stay around for a while in the registry.
        delete this.events[name];
    }
}
```
这个on和emit一个是注册，一个是触发，我们的module在拿到export后便会传入emit方法来出发，从而达到调用我们自身回调的过程，这里的cb其实是一个bind方法，依然是一个闭包。写到这里，requireJS的基本流程差不多完了。
<h3>结束语</h3>
整个requireJS研究下来，发现还是有点难度的，所以本文只是讲的一个基本的流程，有什么不对的地方，还请多多指出。

&nbsp;
---
title: requireJS实现原理研究1
date: 2014-08-12 18:31:29
tags:
- requirejs
categories:
- 381
---
众所周知，Javascript有一个很棒的模块化库requireJS，这个基于AMD规范的js库受到越来越多的程序员喜爱，那么，下面就来谈谈我对requireJS的研究和理解。

<!--more-->

<h3>1. 简单流程概括：</h3>
<ol>
	<li>我们在使用requireJS时，都会把所有的js交给requireJS来管理，也就是我们的页面上只引入一个require.js，把data-main指向我们的main.js。</li>
	<li>通过我们在main.js里面定义的require方法或者define方法，requireJS会把这些依赖和回调方法都用一个数据结构保存起来。</li>
	<li>当页面加载时，requireJS会根据这些依赖预先把需要的js通过document.createElement的方法引入到dom中，这样，被引入dom中的script便会运行。</li>
	<li>由于我们依赖的js也是要按照requireJS的规范来写的，所以他们也会有define或者require方法，同样类似第二步这样循环向上查找依赖，同样会把他们村起来。</li>
	<li>当我们的js里需要用到依赖所返回的结果时(通常是一个key value类型的object),requireJS便会把之前那个保存回调方法的数据结构里面的方法拿出来并且运行，然后把结果给需要依赖的方法。</li>
	<li>以上就是一个简单的流程。</li>
</ol>
&nbsp;
<h3>2. 测试代码</h3>
下面我把一个requireJS小Demo写出来，是下面研究源码的基础：

main.js
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
a.js
```javascript
define(['b'], function(b){
    console.log('a');
    return {
        'text' : 1
    }
})
```
b.js
```javascript
define(function(){
    console.log('b');
    //return 1;
})
```
这些代码都很简单，下面开始正式的研究。

&nbsp;
<h3>3. 开始研究</h3>
<ol>
	<li> <em><strong>关于requireJS预加载</strong></em>。也就是说，我每个模块所依赖的其他模块都会比本模块预先加载，这点可以直接运行测试代码来证明。
```javascript
b b.js:2
a a.js:2
main main.js：8
```
这是控制台打印出的信息，可以看到，加载的顺序确实如上面所说。</li>
	<li><em><strong>requireJS的上下文对象context。</strong></em>翻开requireJS的代码，看到通篇的function定义和其他变量的声明，这些暂时都还不重要，我们只用关心两行代码。
```javascript
//Create default context.
    req({});
```
根据注释可以知道，这段代码初始化了一个上下文对象context，调用的是req方法
```javascript
req = requirejs = function (deps, callback, errback, optional) {

        //Find the right context, use default
        var context, config,
            contextName = defContextName;

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

        if (config &amp;&amp; config.context) {
            contextName = config.context;
        }

        context = getOwn(contexts, contextName);
        if (!context) {
            context = contexts[contextName] = req.s.newContext(contextName);
        }

        if (config) {
            context.configure(config);
        }
        var fg = context.require(deps, callback, errback);
        return fg;
    };
```
这也方法也就是常用的require方法，可以看到这个context只会初始化一次，打印出来就是
```javascript
Module: function (map) {
completeLoad: function (moduleName) {
config: Object
configure: function (cfg) {
contextName: "_"
defQueue: Array[0]
defined: Object
enable: function (depMap) {
execCb: function (name, callback, args, exports) {
load: function (id, url) {
makeModuleMap: function makeModuleMap(name, parentModuleMap, isNormalized, applyMap) {
makeRequire: function (relMap, options) {
makeShimExports: function (value) {
nameToUrl: function (moduleName, ext, skipExt) {
nextTick: function (fn) {
onError: function onError(err, errback) {
onScriptError: function (evt) {
onScriptLoad: function (evt) {
registry: Object
require: function localRequire(deps, callback, errback) {
urlFetched: Object
__proto__: Object
```
当然，这里面有很多东西，但是根据命名，不难理解，这里很多东西都是以后要用到的，例如defQueue，makeRequire等等，不用着急，这些都会在后面说道的。这个方法return了一个fg我是自己调试用的，这个fg是一个function（闭包），程序运行到这里由于我们传入的defs是空，这个function里的大多数逻辑都没有走，所以这个方法就结束了，程序会继续往下运行。</li>
	<li><em><strong>requireJS的引入script。</strong></em>上一步我们得到的第一次初始化的context对象，看上去里面什么还没有，我们继续debug下，程序走到了这里：
```javascript
if (isBrowser) {
    head = s.head = document.getElementsByTagName('head')[0];
    //If BASE tag is in play, using appendChild is a problem for IE6.
    //When that browser dies, this can be removed. Details in this jQuery bug:
    //http://dev.jquery.com/ticket/2709
    baseElement = document.getElementsByTagName('base')[0];
    if (baseElement) {
        head = s.head = baseElement.parentNode;
    }
}
```
这段代码不难理解，跟我上面说道的流程一样，开始寻找html里的head标签了，当然是为了引入script了！然后程序走到了这里：
```javascript
if (isBrowser &amp;&amp; !cfg.skipDataMain) {

	eachReverse(scripts(), function (script) {

	    if (!head) {
	        head = script.parentNode;
	    }

	    dataMain = script.getAttribute('data-main');
	    if (dataMain) {

	        mainScript = dataMain;

	        if (!cfg.baseUrl) {

	            src = mainScript.split('/');
	            mainScript = src.pop();
	            subPath = src.length ? src.join('/')  + '/' : './';

	            cfg.baseUrl = subPath;
	        }

	        mainScript = mainScript.replace(jsSuffixRegExp, '');

	        if (req.jsExtRegExp.test(mainScript)) {
	            mainScript = dataMain;
	        }

	        cfg.deps = cfg.deps ? cfg.deps.concat(mainScript) : [mainScript];

	        return true;
	    }
	});
}
```
这段代码的主要功能就是找到我们之前绑定的data-main的，然后往全局的cfg对象里添加base路径和main，当然如果我们自己通过require的config设置打印出cfg：
```javascript
Object {baseUrl: "./", deps: Array[1]}
baseUrl: "./"
deps: Array[1]
0: "main"
length: 1
__proto__: Array[0]
__proto__: Object
```
这个cfg马上就会用到了。</li>
	<li><em><strong>第二次执行req方法。</strong></em>这时候程序来到了整个文件的倒数第二行
```javascript
//Set up with config info.
    req(cfg);
```
这个时候调用req方法，把刚才的cfg传了进去，然后便又回到了require方法里面，到此为止，html页面上还只有一个script标签和一个require.js，我们的main a b .js都还没有运行，里面的代码都还没有起作用。
再次进入req方法，这时deps不再是｛｝，而是上面cfg，里面有一个main，这是在回头看看刚才return的fg，打印出来：
```javascript
function localRequire(deps, callback, errback) {
    var id, map, requireMod;

    if (options.enableBuildCallback &amp;&amp; callback &amp;&amp; isFunction(callback)) {
        callback.__requireJsBuild = true;
    }

    if (typeof deps === 'string') {
        if (isFunction(callback)) {
            //Invalid call
            return onError(makeError('requireargs', 'Invalid require call'), errback);
        }

        if (relMap &amp;&amp; hasProp(handlers, deps)) {
            return handlers[deps](registry[relMap.id]);
        }


        if (req.get) {
            return req.get(context, deps, relMap, localRequire);
        }

        //Normalize module name, if it contains . or ..
        map = makeModuleMap(deps, relMap, false, true);
        id = map.id;

        if (!hasProp(defined, id)) {
            return onError(makeError('notloaded', 'Module name "' +
                        id +
                        '" has not been loaded yet for context: ' +
                        contextName +
                        (relMap ? '' : '. Use require([])')));
        }
        return defined[id];
    }

    intakeDefines();

    context.nextTick(function () {

        intakeDefines();

        requireMod = getModule(makeModuleMap(null, relMap));

        requireMod.skipMap = options.skipMap;
        requireMod.init(deps, callback, errback, {
            enabled: true
        });

        checkLoaded();
    });

    return localRequire;
}
```
原来程序走到了这里，于是我们继续debug。由于我们传入的defs不为空，所以这次和第一次执行req方法大不一样了，一路走下来，我们发现context.nextTick这个方法，很奇怪的是，找到nextTick定义的地方
```javascript
req.nextTick = typeof setTimeout !== 'undefined' ? function (fn) {
        setTimeout(fn, 4);
    } : function (fn) { fn(); };
```
没错，这里用到了setTimeout来延迟执行一个方法，这是为什么呢？还是4ms，至今没有搞明白！nextTick方法：
```javascript
context.nextTick(function () {
        //Some defines could have been added since the
        //require call, collect them.
        intakeDefines();

        requireMod = getModule(makeModuleMap(null, relMap));

        //Store if map config should be applied to this require
        //call for dependencies.
        requireMod.skipMap = options.skipMap;
        requireMod.init(deps, callback, errback, {
             enabled: true
        });

        checkLoaded();
});
```
这里说一下getModule方法，这个方法返回context里面的Module对象，这个对象是唯一标识的，也就说每个模块对应一个module，module里面存储这当前模块所依赖的模块和当前模块运行的结果。
```javascript
Module = function (map) {
    this.events = getOwn(undefEvents, map.id) || {};
    this.map = map;
    this.shim = getOwn(config.shim, map.id);
    this.depExports = [];
    this.depMaps = [];
    this.depMatched = [];
    this.pluginMaps = {};
    this.depCount = 0;

    /* this.exports this.factory
       this.depMaps = [],
       this.enabled, this.fetched
    */
};
```
这个方法里的intakeDefines方法，可以理解为对上面context里面defQueue的初始化，通过getModule方法，最终会执行Module的init方法，这个defQueue数组里面存的是全局当前的依赖。由于此时defQueue还为空，所以不会初始化。
然后程序接着往下运行，由于我们的deps里面有main，所以我们得到了一个新的Module，是关于main的requireMod，然后执行init方法。</li>
	<li><em><strong>开始引入main.js。</strong></em>进入init（）方法：
```javascript
init: function (depMaps, factory, errback, options) {
	    options = options || {};

	    if (this.inited) {
	        return;
	    }

	    this.factory = factory;

	    if (errback) {
	 this.on('error', errback);
	    } else if (this.events.error) {

	        errback = bind(this, function (err) {
	            this.emit('error', err);
	        });
	    }

	    this.depMaps = depMaps &amp;&amp; depMaps.slice(0);

	    this.errback = errback;


	    this.inited = true;

	    this.ignore = options.ignore;


	    if (options.enabled || this.enabled) {
	        //Enable this module and dependencies.
	        //Will call this.check()

	        this.enable();
	    } else {
	        this.check();
	    }
	},
```
这段代码这么长，一开始我也看不懂的，但是我们可以抽取要点，看最后的this.enable()由于执行了这个方法，再往下debug，中间经过了check（）方法，fetch（）方法，load（）方法，然后进入到了req.load（），在这里感叹一下requireJS确实比较复杂，中间的每个方法都对一些全局变量有修改或者设置，在这里不细致描述，继续我们主要的流程。在req.load（）
```javascript
req.load = function (context, moduleName, url) {
    var config = (context &amp;&amp; context.config) || {},
        node;
    if (isBrowser) {

        node = req.createNode(config, moduleName, url);

        node.setAttribute('data-requirecontext', context.contextName);
        node.setAttribute('data-requiremodule', moduleName);


        if (node.attachEvent &amp;&amp;

                !(node.attachEvent.toString &amp;&amp; node.attachEvent.toString().indexOf('[native code') &lt; 0) &amp;&amp;
                !isOpera) {

            useInteractive = true;

            node.attachEvent('onreadystatechange', context.onScriptLoad);

        } else {
            node.addEventListener('load', context.onScriptLoad, false);
            node.addEventListener('error', context.onScriptError, false);
        }
        node.src = url;

        currentlyAddingScript = node;
        if (baseElement) {
            head.insertBefore(node, baseElement);
        } else {
            head.appendChild(node);
        }
        currentlyAddingScript = null;

        return node;
    } else if (isWebWorker) {
        try {

            importScripts(url);

            context.completeLoad(moduleName);
        } catch (e) {
            context.onError(makeError('importscripts',
                            'importScripts failed for ' +
                                moduleName + ' at ' + url,
                            e,
                            [moduleName]));
        }
    }
};
```
终于，我们看到了证据，requireJS开始往dom里面插script了！打印出参数context, moduleName, url即
```javascript
context：

Module: function (map) {
completeLoad: function (moduleName) {
config: Object
configure: function (cfg) {
contextName: "_"
defQueue: Array[0]
defined: Object
enable: function (depMap) {
execCb: function (name, callback, args, exports) {
load: function (id, url) {
makeModuleMap: function makeModuleMap(name, parentModuleMap, isNormalized, applyMap) {
makeRequire: function (relMap, options) {
makeShimExports: function (value) {
nameToUrl: function (moduleName, ext, skipExt) {
nextTick: function (fn) {
onError: function onError(err, errback) {
onScriptError: function (evt) {
onScriptLoad: function (evt) {
registry: Object
require: function localRequire(deps, callback, errback) {
startTime: 1407753904901
urlFetched: Object


moduleName：
main

url：
./main.js
```
然后，我们的main.js就引入进来了！
```html
<script type="text/javascript" charset="utf-8" async="" data-requirecontext="_" data-requiremodule="main" src="./main.js"></script>
```
当然，localRequire方法最后还有一个checkLoaded();方法，顾名思义就是用来检测是否引入成功，里面还有一个exprier时间，超出则报错。</li>
	<li>OK，这篇文章先写到这里。</li>
</ol>
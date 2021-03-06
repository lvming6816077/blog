---
title: Angular的依赖注入
date: 2014-07-31 17:26:17
tags:
- Angular
categories:
- 341
---
在众多的javascript框架中，多多少少都会使用一些思想，设计模式等等，下面就来说说其中的某些：
<h3>依赖注入：</h3>
什么是依赖注入呢，我的理解，简单点就是说我的东西我自己并不像来拿着，我想要我依赖的那个人来帮我拿着，当我需要的时候，他给我就行了。当然这只是简单的理解，还是用代码解释比较清楚一些。
<!--more-->
这里有一个function，很简单。
```javascript
var a = function(name){
console.log(name);
}```

我们调用它：
```javascript
a('abc')；//abc
```
那么，就像我上面说的，我能不能自己不传参数呢，例如：
```javascript
a();//undefined
```
如何才能实现让别人帮我们注入这个参数呢：
```javascript
var inject = function(name,callback){
  return function(){
     callback(name);
  }
}```
想这样，我们在定义参数的时候这样传：
```javascript
a = inject('abc',a)```
我们再调用a方法：
```javascript
a()；//abc```
这其实就是最简单的依赖注入了，当然这么简单是不行的，其实这是很无意义的，下面我们来看一下高深的angularjs：
```javascript
var MyController = function($scope){
        $scope.test = 1;
}```
上面这段代码定义了angularjs的controller里面用到了scope，这样还看不出问题，在看下面：
```javascript
var MyController = function($scope,$http){
        $scope.test = 1;
        $http.get('');
}```
上面这段代码在原来的基础上增加了http,那么问题就来了，angular在调用controller的时候怎么知道我需要scope还是http还是两个都需要呢，这就牵着到了angular里的依赖注入，那么我们来模拟一下。

假设没有angular的情况下，我们：
```javascript
var MyController = function($scope,$http){
        $scope.test = 1;
        $http.get('');
}
MyController();//<span style="color: #808080;">undefined</span>```
肯定会报错的，然后我们来修改下我们的inject：
```javascript
var inject = {
            dependencies: {},
            register: function(key, value) {
                this.dependencies[key] = value;
            },
            resolve: function(deps, func, scope) {
                var arr = [];
                for (var i = 0 ; i &lt; deps.length ; i++) {
                    if (this.dependencies.hasOwnProperty(deps[i])) {
                       arr.push(this.dependencies[deps[i]])
                    }
                }
                console.log(arr);
                return function(){
                    func.apply(scope || {}, arr);
                }

            }
        }```
然后我们模仿angular来预先注册几个模块：
<pre class="lang:default decode:true ">inject.register('$http', {'get':function(){console.log('get')}});
inject.register('$scope', {'test':''});
inject.register('$location', {'hash':function(){console.log('hash')}});```
然后我们就可以注入了：
```javascript
MyController = inject.resolve(['$http','$scope'],MyController)；
MyController();```
我们只需要http和scope，所以我们只穿了两个，虽然这样看似解决了依赖注入，但是还有很多问题，比如我要交换两个参数的位置就不行了。

于是翻看了angularjs的源码，找到了：
```javascript
var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
.....
function annotate(fn) {
  .....
  fnText = fn.toString().replace(STRIP_COMMENTS, '');
  argDecl = fnText.match(FN_ARGS);
  .....
}```
我们忽略掉一些细节代码，只看我们需要的。annotate方法和我们的resolve方法很像。它转换传递过去的func为字符串，删除掉注释代码，然后抽取其中的参数。让我们看下它的执行结果，修改一下resolve方法：
```javascript
resolve: function(deps, func, scope) {
                
                var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
                var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
                var fnText = func.toString().replace(STRIP_COMMENTS, '');
                var argDecl = fnText.match(FN_ARGS);
                console.log(argDecl);
                

            }```
打印出argDecl：
```javascript
["function ($scope,$http)", "$scope,$http", index: 0, input: "function ($scope,$http){

$scope.test = 1;
$http.get('');
}"]```

可以看到，这个数组拿到了func的参数，argDecl［1］ = "$scope,$http";

根据这个，我们来修改resolve：
```javascript
resolve: function(func, scope) {
                
                var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
                var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
                var fnText = func.toString().replace(STRIP_COMMENTS, '');
                var argDecl = fnText.match(FN_ARGS);
                console.log(argDecl);
                var deps = argDecl[1].split(',');
                var arr = [];
                for (var i = 0 ; i &lt; deps.length ; i++) {
                    if (this.dependencies.hasOwnProperty(deps[i])) {
                       arr.push(this.dependencies[deps[i]])
                    }
                }
                return function(){
                    func.apply(scope || {}, arr);
                }

            }```
OK，这次我们不用在意参数的顺序了，但是angular远比我们要想的多，大多数情况下，我们的js都是要压缩的，所以function的实参会被替换，如果是那样的话，我们这个方法的argDecl［1］ = "$scope,$http";就会是argDecl［1］ = "r,t";类似这样的变量，那么又该怎么解决呢？

angular官方有这样的解释：

为了克服压缩引起的问题，只要在控制器函数里面给$inject属性赋值一个依赖服务标识符的数组，就像：
```javascript
var MyController = ['$scope', '$http', function($scope, $http) { /* constructor body */ }];```
那么，用到我们这个方法里面又该怎么实现呢？那我们在看看angular的源码吧：
```javascript
....
} else if (isArray(fn)) {
    last = fn.length - 1;
    assertArgFn(fn[last], 'fn')
    $inject = fn.slice(0, last);
  } else {
....```
看到了吧，之所以用到数组也是有原因的，把需要的依赖写在方法的前面，于是，应用到我们的reslove方法：
```javascript
resolve: function(func, scope) {
                isArray(func) {
                    var last = func.length - 1;
                    var deps = func.slice(0, last);
                    func = func[last]
                } else {
                    var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
                    var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
                    var fnText = func.toString().replace(STRIP_COMMENTS, '');
                    var argDecl = fnText.match(FN_ARGS);
                    var deps = argDecl[1].split(',');
                }
                
                var arr = [];
                for (var i = 0 ; i &lt; deps.length ; i++) {
                    if (this.dependencies.hasOwnProperty(deps[i])) {
                       arr.push(this.dependencies[deps[i]])
                    }
                }
                return function(){
                    func.apply(scope || {}, arr);
                }

            }```
OK，到这里，便可以用我们的inject来模拟angular的依赖注入了，当然，真正angular的依赖注入还有很多东西，这里就不在详细描述了。

以上观点都是我的个人见解，如有错误欢迎指正！

&nbsp;

完！

&nbsp;

参考资料：<a href="http://www.2cto.com/kf/201401/275236.html" target="_blank">http://www.2cto.com/kf/201401/275236.html</a>

&nbsp;
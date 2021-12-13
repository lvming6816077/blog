---
title: Proxy API--Vue3响应式对象reactive初探
date: 2020-05-26 17:26:17
tags:
- Vue3
- Vue.js
categories:
- 1209

---

`Proxy API`对应的`Proxy`对象是[ES2015](https://www.ecma-international.org/ecma-262/6.0/#sec-proxy-objects)就已引入的一个原生对象，用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。

从字面意思来理解，`Proxy`对象是目标对象的一个代理器，任何对目标对象的操作（实例化，添加/删除/修改属性等等），都必须通过该代理器。因此我们可以把来自外界的所有操作进行拦截和过滤或者修改等操作。

基于`Proxy`的这些特性，常用于：
* 创建一个可“响应式”的对象，例如Vue3.0中的reactive方法。
* 创建可隔离的JavaScript“沙箱”。

<!--more-->
## Proxy常见用法

Proxy语法：
```javascript
const p = new Proxy(target, handler)
```

* target：要使用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
* handler：以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 p 的行为。

例如下面一个很简单的用法：
```javascript
let foo = {
  a: 1,
  b: 2
}
let handler = {
    get:(obj,key)=>{
        console.log('get')
        return key in obj ? obj[key] : undefined
    }
}
let p = new Proxy(foo,handler)
console.log(p.a) // 1
```
上面代码中p就是foo的代理对象，对p对象的相关操作都会同步到foo对象上。

同时Proxy也提供了另一种生成代理对象的方法`Proxy.revocable()`：
```javascript
const { proxy,revoke } = Proxy.revocable(target, handler)
```
该方法的返回值是一个对象，其结构为： `{"proxy": proxy, "revoke": revoke}`，其中:
* proxy：表示新生成的代理对象本身，和用一般方式`new Proxy(target, handler)` 创建的代理对象没什么不同，只是它可以被撤销掉。
* revoke：撤销方法，调用的时候不需要加任何参数，就可以撤销掉和它一起生成的那个代理对象。

例如：
```javascript
let foo = {
  a: 1,
  b: 2
}
let handler = {
    get:(obj,key)=>{
        console.log('get')
        return key in obj ? obj[key] : undefined
    }
}
let { proxy,revoke } = Proxy.revocable(foo,handler)

console.log(proxy.a) // 1

revoke()

console.log(proxy.a) // Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
```
需要注意的是，一旦某个代理对象被撤销，它将变得几乎完全不可调用，在它身上执行任何的可代理操作都会抛出 TypeError 异常。

## Proxy的handler
上面代码中，我们只使用了get操作的handler，即当尝试获取对象的某个属性时会进入这个方法，除此之外Proxy共有接近14个handler也可以称作为钩子，它们分别是：
```
handler.getPrototypeOf()：
在读取代理对象的原型时触发该操作，比如在执行 Object.getPrototypeOf(proxy) 时。

handler.setPrototypeOf()：
在设置代理对象的原型时触发该操作，比如在执行 Object.setPrototypeOf(proxy, null) 时。

handler.isExtensible()：
在判断一个代理对象是否是可扩展时触发该操作，比如在执行 Object.isExtensible(proxy) 时。

handler.preventExtensions()：
在让一个代理对象不可扩展时触发该操作，比如在执行 Object.preventExtensions(proxy) 时。

handler.getOwnPropertyDescriptor()：
在获取代理对象某个属性的属性描述时触发该操作，比如在执行 Object.getOwnPropertyDescriptor(proxy, "foo") 时。

handler.defineProperty()：
在定义代理对象某个属性时的属性描述时触发该操作，比如在执行 Object.defineProperty(proxy, "foo", {}) 时。

handler.has()：
在判断代理对象是否拥有某个属性时触发该操作，比如在执行 "foo" in proxy 时。

handler.get()：
在读取代理对象的某个属性时触发该操作，比如在执行 proxy.foo 时。

handler.set()：
在给代理对象的某个属性赋值时触发该操作，比如在执行 proxy.foo = 1 时。

handler.deleteProperty()：
在删除代理对象的某个属性时触发该操作，即使用 delete 运算符，比如在执行 delete proxy.foo 时。

handler.ownKeys()：
当执行Object.getOwnPropertyNames(proxy) 和Object.getOwnPropertySymbols(proxy)时触发。

handler.apply()：
当代理对象是一个function函数时，调用apply()方法时触发，比如proxy.apply()。

handler.construct()：
当代理对象是一个function函数时，通过new关键字实例化时触发，比如new proxy()。
```
结合这些handler，我们可以实现一些针对对象的限制操作，例如：

* 禁止删除和修改对象的某个属性

```javascript
let foo = {
    a:1,
    b:2
}
let handler = {
    set:(obj,key,value,receiver)=>{
        console.log('set')
        if (key == 'a') throw new Error('can not change property:'+key)
        obj[key] = value
        return true
    },
    deleteProperty:(obj,key)=>{
        console.log('delete')
        if (key == 'a') throw new Error('can not delete property:'+key)
        delete obj[key]
        return true
    }
}

let p = new Proxy(foo,handler)

p.a = 3 // Uncaught Error

delete p.a  // Uncaught Error
```

其中，set方法的receiver通常是 Proxy 本即 p，但是当有一段代码执行 obj.name = "jen"， obj 不是一个 proxy，且自身不含 name 属性，但是它的原型链上有一个 proxy，那么，那个proxy的handler里的set方法会被调用，而此时obj会作为 receiver 这个参数传进来。

* 对属性的修改进行校验
```javascript
let foo = {
    a:1,
    b:2
}
let handler = {
    set:(obj,key,value)=>{
        console.log('set')
        if (typeof(value) !== 'number') throw new Error('can not change property:'+key)
        obj[key] = value
        return true
    }
}

let p = new Proxy(foo,handler)

p.a = 'hello' // Uncaught Error
```

## Proxy和响应式对象reactive
Vue3中的响应式对象：
```javascript
import {ref,reactive} from 'vue'
...
setup(){
  const name = ref('test')
  const state = reactive({
    list: []
  })
  return {
      name,
      state
  }
}
...
```
在Vue3中，composition-api提供了一种创建响应式对象的方法reactive，其内部就是利用了Proxy API来实现的，特别是借助handler的set方法，可以实现双向数据绑定相关的逻辑，这对于Vue2.x中的`Object.defineProperty()`是很大的改变。

* `Object.defineProperty()`只能单一的监听已有属性的修改或者变化，无法检测到对象属性的新增或删除，而Proxy则可以轻松实现。

* `Object.defineProperty()`无法监听属性值是数组类型的变化，而Proxy则可以轻松实现。

例如监听数组的变化：

```javascript
let arr = [1]
let handler = {
    set:(obj,key,value)=>{
        console.log('set')
        return Reflect.set(obj, key, value);
    }
}

let p = new Proxy(arr,handler)
p.push(2)
```

上面代码中`Reflect.set()`用于修改数组的值，可以参考[Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)，但是目前对于多层对象嵌套问题，需要经过一定的处理：

```javascript
let foo = {
    a:1,
    b:2
}
let handler = {
    set:(obj,key,value)=>{
        console.log('set')
        // 双向绑定相关逻辑
        obj[key] = value
        return true
    }
}

let p = new Proxy(foo,handler)

p.a = 3
```

上面代码中，对于简单的对象foo是完全没问题的，但是如果foo是一个复杂对象，里面嵌套的很多对象，那么当去尝试修改里层对象的值时，set方法就不会触发，为了解决这种场景，在Vue3中，采用了**递归**的方式来解决这个问题：

```javascript
let foo = {a:{c:3,d:{e:4}},b:2}
const isObject = (val)=>{
    return val !== null && typeof val === 'object'
}
const createProxy = (target)=>{
    let p = new Proxy(target,{
        get:(obj,key)=>{
            let res = obj[key] ? obj[key] : undefined

            // 判断类型，避免死循环
            if (isObject(res)) {
                return createProxy(res)
            } else {
                return res
            }
        },
        set: (obj, key, value)=> {
          console.log('set')
          obj[key] = value;

        }
    })

    return p
}

let result = createProxy(foo)

result.a.d.e = 6 // 打印出set
```

**当尝试去修改一个多层嵌套的对象的属性时，会触发该属性的上一级对象的get方法**，利用这个就可以对每个层级的对象添加Proxy代理，这样就实现了多层嵌套对象的属性修改问题。

当然，上面这段代码只是Vue3中reactive的一个缩影，更多的细节可以浏览相关[源码](https://github.com/vuejs/vue-next/tree/a5b4332c69146de569ad328cac9224c3cded15c9/packages/reactivity/src)来了解。

就目前来看，Porxy API相关内容是从ES2015才引入的标准，并且业界相关的polyfill也不是很完善，所以使用此API相关的框架要慎重的考虑兼容性问题。



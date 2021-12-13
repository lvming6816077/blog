---
title: Vue3源码解析--响应式原理
date: 2021-11-06 17:26:17
tags:
- Vue3
- 源码解析
categories:
- 914

---


响应式reactivity是Vue 3相对于Vue 2改动比较大的一个模块，也是性能提升最多的一个模块。其核心改变是采用了ES 6的Proxy API来代替Vue2中Object.defineProperty方法来实现响应式，那么什么是Proxy API呢，Vue 3的响应式又是如何实现的，下面将会进行揭晓。
<!--more-->
## Proxy API

Proxy API对应的Proxy对象是ES6就已引入的一个原生对象，用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。 从字面意思来理解，Proxy对象是目标对象的一个代理器，任何对目标对象的操作（实例化，添加/删除/修改属性等等），都必须通过该代理器。因此我们可以把来自外界的所有操作进行拦截和过滤或者修改等操作。 基于Proxy的这些特性，常用于：

-   创建一个可“响应式”的对象，例如Vue3.0中的reactive方法。
-   创建可隔离的JavaScript“沙箱”。

Proxy的基本语法如下代码所示：

```
const p = new Proxy(target, handler)
```

其中，target参数表示要使用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理），handler参数表示以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理p的行为。常见使用方法如下代码所示：

```
let foo = {
 a: 1,
 b: 2
}
let handler = {
    get:(obj,key)=>{
        console.log('get')
        return key in obj ? obj[key] : undefined
    }
}
let p = new Proxy(foo,handler)
console.log(p.a) // 打印1
```

上面代码中p就是foo的代理对象，对p对象的相关操作都会同步到foo对象上，同时Proxy也提供了另一种生成代理对象的方法Proxy.revocable()，如下代码所示：

```
const { proxy,revoke } = Proxy.revocable(target, handler)
```

该方法的返回值是一个对象，其结构为： `{"proxy": proxy, "revoke": revoke}`，其中：proxy表示新生成的代理对象本身，和用一般方式`new Proxy(target, handler)`创建的代理对象没什么不同，只是它可以被撤销掉，revoke表示撤销方法，调用的时候不需要加任何参数，就可以撤销掉和它一起生成的那个代理对象，如下代码所示：

```
let foo = {
 a: 1,
 b: 2
}
let handler = {
    get:(obj,key)=>{
        console.log('get')
        return key in obj ? obj[key] : undefined
    }
}
let { proxy,revoke } = Proxy.revocable(foo,handler)
console.log(proxy.a) // 打印1
revoke()
console.log(proxy.a) // 报错信息：Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
```

需要注意的是，一旦某个代理对象被撤销，它将变得几乎完全不可调用，在它身上执行任何的可代理操作都会抛出TypeError异常。 在上面代码中，我们只使用了get操作的handler，即当尝试获取对象的某个属性时会进入这个方法，除此之外Proxy共有接近14个handler也可以称作为钩子，它们分别是：

```
handler.getPrototypeOf()：
在读取代理对象的原型时触发该操作，比如在执行 Object.getPrototypeOf(proxy) 时。

handler.setPrototypeOf()：
在设置代理对象的原型时触发该操作，比如在执行 Object.setPrototypeOf(proxy, null) 时。

handler.isExtensible()：
在判断一个代理对象是否是可扩展时触发该操作，比如在执行 Object.isExtensible(proxy) 时。

handler.preventExtensions()：
在让一个代理对象不可扩展时触发该操作，比如在执行 Object.preventExtensions(proxy) 时。

handler.getOwnPropertyDescriptor()：
在获取代理对象某个属性的属性描述时触发该操作，比如在执行 Object.getOwnPropertyDescriptor(proxy, "foo") 时。

handler.defineProperty()：
在定义代理对象某个属性时的属性描述时触发该操作，比如在执行 Object.defineProperty(proxy, "foo", {}) 时。

handler.has()：
在判断代理对象是否拥有某个属性时触发该操作，比如在执行 "foo" in proxy 时。

handler.get()：
在读取代理对象的某个属性时触发该操作，比如在执行 proxy.foo 时。

handler.set()：
在给代理对象的某个属性赋值时触发该操作，比如在执行 proxy.foo = 1 时。

handler.deleteProperty()：
在删除代理对象的某个属性时触发该操作，即使用 delete 运算符，比如在执行 delete proxy.foo 时。

handler.ownKeys()：
当执行Object.getOwnPropertyNames(proxy) 和Object.getOwnPropertySymbols(proxy)时触发。

handler.apply()：
当代理对象是一个function函数时，调用apply()方法时触发，比如proxy.apply()。

handler.construct()：
当代理对象是一个function函数时，通过new关键字实例化时触发，比如new proxy()。
```

结合这些handler，我们可以实现一些针对对象的限制操作，例如： 禁止删除和修改对象的某个属性，如下代码所示：

```
let foo = {
    a:1,
    b:2
}
let handler = {
    set:(obj,key,value,receiver)=>{
        console.log('set')
        if (key == 'a') throw new Error('can not change property:'+key)
        obj[key] = value
        return true
    },
    deleteProperty:(obj,key)=>{
        console.log('delete')
        if (key == 'a') throw new Error('can not delete property:'+key)
        delete obj[key]
        return true
    }
}

let p = new Proxy(foo,handler)
// 尝试修改属性a
p.a = 3 // 报错信息：Uncaught Error
// 尝试删除属性a
delete p.a  // 报错信息：Uncaught Error
```

上面代码中，set方法多了一个receiver参数，这个参数通常是Proxy本身即p，场景是当有一段代码执行`obj.name="jen"`，obj不是一个proxy，且自身不含name属性，但是它的原型链上有一个proxy，那么，那个proxy的handler里的set方法会被调用，而此时obj会作为receiver这个参数传进来。 对属性的修改进行校验，如下代码所示：

```
let foo = {
    a:1,
    b:2
}
let handler = {
    set:(obj,key,value)=>{
        console.log('set')
        if (typeof(value) !== 'number') throw new Error('can not change property:'+key)
        obj[key] = value
        return true
    }
}
let p = new Proxy(foo,handler)
p.a = 'hello' // 报错信息：Uncaught Error
```

Proxy也能监听到数组变化，如下代码所示：

```
let arr = [1]
let handler = {
    set:(obj,key,value)=>{
        console.log('set') // 打印set
        return Reflect.set(obj, key, value);
    }
}

let p = new Proxy(arr,handler)
p.push(2) // 改变数组
```

`Reflect.set()`用于修改数组的值，返回布尔类型，这也可以兼容修改数组原型上的方法对应场景，相当于`obj[key] = value`。

## Proxy和响应式对象reactive

在Vue 3中，使用响应式对象方法如下代码所示：

```
import {ref,reactive} from 'vue'
...
setup(){
  const name = ref('test')
  const state = reactive({
    list: []
  })
  return {name,state}
}
...
```

在Vue 3中，Composition API中会经常使用创建响应式对象的方法ref/reactive，其内部就是利用了Proxy API来实现的，特别是借助handler的set方法，可以实现双向数据绑定相关的逻辑，这对于Vue 2中的Object.defineProperty()是很大的改变，主要提升如下：

-   `Object.defineProperty()`只能单一的监听已有属性的修改或者变化，无法检测到对象属性的新增或删除（Vue 2中是采用$set()方法来解决），而Proxy则可以轻松实现。
-   `Object.defineProperty()`无法监听响应式数据类型是数组的变化（主要是数组长度变化，Vue 2中采用重写数组相关方法并添加钩子来解决），而Proxy则可以轻松实现。

正是由于Proxy的特性，在原本使用Object.defineProperty()需要很复杂的方式才能实现的上面两种能力，在Proxy无需任何配置，利用其原生的特性就可以轻松实现。

## ref()方法运行原理

在Vue 3的源码中，所有关于响应式的代码都在vue-next/package/reactivity下面，其中reactivity/src/index.ts里暴露了所有可以使用的方法。我们以常用的ref()方法举例，来看看Vue 3是如何利用Proxy的。 ref()方法的主要逻辑在reactivity/src/ref.ts中，其代码如下：

```
...
// 入口方法
export function ref(value?: unknown) {
  return createRef(value, false)
}
function createRef(rawValue: unknown, shallow: boolean) {
  // rawValue表示原始对象，shallow表示是否递归
  // 如果本身已经是ref对象，则直接返回
  if (isRef(rawValue)) {
    return rawValue
  }
  // 创建一个新的RefImpl对象
  return new RefImpl(rawValue, shallow)
}
...
```

createRef这个方法接收的第二个参数是shallow，表示是否是递归监听响应式，这个和另外一个响应式方法shallowRef()是对应的。在RefImpl构造函数中，有一个value属性，这个属性是由toReactive()方法所返回，toReactive()方法则在reactivity/src/reactive.ts文件中，如下代码所示：

```
class RefImpl<T> {
  ...
  constructor(value: T, public readonly _shallow: boolean) {
    this._rawValue = _shallow ? value : toRaw(value)
    // 如果是非递归，调用toReactive
    this._value = _shallow ? value : toReactive(value)
  }
  ...
}
```

在reactive.ts中，则开始真正创建一个响应式对象，如下代码所示：

```
export function reactive(target: object) {
  // 如果是readonly，则直接返回，就不添加响应式了
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  return createReactiveObject(
    target,// 原始对象
    false,// 是否readonly
    mutableHandlers,// proxy的handler对象baseHandlers
    mutableCollectionHandlers,// proxy的handler对象collectionHandlers
    reactiveMap// proxy对象映射
  )
}
```

其中，`createReactiveObject()`方法传递了两种handler，分别是baseHandlers和collectionHandlers，如果target的类型是Map，Set，WeakMap，WeakSet则会使用collectionHandlers，类型是Object，Array则会是baseHandlers，如果是一个基础对象，也不会创建Proxy对象，reactiveMap则存储所有响应式对象的映射关系，用来避免同一个对象的重复创建响应式。我们在来看看createReactiveObject()方法的实现，如下代码所示：

```
function createReactiveObject(...) {
  // 如果target不满足typeof val === 'object'，则直接返回target
  if (!isObject(target)) {
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`)
    }
    return target
  }
  // 如果target已经是proxy对象或者只读,则直接返回
  // exception: calling readonly() on a reactive object
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }
  // 如果target已经被创建过Proxy对象，则直接返回这个对象
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  // 只有符合类型的target才能被创建响应式
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
  // 调用Proxy API创建响应式
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  // 标记该对象已经创建过响应式
  proxyMap.set(target, proxy)
  return proxy
}
```

可以看到在`createReactiveObject()`方法中，主要做了以下事情：

-   防止只读和重复创建响应式。
-   根据不同的target类型选择不同的handler。
-   创建Proxy对象。

最终会调用new Proxy来创建响应式对象，我们以baseHandlers为例，看看这个handler是怎么实现的，在reactivity/src/baseHandlers.ts可以看到这部分代码，主要实现了这几个handler，如下代码所示：

```
const get = /*#__PURE__*/ createGetter()
...
export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys
}
```

以handler.get为例看看在其内部做了什么操作，当我们尝试读取对象的属性时，便会进入get方法，其核心代码如下所示：

```
function createGetter(isReadonly = false, shallow = false) {
  return function get(target: Target, key: string | symbol, receiver: object) {
    if (key === ReactiveFlags.IS_REACTIVE) { // 如果访问对象的key是__v_isReactive，则直接返回常量
      return !isReadonly
    } else if (key === ReactiveFlags.IS_READONLY) {// 如果访问对象的key是__v_isReadonly，则直接返回常量
      return isReadonly
    } else if (// 如果访问对象的key是__v_raw，或者原始对象只读对象等等直接返回target
      key === ReactiveFlags.RAW &&
      receiver ===
        (isReadonly
          ? shallow
            ? shallowReadonlyMap
            : readonlyMap
          : shallow
          ? shallowReactiveMap
          : reactiveMap
        ).get(target)
    ) {
      return target
    }
    // 如果target是数组类型
    const targetIsArray = isArray(target)
    // 并且访问的key值是数组的原生方法，那么直接返回调用结果
    if (!isReadonly && targetIsArray && hasOwn(arrayInstrumentations, key)) {
      return Reflect.get(arrayInstrumentations, key, receiver)
    }
    // 求值
    const res = Reflect.get(target, key, receiver)
    // 判断访问的key是否是Symbol或者不需要响应式的key例如__proto__,__v_isRef,__isVue
    if (isSymbol(key) ? builtInSymbols.has(key) : isNonTrackableKeys(key)) {
      return res
    }
    // 收集响应式，为了后面的effect方法可以检测到
    if (!isReadonly) {
      track(target, TrackOpTypes.GET, key)
    }
    // 如果是非递归绑定，直接返回结果
    if (shallow) {
      return res
    }

    // 如果结果已经是响应式的，先判断类型，再返回
    if (isRef(res)) {
      const shouldUnwrap = !targetIsArray || !isIntegerKey(key)
      return shouldUnwrap ? res.value : res
    }

    // 如果当前key的结果也是一个对象，那么就要递归调用reactive方法对改对象再次执行响应式绑定逻辑
    if (isObject(res)) {
      return isReadonly ? readonly(res) : reactive(res)
    }
    // 返回结果
    return res
  }
}
```

上面这段代码是Vue 3响应式的核心代码之一，其逻辑相对比较复杂，读者可以根据注释来理解，总结下来，这段代码主要做了以下事情：

-   对于`handler.get`方法来说，最终都会返回当前对象对应key的结果即`obj[key]`，所以该段代码最终会return结果。
-   对非响应式key，只读key等直接返回对应的结果。
-   对于数组类型的target，key值如果是原型上的方法，例如includes，push，pop等，采用Reflect.get直接返回。
-   在effect添加收集监听track，为响应式监听服务。
-   当当前key对应的结果是一个对象时，为了保证set方法能够触发，需要循环递归的对这个对象进行响应式绑定即递归调用reactive()方法。

handler.get方法主要功能是对结果value的返回，那么我们看看handler.set主要做了什么，其代码如下所示：

```
function createSetter(shallow = false) {
  return function set(
    target: object,
    key: string | symbol,
    value: unknown,// 即将被设置的新值
    receiver: object
  ): boolean {
    // 缓存旧值
    let oldValue = (target as any)[key]
    if (!shallow) {
      // 新旧值转换原始对象
      value = toRaw(value)
      oldValue = toRaw(oldValue)
      // 如果旧值已经是一个RefImpl对象且新值不是RefImpl对象
      // 例如var v = Vue.reactive({a:1,b:Vue.ref({c:3})})场景的set
if (!isArray(target) && isRef(oldValue) && !isRef(value)) {
        oldValue.value = value // 直接将新值赋给旧址的响应式对象里
        return true
      }
    }
    // 用来判断是否是新增key还是更新key的值
    const hadKey =
      isArray(target) && isIntegerKey(key)
        ? Number(key) < target.length
        : hasOwn(target, key)
    // 设置set结果，并添加监听effect逻辑
    const result = Reflect.set(target, key, value, receiver)
    // 判断target没有动过，包括在原型上添加或者删除某些项
    if (target === toRaw(receiver)) {
      if (!hadKey) {
        trigger(target, TriggerOpTypes.ADD, key, value)// 新增key的触发监听
      } else if (hasChanged(value, oldValue)) {
        trigger(target, TriggerOpTypes.SET, key, value, oldValue)// 更新key的触发监听
      }
    }
    // 返回set结果 true/false
    return result
  }
}
```

`handler.set`方法核心功能是设置key对应的值即`obj[key] = value`，同时对新旧值进行逻辑判断和处理，最后添加上trigger触发监听track逻辑，便于触发effect。 如果读者感觉上述源码理解比较困难，笔者剔除一些边界和兼容判断，将整个流程进行梳理和简化，可以参考下面这段便于理解的代码：

```
let foo = {a:{c:3,d:{e:4}},b:2}
const isObject = (val)=>{
    return val !== null && typeof val === 'object'
}
const createProxy = (target)=>{
    let p = new Proxy(target,{
        get:(obj,key)=>{
            let res = obj[key] ? obj[key] : undefined

            // 添加监听
            track(target)
            // 判断类型，避免死循环
            if (isObject(res)) {
                return createProxy(res)// 循环递归调用
            } else {
                return res
            }
        },
        set: (obj, key, value)=> {
          console.log('set')
          
          obj[key] = value;
          // 触发监听
          trigger(target)
          return true
        }
    })

    return p
}

let result = createProxy(foo)

result.a.d.e = 6 // 打印出set
```

当尝试去修改一个多层嵌套的对象的属性时，会触发该属性的上一级对象的get方法，利用这个就可以对每个层级的对象添加Proxy代理，这样就实现了多层嵌套对象的属性修改问题，在此基础上同时添加track和trigger逻辑，就完成了基本的响应式流程。我们将在后面章节结合双向绑定来具体讲解track和trigger流程。



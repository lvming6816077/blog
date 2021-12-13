---
title: Vue3源码解析--虚拟DOM
date: 2021-11-11 17:26:17
tags:
- Vue3
- 源码解析
categories:
- 916

---


# Vue3源码解析--虚拟DOM

## 什么是虚拟DOM
在浏览器中，HTML页面由基本的DOM树来组成的，当其中一部分发生变化时，其实就是对应某个DOM节点发生了变化，当DOM节点发生变化时就会触发对应的重绘或者重排，当过多的重绘和重排在短时间内发生时，就会可能引起页面的卡顿，所以改变DOM是有一些代价的，那么如何优化DOM变化的次数以及在合适的时机改变DOM就是开发者需要注意的事情。
<!--more-->
虚拟DOM就是为了解决上述浏览器性能问题而被设计出来的。当一次操作中有10次更新DOM的动作，虚拟DOM不会立即操作DOM，而是和原本的DOM进行对比，将这10次更新的变化部分内容保存到内存中，最终一次性的应用在到DOM树上，再进行后续操作，避免大量无谓的计算量。

虚拟DOM实际上就是采用JavaScript对象来存储DOM节点的信息，将DOM的更新变成对象的修改，并且这些修改计算在内存中发生，当修改完成后，再将JavaScript转换成真实的DOM节点，交给浏览器，从而达到性能的提升。
例如下面一段DOM节点，如下代码所示：
```
<div id="app">
  <p class="text">Hello</p>
</div>
```
转换成一般的虚拟DOM对象结构，如下代码所示：
```javascript
{
  tag: 'div',
  props: {
    id: 'app'
  },
  chidren: [
    {
      tag: 'p',
      props: {
        className: 'text'
      },
      chidren: [
        'Hello'
      ]
    }
  ]
}
```
上面这段代码就是一个基本的虚拟DOM，但是他并非是Vue中使用的虚拟DOM结构，因为Vue要复杂的多。

## Vue 3虚拟DOM

在Vue中，我们写在`<template>`标签内的内容都属于DOM节点，这部分内容会被最终转换成Vue中的虚拟DOM对象VNode，其中的步骤比较复杂，主要有以下几个过程：

* 抽取`<template>`内容进行compile编译。
* 得到AST语法树，并生成render方法。
* 执行render方法得到VNode对象。
* VNode转换真实DOM并渲染到页面。

完整流程如下图：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/447220e690a64adfb3a008573746d95c~tplv-k3u1fbpfcp-watermark.image?)

我们以一个简单的demo为例子，在Vue 3的源码里去寻找，到底是如何一步一步进行了，demo如下代码所示：
```
<div id="app">
  <div>
    {{name}}
  </div>
  <p>123</p>
</div>
Vue.createApp({
  data(){
    return {
      name : 'abc'
    }
  }
}).mount("#app")
```
上面代码中，data中定义了一个响应式数据name，并在`<template>`中使用插值表达式`{{name}}`进行使用，还有一个静态节点`<p>123</p>`。

## 获取`<template>`内容

调用`createApp()`方法，会进入到源码`packages/runtime-dom/src/index.ts`里面的createApp()方法，如下代码所示：

```javascript
export const createApp = ((...args) => {
  const app = ensureRenderer().createApp(...args)
  ...
  app.mount = (containerOrSelector: Element | ShadowRoot | string): any => {
    if (!isFunction(component) && !component.render && !component.template) {
      // 将#app绑定的HTML内容赋值给template项上
      component.template = container.innerHTML

      // 调用mount方法渲染
    const proxy = mount(container, false, container instanceof SVGElement)
    return proxy
  }
  ...
  return app
}) as CreateAppFunction<Element>
```

对于根组件来说，`<template>`的内容由挂载的`#app`元素里面的内容组成，如果项目是采用npm和Vue Cli+Webpack这种前端工程化的方式，那么对于`<template>`的内容则主要由对应的loader在构建时对文件进行处理来获取，这和在浏览器运行时的处理方式是不一样的。

## 生成AST语法树

在得到`<template>`后，就依据内容生成AST语法树。抽象语法树（Abstract Syntax Tree，AST），是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。比如，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现；而类似于if-condition-then这样的条件跳转语句，可以使用带有三个分支的节点来表示。如下代码所示：
```javascript
while b ≠ 0
  if a > b
a := a − b
  else
b := b − a
return a
```
如果将上述代码转换成广泛意义上的语法树，如图所示。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/159a46e93d5f4c87980cfb549c8ab024~tplv-k3u1fbpfcp-watermark.image?)


对于`<template>`的内容，其大部分是由DOM组成，但是也会有`if-condition-then`这样的条件语句，例如`v-if`，`v-for`指令等等，在Vue 3中，这部分逻辑在源码`packages\compiler-core\src\compile.ts中baseCompile`方法，核心代码如下所示：

```javascript
export function baseCompile(
  template: string | RootNode,
  options: CompilerOptions = {}
): CodegenResult {
  ...
  // 通过template生成ast树结构
  const ast = isString(template) ? baseParse(template, options) : template
  ...
  // 转换
  transform(
    ast,
    ...
  )
  return generate(
    ast,
    extend({}, options, {
      prefixIdentifiers
    })
  )
}
```

baseCompile方法主要做了以下事情：

* 生成Vue中的AST对象。
* 将AST对象作为参数传入transform函数，进行转换。
* 将转换后的AST对象作为参数传入generate函数，生成render函数。

其中，baseParse方法用来创建AST对象，在Vue 3中，AST对象是一个`RootNode`类型的树状结构，在源码`packages\compiler-core\src\ast.ts`中，其结构如下代码所示：

```javascript
export function createRoot(
  children: TemplateChildNode[],
  loc = locStub
): RootNode {
  return {
    type: NodeTypes.ROOT, // 元素类型
    children, // 子元素
    helpers: [],// 帮助函数
    components: [],// 子组件
    directives: [], // 指令
    hoists: [],// 标识静态节点
    imports: [],
    cached: 0, // 缓存标志位
    temps: 0,
    codegenNode: undefined,// 存储生成render函数字符串
    loc // 描述元素在AST树的位置信息
  }
}
```
其中，children存储的时后代元素节点的数据，这就构成一个AST树结构，type表示元素的类型NodeType，主要分为HTML普通类型和Vue指令类型等，常见的有以下几种：
```
ROOT,  // 根元素 0
ELEMENT, // 普通元素 1
TEXT, // 文本元素 2
COMMENT, // 注释元素 3
SIMPLE_EXPRESSION, // 表达式 4
INTERPOLATION, // 插值表达式 {{ }} 5
ATTRIBUTE, // 属性 6
DIRECTIVE, // 指令 7
IF, // if节点 9
JS_CALL_EXPRESSION, // 方法调用 14
...
```
`hoists`是一个数组，用来存储一些可以静态提升的元素，在后面的`transform`会将静态元素和响应式元素分开创建，这也是Vue 3中优化的体现，codegenNode则用来存储最终生成的render方法的字符串，loc表示元素在AST树的位置信息。

在生成AST树时，Vue 3在解析`<template>`内容时，会用一个栈`stack`来保存解析到的元素标签。当它遇到开始标签时，会将这个标签推入栈，遇到结束标签时，将刚才的标签弹出栈。它的作用是保存当前已经解析了，但还没解析完的元素标签。这个栈还有另一个作用，在解析到某个字节点时，通过`stack[stack.length - 1]`可以获取它的父元素。

demo代码中生成的AST语法树如下图所示。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/479408c50a0d40a586b31f70d9c3a16d~tplv-k3u1fbpfcp-watermark.image?)


## 生成render方法字符串

在得到`AST`对象后，会进入`transform`方法，在源码`packages\compiler-core\src\transform.ts`中，其核心代码如下所示：

```javascript
export function transform(root: RootNode, options: TransformOptions) {
// 数据组装  
const context = createTransformContext(root, options)
  // 转换代码
  traverseNode(root, context)
  // 静态提升
  if (options.hoistStatic) {
    hoistStatic(root, context)
  }// 服务端渲染
  if (!options.ssr) {
    createRootCodegen(root, context)
  }
  // 透传元信息
  root.helpers = [...context.helpers.keys()]
  root.components = [...context.components]
  root.directives = [...context.directives]
  root.imports = context.imports
  root.hoists = context.hoists
  root.temps = context.temps
  root.cached = context.cached
  if (__COMPAT__) {
    root.filters = [...context.filters!]
  }
}
```

`transform`方法主要是对AST进行进一步转化，为`generate`函数生成`render`方法做准备，主要做了以下事情：

* traverseNode方法将会递归的检查和解析AST元素节点的属性，例如结合helpers方法对@click等事件添加对应的方法和事件回调，对插值表达式、指令、props添加动态绑定等。
* 处理类型逻辑包括静态提升逻辑，将静态节点赋值给hoists，以及根据不同类型的节点打上不同的patchFlag，便于后续diff使用。
* 在AST上绑定并透传一些元数据。

`generate`方法主要是生成`render`方法的字符串code，在源码`packages\compiler-core\src\codegen.ts`中，其核心代码如下所示：
```javascript
export function generate(
  ast: RootNode,
  options: CodegenOptions & {
    onContextCreated?: (context: CodegenContext) => void
  } = {}
): CodegenResult {
  const context = createCodegenContext(ast, options)
  if (options.onContextCreated) options.onContextCreated(context)
  const {
    mode,
    push,
    prefixIdentifiers,
    indent,
    deindent,
    newline,
    scopeId,
    ssr
  } = context
  ...
  // 缩进处理
  indent()
  deindent()
  // 单独处理component、directive、filters
  genAssets()
  // 处理NodeTypes里的所有类型
  genNode(ast.codegenNode, context)
  ...
  // 返回code字符串
  return {
    ast,
    code: context.code,
    preamble: isSetupInlined ? preambleContext.code : ``,
    // SourceMapGenerator does have toJSON() method but it's not in the types
    map: context.map ? (context.map as any).toJSON() : undefined
  }
}
```
`generate`方法的核心逻辑在`genNode`方法中，其逻辑是根据不同的NodeTypes类型构造出不同的`render`方法字符串，部分类型如下代码所示：
```javascript
switch (node.type) {
case NodeTypes.ELEMENT:
case NodeTypes.IF:
case NodeTypes.FOR:// for关键字元素节点
  genNode(node.codegenNode!, context)
  break
case NodeTypes.TEXT:// 文本元素节点
  genText(node, context)
  break
case NodeTypes.VNODE_CALL:// 核心：VNode混合类型节点（AST语法树节点）
  genVNodeCall(node, context)
  break
case NodeTypes.COMMENT: // 注释元素节点
  genComment(node, context)
  break
case NodeTypes.JS_FUNCTION_EXPRESSION:// 方法调用节点
  genFunctionExpression(node, context)
  break
...
```
其中：

* 节点类型NodeTypes.VNODE_CALL对应genVNodeCall方法和ast.ts文件里面的createVNodeCall方法对应，后者用来返回VNodeCall，前者生成对应的VNodeCall这部分render方法字符串，是整个render方法字符串的核心。
* 节点类型NodeTypes.FOR对应for关键字元素节点，其内部是递归调用了genNode方法。
* 节点类型NodeTypes.TEXT对应文本元素节点负责静态文本的生成。
* 节点类型NodeTypes.JS_FUNCTION_EXPRESSION对应方法调用节点，负责方法表达式的生成。

终于，经过一系列的加工，最终生成的render方法字符串结果如下所示：
```javascript
(function anonymous(
) {
const _Vue = Vue
const { createElementVNode: _createElementVNode } = _Vue

const _hoisted_1 = ["data-a"] // 静态节点
const _hoisted_2 = /*#__PURE__*/_createElementVNode("p", null, "123", -1 /* HOISTED */)// 静态节点

return function render(_ctx, _cache) {// render方法
  with (_ctx) {
    const { toDisplayString: _toDisplayString, createElementVNode: _createElementVNode, Fragment: _Fragment, openBlock: _openBlock, createElementBlock: _createElementBlock } = _Vue // helper方法

    return (_openBlock(), _createElementBlock(_Fragment, null, [
      _createElementVNode("div", { "data-a": attr }, _toDisplayString(name), 9 /* TEXT, PROPS */, _hoisted_1),
      _hoisted_2
    ], 64 /* STABLE_FRAGMENT */))
  }
}
})
```

`_createElementVNode`，`_openBlock`等等上一步传进来的`helper`方法。其中`<p>123</p>`这种属于没有响应式绑定的静态节点，会被单独区分，而对于动态节点会使用`createElementVNode`方法来创建，最终这两种节点会进入`createElementBlock`方法进行VNode的创建。

render方法中使用了with关键字，with的作用如下代码所示：
```javascript
const obj = {
  a:1
}
with(obj){
  console.log(a) // 打印1
}
```
在`with(_ctx)`包裹下，我们在data中定义的响应式变量才能正常使用，例如调用`_toDisplayString(name)`，其中name就是响应式变量。

## 得到最终VNode对象

最终，这是一段可执行代码，会赋值给组件`Component.render`方法上，其源码在`packages\runtime-core\src\component.ts`中，如下所示：
```javascript
...
Component.render = compile(template, finalCompilerOptions)
...
if (installWithProxy) { // 绑定代理
   installWithProxy(instance)
}
...
```
`compile`方法是最初`baseCompile`方法的入口，在完成赋值后，还需要绑定代理，执行`installWithProxy`方法，其源码在`runtime-core/src/component.ts`中，如下所示：
```javascript
export function registerRuntimeCompiler(_compile: any) {
  compile = _compile
  installWithProxy = i => {
    if (i.render!._rc) {
      i.withProxy = new Proxy(i.ctx, RuntimeCompiledPublicInstanceProxyHandlers)
    }
  }
}
```
这主要是给`render`里`_ctx`的响应式变量添加绑定，当上面`render`方法里的`name`被使用时，可以通过代理监听到调用，这样就会进入响应式的监听收集`track`，当触发`trigger`监听时，进行`diff`。

在`runtime-core/src/componentRenderUtils.ts`源码里的`renderComponentRoot`方法里会执行`render`方法得到`VNode`对象，其核心代码如下所示：
```javascript
export function renderComponentRoot(){
  // 执行render
  let result = normalizeVNode(render!.call(
        proxyToUse,
        proxyToUse!,
        renderCache,
        props,
        setupState,
        data,
        ctx
      ))
  ...

  return result
}
```
demo代码中最终得到的VNode对象如下图所示。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f3fe695743a49ee99fd69c98ccc2331~tplv-k3u1fbpfcp-watermark.image?)

上图就是通过`render`方法运行后得到的`VNode`对象，可以看到`children`和`dynamicChildren`区分，前者包括了两个子节点分别是`<div>`和`<p>`这个和在`<template>`里面定义的内容是对应的，而后者只存储了动态节点，包括动态`props`即`data-a`属性。同时`VNode`也是树状结构，通过`children`和`dynamicChildren`一层一层递进下去。

在通过`render`方法得到`VNode`的过程也是对指令，插值表达式，响应式数据，插槽等一系列Vue语法的解析和构造过程，最终生成结构化的`VNode`对象，可以将整个过程总结成流程图，便于读者理解，如下图所示。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/447220e690a64adfb3a008573746d95c~tplv-k3u1fbpfcp-watermark.image?)

另外一个需要关注的属性是patchFlag这个是后面进行VNode的diff时所用到的标志位，数字64表示稳定不需要改变。最后得到VNode对象后需要转换成真实的DOM节点，这部分逻辑是在虚拟DOM的diff中完成的，在后面的双向绑定原理解析中进行讲解。



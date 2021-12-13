---
title: Vue3.0--Vue Composition API使用体验
date: 2020-05-22 17:26:17
tags:
- Vue3
- Vue.js
categories:
- 1210
photos: https://pic3.zhimg.com/v2-74bf1c42b3eed22bb78c4a72eb1399c6_1440w.jpg?source=172ae18b
---

Vue3.0目前已经出了beta版本,并在github上进行了开源，叫做[vue-next](https://github.com/vuejs/vue-next)，本文将之前采用Vue2.6开发的todoList小项目改造成为Vue3.0编写，并介绍一下2.x和3.x之间写法的不同之处。

[点击体验](https://www.nihaoshijie.com.cn/mypro/vue3todo/index.html)
Github地址：[Vue.js2.6版本todoList](https://github.com/lvming6816077/vue-todo/)，[Vue.js3.0版本todoList](https://github.com/lvming6816077/vue3todo/)
<!--more-->

Vue3.x适配大部分Vue2.x的组件配置，也就是说以前我们在Vue2.x针对组件的一些配置项，例如：
```
export default {
  name: 'test',
  components: {},
  props: {},
  data () {
    return {}
  },
  created(){},
  mounted () {},
  watch:{},
  methods: {}
}
```
在Vue3.x中也是可以适配的，对应的相关生命周期方法也可正常执行，但是Vue3.x的一大核心是引入了[Vue Composition API](https://composition-api.vuejs.org/zh/api.html)（组合式API）,这使得组件的大部分内容都可以通过`setup()`方法进行配置，同时Vue Composition API在Vue2.x也可以使用，需要通过安装@vue/composition-api来使用：
```javascript
npm install @vue/composition-api
...
import VueCompositionApi from '@vue/composition-api';

Vue.use(VueCompositionApi);
```
下面主要介绍一下采用Vue Composition API来改造采用2.x开发的todoList项目时的新老代码对比。

### 如何创建一个Vue3.0的项目
首先，安装vue cli的最新版本，一般是vue cli 4，安装成功后，调用：
```bash
vue create myapp
```
创建一个基于Vue2.x的项目，然后进入项目的根目录，执行：
```bash
vue add vue-next
```
然后就会自动安装[vue-cli-plugin-vue-next](https://github.com/vuejs/vue-cli-plugin-vue-next)插件，完毕之后，myapp项目就会变成一个基于Vue3.0Beta版本的项目框架。

### 根实例初始化:
在2.x中通过`new Vue()`的方法来初始化：
```javascript
import App from './App.vue'
new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```
在3.x中Vue不再是一个构造函数，通过createApp方法初始化：
```javascript
import App from './App.vue'
createApp(App).use(store).mount('#app')
```


### ref或者reactive代替data中的变量:
在2.x中通过组件data的方法来定义一些当前组件的数据：
```javascript
...
data() {
  return {
    name: 'test',
    list: [],
  }
},
...
```
在3.x中通过ref或者reactive创建响应式对象：
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
ref将给定的值创建一个响应式的数据对象并赋值初始值（int或者string），reactive可以直接定义复杂响应式对象。
### methods中定义的方法也可以写在setup()中:
在2.x中methods来定义一些当前组件内部方法：
```javascript
...
methods: {
  fetchData() {
    
  },
}
...
```
在3.x中直接在setup方法中定义并return：
```javascript
...
setup(){
  const fetchData = ()=>{
      console.log('fetchData')
  }

  return {
      fetchData
  }
}
...
```
### 无法使用EventBus:
在2.x中通过EventBus的方法来实现组件通信：
```javascript
var EventBus = new Vue()
Vue.prototype.$EventBus = EventBus
...
this.$EventBus.$on()  this.$EventBus.$emit()
```

在3.x中移除了`$on, $off`等方法（参考[rfc](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0020-events-api-change.md)），而是推荐使用[mitt](https://github.com/developit/mitt#examples--demos)方案来代替：
```javascript
import mitt from 'mitt'
const emitter = mitt()
// listen to an event
emitter.on('foo', e => console.log('foo', e) )
// fire an event
emitter.emit('foo', { a: 'b' })
```
由于3.x中不再支持prototype的方式设置全局方法，可以通过`app.config.globalProperties.mitt = () => {}`方案。
### setup()中使用props和this:
在2.x中，组件的方法中可以通过this获取到当前组件的实例，并执行data变量的修改，方法的调用，组件的通信等等，但是在3.x中，setup()在beforeCreate和created时机就已调用，无法使用和2.x一样的this，但是可以通过接收`setup(props,ctx)`的方法，获取到当前组件的实例和props：
```javascript
export default {
  props: {
    name: String,
  },
  setup(props,ctx) {
    console.log(props.name)
    ctx.emit('event')
  },
}
```
注意ctx和2.x中this并不完全一样，而是选择性地暴露了一些property，主要有`[attrs,emit,slots]`。
### watch来监听对象改变
2.x中，可以采用watch来监听一个对象属性是否有改动：
```javascript
...
data(){
  return {
    name: 'a'  
  }
},
watch: {
  name(val) {
    console.log(val)
  }
}
...
```
3.x中，在setup()中，可以使用watch来监听：
```javascript
...
import {watch} from 'vue'
setup(){
  let state = reactive({
    name: 'a'
  })
  watch(
    () => state.name,
    (val, oldVal) => {
      console.log(val)
    }
  )
  state.name = 'b'
  return {
      state
  }
}
...
```
在3.x中，如果watch的是一个数组array对象，那么如果调用array.push()方法添加一条数据，并不会触发watch方法，必须重新给array赋值：
```javascript
 let state = reactive({
    list: []
 })
 watch(
    () => state.list,
    (val, oldVal) => {
      console.log(val)
    }
  )
  
  state.list.push(1) // 不会触发watch
  
  state.list = [1] // 会触发watch
```
此问题不知是否是Vue3.x特意加上的，有待正式版出来后在验证。

### computed计算属性：
2.x中：
```javascript
...
computed: {
    storeData () {
      return this.$store.state.storeData
    },
},
...
```
3.x中：
```javascript
...
import {computed} from 'vue'
setup(){
  const storeData = computed(() => store.state.storeData)

  return {
      storeData
  }
}
...
```

当然，对于完整的Vue Composition API，各位同学可以参考[文档](https://composition-api.vuejs.org/zh/api.html)。就目前来说Vue3.0还并没有发布正式版，所以很多用法和实现方式可能还是未知数，但是对于新版本提供的编码方式的新思路，还是很期待的。



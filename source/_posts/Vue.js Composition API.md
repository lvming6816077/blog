---
title: Vue.js Composition API
date: 2021-11-26 17:26:17
tags:
- vue3
- Composition API
categories:
- 1914

---


在Vue 3引入的Composition API翻译过来就叫做组合式API，所谓组合式就是我们可以自由的组合逻辑，即剥离公共逻辑，差异化个性逻辑，维护整体逻辑。我们知道一个大型的Vue应用就是业务逻辑的综合体，而Vue组件就是组成这个综合体的个体。
<!--more-->
通过创建Vue组件，我们可以将界面中重复的部分连同其功能一起提取为可重用的代码段。仅此一项就可以使我们的应用在可维护性和灵活性方面走得相当远。然而，我们的经验已经证明，光靠这一点可能并不够，尤其是当你的应用变得非常大的时候——想想几百个组件。处理这样的大型应用时，共享和重用代码变得尤为重要。

Composition API给我们提供了更加高效的代码逻辑组合能力，整体提示项目的可维护性，是函数式编程的重要体现。

## Composition API 基础

通常，一个Vue组件对象大概是包括一些data属性，生命周期钩子函数，methods，components，props等等的配置项的Object对象，如示例代码所示。

```javascript
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

这种通过选项来配置Vue组件的方式称作配置式API，我们大部分的业务逻辑都是写在这些配置对应的方法或者配置里，这种方式使得每个配置各司其职，data、computed、methods、watch每个组件选项都有自己的业务逻辑。然而，当我们的组件开始变得更大时，逻辑关注点的列表也会增长。尤其对于那些一开始没有编写这些组件的人来说，这会导致组件难以阅读和理解，如图所示。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f9f354a4b414d1c9b70e83eb41a098a~tplv-k3u1fbpfcp-watermark.image?)

如上图是一个大型组件，其逻辑是很复杂的，其中的逻辑关注点按颜色进行分组，当我们关注一条流程逻辑时，可能需要来回的在data、computed、methods、watch之间切换滚动这些代码块，这种碎片化使得理解和维护复杂组件变得困难，虽然在之前章节讲到过Mixin在一定程度上可以抽离出一些组件中的代码，但始终不是最高效的。

为了能够将同一个逻辑关注点相关代码更好的收集在一起，Vue 3引入了与配置式API相对应的Composition组合式API，将上面的配置式API代码转换成组合式API，

```
import {onMounted,reactive,watch} from 'vue'
export default {
    props: {
      name: String,
    },
    name: 'test',
    components: {},
    setup(props,ctx) {
      console.log(props.name)
      console.log('created')
      const data = reactive({
        a: 1
      })
      watch(
        () => data.a,
        (val, oldVal) => {
          console.log(val)
        }
      )
      onMounted(()=>{
        
      })
      const myMethod = (obj) =>{

      }

      retrun {
          data,
          myMethod
      }
    }
}
```



如上所示可以看到，组合式API的代码逻辑都可以写在setup方法中，这使得逻辑更加集中，更加原子化，从而提示可维护性。

## setup 方法

为了开始使用组合式 API，我们首先需要一个可以实际使用它的地方。在Vue 3的组件中，我们将此位置称为setup方法，如示例代码所示。

```
<div id="app">
  <component-b user="John" />
</div>
const componentB = {
  props: {
    user: {
      type: String,
      required: true
    }
  },
  template:'<div></div>',
  setup(props,context) {
    console.log(props.user) // 打印'John'
    return {} // 这里返回的任何内容都可以用于组件的其余部分
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```



### setup方法参数

setup方法中接收两个参数，第一个参数式props，它和之前讲解组件通信中的props一样，可以接收到父组件传递的数据，同样，如果props是一个动态值，那么它就是响应式的，会随着父组件的改变而更新。

但是，因为props是响应式的，你不能使用ES6解构，它会消除prop的响应性。如果需要解构prop，可以在setup方法中使用toRefs函数来完成此操作，如下代码所示：

```
setup(props,context) {
    const { user } = Vue.toRefs(props)
    console.log(user.value) // 打印'John'
}
```

注意，如果是采用npm来管理的项目，可以采用如下import方式引入toRefs，包括后续的Composition API相关的方法：

`import { toRefs } from 'vue'`

如果user是可选的prop，则传入的props中可能没有user。在这种情况下，需要使用toRef替代它，代码如下：

```
setup(props,context) {
    const { user } = Vue.toRef(props,'user')
    console.log(user.value) // 打印'John'
}
```

setup方法的第二个参数是context对象，context是一个普通的JavaScript对象，它暴露组件的三个属性，分别是attrs，slots，emit，并且由于是普通的JavaScript对象，可以之间采用ES6解构，如示例代码所示。

```
<div id="app">
  <component-b attrone="one" @emitcallback="emitcallback">
    <template v-slot:slotone>
      <span>slot</span>
    </template>
  </component>
</div>
const componentB = {

  template:'<div></div>',
  setup(props, { attrs, slots, emit }) {

    // Attribute (非响应式对象)
    console.log(attrs) // 打印 { attrone: 'one' } 相当于this.$attrs

    // 插槽 (非响应式对象)
    console.log(slots.slotone) // 打印{ slotone: function(){} } 相当于this.$slots

    // 触发事件 (方法)
    console.log(emit) // 可调用emit('emitcallback')相当于this.$emit

  },

}
const vm = Vue.createApp({
  components: {
    'component-b': componentB
  },
  methods:{
    emitcallback(){
      console.log('emitcallback')
    }
  }
}).mount("#app")
```

其中，attrs对象是父组件传递给子组件且不在props中定义的的静态数据，它是非响应式的，相当于在没有使用setup方法时之外调用的this.\$attrs效果。

slots对象主要是父组件传递的插槽内容，注意v-slot:slotone需要配置插槽名字，这样slots才能接收到，它是非响应式的，相当于在没有使用setup方法时之外调用的this.\$slots效果。

emit对象主要用来和父组件通信，相当于在没有使用setup方法时之外调用的this.\$emit效果。

### setup方法结合模板使用

如果setup方法返回一个对象，那么该对象的属性以及传递给setup的props参数中的属性就都可以在模板template中访问到，如示例代码所示。

```
<div id="app">
  <component-b user="John" />
</div>
const componentB = {
  props: {
    user: {
      type: String,
      required: true
    }
  },
  template:'<div>{{user}} {{person.name}}</div>',
  setup(props) {

    const person = Vue.reactive({ name: 'Son' })
    // 暴露给 template
    return {
        person
    }
  },

}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```

注意，props中的数据我们不必在setup中返回，Vue会自动的暴露给模板template中使用。

### setup方法执行时机和getCurrentInstance方法

setup方法在组件的beforeCreate之前执行，此时由于组件还没有实例化，是无法之间像配置式API一样直接使用this.xx访问当前实例的上下文对象的，例如data，computed和methods都没法访问到，所以setup在和其它配置式API一起使用时可能会导致混淆，需要格外注意。

但是，Vue在还是在Composition API中提供了getCurrentInstance方法来访问组件实例的上下文对象，如示例代码所示。

```
Vue.createApp({
  setup() {
    Vue.onMounted(()=>{
      const internalInstance = Vue.getCurrentInstance()
      internalInstance.ctx.add()// 打印'methods add'
    })
  },
  methods:{
    add(){
      console.log('methods add')
    }
  }
}).mount("#app")
```

需要注意的是请不要把它当作在像在配置式API中的this的替代方案来随意使用，另外getCurrentInstance只能在setup或生命周期钩子中调用，并且不建议在业务逻辑中使用该方法，可以一些开发第三方库中使用。

## 响应式类方法

在配置式API中，我们一般将需要有响应式的变量定义在data选项的属性里面，而在Vue 3的Composition API的setup方法里面，由于还无法访问到data属性，但是也可以定义响应式变量，主要用到toRef，toRefs，ref，reactive和一些其他方法，其中有些我们之前代码中已经用到过了，下面就来详细介绍一下他们的用法和区别。

### ref和reactive

1\. ref方法

ref方法用于为数据添加响应式状态，可以支持基本的数据类型，也可以支持复杂的对象数据类型，是Vue 3中推荐的定义响应式数据的方法，也是最基本的响应式方法，需要注意的是：

-   获取数据值的时候需要加.value。

-   ref的本质是原始数据的拷贝，改变简单类型数据的值不会同时改变原始数据。

使用方法如实例代码所示。

```
<div id="app">
  <component-b  />
</div>
const componentB = {
  template:'<div>{{name}}</div>',
  setup(props) {

    // 为基本数据类型添加响应式状态
    const name = Vue.ref('John')

    let obj = {count : 0};

    // 为复杂数据类型添加响应式状态
    const state = Vue.ref(obj)

    console.log(name.value) // 打印John

    console.log(state.value.count)// 打印0

    let newobj = Vue.ref(obj.count)

    // 修改响应式数据不会影响原数据
    newobj.value = 1

    console.log(obj.count)// 打印0

    return {
      name
    }
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```

需要注意的是，改变的这个数据必须是简单数据类型，一个具体的值，这样才不会影响到原始数据，如上面的代码中的obj.count。

2\. reactive方法

ref方法用于为复杂数据添加响应式状态，只支持对象数据类型，需要注意的是：

-   获取数据值的时候不需要加.value。

-   reactive的参数必须是一个对象，包括JSON数据和数组都可以，否则不具有响应式。

-   和ref一样，reactive的本质也是原始数据的拷贝。

ref本质也是reactive，ref(obj)等价于reactive({value:obj})，使用方法如示例代码所示。

```
<div id="app">
  <component-b  />
</div>
const componentB = {
  template:'<div>{{state.count}}</div>',
  setup(props) {

    // 为复杂数据类型添加响应式状态
    const state = Vue.reactive({count : 0})

    console.log(state.count)// 打印0

    return {
      state
    }
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```

reactive和ref都是用来定义响应式数据的。reactive更推荐去定义复杂的数据类型，不能直接解构，ref更推荐定义基本类型。ref可以简单地理解为是对reactive的二次包装，ref定义的数据访问的时候要多一个.value。

### toRef和toRefs

1\. toRef方法

toRef方法我们在之前的setup方法中对props的操作已经使用过了，其第一种使用场景用于为原响应式对象上的属性新建单个响应式ref，从而保持对其源对象属性的响应式连接。接收两个参数：原响应式对象和属性名，返回一个ref数据。例如使用父组件传递的props数据时，要引用props的某个属性且要保持响应式连接时就很有用，其第二种使用场景是，接收两个参数：原普通对象和属性名，此时可以对单个属性添加响应式ref，但是这个响应式ref的改变不会更新界面，需要注意的是：

-   获取数据值的时候需要加.value。

-   toRef后的ref数据不是原始数据的拷贝，而是引用，改变结果数据的值也会同时改变原始数据。

-   对于原始普通数据来说，新增加的单个ref改变，数据会更新，但是界面不会自动更新。

使用方法如实例代码所示。

```
<div id="app">
  <component-b user="John" />
</div>
const componentB = {

  template:'<div>{{statecount.count}}</div>',
  setup(props) {

    const state = Vue.reactive({
      foo: 1,
      bar: 2
    })

    const fooRef = Vue.toRef(state, 'foo')

    fooRef.value++
    console.log(state.foo) // 打印2会影响原始数据

    state.foo++
    console.log(fooRef.value) // 打印3会影响fooRef数据

    const statecount = {// 普通数据
      count: 0,
    }

    const stateRef = Vue.toRef(statecount,'count')

    setTimeout(()=>{
      stateRef.value = 1 // 界面不会更新
      console.log(statecount.count) // 打印1 会影响原始数据
    },1000)

    return {
      statecount,
    }
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```

toRef更多的使用场景是对象添加单个响应式属性，而toRefs则是对完整的响应式对象进行转换。

2\. toRefs方法

toRefs方法将原响应式对象转换为普通对象，其中结果对象的每个属性都是指向原始对象相应
属性的ref，另外一个重要使用场景是可以将reactive方法返回的复杂响应式数据ES6解构，需要注意的是：

-   获取数据值的时候需要加.value。

-   toRefs后的ref数据不是原始数据的拷贝，而是引用，改变结果数据的值也会同时改变原始数据。

-   toRefs只接受响应式对象参数，不可接收普通对象参数，否则会警告。

使用方法如实例代码所示。

```
<div id="app">
  <component-b  />
</div>
const componentB = {
  template:'<div>{{max}},{{count}}</div>',
  setup(props) {

    let obj = {
      count: 0,
      max: 100
    }

    const statecount = Vue.reactive(obj)

    const {count,max} = Vue.toRefs(statecount) // 方便解构

    setTimeout(()=>{

      statecount.max++
      console.log(obj.max) // 打印101 会影响原始数据，同时界面更新
    },1000)

    return {
      count,
      max
    }
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```

目前用的最多的还是ref和reactive来创建响应式对象，使用toRefs来转换成可以方便使用的解构的对象。

### 其他响应式类方法

1.  shallowRef方法和shallowReactive方法triggerRef方法

对于复杂对象而言，ref和reactive都属于递归嵌套监听，也就是数据的每一层都是响应式的，如果数据量比较大，非常消耗性能，shallowRef和shallowReactive则是非递归监听只会监听数据的第一层。如实例代码所示。

```
<div id="app">
  <component-b />
</div>
const componentB = {
  template:'<div>{{shallow.person.name}}</div>',
  setup(props) {

    const shallow = Vue.shallowRef({
      greet: 'Hello, world',
      person:{
        name:'John'
      }
    })

    setTimeout(()=>{
      // 这不会触发更新，因为 ref 是浅层的
      shallow.value.person.name = 'Ted' 
      // 当调用triggerRef强制更新
      Vue.triggerRef(shallow)
    },1000)

    return {shallow}

  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```

triggerRef可以强制触发之前没有被监听到的更新，另外shallowReactive没有类似triggerRef的方法。

2. readonly方法和shallowReadonly和isReadonly

从字面意思上来理解，readonly表示只读可以将响应式对象标识成只读，当尝试修改时则会抛出警告，同样shallowReadonly方法设置第一层只读，isReadonly方法判断是否为只读对象，如示例代码所示。

```
<div id="app">
  <component-b />
</div>
const componentB = {
  template:'<div></div>',
  setup(props) {

    const obj = Vue.readonly({ foo: { bar: 1 } })

    console.log(Vue.isReadonly(obj)) // true

    obj.foo.bar = 2 // 失败警告：Set operation on key "bar" failed: target is readonly.
    const sobj = Vue.shallowReadonly({ foo: { bar: 1 } })

    sobj.foo.bar = 2 // 第二层可以修改

    return {}

  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```



3. isRef方法和isReactive方法和isProxy

isRef方法判断是否是ref方法返回对象，isReactive方法判断是否是reactive方法返回对象，isProxy用于判断是否是reactive方法或者ref方法返回对象。

4. toRaw方法和makeRaw方法

toRaw方法可以返回一个响应式对象的原始普通对象，可用于临时读取数据而无需承担代理访问/跟踪的开销，也可用于写入数据而避免触发更改。

makeRaw方法，可以标记并返回一个对象，使其永远不会成为响应式对象。如示例代码所示。

```
<div id="app">
  <component-b />
</div>
const componentB = {
  template:'<div>{{reactivecobj.bar}}</div>',
  setup(props) {
    const obj = { foo : 1}
    const reactivecobj = Vue.reactive(obj)
    const rawobj = Vue.toRaw(reactivecobj)

    console.log(obj === rawobj) // true

    setTimeout(()=>{
      rawobj.bar = 2 // 不会触发响应式更新
    },1000)

    const foo = {a:1} // foo无法通过reactive成为响应式对象

    console.log(isReactive(reactive(foo))) // false

    return {reactivecobj}
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  }
}).mount("#app")
```



## 监听类方法

之所以叫做监听（侦听）类方法，主要是这章我们介绍的方法其作用类似于配置式API中使用的watch方法，computed方法等等。监听类方法主要使用场景是提供对于响应式数据改变的追踪和影响，并提供一些钩子函数。本章我们主要介绍Composition API 中的computed和watch方法。

### computed方法

在配置式API中，computed是指计算属性，计算属性里可以完成各种复杂的逻辑，包括运算、函数调用等，只要最终返回一个结果就可以。计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值。Composition API 中的computed也是类似的，使用方法如实例代码所示。

```
<div id="app">
  {{info}}
</div>
Vue.createApp({
  setup() {
    const state = Vue.reactive({
      name: "John",
      age: 18
    });
    const info = Vue.computed(() => { // 创建一个计算属性，依赖name和age
      console.log('computed')
      return state.name + ',' + state.age
    });

    info.value = 1 // 抛出警告
setTimeout(()=>{
      state.age = 20 // info动态修改
    },1000)

    setTimeout(()=>{
      state.age = 20 // 第二次走缓存
    },2000)

    return {info}

  }
}).mount("#app")
```



上面代码中，计算属性info依赖state中的age和name，当他们发生变化时，会导致info变化，同时如果每次变化的值相同，则走缓存，不会再次执行computed里的方法，这和配置式API里的computed是一致的。同时info是也是一个不可变的响应式对象，尝试修改会抛出警告。

computed方法也可以接收一个对象，分别配置get和set方法，这样返回的可被修改，对应调用set方法，如下代码所示：

```
const info = Vue.computed({
  get: () => state.name + ',' + state.age,
  set: val => {
    state.age = val - 1
  }
});
info.value = 21
```



12.5.3 watchEffect方法

watchEffect方法可以显示的监听这些变化，参数是一个函数，这个函数里所依赖的响应式对象如果发生变化，都会触发到这个函数，如示例代码所示。

```
<div id="app">
  {{info}}
</div>
Vue.createApp({
  setup() {
    const state = Vue.reactive({
      name: "John",
      age: 18
    });
    const count = Vue.ref(0)
    const countNo = Vue.ref(0)
    const info = Vue.computed(() => { // 创建一个计算属性，依赖name和age
      return state.name + ',' + state.age
    });

    Vue.watchEffect(()=>{
      console.log('watchEffect')
      console.log(info.value) // 依赖了info
      console.log(count.value) // 依赖了count
      
    })
    setTimeout(()=>{
      state.age = 20 // 触发watchEffect
    },1000)
    setTimeout(()=>{
      count.value = 3 // 触发watchEffect
    },2000)
    setTimeout(()=>{
      countNo.value = 5 // 不触发watchEffect
    },3000)

    return {info}
  }
}).mount("#app")
```



当watchEffect在组件的setup方法或生命周期钩子被调用时，侦听器会被链接到该组件的生命周期，并在组件卸载时自动停止在，一些情况下，也可以显式调用返回值以停止侦听，如下代码所示：

```
const stop = watchEffect(() => {
  /* ... */
})

...
stop()
```



### watch方法

在配置式API中，watch是指监听器，Composition
API中同样提供了watch方法，其使用场景和用法是一致的，主要是对响应式对象变化的监听。但是和watchEffect相比有些类似，主要区别是：

-   watch需要侦听特定的数据源，并在回调函数中执行副作用。

-   默认情况下，它也是惰性的，即只有当被侦听的源发生变化时才执行回调。

-   可以访问侦听状态变化前后的值。

watch监听单个数据源，第一个参数可以是返回值的getter函数，也可以是一个响应式对象，第二个参数是触发变化的回调函数。

```
Vue.createApp({
  setup() {
    // 侦听一个 getter
    const state = Vue.reactive({ count: 0 })

    Vue.watch(() => state.count,
      (count, prevCount) => {
        console.log(count, prevCount)
      }
    )
    // 直接侦听ref
    const count = Vue.ref(0)
    Vue.watch(count, (count, prevCount) => {
      console.log(count, prevCount)
    })

    setTimeout(()=>{
      state.count = 1
      count.value = 2
    })

    return {}
  }
}).mount("#app")
```



watch监听多个数据源，第一个参数为多个响应式对象的数组，第二个参数是触发变化的回调函数。

```
Vue.createApp({
  setup() {
    const state = Vue.reactive({ name: 'John' })
    const count = Vue.ref(0)

    Vue.watch([count,state], (count, prevCount) => {
      console.log(count, prevCount)
      // [2,{name:"Ted"}]   [0,{name:"John"}]
    })

    setTimeout(()=>{
      state.name = 'Ted'
      count.value = 2
    })

    return {}
  }
}).mount("#app")
```



watch监听复杂响应式对象时，如果要完全深度监听，需要添加deep:true配置，同时第一个参数需要为一个
getter方法，同时采用深度复制，如示例代码所示。

```
Vue.createApp({
  setup() {
    const state = Vue.reactive({
      name: "John",
      age: 18,
      attributes: { 
        attr: 'efg',
      }
    });

Vue.watch(()=>JSON.parse(JSON.stringify(state)),// 利用深度复制
(currentState, prevState) => {
        console.log(currentState.attributes.attr)// abc
        console.log(prevState.attributes.attr)// efg
      },{ deep: true })

    setTimeout(()=>{
        state.attributes.attr = 'abc'
    },1000)

    return {}
  }
}).mount("#app")
```



需要注意，深度监听需要对原始state进行深度复制并返回，可以采用JSON.parse()，JSON.stringify()的方法进行复制，也可以采用一些第三方库，例如lodash.cloneDeep方法。

## 生命周期类方法

生命周期方法，通常叫做生命周期钩子，在配置式API中我们已经了解了具体的生命周期方法，在Composition API的setup方法里面同样有对应的生命周期方法，他们的对应关系如下所示：

```
beforeCreate -> 使用 setup()

created -> 使用 setup()

beforeMount -> onBeforeMount

mounted -> onMounted

beforeUpdate -> onBeforeUpdate

updated -> onUpdated

beforeUnmount -> onBeforeUnmount

unmounted -> onUnmounted

errorCaptured -> onErrorCaptured

renderTracked -> onRenderTracked

renderTriggered -> onRenderTriggered

activated -> onActivated

deactivated -> onDeactivated
```

由于setup方法在组件的beforeCreate和created之前执行，所以不在提供对应的钩子方法，这些生命周期钩子注册函数只能在setup方法内同步使用，因为它们依赖于内部的全局状态来定位当前活动的实例
(此时正在调用其setup的组件实例)，在没有当前活动实例的情况下，调用它们将会出错。同时，在这些生命周期钩子内同步创建的侦听器和计算属性也会在组件卸载时自动删除，这点和配置式API式一致的。如示例代码所示。

```
const MyComponent = {
  setup() {
    Vue.onMounted(() => {
      console.log('mounted!')
    })
    Vue.onUpdated(() => {
      console.log('updated!')
    })
    Vue.onUnmounted(() => {
      console.log('unmounted!')
    })
  }
}
```



## methods方法

除了上面所讲解的方法之外，还有一类就是使用最多的对应配置式API中的methods类方法了，这类方法主要结合模板template中的一些回调事件使用，如示例代码所示。

```
<div id="app">
  {{count}}
  <button @click="add">点我+1</button>
</div>
Vue.createApp({
  setup() {
    const count = Vue.ref(0)

    const add = ()=>{
      count.value++
    }

    return { count,add }
  }
}).mount("#app")
```



上面代码中，在setup方法中返回了add方法，这样在模板template中就可以进行绑定，当click事件触发时，会进入这个方法。

当结合配置式API使用时，如果在组件的methods中也配置了同名的方法，那么会优先执行setup中定义的，methods中定义的方法将不会执行，如下代码：

```
Vue.createApp({
  setup() {
    const count = Vue.ref(0)

    const add = ()=>{
      count.value++
    }

    return { count,add }
  },
  methods:{
    add(){} // 不会触发
  }
}).mount("#app")
```



同样，在进行组件通信时，如果遇到同名的方法，优先以setup中定义并返回的方法为主，如示例代码所示。

```
<div id="app">
  <component-b @add="add"/>
</div>
const componentB = {
  template:'<div></div>',
  setup(props,{emit}) {
    emit('add') // 通知父组件
  }
}
Vue.createApp({
  components: {
    'component-b': componentB
  },
  setup() {
    const add = ()=>{
      console.log('setup add')
    }
    return { add }
  },
  methods:{
    add(){
      console.log('methods add') // 不会触发
    }
  }
}).mount("#app")
```



上面代码中，当子组件调用emit通知父组件时，会调用父组件setup方法中的add方法，而不会调用methods中定义的。

## Provide / Inject

provide（提供）和inject（注入）也可以在Composition API的setup方法里面使用，来实现跨越层级的组件通信。

provide方法接受两个参数，第一个参数是提供数据的key，第二个参数是值value，可以是对象，方法等等，如示例代码所示。

```
<div id="app">
  <component-b />
</div>
Vue.createApp({
  components: {
    'component-b': componentB
  },
  setup() {
    Vue.provide('location', 'North Pole')
    Vue.provide('geolocation', {
      longitude: 90,
      latitude: 135
    })
  }
}).mount("#app")
```



inject方法接受两个参数，第一个参数是需要注入的数据的key，第二个参数是默认值（可选），如示例代码所示。

```
const componentB = {
  template:'<div>{{userLocation}}</div>',
  setup() {
    const userLocation = Vue.inject('location', 'The Universe')
    const userGeolocation = Vue.inject('geolocation')
    
console.log(userGeolocation)
    return {
      userLocation,
      userGeolocation
    }
  },
}
```



和之前配置式API不同的是，我们可以在provide值时使用ref或reactive方法，来增加provide值和inject值之间的响应性，这样，当provide的数据发生变化时，inject也能实时接收到。如示例代码所示。

```
const componentB = {

  template:'<div>{{userLocation}}</div>',
  setup() {

    const userLocation = Vue.inject('location', 'The Universe')
    const userGeolocation = Vue.inject('geolocation')

    console.log(userGeolocation)

    return {
      userLocation,
      userGeolocation
    }
  },

}
Vue.createApp({
  components: {
    'component-b': componentB
  },
  setup() {
    const location = Vue.ref('North Pole')
    const geolocation = Vue.reactive({
      longitude: 90,
      latitude: 135
    })

    Vue.provide('location', location)
    Vue.provide('geolocation', geolocation)

    setTimeout(()=>{
      location.value = 'China'
    },1000)
  }
}).mount("#app")
```



通常情况下，只允许在provide的组件内去修改响应式的provide数据，但是如果需要在被inject里面修改provide的值，则需要provide一个回调方法，然后在被inject的组件内调用。如示例代码所示。

```
const componentB = {
  template:'<div>{{userLocation}}</div>',
  setup() {
    const userLocation = Vue.inject('location', 'The Universe')
    const updateLocation = Vue.inject('updateLocation')

    setTimeout(()=>{
      updateLocation('China')
    },1000)

    return {
      userLocation,
    }
  },

}
Vue.createApp({
  components: {
    'component-b': componentB
  },
  setup() {
    const location = Vue.ref('North Pole')

    const updateLocation = (v) => {
      location.value = v
    }

    Vue.provide('location', location)
    Vue.provide('updateLocation', updateLocation)

  }
}).mount("#app")
```



最后，如果要确保通过provide传递的数据不会被inject的组件更改，可以使用readonly方法，如下所示：

```
const location = Vue.ref('North Pole')

Vue.provide('location', Vue.readonly(location))
```



## 本章小结

在本章中，讲解了Vue 3引入的Composition API相关知识，主要内容包括：Composition
API基础，setup方法，响应式类方法，监听类方法，生命周期类方法，methods方法，Provide和Inject。其中setup方法是Composition API的重点，所有相关Composition API新提供的接口都需要在setup中来使用，而响应式类方法中ref方法和reactive方法常被用来定义响应式对象，监听类方法中的computed方法和watch方法则提供了监听到响应式对象变化的时机，生命周期类方法基本和配置式API的使用类似，methods方法则需要注意同名的情况，最后Provide和Inject也是在setup方法中实现组件通信的重要工具。

在后续的实战项目中，我们会大量的使用Composition API，所以学好本章内容非常重要。

下面来检验一下读者对本章内容的掌握程度：

-   setup方法中接受的两个参数，他们的作用分别是什么？Vue.js中父子组件如何通信？

-   如果需要定义基本类型数据为响应式，应该调用哪个方法。

-   watch和watchEffect的区别和各自的使用场景是什么？

-   如何在被inject的组件中修改provide的数据。

-   如果在模板中调用setup方法和配置式API中methods定义的方法同名会怎么样？






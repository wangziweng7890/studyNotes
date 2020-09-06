# 一.vue-cli搭建项目

使用步骤

1.安装

```
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

2.创建项目

```
vue create hello-world
```

3.运行项目

```
npm run serve
# OR
yarn serve
```

4.打包项目

```
npm run build
```

# 二.Vuex的使用

## 1.什么是Vuex

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态 。

## 2.vueX的基本使用

[安装](https://vuex.vuejs.org/zh/installation.html) Vuex 之后，让我们来创建一个 store。创建过程直截了当——仅需要提供一个初始 state 对象和一些 mutation： 

```
// 如果在模块化构建系统中，请确保在开头调用了 Vue.use(Vuex)
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

现在，你可以通过 `store.state` 来获取状态对象，以及通过 `store.commit` 方法触发状态变更： 

```
store.commit('increment')
console.log(store.state.count) // -> 1
```

## 3.核心概念

### 2.getter

**（1）基本概念**

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。 

**（2）用法**

Getter 接受 state 作为其第一个参数： 

```
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

**通过属性访问**

Getter 会暴露为 `store.getters` 对象，你可以以属性的形式访问这些值： 

```
this.$store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

注意，getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的。 

### 3.mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数： 

```
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

你不能直接调用一个 mutation handler。这个选项更像是事件注册：“当触发一个类型为 `increment` 的 mutation 时，调用此函数。”要唤醒一个 mutation handler，你需要以相应的 type 调用 **store.commit** 方法： 

```
store.commit('increment')
```

#### 提交载荷（Payload）

你可以向 `store.commit` 传入额外的参数，即 mutation 的 **载荷（payload）**： 

```
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}

store.commit('increment', 10)
```

在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读

#### 对象风格的提交方式

提交 mutation 的另一种方式是直接使用包含 `type` 属性的对象：

```
store.commit({
  type: 'increment',
  amount: 10
})

mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

#### Mutation 必须是同步函数

一条重要的原则就是要记住 **mutation 必须是同步函数**。 

```
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

现在想象，我们正在 debug 一个 app 并且观察 devtool 中的 mutation 日志。每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。 

注：devtools 是vue调试插件

### 4.action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。(更改 Vuex 的 store 中的状态的唯一方法是提交 mutation)
- Action 可以包含任意异步操作。

Actions接受一个context对象参数，该参数具有和store实例相同的属性和方法，所以我们可以通过context.commit()提交mutations中的方法，或者可以通过context.state和context.getters去获取state和getters。 

```
actions : {
    //这里的actionAdd是组件中和所触发的事件相对应的方法
    actionAdd(context){
        context.commit('add')//这里的add是mutations中的方法
    },
    //这里是通过参数解构来简化代码。
    actionReduce( { commit } ){
        commit('reduce')
    }
}
```

- context作为上下文对象，可以简单的理解成store实例，有共享store实例的属性和方法的权利，但是切记：context并不是store实例本身。
- { commit } 这里直接把commit为属性的对象传过来，可以让代码简单明了。

# 插槽

定义：插槽，也就是slot，是组件的一块HTML模板，这块模板显示不显示、以及怎样显示由父组件来决定。

但是插槽显示的位置确由子组件自身决定，slot写在组件template的什么位置，父组件传过来的模板将来就显示在什么位置。

`v-slot` 也有缩写， (`v-slot:`) 替换为字符 `#`。例如 `v-slot:header` 可以被重写为 `#header`： 

## 单个插槽 | 默认插槽 | 匿名插槽

单个插槽可以放置在组件的任意位置，但是就像它的名字一样，一个组件中只能有一个该类插槽。相对应的，具名插槽就可以有很多个，只要名字（name属性）不同就可以了。 

用法：

```
父组件：
<template>
    <div class="father">
        <h3>这里是父组件</h3>
        <child> 
            <div class="tmpl">
              <span>菜单1</span>
              <span>菜单2</span>
              <span>菜单3</span>
            </div>
        </child>
    </div>
</template>
```

```
子组件：
<template>
    <div class="child">
        <h3>这里是子组件</h3>
        <slot></slot>
    </div>
</template>
```

## 具名插槽

匿名插槽没有name属性，所以是匿名插槽，那么，插槽加了name属性，就变成了具名插槽。具名插槽可以在一个组件中出现N次，出现在不同的位置。下面的例子，就是一个有两个具名插槽和单个插槽的组件，这三个插槽被父组件用同一套css样式显示了出来，不同的是内容上略有区别。 

```
父组件：
<template>
  <div class="father">
    <h3>这里是父组件</h3>
    <child>
      <div class="tmpl" slot="up">  //注意使用新写法v-slot:up
        <span>菜单1</span>
        <span>菜单2</span>
        <span>菜单3</span>
      </div>
      <div class="tmpl" slot="down">
        <span>菜单-1</span>
        <span>菜单-2</span>
        <span>菜单-3</span>
      </div>
      <div class="tmpl">
        <span>菜单->1</span>
        <span>菜单->2</span>
        <span>菜单->3</span>
      </div>
    </child>
  </div>
</template>
```

```
子组件：
<template>
  <div class="child">
    // 具名插槽
    <slot name="up"></slot>
    <h3>这里是子组件</h3>
    // 具名插槽
    <slot name="down"></slot>
    // 匿名插槽
    <slot></slot>
  </div>
</template>
```

## 作用域插槽 | 带数据的插槽

最后，就是我们的作用域插槽。这个稍微难理解一点。官方叫它作用域插槽，实际上，对比前面两种插槽，我们可以叫它带数据的插槽。什么意思呢，就是前面两种，都是在组件的template里面写 

```
//父组件：
<template>
  <div class="father">
    <h3>这里是父组件</h3>
    <!--第一次使用：用flex展示数据-->
    <child>
      <template slot-scope="user"> v-slot:aaa='user'
        <div class="tmpl">
          <span v-for="item in user.data">{{item}}</span>
        </div>
      </template>
    </child>
  </div>
</template>
```

```
//子组件：
<template>
  <div class="child">

    <h3>这里是子组件</h3>
    // 作用域插槽
    <slot name='aaa' :data="data"></slot>
  </div>
</template>

 export default {
    data: function(){
      return {
        data: ['zhangsan','lisi','wanwu','zhaoliu','tianqi','xiaoba']
      }
    }
}
```

# **nextTick**

```
this.nextTick(callBack)//数据发生变化，更新后触发
this.$nextTick(callBack)//dom发生变化，更新后触发
```



# Vue.mixin 

混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。 

例:

```
// 定义一个混入对象
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 定义一个使用混入对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"
```

****选项合并****

当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。比如，`data`在内部会进行递归合并，并在发生冲突时以**组件`data`优先**。

```
var mixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
}

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```

同名**钩子函数**将合并为一个数组，因此都将被调用。另外，**混入对象的钩子将在组件自身钩子之前**调用。

```
var mixin = {
  created: function () {
    console.log('混入对象的钩子被调用')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('组件钩子被调用')
  }
})

// => "混入对象的钩子被调用"
// => "组件钩子被调用"
```

值为对象的选项，例如 `methods`、`components` 和 `directives`，将被合并为同一个对象。两个对象键名冲突时，取组件对象的键值对。 

```
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}
var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})
vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "from self"
```

**全局混入**

```
// 为自定义的选项 'myOption' 注入一个处理器。
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

[自定义选项合并策略](https://cn.vuejs.org/v2/guide/mixins.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E9%80%89%E9%A1%B9%E5%90%88%E5%B9%B6%E7%AD%96%E7%95%A5)

略...

# 自定义指令

有的情况下，你仍然需要对普通 DOM 元素进行底层操作，这时候就会用到自定义指令。 

```
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
//局部注册
directives: {
  focus: {
    // 指令的定义
    inserted: function (el) {
      el.focus()
    }
  }
}
```

使用

```
<input v-focus>
```

## [钩子函数](https://cn.vuejs.org/v2/guide/custom-directive.html#%E9%92%A9%E5%AD%90%E5%87%BD%E6%95%B0)

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

## [钩子函数参数](https://cn.vuejs.org/v2/guide/custom-directive.html#%E9%92%A9%E5%AD%90%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0)

指令钩子函数会被传入以下参数： 

- `el`：指令所绑定的元素，可以用来直接操作 DOM 。
- `binding`：一个对象，包含以下属性： 
  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"`中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。
- `vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://cn.vuejs.org/v2/api/#VNode-%E6%8E%A5%E5%8F%A3) 来了解更多详情。
- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

例

```
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
```

```
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

# vue常用组件封装

## (1)vue返回上一页面时回到原先滚动的位置

方法一

思路：因为vue是单页面应用，进入其他页面时会销毁该页面，用keep-alive不让其刷新，具体实现为 

```
  <div id="app">
    <keep-alive>
        <router-view v-if="$route.meta.keepAlive"></router-view>
    </keep-alive>
    <router-view v-if="!$route.meta.keepAlive">   	
    </router-view>      
  </div>
```

把需要这个需求的页面在路由里面配置一下就可以了 

```
{
  path: '/guessContent',
  name: 'guessContent',
  meta: {
    keepAlive : true 
  }
},
```

方法二

beforeEach里把from的scrollY保存到vuex里，然后afterEach里取出to的scrollY，进行跳转，window.scrollTo 

scrollTo方法外面套一个setTimeout，时间0 或者使用$nextTick;

方法三(坑点,页面渲染会比滚动慢,所以达不到滚动效果)

vue官方提供的进阶功能——滚动行为，通过这个功能，可以自定义路由切换时页面如何滚动。 

使用前端路由，当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。 `vue-router` 能做到，而且更好，它让你可以自定义路由切换时页面如何滚动。 

```
const router = new Router({
    mode: "history",
    routes: routes,
    scrollBehavior(to, from, savedPosition) {
        console.log(savedPosition)
        if(savedPosition) {
            return savedPosition		//这里要实现要设定时器,官方坑点
        } else {
            return {
                x: 0,
                y: 0
            }
        }
    }
});
```

注意：该功能只能在history的路由模式下有效，对于某些页面操作可能会影响上一个页面的展示，那么上一个页面就不应该做滚动处理，这个时候就可以根据路由的来源和去向来判断是否需要滚动了。 

## (2)Vue 页面加载数据之前增加 `loading` 动画

### 创建组件

1. 新建 `.vue` 文件： `src -> components -> laoding -> index.vue`
2. 编辑 `index.vue` 文件

```
<template>
<div class="loading"></div>
</template>

<script>
export default {
  name: 'Loading' // 定义的组件名称 使用时写法：loading
}
</script>

<style scoped>
.loading {
  position: fixed;
  left: 0;
  top: 0;
  background: url('~@/assets/img/loading-ball.svg') center center no-repeat #fff;
  width: 100vw;
  height: 100vh;
  z-index: 1000;
}
</style>
```

### 使用组件

通过自定义一个变量 `isLoading` 初始化为 `true` ，在数据请求成功之后将变量改为 `false` ，在 `template` 中通过变量来控制是否显示隐藏，使用 `vue` 自带的 [过渡效果](https://cn.vuejs.org/v2/guide/transitions.html#%E5%8D%95%E5%85%83%E7%B4%A0-%E7%BB%84%E4%BB%B6%E7%9A%84%E8%BF%87%E6%B8%A1) 增加用户体验 

- 需要在全局的 `css` 中加入过渡需要的样式

```
.fade-enter,
.fade-leave-active {
  opacity: 0;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
```

- `.vue` 文件使用代码示例片段

```
<template>
  <div>
    <transition name="fade">
      <loading v-if="isLoading"></loading>
    </transition>
  </div>
</template>
<script>
import Loading from '@/components/loading'

export default{
  components:{ Loading  },
  data(){
    return{
      isLoading: true
    }
  },
  mounted() {
    const me = this
    // 初始化页面数据
    me.loadPageData()
  },
  methods:{
    loadPageData: function() {
      // axios 请求页面数据 .then 中将状态值修改  this.isLoading = false
    },
  }
}
<script>
```

​	

## (3)登陆页判断

采用路由元信息来判断哪个页面需要登陆,如果需要,则在判断是否已经登陆

```
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      children: [
        {
          path: 'bar',
          component: Bar,
          // a meta field
          meta: { requiresAuth: true }	//使用meta标记不需要登陆的页面
        }
      ]
    }
  ]
})
```



```
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
```



# vue-原理解析

## (1)v-mode解析

基本用于input,select,textarea 

```HTML
<input type='text' v-model='mes' />
```

它本质是一个语法糖,上面的代码其实是下面代码的语法糖

```php+HTML
<input v-bind:value='mes' v-on:input="mes=$event.target.value">
```

v-model会自动在子组件里面添加一个mode属性,绑定value元素和一个input事件,如果你想更改这绑定元素和事件,就要显示的在子组件里面声明一个mode属性.

**父组件**

```html
<template>
    <HelloWorld v-model="foo" :checked2="wo"></HelloWorld>
</template>

//上面这段代码可以写成
    <HelloWorld :checked1='foo' @changeE='foo=$event' :checked2="wo"></HelloWorld>
```

**子组件(HelloWorld.vue)**

```html
<template>
  <div>
    <input type="checkbox" @change='newValue($event.target.checked)' :checked='checked1' />
    {{checked1}}=========={{checked2}}
  </div>
</template>
```

```js
<script>
export default {
  props:['checked1','checked2'],
  model:{
    prop:'checked1',  //绑定父元素的属性
    event:'changeE'	//此事件触发时可以更改与prop绑定的父元素的值
  },
  methods: {
    newValue(val){
      this.$emit('changeE',val) //事件名称随便写什么都行
    }
  },
}
</script>
```

# vue-Router

## 动态路由匹配

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 `User` 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。那么，我们可以在 `vue-router` 的路由路径中使用“动态路径参数”(dynamic segment) 来达到这个效果： 

```
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}

像 /user/foo 和 /user/bar 都将映射到相同的路由。
	
const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

## 响应路由参数的变化

注意，当使用路由参数时，例如从 `/user/foo` 导航到 `/user/bar`，**原来的组件实例会被复用**。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。**不过，这也意味着组件的生命周期钩子不会再被调用**。 

复用组件时，想对路由参数的变化作出响应的话，你可以简单地 watch (监测变化) `$route` 对象： 

```
const User = {
  template: '...',
  watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```

或者使用 2.2 中引入的 `beforeRouteUpdate` [导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)： 

```
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next() 不要忘记加next()
  }
}
```

beforeRouteEnter是新进入的一个路由，比如进入/login登录界面，会触发beforeRouteEnter这个钩子；

而beforeRouteUpdate是路由更新时触发，从主页进入登录界面不会触发这个钩子函数，一个父路由下的子路由跳转会触发这个钩子函数。

## 捕获所有路由或 404 Not found 路由

常规参数只会匹配被 `/` 分隔的 URL 片段中的字符。如果想匹配**任意路径**，我们可以使用通配符 (`*`)： 

```
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

当使用*通配符*路由时，请确保路由的顺序是正确的，也就是说含有*通配符*的路由应该放在最后。路由 `{ path: '*' }` 通常用于客户端 404 错误。 

## 匹配优先级

有时候，同一个路径可以匹配多个路由，此时，匹配的优先级就按照路由的定义顺序：谁先定义的，谁的优先级就最高。 

## 编程式的导航

## `router.push(location, onComplete?, onAbort?)`

```
// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

**注意：如果提供了 path，params 会被忽略，上述例子中的 query 并不属于这种情况。取而代之的是下面例子的做法，你需要提供路由的 name 或手写完整的带有参数的 path：** 

```
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

## `router.go(n)`

这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步，类似 `window.history.go(n)`。

## 命名路由

有时候，通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候。你可以在创建 Router 实例的时候，在 `routes` 配置中给某个路由设置名称。 

```
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

```
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

```
router.push({ name: 'user', params: { userId: 123 }})
```



## 命名视图

有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` (侧导航) 和 `main` (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`。 

```
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components`配置 (带上 s)：

```
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

## 导航守卫

### (1)**全局前置守卫**

router.beforeEach((to,from,next)=>{

})

- 一般常用来做登陆页面跳转和是否登陆认证

### (2)全局解析守卫

router.beforeResolve 

### (3)全局后置钩子

这些钩子不会接受 `next` 函数也不会改变导航本身： 

afterEach()方法常用的地方是自动让页面回到顶端

- 如果一个页面较长,滚动某个位置后跳转.这时新加载的路由页面的滚动的位置默认是上一个页面的位置

  ```
  router.afterEach((to, from) => {
    //将滚动条恢复到最顶端
    window.scrollTo(0, 0);
  })
  ```

  



### (4)路由独享的守卫

你可以在路由配置上直接定义 `beforeEnter` 守卫 

### (5)组件内的守卫

最后，你可以在路由组件内直接定义以下路由导航守卫：

- `beforeRouteEnter`
- `beforeRouteUpdate` (2.2 新增)
- `beforeRouteLeave`

```
beforeRouteEnter(to,from,next){
    //在渲染该组件的路由的被确认之前调用
    //不能获取到组件this实例
    //当守卫执行时,组件实例还没调用
}
beforeRouteUpdate(to,from,next){
    //在当前路由改变,但是该组件被复用时调用
    //一般动态路由匹配或者当此组件被重复渲染时
    //
}
beforeRouteLeave(to,from,next){
    //当导航离开此组件时调用,可以访问this实例
    //这个离开守卫通常用来禁止用户在还未保存修改前突然离开。该导航可以通过 next(false) 来取消。
      const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (answer) {
    next()
  } else {
    next(false)
  }
}

```

**注意**

​	beforeRouteEnter支持给next传递回调的唯一守卫

# 动态组件

让多个组件使用同一个挂载点，并动态切换，这就是动态组件。 

```
//用法
//通过使用保留的 <component> 元素，动态地绑定到它的 is 特性，可以实现动态组件
<component :is="currentView"></component> //currentView对应组件名
```

**与缓存相结合**

【`include和``exclude`】

`　　include` 和 `exclude` 属性允许组件有条件地缓存。二者都可以用逗号分隔字符串、正则表达式或一个数组来表示

```
<!-- 逗号分隔字符串 -->
<keep-alive include="a,b">
  <component :is="view"></component>
</keep-alive>
<!-- 正则表达式 (使用 v-bind) -->
<keep-alive :include="/a|b/">
  <component :is="view"></component>
</keep-alive>
<!-- Array (use v-bind) -->
<keep-alive :include="['a', 'b']">
  <component :is="view"></component>
</keep-alive>
```

　　匹配首先检查组件自身的 `name` 选项，如果 `name` 选项不可用，则匹配它的局部注册名称（父组件 `components` 选项的键值）。匿名组件不能被匹配

```
  <keep-alive include="home,archive">
    <component :is="currentView"></component> 
  </keep-alive>
```

# **父向子传值**

**！注意点**

1.props是单向数据流：父组件的数据变化，通过props实时反应在子组件中，反之不然

2.不允许在子组件中直接操作props

3.可以变相操作props

（1）在data中声明局部变量，并用props初始化，弊端：局部变量不随着props更新而更新

（2）在computed中对props值转换后输出 
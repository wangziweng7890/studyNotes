# Vue渲染wather执行流程

每当属性改变时，触发set拦截，在set拦截里，该属性对应的Dep实例去notify 它收集的watcher调用update方法，在update方法里，把wather实例传入queueWathcer进行收集放入queue数组，该数组会对重复放入的watcher进行去重。放入数组之后，在把flushSchedularQueue方法放入nextTick去执行。flushSchedularQueue会拿到queue里的wathcer遍历执行。nextTick是一个将传入的回调进行延迟执行的方法。它会从promise，MutationObserver,setImmediate,setTimeout选则一个延迟方案，优先从左往右选。在将选出的方法放到TimerFunc中进行延迟。nextTick传入的回调会放入一个callbacks的数组里。第一次触发nextTick，会直接触发TimerFunc方法，并将pending置为true,等TimeFunc执行完在把pending置为false。这样做是为了保证一次事件循环或者中，一个wathcer只会执行一次。不然的话，在一个方法里改变了多个属性的值，属性A变了，通知渲染Watcher执行，属性B变了，又通知渲染Watcher执行，效率很不好。

# keep-alive原理

它是一个抽象组件，在created阶段创建 caches 对象来保存 需要缓存组件vnode,在mouted阶段使用$watch 来观察 include和exclude属性，如果发生变化，则对caches里面的缓存的的不满足的vnode进行清楚，在beforedestroy阶段清楚所有caches。它的缓存收集是在render方法中，它首先拿第一个插槽组件，判断是否满足include和exclude，不满足直接返回 vnode本身，不进行缓存。否则拿到vnode的key值，判断是否已加入缓存，没加入则加入，否则将之前缓存的componentInstance赋值给vnode的componentInstance。最后返回vnode

# Vue-router 基本过程

router-link 去改变url的值-》触发hashChange事件-》拿到最新url赋值给响应式数据-》响应式数据通知收集router-view的渲染watcher去重新执行

# Vuex基本实现

- 将vuex构造函数options里的state属性放入响应式数据的$$state里
- 声明一个state属性，用属性赋值器getter进行拦截，返回的其实是响应式数据$$state
- 遍历options里的actions和mutations将原函数包装一层，分别传入 this,payLoad 和 state, payLoad。然后将它们分别赋值到vuex实例的actions和mutions对象。
- 声明commit和dispatch方法接受type,和payload参数，它们会找到`this.mutions[type][playload]`去执行
- 对于getters也遍历包装 ，最后赋值到vue的计算属性上去，然后通过defineProperty对getters进行拦截，通过getter[key]拿到的其实是 this._vm[key]



# diff算法

diff算法就是帮助我们只更新dom的某一块而不是全部更新。

vue的会为每一个组件创建一个vnode对象，当组件发生改变时，会生成一个新的vnode。然后对新老vnode进行比较，发现有不一样的地方就修改到真实dom上。

diff过程就是调用patch函数，一边比较新旧节点，一边给真实dom打补丁的过程。

diff在比较新旧节点的时候，只会同级之间比较，不会跨级比较

patch函数首先会判断两个vnode当前节点是否为一致（通过key,tagName,isComment等来判断），是的话就调用patchVnode，否则的话直接用新vnode生成一个真实dom替换老的dom

pachVnode主要做了这几件事

- 找到oldvnode的真实dom，称为el
- 比较vnode和oldvnode是否指向同一对象，如果相等，直接返回
- 如果他们都有文本且不相同，将el的文本设置为vnode的
- 如果属性不一致，则更新el的属性值为vnode的
- 如果oldValue有子节点，vnode没有，则删除el的子节点
- 如果oldValue没有子节点，vnode有，则将vnode的子节点真实化后添加到el上
- 如果两者都有子节点，那么通过updateChildren来比较子节点

updateChildren主要采用双指针算法，分别为新旧vnode的子节点数组创建start和end两个指针，分别指向数组的头和尾。每次比较都会移动其中一方或双方的指针（头指针向后，尾指针向前移），直到一方的头指针大于尾指针（说明其中一方所有子节点已经全部进行了比较）

​	具体的比较过程主要有五种比较方案，对旧子节点数组和新子节点数组进行头头比较，尾尾，头尾，尾头，还有key值比较。

- 头头比较成功，继续递归patch函数比较当前新老子节点，然后往后移动头指针
- 尾尾比较成功，继续递归patch函数比较当前新老子节点，尾指针前移
- 头尾比较成功，继续递归，然后将头指针指向的旧子节点移动到尾指针指向的旧子节点后面，移动指针
- 尾头比较成功，继续递归，将尾指针指向的旧子节点移动到头指针指向的新子节点前面
- 最后进行key值比较，在进行比较前，我们会为旧子节点数组创建一个 key值和下标index的映射表，然后去查找新头指针指向的子节点的key是否能找到映射,然后将新节点头指针后移。
  - 如果没找到，说明是新节点，则根据该节点创建真实dom插入老头指针节点所指的节点dom之前
  - 如果找到，就将该位置的老节点的dom移动到头指针指向的老节点dom前面，并将该位置的老节点位置置为null,在递归调用patch
- 循环指向完毕后，
  - 如果新节点数组头指针小于等于尾指针，说明，新节点的元素还有多的没插入，于是将它们从左往右依次插入 **新尾指针+1** 指向的节点 的前面
  - 如果旧点头指针小于尾结点，则依次移除多余元素



# 自定义指令 钩子函数

bind:指令绑定到元素时调用

inserted:被绑定节点插入到父元素调用

update:组件vnode更新时调用

componentUpdated：组件及子组件vnode更新完毕后调用

unbind：与元素解绑时调用

钩子函数会传入以下参数

- el:被绑定的元素
- binding: 
  - name:指令名
  - value:指令值
  - oldValue:指令绑定的前一个值（仅在update和componentUpdated中有用）
  - arg：传给指令的参数
  - modifiers:一个包含修饰符的对象
- vnode:
- oldVnode:



# vue 父子组件的生命周期顺序

**1. 加载渲染过程**

父组件的beforeCreate、created、beforeMount --> 所有子组件的beforeCreate、created、beforeMount --> 所有子组件的mounted --> 父组件的mounted

**2. 子组件更新过程**
父beforeUpdate->子beforeUpdate->子updated->父updated

**3.销毁过程**
父beforeDestroy->子beforeDestroy->子destroyed->父destroyed

## **2.导航守卫全解析**

beforeEach-->beforeEnter-->beforeRouterEnter-->beforeResolve-->afterEach -->组件生命周期beforeCreate.....  -->beforeRouterEnter里next的回调
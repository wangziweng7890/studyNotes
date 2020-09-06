# 一.基本语法

## (1)兄弟组件传值

1.首先声明一个公共bus.js

```
import {EventEmitter} from 'events'
const bus = new EventEmitter()
export {bus}
```

2.通过bus传值

```
    sendValue = () => {
    //给bus注册一个事件,并传值
        bus.emit('myevent',{name:'小昭',age:30})
    }
```

3.通过bus接值

```
import {bus} from './bus'
componentWillMount(){
    //在组件挂载前声明监听事件,通过方法接收
        bus.on('myevent',(data)=>{
            this.setState({
                name:data.name,
                age:data.age
            })
        })
    }
```

**注意:this.setState方法是异步的,它的第二个参数将会被作为回调函数执行**



## (2)父子组件传值

### 1.父向子通过props

```
无状态组件（函数式组件）
	props 拿到父组件传递过来的值,通过参数获取
	应用场景：负责展示

有状态组件（类组件）
	props 拿到父组件传递过来的值
	state 自己内部可以拥有值
		<Son initCount={this.state.initCount}/>
	应用场景：当业务比较复杂的时候，使用它，比较合适

当父组件重新给子组件传值时,必须通过钩子函数componentWillReceiveProps(props)
```

**注意：**
	**props的值，只能用，不能改，自己内部的数据state可以改**

### 2.子向父通过回调函数

```
//getValue为父组件方法
    getValue = value => {
        console.log('我是父组件，我现在得到了值',value)
    }
//
<Son initCount={this.state.initCount} callback={this.getValue}/>
```

```
//在子组件中通过props接收方法并调用
        this.props.callback(this.state.count)
```

## (3)条件渲染 & 列表渲染

```
条件渲染:
	注意：
		在JSX中，javascript表达式或是逻辑，需要使用`{}`包裹起来
	React:
		if 
		三目表达式
		&&
列表渲染:
	v-for
	
	写表达式，利用数组的一个方法map
```

## (4)受控组件 & 非受控组件

由于React中没有像 Vue、小程序中有指令，所以获取值的方式不一样，
但是主要分为两种获取值的方式，一种是受控组件的形式，一种是非受控组件的形式

都是针对表单，都是为了获取表单元素的值
	input checkbox radio textarea ...

受控就是受到 state 的控制
	步骤：
		1、定义好模型
		2、在表单元素上写好 **value onChange**
		3、当用户输入了值之后，我们需要把表单元素的值，赋值给state
		4、模型更改之后，视图也会发生相应的变化

非受控组件使用场景
	非受控组件React推荐我们使用ref方式获取dom节点，不推荐直接
操作dom
	获取input的焦点，让它聚焦
	文件上传，获取选中的文件

**受控组件例**

```
用户名:<input value={this.state.username} onChange={this.changeValue} name="username" type="text"/>
```

**非受控组件**

```
<input type="file" name="" ref={this.fileName} onChange={this.change} />
<button onClick={this.upload}>上传文件</button>
//在this.upload方法里通过this.fileName.current获取到非受控组件的
```

## (5)react-router-dom与路由之间传值

**核心概念:**

​	Router

​		BrowserRouter h5 history 模式

​		HashRouter hash模式

​	Link

​		声明式导航

​	push

​		编程式导航 this.props.history.push('路径')

​	Route

​		占位的同时设置路由规则 

​	Switch

​		为了包括重定向和404,一定要写在最后

​	Redirect

​		重定向

​	react中的匹配规则是模糊匹配,从上往下,可匹配多个路由

​	exact 精准匹配

**传参**

传

```
<Link to={{pathname:'/newsdetail',search:'?newsId=1001&name=张三'}}>
```

收

```
const params = new URLSearchParams(props.location.search)
params.get('name')
```

# Redux

**基本概念**

用来进行全局状态管理的一个第三方包，它是跟框架无关，也就是说可以用在 React、Vue、Angualr

**Store**
	仓库，他负责两件事，第一，组件需要值的时候，由它提供给组件去使用。第二，它负责跟reducer打交道，它会把旧的值和此次的action，传递给reducer，reducer做完之后，会把最新的结果返回给store

```js
store中用得最多的三个方法
	store.getState()  组件调用该方法就可以获取到仓库中的值
            
    store.subscribe(()=>{// 监听仓库中值的变化
        this.setState({
           goodsList:store.getState(),
            totalPrice:this.calcTotalPrice(store.getState())
        })
    })

	store.dispatch(action) 触发action，更改仓库中的数据
	action为一个对象
	{
        type:''  //type为调用的行为
        payload	//载荷
	}
```
**reducers**
	干活的，它其实就是一个纯函数,store会把action传入到reducers让它进行处理
		纯函数：没有副作用的函数
		你给一个函数传递固定的输入，永远得到的是固定的输出【结果可控】
		函数里面只能通过参数引用外部的变量

## redux-thunk 解决异步action问题

```js
//步骤
	//1.安装redux-thunk 
	//2.更改store.js中的代码
		const store = createStort(
			reducers,
			compose(
				applyMiddleware(thunk),
				        window.__REDUX_DEVTOOLS_EXTENSION__ && 												window.__REDUX_DEVTOOLS_EXTENSION__()
				)
		)
	//3.可以对store.dispatch(asyncAddGoods(goods))传入一个回调函数执行异步操作
       asyncAddGoods(goods){
       	return (dispatch2)=>{//	dispatch2为store.dispatch()调用时传入
          setTimeout(()=>{
            dispatch2({
              type:'ADD',
              good:goods
            })
          },2000)
        }
      }
```

## react-redux

**为了弥补redux的不足**

```
redux的不足支持【体现在视图的操作上】
	1、我们在组件中要想从仓库中获取值，或是更改仓库中的值
		都必须先导入store

	2、我们redux默认都需要通过 store.subscribe 监听仓库中值的更改
		vuex中提供了getters的方式，获取仓库中变更的值
```

上面这些问题，我们用react-redux就能解决

```
redux为react定制的一个包，解决我们
	store.getState，并且还要配合 store.subscribe
	store.dispatch(action)

它只要在根组件中注入 store，然后组件中通过 react-redux提供的一个函数绑定一样，就可以通过 this.props.xxx 获取数据或是 this.props.xxx 更改数据

注意点：
	react-redux 主要是简化view的代码书写
		每个组件不需要导入 store

	但是我们redux的那些代码 store.js actionCreators actionTypes reducers 还是使用之前的
```

**使用步骤**

```
1.安装包 react-redux
一.根组件
	2.在根组件导入store.js
	3.在根组件导入import {Provider} from 'react-redux'
	4.通过Provider把store注入到需要的组件中的props
		<Provider store={store}><Index /></Provider>
二.被注入组件
	5.在被注入组件中导入	import {connect} from 'react-redux'
		connect的作用是建立仓库和组件的props的对应关系
	6.在被注入组件生命两个函数,用来从仓库中获取值和更改仓库中的值
		1.const mapStateToProps = storeState => {
		//storeState为仓库的值
   		 // 这个地方返回的对象，都将来会挂在到组件的props上面
   			 return {
        			goodsList:storeState,
        			total:calcTotal()
    		}
		}
		2.const mapDispatchToProps = dispatch => {
  			return {
    			addGoods:function(goods){
     		 		// 触发同步添加购物车的action
      				dispatch(addGoods(goods))
    			},
    			asyncAddGoods(goods){
      				// 触发异步添加购物车的action
      				dispatch((dispatch2)=>{
                        setTimeout(()=>{
                            dispatch2({
                                type:'',
                                play:''
                            })
                        },2000)
      				})
    			}
 			 }
		}
	7.导出组件
	/**
 	* 参数1【都是箭头函数】  从仓库中获取值
 	* 参数2【都是箭头函数】  更改仓库中的值
 	* 参数不需要的话要写null
 	*/
		export default connect(null,mapDispatchToProps)(GoodsList);
```

**购物车案例**

reducers.js

```
import {ADD_GOODS,UPDATE_GOODS,DELETE_GOODS} from './actionTypes'
/**
 * previousState 上一次的状态
 * action 此次要出发的动作
 */
const defaultState = JSON.parse(localStorage.getItem('GOODS') || '[]')

export default (previousState = defaultState,action) => {
    console.log(previousState,action)
    switch (action.type) {
        case ADD_GOODS: // 添加商品
            const addGoodsList = JSON.parse(JSON.stringify(previousState))

            const goods = addGoodsList.find(item => item.id === action.goods.id)
            if (goods){ // 判断新添加的对象是否存在，如果存在就把数量相加
                goods.num += action.goods.num
            } else { // 之前不存在，添加到数组中
                addGoodsList.push(action.goods)
            }

            return addGoodsList

        case UPDATE_GOODS:// 修改商品
            const updateGoodsList = JSON.parse(JSON.stringify(previousState))
            
            const updateGoods = updateGoodsList.find(item => item.id === action.goods.id)
            updateGoods.num = action.goods.num
            
            return updateGoodsList
            
        case DELETE_GOODS: //删除商品
            const deleteGoodsList = JSON.parse(JSON.stringify(previousState))

            return deleteGoodsList.filter(item => item.id != action.id)

        default:
            return previousState
    }
}
```

store.js

```
import {createStore} from 'redux'

//导入reducers.js
import reducers from './reducers'

const store = createStore(
    reducers,
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
)

export default store
```



# 生命周期

一.初始化阶段

1. componentWillMount :	只会执行一次  	发送请求
2. `render 初次渲染`
	. `componentDidMount :	只会执行一次		拿到渲染后的dom`

二.运行阶段

​	注意点:不能在运行阶段改变state的值

​	props变了

​		componentWillReceiveProps-->shouldComponentUpdate-->componentWillUpdate

​		-->render-->componentDidUpdate

​	state变了

​		shouldComponentUpdate-->componentWillUpdate-->render-->componentDidUpdate

三.销毁阶段

​	componentWillUnmount 组件即将销毁

# **脚手架生成项目**

```
1.装包
	cnpm i create-react-app -g
2.创建
	create-react-app 项目名称
```

# 租房项目

## 模块分类

1.首页

2.房屋列表

3.地图找房

```
/*  实现地图搜房功能主要是几个步骤：
	1.根据当前位置设置中心点，显示中心点附近房源
	2.设置
	通过搜索地点，将地图移动到中心
    1、添加聚合点marker（利用接口返回数据用marker来显示相应的数量来模拟聚合效果）；
    2、根据鼠标悬停事件来显示隐藏行政区划边界线（百度地图Polygon方法）；
    3、点击marker或鼠标滚轮事件来扩大地图级别显示所有的点（addEventListener(event: String, handler: Function)）；
    4、通过拖拽地图根据视野来加载对应的点，提高加载点位性能（百度地图GeoUtils）；
    5、给具体的点位添加信息窗体事件。
*/
--------------------- 
作者：w冰淇淋 
来源：CSDN 
原文：https://blog.csdn.net/wbx_wlg/article/details/85783774 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

4.咨询
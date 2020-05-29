# React

- 只实现了从state视图的数据单向绑定，未实现双向绑定
- 要实现从页面到state必须手动触发onChange方法，在方法内更新state，state改变了会自动刷新视图

## webpack 4.x

###  下载

`npm install webpack webpack-cli`

- **webpack ：**核心
- **webpack-cli：** 打包命令（4.x后命令被单独放在webpack-cli）

### 配置

- 约定大于配置
  + 指定默认 en [vue.html](../../06.拓展复习/02.webpack+vue/vue.html) try 文件为 /src/index.js
  + 指定默认 output 文件为 /dist/main.js

```javascript
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
// 创建插件实例
const htmlWebpackPlugin = new HtmlWebpackPlugin({
  template: path.join(__dirname, './src/index.html'),// 源文件
  filename: 'index.html' // 创建到内存中到文件
})
module.exports = {
  mode: 'development',// 配置当前环境
  plugins: [// 配置webpack插件
    htmlWebpackPlugin
  ],
  module: { // 配置第三方包
    rules: [
      { test: /\.js|jsx$/, use: 'babel-loader', exclude: /node_modules/ },
      // modules 参数 表示：为普通的样式表启用模块化，每个模块导入的css文件都会作为一个变量return 回来，只针对class和id生效，对标签无效
      // localIndentName:自定义类名
      // 不想被模块化的类名 :global(.title){}
      { test: /\.css$/, use: ['style-loader','css-lodaer?modules&localIndentName=[path][name]-[local]-[hash:5]'] }
    ]
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json'], // 扩展，后缀名可省略不写
    alias: { // 目录别名
      '@': path.join(__dirname, './src')
    }
  }
}
```

## 安装

- **react:** 创建组建，创建虚拟DOM
- **react-dom:** DOM以及虚拟DOM操作相关

`npm install react react-dom -S`

## 基本使用

```javascript
// 1.导入包
import React from 'react'
import ReactDOM from 'react-dom'
// 2.创建虚拟Dom
// const myH1 = React.createElement('组建名',组建属性对象,子节点-包括文本)
const myH1 = React.createElement('h1',{id:'myH1'},'我是一个h1');
const myDiv = React.createElement('div',null,myH1);
// 3.渲染虚拟DOM
ReactDOM.render(myDiv,document.querySelector('#app'))
```

## 使用jsx

- 装包---babel 7.x的配置方法，8.x完全变了

```javascript
npm install babel-core babel-loader babel-plugin-transform-runtime -D
npm install babel-preset-env babel-preset-stage-0 -D
npm install babel-preset-react -D
```

- 配置.babelrc

```json
{
  "presets":["env","stage-0","react"],
  "plugins":["transform-runtime"]
}
```

- 配置webpack.config.js

```javascript
module: {
  rules:[
    {test:/\.js|jsx$/,use:'babel-loader',exclude:/node_modules/}
  ]
}
```

## React组件

### 无状态组件

> 没有 state 的状态，生命周期函数，性能更高

```js
import React from 'react'; // 但是函数必须引入 react 包
export default const Person = ()=>{
	render(){
    return (<div onClick={this.show}>
      <h1 onClick={this.showh1}>h1</h1>
        {/* this传参数第二种 */}
      <h2 onClick={this.showh2.bind(this.param)}>h2</h2>
      <h3 onClick={()=>{this.showh3(param)}}>h3</h3>
    </div>)
  }
	show = ()=>{} 
}
```



### 有状态组件

```jsx
import React from 'react';
export default class Person extends React.Component {
  construcor(props){ // 需要定义state时候才需要constructor
    super(props)
    this.state = {}
    // this传参数第一种
    this.showh1 = this.showh1.bind(this,param)
	}
  
  {/* 指定组件的默认值*/}
  static defaultProps = {
    msg:'default value'
  }

 {/* 指定组件的默认值的类型，v15版本后单独从React包中抽离出来prop-types包*/}
  static propTypes = {
    msg: ReactTypes.number
  }
	render(){
    return (<div onClick={this.show}>
      <h1 onClick={this.showh1}>h1</h1>
        {/* this传参数第二种 */}
      <h2 onClick={this.showh2.bind(this.param)}>h2</h2>
      <h3 onClick={()=>{this.showh3(param)}}>h3</h3>
    </div>)
  }
	{/* 不传值使用这种方式最简单 */}
	show = ()=>{} 
	{/* 传值方法-事件被调用的对象不一定 */}
	{/* 使用  show(){}  ，需要通过bind让函数内部的this指向之前的参数 */}
	{/* this.show.bind(this) 可以在constructor中重新复制函数，也可以直接写在div绑定的onClick中*/}
	
	showh1(param){} 
  showh2(param){} 
	{/* 使用箭头函数保证this的指向 , 调用showh3是全局调用的，即当前组件*/}
  showh3(param){} 
}
```

## 绑定事件的三种方式

- 使用箭头函数没有bind的方式性能高，但是更方便
- 通过对象e获取元素没有ref性能高

```javascript
export default class Person extends React.Component {
  construcor(props){ // 需要定义state时候才需要constructor
    super(props)
    this.state = {}
    // 2.1 this传参数第一种
    this.showh1 = this.showh1.bind(this,param)
	}

	render(){
    return (<div onClick={this.show}>
      <h1 onClick={this.showh1}>h1</h1>
        {/* 2.2 this传参数第二种 */}
      <h2 onClick={this.showh2.bind(this,param)}>h2</h2>
      <h3 onClick={()=>{this.showh3(param)}}>h3</h3>
    </div>)
  }
	{/* 1 不传值使用这种方式最简单，但是如果多次调用，每次调用都会新建一个匿名函数，效率不高 */}
	show = ()=>{} 
	{/* 2 传值方法-事件被调用的对象不一定 */}
	{/* 2.1 使用  show(){}  ，需要通过bind让函数内部的this指向之前的参数 */}
	{/* 2.2 this.show.bind(this) 可以在constructor中重新复制函数，也可以直接写在div绑定的onClick中*/}
	{/*事件对象的第一个参数默认是event*/}
	showh1(event,param){} 
  showh2(event,param){} 
	{/* 2.3 使用箭头函数保证this的指向 , 调用showh3是全局调用的，即当前组件*/}
  showh3(param){} 
}
```



## state

- state 不能直接修改，通过setState来修改
- setState是assign修改对象
- setState是异步的，要得到实时修改的结果，在第二个参数中

```javascript
this.setState({
  msg:'hh'
},function(){
  console.log(this.state.msg)
})
```

## 生命周期

### 创建阶段

- static defaultProps={} 设置默认值
- this.state={} 
- componentWillMount 可以拿到state，props，实例方法
- render（虚拟DOM是在render函数中创建的）
- componentDidMount

### 更新阶段

- componentWillReceiveProps（props发生改变时先触发这个函数，组件第一次被渲染时候不会触发）

```javascript
componentWillReceiveProps(nextProps){
  // 2.this.props.msg获取到的是上一次的props
  console.log(this.props.msg)
  console.log(nextProps.msg)
}
```



- shouldComponentUpdate（state改变直接触发这个函数）(return true or false)

```javascript
shouldComponentUpdate(nextProps,nextState){
  // 1.若返回false则不会继续执行后续生命周期函数render创建DOM，而是直接退回运行中，但是state已经被修改
  // 2.this.state.msg获取到的是上一次的state
  console.log(this.state.msg)
  console.log(nextState.msg)
}
```



- componentWillUpdate(nextProps,nextState)
- render

```javascript
// 此时页面还是旧的virtual DOM
```



- componentDidUpdate(prevProps,prevState)

### 销毁阶段

- componentWillUnmounted

## context组件共享数据--不怎么用

### 父组件暴露数据

```javascript
getChildContext(){
  return {
    msg:'hhhh'
  }
}
```

### 父组件规定数据类型

```javascript
static childContextTypes= {
  msg: ReactTypes.string
}
```

### 子组件检验数据类型

```javascript
static contextTypes = {
  msg: ReactTypes.string
}
```

### 子组件使用共享数据

```javascript
this.context.msg
```

## react-router-dom

- npm install react-router-dom -D

### 导入

```jsx
// HashRouter:路由根容器元素，包含所有路由，一个项目只出现一次，只能包含一个元素
// Route：既是路由规则，又是占位符
// Link：路由链接
// Switch:匹配到路由不再向下匹配
import {HashRouter,Route,Link,Switch} from 'react-router-dom'
class App extends React.component{
  render() {
    return  (
      // 启用路由
  		<HashRouter>
     		<div></div>
    	</HashRouter>
  	)
  }
}
        
// Link：路由链接
// exact 启用精确匹配
<Link to="/home">首页</Link>
<Route path="/home" component={Home} exact></Route>
```

## redux 状态管理

> Redux 支持 React、Angular、Ember、jQuery 甚至纯 JavaScript。
>
> Redux 和 React 之间没有关系。

> flux也是状态管理器，redux借鉴了很多flux的东西，但是在flux上优化并提升了很多方法，flux已经承认redux，所以flux已经处于被淘汰的状态

> 创建一个公共仓储 store 来统一管理组建之间需要公用的数据
>
> redux提供单一数据源
>
> redux开始加载回默认出发一次reducer，来初始化store中的state树 

### 基本流程

- 创建一个用于处理数据 state 更新的 reducer，reducer 是一个描述和处理如何更新数据 state 的纯函数

  - 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。
  - 所有的修改都被集中化处理，且严格按照顺序执行，因此不用担心 race condition 的出现
  - reducer 必须是纯函数，只能依赖自己传入的参数决定，内部不建议使用任何有副作用的操作，如：
    - 修改传入参数；
    - 执行有副作用的操作，如 API 请求和路由跳转；
    - 调用非纯函数，如 `Date.now()` 或 `Math.random()`。
  - redux 提供了一个api就是 combineReducers，用于拆分复杂的 redux

  ```javascript
  import { createStore, combineReducers, applyMiddleware } from 'redux'
  const defaultState = {}
  const reducer1 = function(state = defaultState, action) {
    // 注意：不要直接修改 state，而是传回一个处理后副本
    // 或者使用deepCope: JSON.parse(JSON.stingify(state))
     switch (action.type) {
      case SET_VISIBILITY_FILTER:
        return Object.assign({}, state, {
          visibilityFilter: action.filter
        })
      default:
        return state
    }
  }
  const reducer2 = function(state = defaultState, action) {
    return {
      user: {
        name: 'redux2'
      }
    }
  }
  
  var rootReducer = combineReducers({
    reducer1,
    reducer2
  })
  
  var store = createStore(rootReducer)
  ```

  

- 创建一个公共仓储 store，同时传入 根reducer 函数用于处理数据 state 的更新，store提供三个方法

  > store 必须是唯一的
  >
  > 只有 store 可以改变自己的内容 state

  > 第二个参数是可选的, 用于设置 state 初始状态。这对开发同构应用时非常有用，服务器端 redux 应用的 state 结构可以与客户端保持一致, 那么客户端可以将从网络接收到的服务端 state 直接用于本地数据初始化。
  >
  > ```js
  > let store = createStore(todoApp, window.STATE_FROM_SERVER)
  > ```

  - getState()：获取 store 中的 state
  - dispatch(action)：通过action通知 store 更新 state 的意图
  - subscribe(listener)：注册一个订阅者

  ```javascript
  const store = createStore(reducers,other)
  ```

- 组件获取 store 中的数据 state 使用`store.getState()`

```js
construct(props){
  super(props)
  this.state = store.getState()
}
```



- 更改 store 中的数据 state

  - 1 创建一个用于数据 state 更新的对象 action的对象，来描述需要修改 state 的意图，并不描述怎么更新state

  ```javascript
  const action = {
    type:'需要更改的 state 的 key 的映射', // 除type外，其他字段都可自行定义
    text：'啊啊啊 '
  }
  ```

  - 2 同时通过 `store.subscribe` 注册一个订阅者 listencer，用于当数据 state 更新时候，需要进行的操作，并返回一个注销订阅者的方法 `unsubscribe`

  > 可以在订阅者中更新组件（包括修改组件 state 的值）

  ```javascript
  construct(props){
    super(props)
    this.state = store.getState()
    const unsubscribe = store.subscribe(() => // listencer函数体也可以放在下面定义
    	this.setState(store.state)
  	)
  }
  ```

  - 3 通过 dispatch 通知 store 我来更改数据 state 了，并将 action 传入 store

  ```javascript
  store.dispatch(action);
  ```

  - 4 store 接收到 action 后，进行以下步骤
    1. 调用 reducer 函数，直接将数据 state 的2种状态 previousState, action 交给 根reducer 来处理
    2. 根 reducer 拿到2种状态后开始处理（若有子 reducer 就分发处理），并将处理完后的值作为 newState返回给 store（reducer 只处理数据，并不修改 state）
    3. store 更新自己的数据 state，同时调用订阅者 listener（store.subscribee 传入的 callback）
  - 5 store 的数据 state 发生改变，但组件中的 state 并未改变，被调用的订阅者 listener 来重新更新组件
  - 6 通过 step2 中返回的方法  unsubscribe 注销订阅者

### 项目中的优化技巧

1. action 中的 type 一般都单独抽离：`actionTypes.js`文件中导出常量
   1. 方便维护，复用性强
   2. 这样如果 type 的常量变量名写错，就会直接报错，xx is not defined。原来的方法不会报错，因为直接走 `return state`了
2. action 单独抽离：`createAction.js` 返回 action 工厂函数，并引入 step1 中的文件
3. state一般分为三类：
   1. domainData:服务端获取的数据，例如：用户信息，数据列表
   2. UI state：决定页面显示的状态，例如：受控组件，弹窗
   3. App state：是否loading，路由

### react-redux

> 统一解决redux的监听 subscribe 和取消监听 unsubscribe
>
>  Redux 本身和 React 没有关系，只是数据处理中心，是 React-Redux 让他们联系在一起。`
> `React-redux提供两个方法：connect 和 Provider。

- connext
  - connect连接React组件和Redux store。connect实际上是一个高阶组件函数，返回一个新的已与 Redux store 连接的组件类。
   - 负责接受一个组件，把state里的一些数据放进去，返回一个新的组件；
   - 数据变化的时候，能够通知组件。
   - `mapStateToProps(state,ownProps)`：
      - state: redux 中的 store
     - ownProps: 自己的 props，不能被覆盖了
      - 从Redux状态树中提取需要的部分作为props传递给当前的组件。state 的接收方
      - 返回值为
   - `mapDispatchToProps(dispatch,ownProps)`：
      - dispatch: store.dispatch()
      - ownProps: 自己的 props，不能被覆盖了
      - 将需要绑定的响应事件 dispatch(action) ,作为props传递到组件上。 state 的发送方
      - 返回一个object，内部key: 函数名；value：函数体，内部是一个dispatch(action)
      - contect 函数会把整个返回的 object 挂载（assign）到 props 上面

```javascript
import {content} from 'react-redux'
// 接收方
const mapStateToProps = (state)=>{
  return state
}
//发送方
const mapDispatchToProps = (dispatch)=>{
  // 返回一个object，内部key: 函数名；value：函数体，内部是一个dispatch(action)
  return {
    sendAction(){
      dispatch({
        type:'ADD_ACTION'
      })
    }
  }
}
export default connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)
```

- Provider
  - **Provider**把store放到context里，所有的子元素可以直接取到store，本质上 Provider 就是给 connect 提供 store 用的。
  - 原理：使用React的context，context可以实现跨组件之间的传递。

```jsx
// store 交给 Provider 管理
import store from '@/store'
import {Provider} from 'react-redux'
export default function App(){
  return (
  	<Provider store={store}>
			<div></div>
		</Provider>
  )
}
```



## 中间件 ，其实开启 redux 的 Chrome 插件的参数（other）也是中间件

> 1. 因为 reducer 是纯函数，action 是普通对象，所以增加中间件来扩展功能
> 2. 处理日志
> 3. 崩溃报告
> 4. 异步不建议放入中间件

```js
// applyMiddleware 控制中间件
// compose增强函数，可以加入多个中间件
import {createrStore ,applyMiddleware , compose} from 'redux'
import {reduxThunk} from 'redux-thunk'
const store = createrStore(reducer, applyMiddleware(thunk))
```



### redux-thunk：action处理异步请求

1. 下载包

```js
npm i -S redux-thunk
```

2. 引入并初始化

```js
// applyMiddleware 控制中间件
// compose增强函数，可以加入多个中间件
import {createrStore ,applyMiddleware , compose} from 'redux'
import {reduxThunk} from 'redux-thunk'
const store = createrStore(reducer, applyMiddleware(thunk))
```

### redux-saga：类似 redux-thunk



## 其他

### render函数的return中必须是一个根组件，如果并列关系

```html
<>
<div>
  1
</div>
<div>
  2
</div>
</>
```



### ref：尽量是用ref而不是通过选择器获取元素

```html
<Hello ref="hello">
  我是一个h3
</Hello>
```

```javascript
this.refs.hello.innerHTML;
```

### 修改this指向的方式：bind | call/apply

- bind的返回值：改变了this指向后的函数的拷贝
- bind并不会改变原函数的this，只是改变了拷贝函数的this
- bind 只修改this指向，不立即调用，call/apply不仅修改指向，并立即调用
- bind(this,param,param...) 第一个参数是this，后面为传入函数内部的参数

###  在promise中return另一个promise才能继续 .then

### 高阶组件

- 是一个函数，而不是组件
- 接收一个组件 oldCompt 作为参数，返回一个新组件 newCompt，并将传入的组件在其内部渲染
- 虽然看起来没什么区别，但可以在 newCompt 启动时候做一些启动前的工作

```js
export default (oldCompt,name)=>{
  class newCompt extends Component{
    contructor(props){
      super(props)
    }
    render(){
      return <oldCompt data={name} />
    }
  }
    return newCompt
}
```



















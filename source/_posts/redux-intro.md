title: Redux
date: 2015-11-25 18:29:41
tags: [Redux, React]
categories: React
---

之前接触了一下 React，配合 Flux，单向数据流，让程序的结构看起来很清晰。但是 Flux 需要写很多"重复"的代码，看起来很麻烦很复杂。不过 Flux 的替换方案有很多，例如：[alt](https://github.com/goatslacker/alt)、[reflux](https://github.com/reflux/refluxjs)等，最热门应该是 [Redux](https://github.com/rackt/redux/) 了。

> Redux is a predictable state container for JavaScript apps.

官方介绍 Redux 是 JavaScript 状态容器，提供可预测化的状态管理。可以跟大部分的 View 层框架配合使用，不限于 React。

Redux 只关心如何管理 state，它的顶层暴露的 API 只有 5 个，非常简单。

### API

为了更具体的说明，写了一个 [redux-demo](https://github.com/booxood/redux-demo)，

    commit 5cf4f88adf1c57ca54ef7cd73a6278056db52ebe
    Author: Liucw
    Date:   Wed Nov 25 18:05:26 2015 +0800

        applyMiddleware

    commit 3ecad1d75a4fca5a79a300641db972202c1a47e7
    Author: Liucw
    Date:   Wed Nov 25 16:16:49 2015 +0800

        combineReducers

    commit df0de5b1a9730d286ebf880adc379d7a03638d87
    Author: Liucw
    Date:   Wed Nov 25 16:00:09 2015 +0800

        store

一次提交试验一个 API，可以 checkout 到对应的 commit 上运行程序。

#### createStore

`createStore(reducer, [initialState])`

创建一个 Store，里面存放着 state。一般来说一个应用只有一个 Store。

参数

- reducer：一个函数，接受两个参数（当前的 state 和 action），按 action 处理完后返回一个新的 state
- [initialState]：初始化 state。

返回值

- Store：保存了应用所有 state 的对象。改变 state 的惟一方法是 dispatch action。也可以 subscribe 监听 state 的变化，然后更新 UI。

Store 对象有几个方法

- getState()：返回当前的 state
- dispatch(action)：转发一个 action，会触发相应的 reducer
- subscribe(listener)：添加一个监听器，每当 dispatch action 的时候就会触发 listener
- replaceReducer(nextReducer)：替换 store 当前用来计算 state 的 reducer（感觉用不到...）

可以切到 `df0de5b1a9730d286ebf880adc379d7a03638d87` 运行一下，看下结果

```
$ git checkout df0de5b1a9730d286ebf880adc379d7a03638d87
$ node index.js
s0: []
store.subscribe: [ 'add 1' ]
s1: [ 'add 1' ]
unsubscribe
s2: [ 'add 1', 'add 2' ]
```

#### combineReducers

`combineReducers(reducers)`
combineReducers 可以把一个由多个 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后对这个 reducer 调用 createStore 产生唯一的 Store。

```
import { todos, counter, add } from './reducer'

let counters = combineReducers({
  counter,
  add
})

let reducers = combineReducers({
  todos,
  counters
})

let store = createStore(reducers)
```

定义了三个 Reducer，先将 counter 和 add 进行 combine 产生 counters，再将 counters 和 todos 进行 combine 得到最终的 reducers。执行一下，看下返回的结果

```
$ git checkout 3ecad1d75a4fca5a79a300641db972202c1a47e7
$ node index.js
s0: { todos: [], counters: { counter: 0, add: 0 } }
store.subscribe: { todos: [ 'add 1' ], counters: { counter: 0, add: 0 } }
store.subscribe: { todos: [ 'add 1' ], counters: { counter: 1, add: 0 } }
store.subscribe: { todos: [ 'add 1' ], counters: { counter: 1, add: 1 } }
s1: { todos: [ 'add 1' ], counters: { counter: 1, add: 1 } }
unsubscribe
s2: { todos: [ 'add 1', 'add 2' ],
  counters: { counter: 1, add: 1 } }
```

最后得到的 state 的结构与 combine 时的结构相同。

#### applyMiddleware

applyMiddleware 可以为 store 的 dispatch 包装一些 middleware。多个 middleware 可以被组合到一起使用，形成 middleware 链。其中，每个 middleware 都不需要关心链中它前后的 middleware 的任何信息。

```
// ./action.js
export function thunkAction() {
  return function(dispatch) {
    return new Promise((resolve, reject) => resolve(dispatch({type: 'ADD'})))
  }
}

// ./store.js
import thunk from 'redux-thunk'
import { thunkAction } from './action'

let createStoreWithMiddleware = applyMiddleware(...[thunk, logger])(createStore);
let store = createStoreWithMiddleware(add)

store.dispatch(thunkAction()).then(() => console.log('done:', store.getState()))
```

应用了 `redux-thunk` middleware 后，可以 dispatch thunkAction。
最后的运行结果
```
[2015-11-26 11:02:33][LOG]s0: 0
[2015-11-26 11:02:33][INFO]will dispatch: { type: 'ADD' }
[2015-11-26 11:02:33][LOG]store.subscribe: 1
[2015-11-26 11:02:33][INFO]state after dispatch: 1
[2015-11-26 11:02:33][LOG]s1: 1
[2015-11-26 11:02:33][LOG]done: 1
```

#### bindActionCreators

将 action 与 Store.dispatch 绑定起来。

> 惟一使用 bindActionCreators 的场景是当你需要把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，而且不希望把 Redux store 或 dispatch 传给它。

#### compose

将多个函数从右到左组合在一起。

> compse 做的只是让你不使用深度右括号的情况下来写深度嵌套的函数。不要觉得它很复杂。


### 总结

其实 Redux 提供的功能很简单，就是创造一颗 state 树，定义修改这颗树的规则。

### 参考

http://rackt.org/redux/
http://camsong.github.io/redux-in-chinese/index.html

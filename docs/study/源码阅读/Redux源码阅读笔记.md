## 概述

官方概述：`Redux` 是 `JavaScript` 应用的状态容器，提供可预测的状态管理。

可以让你开发出行为稳定可预测的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试。不仅于此，它还提供超爽的开发体验，比如有一个时间旅行调试器可以编辑后实时预览。

`Redux` 除了和 `React` 一起用外，还支持其它界面库。它体小精悍（只有2kB，包括依赖），却有很强大的插件扩展生态。

我将其`fork`了一份，由于主分支上`src`文件夹的`commit`显示`Port TS type fixes from the 4.x branch`，主分支下的`src`文件为`.ts`，因此我切换到了`4.x`分支，`commit id`为[795a11c](https://github.com/time0verflow/redux/commit/795a11c8233963bbfc37c13fde1b44d50159a8b8)。



## 使用

`redux`向外暴露了5个接口，分别为

+ `createStore`,  创建一个 `Redux store` 来以存放应用中所有的 `state`。应用中应有且仅有一个 `store`。

+ `combineReducers`,   合并reducer

+ `bindActionCreators`,作用是将单个或多个ActionCreator转化为dispatch(action)的函数集合形式。

开发者不用再手动`dispatch(actionCreator(type))`，而是可以直接调用方法

+ `applyMiddleware`,      添加中间件,实现对于`redux`的扩展

+ `compose` 从右到左来组合多个函数。这是函数式编程中的方法，为了方便，被放到了 Redux 里。当需要把多个 store 增强器 依次执行的时候，需要用到它。

我的理解：redux的工作流程大概是`dispatch(action)=>reducer=>new state=>new UI`

## 目录结构

只关注`src`下的文件，目录结构非常简单，主目录下是暴露的5个接口和入口文件`index.js`,`utils`是相关的工具函数

```js
src
├─ applyMiddleware.js
├─ bindActionCreators.js
├─ combineReducers.js
├─ compose.js
├─ createStore.js
├─ index.js    
└─ utils
       ├─ actionTypes.js
       ├─ formatProdErrorMessage.js
       ├─ isPlainObject.js
       ├─ kindOf.js
       ├─ symbol-observable.js
       └─ warning.js
```

## index.js

```js
import createStore from './createStore'
import combineReducers from './combineReducers'
import bindActionCreators from './bindActionCreators'
import applyMiddleware from './applyMiddleware'
import compose from './compose'
import warning from './utils/warning'
import __DO_NOT_USE__ActionTypes from './utils/actionTypes'

/*
 * This is a dummy function to check if the function name has been altered by minification.
 * If the function has been minified and NODE_ENV !== 'production', warn the user.
 */
function isCrushed() {}

//isCrushed.name!=='isCrushed'用来判断是否压缩过
//不是production环境且压缩了,给出warning
if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning(
    'You are currently using minified code outside of NODE_ENV === "production". ' +
      'This means that you are running a slower development build of Redux. ' +
      'You can use loose-envify (https://github.com/zertosh/loose-envify) for browserify ' +
      'or setting mode to production in webpack (https://webpack.js.org/concepts/mode/) ' +
      'to ensure you have the correct code for your production build.'
  )
}

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes,
}
```

## createStore.js

这是`redux`的核心部分,`redux`源码每部分的注释都非常清楚

大致用法为

```js
const store=createStore(rootReducer,initState,applyMiddleWare(thunk));
```

这是源码对于函数签名的介绍

```js
/**
 * Creates a Redux store that holds the state tree.
 * The only way to change the data in the store is to call `dispatch()` on it.
 *
 * There should only be a single store in your app. To specify how different
 * parts of the state tree respond to actions, you may combine several reducers
 * into a single reducer function by using `combineReducers`.
 * 创造一个Redux store接收state tree
 * 改变容器中data的唯一方法是调用dispatch()
 * 
 * app中应该只有唯一store，为了区分不同部分的state tree来响应action,你可能要用 combineReducers combine几个
 * reducesrs 放入一个单独的reducer中
 * 
 * @param {Function} reducer A function that returns the next state tree, given
 * the current state tree and the action to handle.
 * 一个函数返回新的state tree,传入当前state tree和 action
 * 
 *
 * @param {any} [preloadedState] The initial state. You may optionally specify it
 * to hydrate the state from the server in universal apps, or to restore a
 * previously serialized user session.
 * If you use `combineReducers` to produce the root reducer function, this must be
 * an object with the same shape as `combineReducers` keys.
 * 
 * @param {Function} [enhancer] The store enhancer. You may optionally specify it
 * to enhance the store with third-party capabilities such as middleware,
 * time travel, persistence, etc. The only store enhancer that ships with Redux
 * is `applyMiddleware()`.
 *
 * @returns {Store} A Redux store that lets you read the state, dispatch actions
 * and subscribe to changes.
 */
export default function createStore(reducer, preloadedState, enhancer) {...}
```

首先先定义了参数的规范

```js
  //如果preloadedState和enhancer都是函数，不支持 
  if (
    (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
    (typeof enhancer === 'function' && typeof arguments[3] === 'function')
  ) {
    throw new Error(
      'It looks like you are passing several store enhancers to ' +
        'createStore(). This is not supported. Instead, compose them ' +
        'together to a single function. See https://redux.js.org/tutorials/fundamentals/part-4-store#creating-a-store-with-enhancers for an example.'
    )
  }

  //preloadedState为function,enhancer为undefined的时候，说明没有initState，有middleware
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  //enhancer存在，必为function
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error(
        `Expected the enhancer to be a function. Instead, received: '${kindOf(
          enhancer
        )}'`
      )
    }

    /**
     * 传入符合参数类型的参数，enhancer这个函数执行返回了一个加强版的createStore函数
     */
    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error(
      `Expected the root reducer to be a function. Instead, received: '${kindOf(
        reducer
      )}'`
    )
  }
```

定义的一些变量

```js
let currentReducer = reducer  //临时reducer
let currentState = preloadedState  //临时initState
let currentListeners = []  //监听队列
let nextListeners = currentListeners //监听队列的浅拷贝
let isDispatching = false  //是否正在执行dispatch
```


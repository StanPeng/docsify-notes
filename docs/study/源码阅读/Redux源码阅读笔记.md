## 概述

官方概述：`Redux` 是 `JavaScript` 应用的状态容器，提供可预测的状态管理。

可以让你开发出行为稳定可预测的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试。不仅于此，它还提供超爽的开发体验，比如有一个时间旅行调试器可以编辑后实时预览。

`Redux` 除了和 `React` 一起用外，还支持其它界面库。它体小精悍（只有2kB，包括依赖），却有很强大的插件扩展生态。

我将其`fork`了一份，由于主分支上`src`文件夹的`commit`显示`Port TS type fixes from the 4.x branch`，主分支下的`src`文件为`.ts`，因此我切换到了`4.x`分支，`commit id`为[795a11c](https://github.com/time0verflow/redux/commit/795a11c8233963bbfc37c13fde1b44d50159a8b8)。



## 使用

`redux`向外暴露了5个接口，分别为

+ `createStore`,  创建一个 `Redux store` 来以存放应用中所有的 `state`。应用中应有且仅有一个 `store`。

+ `combineReducers`,   合并`reducer`

+ `bindActionCreators`,作用是将单个或多个`ActionCreator`转化为`dispatch(action)`的函数集合形式。

开发者不用再手动`dispatch(actionCreator(type))`，而是可以直接调用方法

+ `applyMiddleware`,      添加中间件,实现对于`redux`的扩展

+ `compose` 从右到左来组合多个函数。这是函数式编程中的方法，为了方便，被放到了 Redux 里。当需要把多个 store 增强器 依次执行的时候，需要用到它。

我的理解：`redux`的工作流程大概是`dispatch(action)=>reducer=>new state=>new UI`

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

生成的`store`也有几个方法

- `dispatch`:派发动作，把`subscribe`收集的函数依次遍历执行
- `subscribe`:订阅收集函数存在数组中，等待触发`dispatch`依次执行，返回一个取消订阅的函数，可以取消监听
- `getState`:获取存在`createStore`函数内部闭包的对象

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

`createStore`的结构大概是这样的

```js
export default function createStore(reducer, preloadedState, enhancer) {
    let currentReducer = reducer  //临时reducer
    let currentState = preloadedState  //临时initState
    let currentListeners = []  //监听队列
    let nextListeners = currentListeners //监听队列的赋值
    let isDispatching = false  //是否正在执行dispatch
    
    /**监听队列浅拷贝**/
    function ensureCanMutateNextListeners() {
        if (nextListeners === currentListeners) {
         	nextListeners = currentListeners.slice()
    	}
  	}
    
    function getState() {return currentState}
    function subscribe(listener){}
    function replaceReducer(nextReducer){}
    function observable(){}
    
    dispatch({ type: ActionTypes.INIT })
    return {
        dispatch,
        subscribe,
        getState,
        replaceReducer,
        [$$observable]: observable,
  	}
}
```

### store.dispatch(action)

```js
  function dispatch(action) {
    //判断action是否是单纯对象([[Prototype]]是否指向Object.prototype)
    if (!isPlainObject(action)) {
      throw new Error(
        `Actions must be plain objects. Instead, the actual type was: '${kindOf(
          action
        )}'. You may need to add middleware to your store setup to handle dispatching other values, such as 'redux-thunk' to handle dispatching functions. See https://redux.js.org/tutorials/fundamentals/part-4-store#middleware and https://redux.js.org/tutorials/fundamentals/part-6-async-logic#using-the-redux-thunk-middleware for examples.`
      )
    }

    //判断action.type是否存在，否则报错
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. You may have misspelled an action type string constant.'
      )
    }

    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
       //将状态更改为正在派发
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      //调用完后置为false
      isDispatching = false
    }

    //将收集的函数依次拿出来调用
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    //返回最终的action
    return action
  }
```

### store.getState()

```js
  function getState() {
    //判断正在dispatch中，则报错
    if (isDispatching) {
      throw new Error(
        'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
      )
    }

    return currentState
  }
```

### store.subscribe(listen)

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

显然，只要把 View 的更新函数（对于 React 项目，就是组件的`render`方法或`setState`方法）放入`listen`，就会实现 View 的自动渲染。

`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。

```js
  function subscribe(listener) {
    //参数校验，listener不是函数则报错
    if (typeof listener !== 'function') {
      throw new Error(
        `Expected the listener to be a function. Instead, received: '${kindOf(
          listener
        )}'`
      )
    }

    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api/store#subscribelistener for more details.'
      )
    }

    //是否订阅更改为true
    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    //返回取消订阅的函数
    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api/store#subscribelistener for more details.'
        )
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
      currentListeners = null
    }
  }
```

## applyMiddleware.js

关于中间件为什么这么设计我见过讲的比较好的文章是[middleware和applyMiddleware](https://zhuanlan.zhihu.com/p/57407920)

中间件的具体使用方法为

```js
const store = createStore(cRducer, applyMiddleware(thunk,logger))
```

在`createStore.js`的源码中，当中间件存在时，将会这么使用

```js
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
```

`applyMiddleware`这一方法大致为

```js
export default function applyMiddleware(...middlewares) {
      return (createStore) => (...args) => {
    		const store = createStore(...args)
			//省略了一些代码
                return {
                  ...store,
                  dispatch,
                }
       }
}
```

可以看出`applyMiddleWare`对`createStore`进行了一些加强，同时对于`store`的修改体现在`store.dispatch`上，因此`middleware`本质上是对`dispatch`的修改其他地方并没有发生改变

需要思考的是如何对多个中间价实现级联

完整代码为

```js
/** 
 * @param {...Function} middlewares The middleware chain to be applied. 可以被应用的中间件
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  return (createStore) => (...args) => {
    //先建立一个store
    const store = createStore(...args)
    let dispatch = store.dispatch

    //暴露出store中的getState和dispatch
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args),
    }
    
    //这里相当于给middleware剥了一层皮,接下来只需要传入dispatch就可以了
    const chain = middlewares.map((middleware) => middleware(middlewareAPI))
    //依次调用所有中间件修改dispatch，相当于将最原始的store.dispatch作为next给第一个middleware，
    //并返回一个新的dispatch作为next传递给下一个中间件，将最后一个中间件返回的作为最终的dispatch
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch,
    }
  }
}
```

为了理解这一过程，查看`redux-thunk`中间件的实现

`redux-thunk`主要的功能就是可以让我们`dispatch`一个函数，而不只是普通的 `Object`

由于`redux-thunk`的代码量非常少，直接把它的代码贴上来看一下。这里是`v2.3.0`的代码：

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

它的核心代码其实只有两行，就是判断每个经过它的`action`：如果是`function`类型，就调用这个`function`（并传入 `dispatch `和 `getState` 及 extraArgument 为参数），而不是任由让它到达 `reducer`，因为` reducer` 是个纯函数，`Redux` 规定到达 `reducer` 的 `action` 必须是一个 `plain object` 类型。`next`指代的是上一个修改后的`store.dispatch`函数

## compose.js

这是将多个函数组合执行变成一个新的函数，将上一个函数的返回结果作为下一个函数的参数

```js
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return (arg) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

根据源码中的注释部分，**compose(f, g, h) is identical to doing \* (...args) => f(g(h(...args)))**

做一个实验

```js
const multi10(x){
  return x*10
}

const add10(x){
    return x+10
}

const minus2(x){
    return x-2;
}

const calc=compose(multi10,add10,minus2)

console.log(calc(2))  //28

const calc2=compose(add10,minus2,multi10)

console.log(calc2(2))  //100
```

## combineReducers.js

这个方法的作用就是合并多个`reducer`为一个函数`combination`，可以看成是一个总的`reducer`

函数大致为

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})
```

最终会等价为

```js
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

先进行一遍过滤，把非function的reducer过滤掉得到finalReducer

```js
 const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)
```

最后返回的是一个新的`reducer`

```js
  return function combination(state = {}, action) {
    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      //获取state[key],传入reducer的key===store的key
      const previousStateForKey = state[key]
      //用reducer[key]来处理，得到下一个状态
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const actionType = action && action.type
        throw new Error(
          `When called with an action of type ${
            actionType ? `"${String(actionType)}"` : '(unknown type)'
          }, the slice reducer for key "${key}" returned undefined. ` +
            `To ignore an action, you must explicitly return the previous state. ` +
            `If you want this reducer to hold no value, you can return null instead of undefined.`
        )
      }
      //根据key更新store的值
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
     
     //如果整个循环都没有被更新过，返回state
    return hasChanged ? nextState : state
  }      
```

## bindActionCreators.js

`bindActionCreators` 也是 `Redux` 的一个方法，它接收一个 `actionCreator` 和 `dispatch`，并返回一系列方法，调用对应的方法能帮我们自动 `dispatch` 对应的`action`,这个方法主要用于其他库，如`react-redux`

关于其使用可以参考[bindActionCreators](https://zhuanlan.zhihu.com/p/40291118)

当`actionCreator`是一个单独的函数时

```js
function bindActionCreator(actionCreator, dispatch) {
  return function () {
    return dispatch(actionCreator.apply(this, arguments))
  }
}
```

返回一个函数，当调用返回的函数时将自动`dispatch`相应的`action`

```js
function bindActionCreators(actionCreators, dispatch) {
  const boundActionCreators = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```

单`actionCreator`是一个对象时,批量生成`actionCreator`为函数时的状态，即{key:function}键值对类型


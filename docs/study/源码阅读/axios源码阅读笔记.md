## 概述

### Axios 是什么?

Axios 是一个基于 *[promise](https://javascript.info/promise-basics)* 网络请求库，作用于[`node.js`](https://nodejs.org/) 和浏览器中。 它是 *[isomorphic](https://www.lullabot.com/articles/what-is-an-isomorphic-application)* 的(即同一套代码可以运行在浏览器和node.js中)。在服务端它使用原生 node.js `http` 模块, 而在客户端 (浏览端) 则使用 XMLHttpRequests。

### 特性

- 从浏览器创建 [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- 从 node.js 创建 [http](http://nodejs.org/api/http.html) 请求
- 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- 拦截请求和响应
- 转换请求和响应数据
- 取消请求
- 自动转换JSON数据
- 客户端支持防御[XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

[官网](https://axios-http.com/zh/)有详细的使用说明

我选择的版本为0.21.1

 `commit Id`为[`a64050a`](https://github.com/axios/axios/commit/a64050a6cfbcc708a55a7dc8030d85b1c78cdf38)

该版本最后更新时间为December 21, 2020

## 目录结构

```txt
📦lib
 ┣ 📂adapters   //适配器
 ┃ ┣ 📜http.js  //axios中node端使用的请求函数
 ┃ ┣ 📜README.md
 ┃ ┗ 📜xhr.js  //axios中浏览器使用的请求函数
 ┣ 📂cancel
 ┃ ┣ 📜Cancel.js  //定义了取消请求返回的信息结构
 ┃ ┣ 📜CancelToken.js  //定义了用于取消请求的主要方法
 ┃ ┗ 📜isCancel.js   //判断是否取消请求的信息
 ┣ 📂core
 ┃ ┣ 📜Axios.js   //Axios类
 ┃ ┣ 📜buildFullPath.js
 ┃ ┣ 📜createError.js   //生成指定的error
 ┃ ┣ 📜dispatchRequest.js  //发起请求的地方
 ┃ ┣ 📜enhanceError.js    //指定error对象的toJSON方法
 ┃ ┣ 📜InterceptorManager.js  //InterceptorManager类，拦截器类
 ┃ ┣ 📜mergeConfig.js   //合并配置项
 ┃ ┣ 📜README.md
 ┃ ┣ 📜settle.js  //根据请求状态，处理Promise
 ┃ ┗ 📜transformData.js  //使用default.js中transformRequest和transformResponse对响应以及请求进行格式化
 ┣ 📂helpers
 ┃ ┣ 📜bind.js // 工具函数
 ┃ ┣ 📜buildURL.js  //将params参数
 ┃ ┣ 📜combineURLs.js  //组合baseurl
 ┃ ┣ 📜cookies.js  //封装了读取，写入，删除cookies的方法
 ┃ ┣ 📜deprecatedMethod.js
 ┃ ┣ 📜isAbsoluteURL.js //判断是否为绝对路径（指的://或//开头的为绝对路径）
 ┃ ┣ 📜isAxiosError.js
 ┃ ┣ 📜isURLSameOrigin.js // 检测当前的url与请求的url是否同源
 ┃ ┣ 📜normalizeHeaderName.js  // 对对象属性名的进行格式化，删除，新建符合大小写规范的属性
 ┃ ┣ 📜parseHeaders.js // 将getAllResponseHeaders返回的header信息转化为对象
 ┃ ┣ 📜README.md
 ┃ ┣ 📜spread.js
 ┃ ┗ 📜validator.js
 ┣ 📜axios.js
 ┣ 📜defaults.js // axios中默认配置
 ┗ 📜utils.js // 一些工具方法
```

## 运作流程

![axios.drawio](C:\Users\peng8\Downloads\axios.drawio.png)

## 发起请求

### 发起请求的方式

```js
//方式一  axios(url[, config])
// Send a GET request (default method)
axios('/user/12345');

//方式二  axios(config)
// Send a POST request
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});

// GET request for remote image in node.js
axios({
  method: 'get',
  url: 'http://bit.ly/2mTM3nY',
  responseType: 'stream'
})
  .then(function (response) {
    response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
  });

//方式三 为所有支持的请求方法提供了别名
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])

//方式四 创建实例
const instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});

//The available instance methods are listed below. The specified config will be merged with the instance config
axios#request(config)
axios#get(url[, config])
axios#delete(url[, config])
axios#head(url[, config])
axios#options(url[, config])
axios#post(url[, data[, config]])
axios#put(url[, data[, config]])
axios#patch(url[, data[, config]])
axios#getUri([config])
```

### 源码分析

入口文件,`lib`目录下的`axios.js`

```js
'use strict';

var utils = require('./utils');
var bind = require('./helpers/bind');
var Axios = require('./core/Axios');
var mergeConfig = require('./core/mergeConfig');
var defaults = require('./defaults');

/**
 * Create an instance of Axios
 *
 * @param {Object} defaultConfig The default config for the instance
 * @return {Axios} A new instance of Axios
 */
function createInstance(defaultConfig) {
  // 创建axios实例
  var context = new Axios(defaultConfig);
  // 将instance指向Axios.prototype.request方法
  // bind函数是工具类函数，用于改变this指向，
  var instance = bind(Axios.prototype.request, context);
  // 把Axios.prototype上的方法扩展到instance上，指定上下文是context
  utils.extend(instance, Axios.prototype, context);

  // 把context上的方法扩展到instance上
  utils.extend(instance, context);

  // 导出instance对象
  return instance;
}

// Create the default instance to be exported
/**
 * 当调用axios()时,实际上是执行了createInstance返回的
 * 一个指向Axios.prototype.request的方法
 */
var axios = createInstance(defaults);


// 下面都是为 axios 实例化的对象增加不同的方法。
// Expose Axios class to allow class inheritance
axios.Axios = Axios;

// 添加create方法,返回createInstance函数，参数为自定义配置+默认配置
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// Expose Cancel & CancelToken
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');

// Expose all/spread
axios.all = function all(promises) {
  return Promise.all(promises);
};
axios.spread = require('./helpers/spread');

// Expose isAxiosError
axios.isAxiosError = require('./helpers/isAxiosError');

module.exports = axios;

// Allow use of default import syntax in TypeScript
module.exports.default = axios;
```

由于调用`axios()`本质上是调用`Axios.prototype.request`方法，所以接下来看`Axios.prototype.request`方法

```js
Axios.prototype.request = function request(config) {
  /*eslint no-param-reassign:0*/
  // Allow for axios('example/url'[, config]) a la fetch API
  /**
   * 第一种调用方式 axios(url[, config]),判断参数是否为字符串，
   * 发起一个 GET 请求 (默认请求方式)
   * axios('/user/12345');
   */
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    /*
      第二种调用方式，axios(config)
      发起一个post请求
      axios({
        method: 'post',
        url: '/user/12345',
        data: {
          firstName: 'Fred',
          lastName: 'Flintstone'
        }
      });
     */
    config = config || {};
  }
  ...
}
  
// 方式三 && 方式四
// 遍历设置别名

// Provide aliases for supported request methods
utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, config) {
    return this.request(mergeConfig(config || {}, {
      method: method,
      url: url,
      data: (config || {}).data
    }));
  };
});

utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, data, config) {
    return this.request(mergeConfig(config || {}, {
      method: method,
      url: url,
      data: data
    }));
  };
});
```

这四种调用方式本质上都是调用`Axios.prototype.request`

## 请求响应拦截器

### 使用方法

[拦截器使用方法](https://axios-http.com/zh/docs/interceptors)

在请求或响应被 then 或 catch 处理前拦截它们。

```js
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```

如果你稍后需要移除拦截器，可以这样：

```js
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

可以给自定义的 axios 实例添加拦截器。

```js
const instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

### 源码分析

```js
// lib/core/Axios
// 每个Axios实例上都有interceptors属性，该属性有request、response属性
// 都是InterceptorManager的实例来管理拦截器
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

// 对InterceptorManager构造函数的分析


function InterceptorManager() {
  this.handlers = [];  // 拦截器
}

/**
 * Add a new interceptor to the stack
 *
 * @param {Function} fulfilled The function to handle `then` for a `Promise`
 * @param {Function} rejected The function to handle `reject` for a `Promise`
 *
 * @return {Number} An ID used to remove interceptor later
 */
// 往拦截器中加入拦截方法
InterceptorManager.prototype.use = function use(fulfilled, rejected, options) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected,
    synchronous: options ? options.synchronous : false,
    runWhen: options ? options.runWhen : null
  });
  return this.handlers.length - 1;
};

/**
 * Iterate over all the registered interceptors
 *
 * This method is particularly useful for skipping over any
 * interceptors that may have become `null` calling `eject`.
 *
 * @param {Function} fn The function to call for each interceptor
 */
// 依次对handles中的拦截器对象执行fn
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};

```

  ![img](data:image/png;base64,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mTMpLS3lxRdfZNy4cTjnePHFF8nlcowfP544jikrKyNJEgqFAh/4wAdYv3495eXlpGlKkiT8+c9/Zv369RxwwAH06dMH7z0NDQ2UlpbSp08f4jhm6tSpVFdX8+yzzzJ9+nT69u3LjTfeyLJly0jTlHnz5vHAAw/Qu3dvrrnmGg466CCmTJnC3XffzX333UdTUxNdXV00NTVx8803UygUyCxZsoTOzk6ampqQxDPPPMMpp5zC2LFj6devH3vvvTcrVqxgzZo1TJgwAe89paWl7L333vzlL39h2bJlDBkyhNGjR5MpFotkil1FXnjhBRYvXkxZWRkzZ86kqqoKSRSLRZ566ileeukl2tvaGTlqJJMnTyafz9Pe3s5TTz3FsGHDWLduHc899xyFQoGZM2dSWVmJc44kSXjuuedYtmwZ/fr1I5/PI4mMJMwMw0iSBOcc69av4/HHH6etrY1hw4YxduxYcrkckmhqauLpp59m6dKl9Krtxd777M3AgQOJoohMa2srN9xwA8OGDWP06NFUVFRgZpgZGUkEOx9J9ERMEARBEAQ9csstt/DQQw/xyiuv8MUvfpGamhpuuOEGkiTBOUdjYyOZP/zhD0giiiLa2trYf//9GT58OIsXL+aTn/wk1VXVICgpKWHBggWMHz+evffem1deeYVLL72Ugw8+mH79+pE58MADGTZsGIsWLWL//fdn+PDhLFu2jNraWiIXkaQJhx12GLlcDjNDElEUceCBBzJ58mT+4z/+g0MOOYRnn32WiooK9ttvP5xz3HjjjSxevJjzzjuPF198kd133526ujra29tpbm7myCOPRBKSWLFiBddccw1z5sxh9nGzaWlt4Wtf+xpHHHEENTU1NDc3I0Sapvzu97/jgQcfQBKrV69mxowZzJ07l5qaGm677TauuOIKBg4cSJqmrFixgmOPPZaTTz6ZhoYG5s2bR3l5OU1NTeRyOVavXs2iRYs466yz6NOnDwsWLGDevHn07duXKIpwztHR0YEkJOGcwzmHc44nnniCSy65hNbWVioqKli7di2HHnoo55xzDl1dXfzbv/0bjz76KH379qWtrY2f/+LnnH/++YwfP55MZWUlc+fOpVAoUF5eTkYSQZCJCYK/gySCIAh2FUcffTTTpk3j3//932lubmbG9BnMmTOHV155hW5RFBHHMe3t7fTv35+RI0cS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ZWcxYuQIKisrKSkuYfHixXTs2IHysnLOOussBg8ehOO49OjRg85dOiNGyMnJYtjRR7GnnJxsvvKVUbTq3r0rVVX/TZrjOKSdcMI4VBVjDMOHH82e8vJyGTZsKCi7ZMYyOfmkk/B9H2MMxhg6duqICLuMHDWSVo4TZsCA/vTt2wdVxXVdRIQ0QejevStVVV2w1kdEcBwHESHNcYRA4J9xCQQCgc/Bpk2beOKJJxg0aBBjx46lc+fO3HXXXbz44otMmTKFgoICfve733HttdfSp08fotEov/jFLygsLOSqq65i586dXH/99eTl5TFp0iTefvtt3nzzTb761a9yySWXsGDBAu677z5ycnI47bTT+PWvf80bb7zBeeedR3FxMbNnz2batGlUVFRQVVXFu+++yyuvvMKECRO44IILeOutt7jnnnuorq6murqaW2+9lQ4dOjBlyhTWrl3Lr371K2bMmMHkyZOZMWMG06dP59xzz6VLly68+OKL3HbbbfTs2ZP+/fvzWYkIIsIjjzxCTk4OgtAW+NZHRIhEIkQiERKJBCJCbW0tTz31FEuWLKFfv35UV1ezc+dO9nmigAIKInx6AhjAICKEwy4oqG9xjMPYMWNwXBcRQQQ6d67kO5dcDCpkRMOoQruydrRrV4oxhm7du2GtxTGGNAUE8K0iAqqAghgFhBNPPhE94XjccAgUMjJLKGlfirUGcQzWV4wj7EnYrWOnjnTuXIkqu6gqIkJV1yqqulaRlpOTy+TJkzBicByDWgiHQ3z962cTiURwHJeDD+6JMQ6hkMu3vnUBsVgMz/Opru6F64b4LBzHYU8igojwSQRQ/kpEcF2XT8t1XT6J4xgcxxAIfFYugUDgX1JVVJX9mYjwn1BViouLufTSSznssMOoqanh4Ycf5sILL+SYY45BVYnFYnznO99h5syZjB07lo0bN9K+fXvatWtHaWkpP//5z3FdFxFBVamurub888+nrKyMvn378uqrr/L888/Tu3dvHn30US699FImTJiAIBx++OHMmDGDp556igsuuADP8+jbty/nnnsuFRUVDBgwgIceeohVq1bRuXNntmzZQlVVFUVFRXTu3Jnq6mocx8ExDtOnT2fw4MGcdNJJhEIhKisrefvtt/nNb35D7969CYVCfBbGGIwxPPTQQ7iuS1tTX19PKpVCVbHWYozBWsv69etp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)

Axios与`InterceptorManager`之间的关系

如何保证`请求拦截器==>http请求==>响应拦截器`，这个再`Axios.prototype.request`中进行了实现

```js
  
	// filter out skipped interceptors
  // 请求拦截队列
  var requestInterceptorChain = [];
  var synchronousRequestInterceptors = true;
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    // 如果 runWhen 的结果是 false ，那么不执行这个拦截器
    if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
      return;
    }

    // 默认拦截器是异步的，并且只有所有的拦截器都是同步的，才会同步执行
    synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;
    // 将请求拦截器的成功方法和请求拦截器的失败方法加入拦截器队首
    requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  var responseInterceptorChain = [];
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    // 将响应拦截器的成功方法，响应拦截器的失败方法加入队尾
    responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
  });

  var promise;
  // 如果拦截器是异步执行
  if (!synchronousRequestInterceptors) {
    // 初始化一个执行链，将执行http请求加入队列中间
    var chain = [dispatchRequest, undefined];
    // 把请求拦截器放在链首
    // 把响应拦截器放在链尾
    Array.prototype.unshift.apply(chain, requestInterceptorChain);
    // ???concat()方法不会改变原数组，尚不清楚这行代码是干什么的
    chain.concat(responseInterceptorChain);
	//补充：再0.24.0版本中进行了修复
    //该行代码修改为chain=chain.concat(responseInterceptorChain)      
    promise = Promise.resolve(config);
    // 按照队列一次执行拦截器中的相关函数
    while (chain.length) {
      promise = promise.then(chain.shift(), chain.shift());
    }

    return promise;
  }

  // 如果拦截器是同步执行
  var newConfig = config;
  // 循环执行完所有的请求拦截器
  while (requestInterceptorChain.length) {
    var onFulfilled = requestInterceptorChain.shift();
    var onRejected = requestInterceptorChain.shift();
    try {
      newConfig = onFulfilled(newConfig);
    } catch (error) {
      onRejected(error);
      break;
    }
  }

  // 派发请求
  try {
    promise = dispatchRequest(newConfig);
  } catch (error) {
    return Promise.reject(error);
  }

  // 把所有的响应拦截器追加到 dispatchRequest 返回的 promise 后面
  while (responseInterceptorChain.length) {
    promise = promise.then(responseInterceptorChain.shift(), responseInterceptorChain.shift());
  }

  return promise;
};
```

执行链最终的结果，同时由源码可以了解到，先加入的请求拦截器会后执行，先加入的响应拦截器会先执行，类似洋葱模型

```js
chain = [... 请求拦截器的成功方法，请求拦截器的失败方法，dispatchRequest， undefined, 响应拦截器的成功方法，响应拦截器的失败方法 ...]。
```

## 派发请求

### 源码分析

直接查看`lib/core/dispatchRequest`

这个函数主要做的事情

1. 如果已经取消，则 `throw` 原因报错，使`Promise`走向`rejected`。
2. 确保 `config.header` 存在。
3.  利用用户设置的和默认的请求转换器转换数据
4. 拍平 `config.header`。
5. 删除一些 `config.header`
6. 返回适配器`adapter`（`Promise`实例）执行后 `then`执行后的 `Promise`实例。返回结果传递给响应拦截器处理。

```js
/**
 * Throws a `Cancel` if cancellation has been requested.
 * 抛出错误原因，使`Promise`走向`rejected`
 */
function throwIfCancellationRequested(config) {
  if (config.cancelToken) {
    config.cancelToken.throwIfRequested();
  }
}

/**
 * Dispatch a request to the server using the configured adapter.
 *
 * @param {object} config The config that is to be used for the request
 * @returns {Promise} The Promise to be fulfilled
 */
module.exports = function dispatchRequest(config) {
  // 取消相关
  throwIfCancellationRequested(config);

  // Ensure headers exist
  // 确保headers存在
  config.headers = config.headers || {};

  // Transform request data
  // 转换请求数据
  config.data = transformData.call(
    config,
    config.data,
    config.headers,
    config.transformRequest
  );

  // Flatten headers
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers
  );

  // 数组中的方法删除 headers
  utils.forEach(
    ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
    function cleanHeaderConfig(method) {
      delete config.headers[method];
    }
  );

      // adapter 适配器部分
  var adapter = config.adapter || defaults.adapter;

  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);

    // Transform response data
    response.data = transformData.call(
      config,
      response.data,
      response.headers,
      config.transformResponse
    );

    return response;
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

      // Transform response data
      if (reason && reason.response) {
        reason.response.data = transformData.call(
          config,
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }

    return Promise.reject(reason);
  }
```

适配器部分后续再进行分析

## 转换请求响应数据

### 源码分析

在派发请求时可以看到这样操作

```js
  // 转换请求数据
  config.data = transformData.call(
    config,
    config.data,
    config.headers,
    config.transformRequest
  );

    // 转换响应数据
    response.data = transformData.call(
      config,
      response.data,
      response.headers,
      config.transformResponse
    );
```

上述所使用的方法均位于`lib/core/transformData`中

```js
/**
 * Transform the data for a request or a response
 *
 * @param {Object|String} data The data to be transformed
 * @param {Array} headers The headers for the request or response
 * @param {Array|Function} fns A single function or Array of functions
 * @returns {*} The resulting transformed data
 */
module.exports = function transformData(data, headers, fns) {
  var context = this || defaults;
  /*eslint no-param-reassign:0*/
  utils.forEach(fns, function transform(fn) {
    data = fn.call(context, data, headers);
  });

  return data;
};
```

上述方法主要是遍历`config.transformRequest/transformResponse`数组中的方法，`config/response.data`和`config/response.headers`作为参数，依次执行这些方法

查看`config.transformRequest`和`config.transformResponse`中的方法为

```js
//转换请求数据  
transformRequest: [function transformRequest(data, headers) {
    normalizeHeaderName(headers, 'Accept');
    normalizeHeaderName(headers, 'Content-Type');
    // 判断data类型
    if (utils.isFormData(data) ||
      utils.isArrayBuffer(data) ||
      utils.isBuffer(data) ||
      utils.isStream(data) ||
      utils.isFile(data) ||
      utils.isBlob(data)
    ) {
      return data;
    }
    if (utils.isArrayBufferView(data)) {
      return data.buffer;
    }
    if (utils.isURLSearchParams(data)) {
      setContentTypeIfUnset(headers, 'application/x-www-form-urlencoded;charset=utf-8');
      return data.toString();
    }

    // 如果data是对象，或Content-Type 设置为 application/json
    if (utils.isObject(data) || (headers && headers['Content-Type'] === 'application/json')) {
      // 设置Content-Type为application/json
      setContentTypeIfUnset(headers, 'application/json');
      // 将data转换json字符串返回
      return JSON.stringify(data);
    }
    return data;
  }],
    
  // 转换响应数据
  transformResponse: [function transformResponse(data) {
    var transitional = this.transitional;
    var silentJSONParsing = transitional && transitional.silentJSONParsing;
    var forcedJSONParsing = transitional && transitional.forcedJSONParsing;
    var strictJSONParsing = !silentJSONParsing && this.responseType === 'json';

    if (strictJSONParsing || (forcedJSONParsing && utils.isString(data) && data.length)) {
      try {
        // 将data转换为json对象并返回
        return JSON.parse(data);
      } catch (e) {
        if (strictJSONParsing) {
          if (e.name === 'SyntaxError') {
            throw enhanceError(e, this, 'E_JSON_PARSE');
          }
          throw e;
        }
      }
    }

    return data;
  }],
```

转换请求和响应数据，由源码可以看出，官方文档功能介绍中的自动转换JSON数据就是在这个过程中完成的

### 新增/重写转换方法

```js
import axios from 'axios'
//重写转换请求数据的过程
axios.default.transformRequest=[(data,headers)=>{
    ...
    return data
}]

// 新增转换方法
axios.default.transformRequest.push((data,headers)=>{
    ...
    return data
})
```

## 适配器(adapter)处理请求

```js
// 在dispatchRequest中引入适配器
// adapter 适配器部分
var adapter = config.adapter || defaults.adapter;

return adapter(config).then(...)
```

在`defaults.js`中查看适配器的默认配置

```js
function getDefaultAdapter() {
  var adapter;
  // 判断当前环境
  if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    // 浏览器环境，使用xhr请求
    adapter = require('./adapters/xhr');
  } else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // For node use HTTP adapter
    // node环境，使用http模块
    adapter = require('./adapters/http');
  }
  return adapter;
}


var defaults = {
	...
  // adapter属性
  adapter: getDefaultAdapter(),
   ...
}
```

在浏览器中运行的机制

### 浏览器环境适配器

```js
// adapters/xhr

module.exports = function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    var requestData = config.data;
    var requestHeaders = config.headers;
    var responseType = config.responseType;

    // 判断是否是FormData对象, 如果是, 删除header的Content-Type字段，让浏览器自动设置Content-Type字段
    if (utils.isFormData(requestData)) {
      delete requestHeaders['Content-Type']; // Let the browser set it
    }

    // 创建xhr对象
    var request = new XMLHttpRequest();

    // 设置http请求头中的Authorization字段
    // 格式Authorization: <type> <credentials>
    // 更多内容可查看https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication
    // HTTP basic authentication
    if (config.auth) {
      var username = config.auth.username || '';
      var password = config.auth.password ? unescape(encodeURIComponent(config.auth.password)) : '';
      // 使用btoa方法base64编码username和password
      requestHeaders.Authorization = 'Basic ' + btoa(username + ':' + password);
    }

    // 拼接路径
    var fullPath = buildFullPath(config.baseURL, config.url);
    // 初始化请求方法
    // open(method:请求的http方法,url:请求的url地址,asyn:是否异步执行操作)
    request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);

    // Set the request timeout in MS
    // 设置超时时间
    request.timeout = config.timeout;

    // 资源加载进度停止(加载完毕)时触发的函数
    function onloadend() {
      if (!request) {
        return;
      }
      // Prepare the response
      // getAllResponseHeaders方法会返回所有的响应头
      var responseHeaders = 'getAllResponseHeaders' in request ? parseHeaders(request.getAllResponseHeaders()) : null;
      // 如果没有设置数据响应类型（默认为“json”）或者responseType设置为text时，获取request.responseText值否则是获取request.response
      // responseType是一个枚举类型，手动设置返回数据的类型 更多请参考 https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/responseType
      // responseText是全部后端的返回数据为纯文本的值 更多请参考 https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/responseText
      // response为正文，response的类型取决于responseType 更多请参考 https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/response
      var responseData = !responseType || responseType === 'text' ||  responseType === 'json' ?
        request.responseText : request.response;
      var response = {
        data: responseData,  // 响应正文
        status: request.status,  // 响应状态
        statusText: request.statusText,  // 响应状态的文本信息
        headers: responseHeaders, // 响应头
        config: config,
        request: request
      };

      // status>=200&&status<300 || status=304 resolve
      // 否则reject
      settle(resolve, reject, response);

      // Clean up request
      request = null;
    }

    // polyfill
    if ('onloadend' in request) {
      // Use onloadend if available
      request.onloadend = onloadend;
      // 不支持onloadend方法则监听statechange
    } else {
      // Listen for ready state to emulate onloadend
      request.onreadystatechange = function handleLoad() {
        if (!request || request.readyState !== 4) {
          return;
        }

        // The request errored out and we didn't get a response, this will be
        // handled by onerror instead
        // With one exception: request that using file: protocol, most browsers
        // will return status as 0 even though it's a successful request
        // 只读属性 XMLHttpRequest.status 返回了XMLHttpRequest 响应中的数字状态码。
        // status 的值是一个无符号短整型。在请求完成前，status的值为0。
        // 值得注意的是，如果 XMLHttpRequest 出错，浏览器返回的 status 也为0。
        // 但是file协议除外，status等于0也是一个成功的请求
        if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
          return;
        }
        // readystate handler is calling before onerror or ontimeout handlers,
        // so we should call onloadend on the next 'tick'
        setTimeout(onloadend);
      };
    }

    // Handle browser request cancellation (as opposed to a manual cancellation)
    // ajax中断时触发
    request.onabort = function handleAbort() {
      if (!request) {
        return;
      }

      reject(createError('Request aborted', config, 'ECONNABORTED', request));

      // Clean up request
      request = null;
    };

    // Handle low level network errors
    // ajax失败时触发
    request.onerror = function handleError() {
      // Real errors are hidden from us by the browser
      // onerror should only fire if it's a network error
      reject(createError('Network Error', config, null, request));

      // Clean up request
      request = null;
    };

    // Handle timeout
    // ajax请求超时时调用
    request.ontimeout = function handleTimeout() {
      var timeoutErrorMessage = 'timeout of ' + config.timeout + 'ms exceeded';
      if (config.timeoutErrorMessage) {
        timeoutErrorMessage = config.timeoutErrorMessage;
      }
      reject(createError(
        timeoutErrorMessage,
        config,
        config.transitional && config.transitional.clarifyTimeoutError ? 'ETIMEDOUT' : 'ECONNABORTED',
        request));

      // Clean up request
      request = null;
    };

    // Add xsrf header
    // This is only done if running in a standard browser environment.
    // Specifically not if we're in a web worker, or react-native.
    // 判断当前是为标准浏览器环境，如果是，添加xsrf头
    // 什么是xsrf header？ xsrf header是用来防御CSRF攻击
    // 原理是服务端生成一个XSRF-TOKEN，并保存到浏览器的cookie中，在每次请求中ajax都会将XSRF-TOKEN设置到request header中
    // 服务器会比较session中的XSRF-TOKEN与header中XSRF-TOKEN是否一致
    // 根据同源策略，非同源的网站无法读取修改本源的网站cookie，避免了伪造cookie
    if (utils.isStandardBrowserEnv()) {
      // Add xsrf header
      // withCredentials设置跨域请求中是否应该使用cookie
      var xsrfValue = (config.withCredentials || isURLSameOrigin(fullPath)) && config.xsrfCookieName ?
      // 读取cookie中的XSRF-TOKEN
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        // 在request header中设置XSRF-TOKEN
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }

    // Add headers to the request
    if ('setRequestHeader' in request) {
      utils.forEach(requestHeaders, function setRequestHeader(val, key) {
        if (typeof requestData === 'undefined' && key.toLowerCase() === 'content-type') {
          // Remove Content-Type if data is undefined
          delete requestHeaders[key];
        } else {
          // Otherwise add header to the request
          request.setRequestHeader(key, val);
        }
      });
    }

    // Add withCredentials to request if needed
    if (!utils.isUndefined(config.withCredentials)) {
      request.withCredentials = !!config.withCredentials;
    }

    // Add responseType to request if needed
    if (responseType && responseType !== 'json') {
      request.responseType = config.responseType;
    }

    // Handle progress if needed
    if (typeof config.onDownloadProgress === 'function') {
      request.addEventListener('progress', config.onDownloadProgress);
    }

    // Not all browsers support upload events
    // request.upload XMLHttpRequest.upload 属性返回一个 XMLHttpRequestUpload对象，用来表示上传的进度
    if (typeof config.onUploadProgress === 'function' && request.upload) {
      request.upload.addEventListener('progress', config.onUploadProgress);
    }

    if (config.cancelToken) {
      // Handle cancellation
      // 取消请求
      config.cancelToken.promise.then(function onCanceled(cancel) {
        if (!request) {
          return;
        }

        request.abort();
        reject(cancel);
        // Clean up request
        request = null;
      });
    }

    if (!requestData) {
      requestData = null;
    }

    // Send the request
    request.send(requestData);
  });
};
```

## 工具方法

### bind

`axios/lib/helpers/bind.js`

```js
module.exports = function bind(fn, thisArg) {
  return function wrap() {
    var args = new Array(arguments.length);
    for (var i = 0; i < args.length; i++) {
      args[i] = arguments[i];
    }
    return fn.apply(thisArg, args);
  };
};
```

这个方法的目的是为了生成改变fn指向的函数，使fn中的this指向thisArg

这个暂时不清楚为什么要手动将arguments转数组，apply是支持`arguments`的

### utils.extend

`axios/lib/utils`

```js
/**
 * Extends object a by mutably adding to it the properties of object b.
 *
 * @param {Object} a The object to be extended
 * @param {Object} b The object to copy properties from
 * @param {Object} thisArg The object to bind function to
 * @return {Object} The resulting value of object a
 */
function extend(a, b, thisArg) {
  forEach(b, function assignValue(val, key) {
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
      a[key] = val;
    }
  });
  return a;
}
```

遍历b对象，将b对象上的方法和属性复制到a对象上,如果是函数，调用`bind`将函数中的`this`指向`thisArg`

### utils.foreach

`axios/lib/utils`

```js
/**
 * Iterate over an Array or an Object invoking a function for each item.
 *
 * If `obj` is an Array callback will be called passing
 * the value, index, and complete array for each item.
 *
 * If 'obj' is an Object callback will be called passing
 * the value, key, and complete object for each property.
 *
 * @param {Object|Array} obj The object to iterate
 * @param {Function} fn The callback to invoke for each item
 */
function forEach(obj, fn) {
  // Don't bother if no value provided
  // 判空操作
  if (obj === null || typeof obj === 'undefined') {
    return;
  }

  // Force an array if not already something iterable
  if (typeof obj !== 'object') {
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }

  // 是数组则用for循环，调用fn函数，参数为Array.prototype.forEach的前三个参数
  if (isArray(obj)) {
    // Iterate over array values
    for (var i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else {
    // Iterate over object keys
    // for in遍历对象会遍历原型链上的属性
    // 以hasOwnProperty来过滤自身属性
    for (var key in obj) {
      if (Object.prototype.hasOwnProperty.call(obj, key)) {
        fn.call(null, obj[key], key, obj);
      }
    }
  }
}
```

这个是`aixos`中运用最多的方法，迭代器模式

## 取消请求

### 使用方法

第一种

```js
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function (thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

axios.post('/user/12345', {
  name: 'new name'
}, {
  cancelToken: source.token
})

// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');
```

第二种

```js
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

// 取消请求
cancel();
```

查看如何生成`config.cancelToken`

### 源码分析

第一种请求流程

```js
const CancelToken=axios.CancelToken
const source=CancelToken.source()
source.cancel('message:取消请求')
```

`CancelToken.source`的实现

```js
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};
```



## 参考资料

https://juejin.cn/post/6844904019987529735

https://juejin.cn/post/6844903910650413070


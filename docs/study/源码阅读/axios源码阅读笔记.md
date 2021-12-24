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
  //创建axios实例
  var context = new Axios(defaultConfig);
  //将instance指向Axios.prototype.request方法
  //bind函数是工具类函数，用于改变this指向，当
  var instance = bind(Axios.prototype.request, context);
  //把Axios.prototype上的方法扩展到instance上，指定上下文是context
  utils.extend(instance, Axios.prototype, context);

  // 把context上的方法扩展到instance上
  utils.extend(instance, context);

  //导出instance对象
  return instance;
}

// Create the default instance to be exported
var axios = createInstance(defaults);

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

当调用`axios()`时，实际上是执行了`createInstance`返回的一个指向`Axios.prototype.requeset`的函数。添加`create`方法支持用户自定义配置创建，并且最终也是执行力`Axios.prototype.request`方法


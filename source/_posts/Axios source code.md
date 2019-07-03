---
title: Axios 源码学习
date: 2018-11-26
tags: 
  - source code
categories: notes
---


## Main file

通过 `package.json` 找到入口文件 `lib/axios.js`。

这里导出了 `Axios` 的实例，并在实例上挂载了 `create`、`Cancel`、`CancelToken`、`isCancel`、`all`、`spread` 等属性方法以及  `Axios` 构造函数。





## Axios class

从入口文件顶部可以看到 Axios 类的构造函数位于 `core/Axios.js`。



构造函数内部定义两个实例属性：`defaults`、`interceptors`。

`defaults` 为传入构造函数的参数。 

`interceptors` 的两个属性分别是 `request` 与 `response`，默认值都是 `InterceptorManager` 类的实例。



在 `Axios` 原型上定义有 `request`、`getUri` 两个方法。



在使用 axios 时通常有两种方式发起请求：

1.  `axios.request(config)` 或 `axios.request(url[, config])`
2. `axios[method](url, config)` 或 `axios[method](url, data, config)`



这里在 `Axios` 内部将 `delete`、`get`、`head`、`options` 四种请求方式封装成不需要 `data` 参数形式，

将 `post`、`put`、`patch`三种请求方式封装成需要 `data` 参数形式，

并最终调用 `Axios.request` 方法 Dispatch request。





### InterceptorManager

`InterceptorManager` 类用管理拦截器。

实例属性 `handlers` 保存所有定义的拦截器。

`interceptorManager.use` 添加拦截器到 `handlers` 数组，并返回其索引。

`interceptorManager.eject` 通过 id (索引)移除拦截器。

`interceptorManager.foeEach` 遍历所有拦截器。





### axios.request

`axios.request` 用来调度拦截器与控制请求发送。

在 `request` 方法内部首先对参数进行了兼容性处理，兼容 `axios.request(config)`  与 `axios.request(url[, config])` 两种形式的调用。

再将参数 `config` 与 `this.defaults`  属性，也就是实例化 `Axios` 类时传入的默认配置进行合并。



下面是 `request` 方法核心代码：

```javascript

  var chain = [dispatchRequest, undefined];
  var promise = Promise.resolve(config);

  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }
```

 `chain` 为待执行队列，默认有一个 `dispatchRequest` 用来发送请求。

`promise` 是一个`resolved` 的 `Promise` 对象，默认返回 `config`  。



随后遍历所有拦截器，并将 request interceptors handlers 添加至执行队列头部，将 response interceptors handlers 添加到执行队列尾部。



最后遍历 `chain` 队列，执行 request 拦截器时 `promise` 变量中默认 resolve 的 `config` 对象被传入拦截器，最先添加的拦截器最后执行，拦截器第二个回调 catch 先执行拦截器中的错误，直到被 `config` 传入 `dispatchRequest`。请求响应 response 依次传入 response 拦截器，执行顺序与 request 拦截器相反。





## dispatchRequest

Axios 类用来调度拦截器与 `dispatchRequest` 方法，实际的请求则是由 `dispatchRequest` 处理请求 `config` 对象后适配不同平台然后进行发送的。



`dispatchRequest` 主要是对 `config` 对象进行一下处理：

- 处理请求 `config.url`，优先使用绝对路径
- 设置默认请求头为 `{}`
- 格式化 request data
- 处理默认 `config.headers` 
- 调用适配后的方法发送请求
- 在最终发送请求的 Promise 回来后对 response 格式化，并 catch 错误





## adapter

axios 根据 `process` 判断是否为 node.js 环境，通过[适配器模式](https://zh.wikipedia.org/zh-hans/%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F)兼容 node.js 与浏览器环境。



### XHR

`lib/adapters/xhr.js` 的 `xhrAdapter` 方法适配浏览器环境。

接受一个 `config` 对象，并返回一个 `Promise` 对象。

请求最终通过 [`XMLHttpRequest` 类](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)实例 `request` 发送，方法内部通过 `config` 对象配置 `request`。



调用 `request.open` 前做了如下判断：

- 判断 request data 是否为 formData 格式
  - 如果是，删除 `Content-Type` 头，浏览器会自动配置
- 判断是否配置 `config.auth` HTTP basic authentication（[基本认证](https://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)）
  - 如果是，将认证信息 Base64 编码在 `Authorization` 头内



调用 request.open 后配置 XHR Event handler:

- readystatechange 事件
  - 过滤未完成的请求 [`readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/readyState) 不为 `4`
  - 处理 `file:` 协议请求成功 [`status`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/status) 为 `0`
  - 包装 `response`
  - 通过 `settle` 方法  resolev promise
- abort 事件
  - 处理浏览器终止请求，抛出错误
- error 事件
  - 抛出错误
- timeout 事件
  - 抛出错误



继续根据 `config` 对象配置 `request`：

- 添加 xsfr 头
- 添加其余请求头
- 配置 `request.withCredentials` 属性
- 配置 `request.responseType` 属性
- 配置 progress 事件
- 配置 upload 事件
- 配置 abort 后的处理



最后调用 `request.send` 发起 XHR 请求。
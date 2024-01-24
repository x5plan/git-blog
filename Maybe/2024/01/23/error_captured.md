# 前端监控错误捕获部分



错误监控可以从几个方面来进行分析：

- What，发生了什么错误：逻辑错误，数据错误，请求错误等。
- When，出现的时间段，如时间戳。
- Who，影响的IP，报错的事件数。
- Where，出现在那些页面，代码的第几行，第几列，出错文件。
- Why, 错误的原因是什么，SourceMap，。
- How，怎么定位到错误位置并解决。、

<img src="https://yunding-ljt.oss-cn-beijing.aliyuncs.com/%E9%94%99%E8%AF%AFshang.png" style="zoom: 80%;" />

## 错误分类

​        错误监控主要可以分为以下几种：**脚本错误监控**、**请求错误监控**以及**资源错误监控**， 而脚本错误又可以大致分为两种：编译时错误以及运行时错误，而编译时错误基本上可以通过lint工具来对大部分代码进行检查，比如**eslint**，可以保证代码不出现低级的错误。所以一般我们说的脚本错误监控指的就是运行时错误监控。

​		脚本错误监控，在JavaScript中我们一般会通过  `**try catch**` 来捕捉错误，但是我们无法在所有代码片段里都嵌入**try catch**。

### 常规脚本错误

只要页面出现脚本错误时，就会产生`onerro`r事件，我们只需要捕捉该事件即可。

**onerror  :**  当JavaScript运行时错误（包括语法错误）发生时，window会触发一个`ErrorEvent`接口的error事件，并执行`window.onerror()。`

​		我们可以使用 `window.onerror` 来捕获全局的 `JS运行异常`,`window.onerror` 是一个全局变量，默认值为null。**当有js运行时错误触发时，window会触发error事件**，并执行 `window.onerror()`,借助这个特性，我们对 `window.onerror` 进行重写就可以捕获到代码中的异常；

```js
/**
 * @description window.onerror 全局捕获错误
 * @param event 错误信息，如果是
 * @param source 错误源文件URL
 * @param lineno 行号
 * @param colno 列号
 * @param error Error对象
 */
window.onerror = function (event, source, lineno, colno, error) {
  // 上报错误
  // 如果不想在控制台抛出错误，只需返回 true 即可
};

```

**window.addEventListener('error')**:	除了可以捕获到`js`运行异常之外，还可以同时捕获到静态资源加载异常，但必须要设置捕获阶段，即第三个参数为`true`。并且`addEventListener('error')`只有一个保存所有错误信息的参数。

```js
window.addEventListener(
				"error",
				function (event) {
					console.log(
						"window.addEventListener:",
						event,
						"==e.error:",
						event.error
					);
					if (event.target && (event.target.src || event.target.href)) {
						//静态资源
						console.log({
							type: "error", //resource
							errorType: "resourceError",
							filename: event.target.src || event.target.href, //加载失败的资源
							tagName: event.target.tagName, //标签名
						});
					}
				},
				true
			); ////由于网络请求不会冒泡，必须在捕获阶段处理

```

**更推荐使用第二种方法**：方法一可以被 `window.onerror` 重新覆盖；方法二可以同时处理静态资源错误。

#### js常见的错误类型

|      类型      |          含义          | 说明                                 |
| :------------: | :--------------------: | ------------------------------------ |
|  SyntaxError   |        语法错误        |                                      |
| ReferenceError |        引用错误        | 引用不存在的变量之类                 |
|   RangeError   |      有效范围错误      | 数组变量超出其有效范围或参数超出范围 |
|   TypeError    |        类型错误        | 类型错误 如：`let foo = 3;foo();`    |
|    URIError    |    URL处理函数错误     |                                      |
| AggregateError | 包含多个错误信息的错误 | 内包含多个错误的错误信息             |

### 静态资源加载异常

一般检测静态资源的方法会使用**window.addEventListener('error')**，

> 以下是从网上找的示例代码：
>
> ```ts
> // 静态资源错误的 ErrorTarget
> export interface ResourceErrorTarget {
>   src?: string;
>   tagName?: string;
>   outerHTML?: string;
> }
> 
> // 初始化 静态资源异常 的数据获取和上报
> initResourceError = (): void => {
>   const handler = (event: Event) => {
>     event.preventDefault(); // 阻止向上抛出控制台报错
>     // 如果不是跨域脚本异常,就结束
>     if (getErrorKey(event) !== mechanismType.RS) return;
>     const target = event.target as ResourceErrorTarget;
>     const exception = {
>       // 上报错误归类
>       mechanism: {
>         type: mechanismType.RS,
>       },
>       // 错误信息
>       value: '',
>       // 错误类型
>       type: 'ResourceError',
>       // 用户行为追踪 breadcrumbs 在 errorSendHandler 中统一封装
>       // 页面基本信息 pageInformation 也在 errorSendHandler 中统一封装
>       // 错误的标识码
>       errorUid: getErrorUid(`${mechanismType.RS}-${target.src}-${target.tagName}`),
>       // 附带信息
>       meta: {
>         url: target.src,
>         html: target.outerHTML,
>         type: target.tagName,
>       },
>     } as ExceptionMetrics;
>     // 一般错误异常立刻上报，不用缓存在本地
>     this.errorSendHandler(exception);
>   };
>   window.addEventListener('error', (event) => handler(event), true);
> };
> 
> ```
>

### 系统错误

上报的错误中本身有错误

### Promise异步异常

而当抛出 `Promise异常` 时，会触发 `unhandledrejection` 事件以及`rejectionhandled`，   所以我们只需要去监听它就可以进行 `Promise 异常` 的捕获了，不过值得注意的一点是：**相比与上面所述的直接获取报错的行号、列号等信息**，`Promise异常` 我们只能捕获到一个 `报错原因` 而已；

**当 `Promise` 被 reject 且没有 reject 处理器的时候，会触发 `unhandledrejection` 事件。**

**当 `Promise` 被 reject 且有 reject 处理器的时候，会触发 `rejectionhandled` 事件。**

`rejectionhandled：`	当`Promise`被`rejected`但`rejection`处理器没有及时处理时会在全局触发`rejectionhandled` 事件

```js
window.addEventListener("unhandledrejection", function (e) {
  //阻断异常继续抛出
  e.preventDefault();
  console.log("unhandledrejection捕获到promise错误的原因是：", e.reason);
  console.log("unhandledrejection Promise 对象是：", e.promise);
  return true;
});

//promise异常被处理了
window.addEventListener("rejectionhandled", (e) => {
  // rejected的原因
  console.log("rejectionhandled:", e.reason);
});

const p1 = new Promise((resolve, reject) => {
  reject("promise error1");
});

setTimeout(() => {
  p1.catch((e) => {
    console.log("catch捕获到promise1 错误:", e);
  });
}, 1000);

```

`unhandledrejection` ：当`Promise` 被 `reject` 且没有 `reject 处理器`的时候，会触发 `unhandledrejection` 事件；这可能发生在 window 下。

> ```js
>   window.addEventListener("unhandledrejection", (event) => {
>     _sendError({
>       topic: "promise",
>       // reject 返回错误
>       message: event.reason,
>       stack: event.reason.stack ? event.reason.stack : '' ,
>     });
>   });
> 
> ```

以下是从网上搜集的代码片段：

```tsx
// 初始化 Promise异常 的数据获取和上报
initPromiseError = (): void => {
  const handler = (event: PromiseRejectionEvent) => {
    event.preventDefault(); // 阻止向上抛出控制台报错
    const value = event.reason.message || event.reason;
    const type = event.reason.name || 'UnKnowun';
    const exception = {
      // 上报错误归类
      mechanism: {
        type: mechanismType.UJ,
      },
      // 错误信息
      value,
      // 错误类型
      type,
      // 解析后的错误堆栈
      stackTrace: {
        frames: parseStackFrames(event.reason),
      },
      // 用户行为追踪 breadcrumbs 在 errorSendHandler 中统一封装
      // 页面基本信息 pageInformation 也在 errorSendHandler 中统一封装
      // 错误的标识码
      errorUid: getErrorUid(`${mechanismType.UJ}-${value}-${type}`),
      // 附带信息
      meta: {},
    } as ExceptionMetrics;
    // 一般错误异常立刻上报，不用缓存在本地
    this.errorSendHandler(exception);
  };

  window.addEventListener('unhandledrejection', (event) => handler(event), true);
};

```

### Http请求异常

http请求异常也就是异步请求http接口时的异常，在出现异常时，是会报错误码的；而我们现在调用的接口一般是通过async/await这种基于Promise的解决异步的最终方案；



假如说请求了一个接口地址报了500，因为是基于 `Promise` 调用的接口，我们**能够**在上文的 `Promise异常` 捕获中，获取到一个错误信息，但是如上文所说，Promise异常捕获没办法获取报错的行列，我们只知道报错信息，但是是不知道哪里的接口报错了。



所以我们不仅需要知道**错误信息**，还需要知道**错误位置**，**请求状态码，**以及请求参数等等。



为了实现上述的监控需求，我们需要了解到：现在异步请求的底层原理都是调用的`XMLHttpRequest或者fetch`，我们只需要对这两个方法都进行劫持，就可以往接口请求的过程中加入我们所需要的一些参数捕获；

### 跨域脚本错误

跨域脚本错误一般较难遇到，举个例子：比如说我们新建一个`texterror.js` 文件到 `项目B` 的 public 目录下以供外部访问;

```js
// 新建的 texterror.js 文件
const a = new Array(-1);

```

上面写在`texterror.js`文件里的这行代码它会被捕获在js运行异常中,且错误类型为RangeError;而我们从`项目A`中引入它

```js
// 项目B的地址，和项目A端口不同；
<script async src="http://xxxxxx:8081/texterror.js"> </script>

```

这时候控制台会报"Script error"的错误,而且**获取不到其他信息**.

其实这是浏览器的一个`安全机制`：**当跨域加载的脚本中发生语法错误时，浏览器出于安全考虑，不会报告错误的细节，而只报告简单的 `Script error`。浏览器只允许同域下的脚本捕获具体错误信息，而其他脚本只知道发生了一个错误，但无法获知错误的具体内容（控制台仍然可以看到，JS脚本无法捕获）**，我们上文通过项目A去加载项目B的文件，自然产生了跨域；

#### 处理跨域

其实对于跨域问题来说 , 网上许多人都说没必要去捕获第三方脚本错误,可以不捕获不上报

还是找到了一种获取错误详情的办法:

> 我们只需要 **`开启跨域资源共享CORS（Cross Origin Resource Sharing）`**，就可以捕获错误了~我们需要分两步来进行实现：
>
> 1. **添加`crossorigin="anonymous"`属性。**
>
> ```js
> js
> 复制代码<script src="http://xxxxxxxx/texterror.js" crossorigin="anonymous"></script>
> ```
>
> 1. **添加`跨域HTTP响应头`。**
>
> ```js
> js
> 复制代码Access-Control-Allow-Origin: *
> ```
>
> 这两步完成后，允许了跨域，我们就可以在错误捕获脚本中获取到具体的错误信息拉！

但是在后期总结时又发现了另一种方法:

即将所有的脚本全部放到同一源下,但是,该方案会放弃CDN，降低性能。

以上第一种方法基本可以完美解决跨域脚本错误捕获的问题，但是，他会有兼容性问题，`crossorigin`属性对于IE和Safari支持程度不高。 

### 框架错误捕获

#### Vue错误捕获

- `Vue2`如果是在组件渲染阶段时出现运行错误，错误将会被传递至全局的`Vue.config.errorHandler`配置函数。
- `Vue3`则是传到`app.config.errorHandler`配置函数中去。

```js
/**
 * 全局捕获Vue错误，直接扔出给onerror处理
 */
Vue.config.errorHandler = function (err) {
  setTimeout(() => {
    throw err
  })
}

```



#### React错误捕获

`React` 一样也有官方提供的错误捕获，见文档：[zh-hans.reactjs.org/docs/react-…](https://link.juejin.cn/?target=https%3A%2F%2Fzh-hans.reactjs.org%2Fdocs%2Freact-component.html%23componentdidcatch)

和 `Vue` 不同的是，我们需要**自己定义一个高阶组件暴露给项目使用**

> ```js
> import * as React from 'react';
> class ErrorBoundary extends React.Component {
>   constructor(props) {
>     super(props);
>   }
>   // ...
>   componentDidCatch(error, info) {
>     // "组件堆栈" 例子:
>     //   in ComponentThatThrows (created by App)
>     //   in ErrorBoundary (created by App)
>     //   in div (created by App)
>     //   in App
>   }
>   // ...
> }
> 
> ```

> 使用时：
>
> ```js
> import React from "react";
> 
> <ErrorBoundary>
>   <Example />
> </ErrorBoundary>;
> 
> ```

# PerformanceObserver

`performance` API有许多作用，**可以根据传入的监控事件返回对应的`PerformanceEntry`对象**，具体可以参考：

[性能监测对象 - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceObserver)

> **`erformanceObserver`** 用于*监测*性能度量事件，在浏览器的性能时间轴记录新的 [performance entry](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceEntry) 的时候将会被通知。

而在关于资源监听方面的作用主要体现在你可以使用`performance API` 配合 `PerformanceObserver`，在脚本加载结束后处理`entry`来实现通过Observer去监听新的请求，而不用去重写`XHR`，`fetch`等对象。

​		需要注意的是，`PerformanceObserver`并不支持所有的`entry`类型，可以通过`PerformanceObserver.supportedEntryTypes`来获取当前浏览器所支持的`entry`类型。

entry 类型大体有：

- paint类型的性能条目表示页面绘制事件，如首次渲染（first-paint）和主要内容渲染（first-contentful-paint）。这些条目提供了关于页面渲染的时间和相关信息。
- resource类型的性能条目表示加载的资源，如脚本、样式表、图像等。这些条目提供了关于资源加载的时间、大小和其他相关信息。
- navigation类型是导航相关
- longtask是长任务类型
- meature是自定义测量

**示例代码：**

> ```js
> const onInitial = async () => {
>     const handler = (list) => {
>         axios.post('/report', {
>             data: list.getEntries();
>         })
>     };
>     const observer = new PerformanceObserver(handler);
>     observer.observe({ entryTypes: PerformanceObserver.supportedEntryTypes });//获取浏览器支持的entry类型
>     await axios.post('/report', {
>         data: performance.getEntries().map(entry => {//	getEntries(): 返回一个数组，包含观察到的性能条目。每个																		性能条目都是 PerformanceEntry 对象的实																	 例，它提供了有关特定事件或测量的详细信息。
>             entry.toJSON();
>         })
>     });
> };
> onInitial();
> 
> ```
>
> 

## 异常上报

### AJAX请求上报

```js
axios.post(url, data);
```

但因为请求异步可被`cancel`，建议使用`XMLHttpRequest`发送同步请求

```js
const syncReport = (url, { data = {}, headers = {} } = {}) => {
  const xhr = new XMLHttpRequest();
  xhr.open('POST', url, false);
  xhr.withCredentials = true;//跨域请求是否提供凭据（cookie）
  Object.keys(headers).forEach((key) => {
    xhr.setRequestHeader(key, headers[key]);
  });
  xhr.send(JSON.stringify(data));
};

```

**缺点**：延迟页面卸载；

### 图片上报

由于图片天然可跨域，又能兼容所有的浏览器，而js和css等其他资源文件则可能出现安全拦截和跨域加载问题。

```js
let img = new Image()
img.src='请求的url'
```

但由于是一个get请求，上报的数据量在不同的浏览器下上限不一致（2kb-8kb），这就可能出现超出长度限制而无法上报完整数据的情况。因此，图片上报也是一个“不安全”的方式。

> 有一种可以将图片格式改为.gif格式的对策，可以有效节约上报体积

**缺点**：延迟页面卸载，部分浏览器丢点，GET请求长度受限制。

### sendBeacon

`navigator.sendBeacon()` 方法可用于通过 `HTTP` POST 将**少量数据**`异步`传输到 Web 服务器。

使用 **`sendBeacon()`** 方法会使用户代理在有机会时**异步**地向服务器发送数据，同时不会延迟页面的卸载或影响下一导航的载入性能，这意味着：

- 数据发送是可靠的。
- 数据异步传输。
- 不影响下一导航的载入。

```js
navigator.sendBeacon(url, data);
```




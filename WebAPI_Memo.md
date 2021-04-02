# Web API Memo

## Web Workers API

### Web Workers concepts and usage

**Web Workers** 是从 web application 的主线程(通常是 UI 线程)中分离出来一个单独的线程, 用来处理那些耗费资源的计算, 从而可以保证主线程的流畅.

使用构造器创建一个 worker, 然后运行一个 js 文件.

在 worker 中不能直接操纵 DOM, 并且有些 `window` 对象的属性和方法也无法使用, 具体详见 [worker golbal context and functions](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API#worker_global_contexts_and_functions) 和 [supported web APIs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API#supported_web_apis)

worker 线程和主线程间的通信方式为: 发送信息 `postMessage()`, 接收/响应信息 `onmessage` 事件控制器 (信息的内容再 `Message` 事件对象的 `data` 属性中).

Workers 可以繁衍新的 workers. 另外也可以使用 `XMLHttpRequest`.

==Worker types==

有几种不同类型的 workers:

- 被单个脚本使用的专属 workers. (Dedicated workers are works that are utilized by a single script.)
- 被多个脚本在不同窗口中使用的共享 workers. (Shared workers are works that can be utilized by multiple scripts running in different windows, IFrames)
- Service Workers 作为 web application, the browser, and the network 间的代理服务器只用. 其目的是创建良好的离线体验.
- Chrome Workers 主要用于开发 add-ons
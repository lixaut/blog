
`进程`是 cpu 分配资源的最小单位

`线程`是程序执行的最小单位

### 🏷️ 浏览器

浏览器是多进程的，主要包含以下进程：
- Browser 进程（主进程）
- 第三方插件进程
- GPU 进程
- 渲染进程（重点）

🔔 渲染进程

渲染进程（renderer）是多线程的，用来实现页面的渲染、JS 的执行和事件循环，主要有以下几个线程：
- GUI 渲染线程
- JS 引擎线程
- 事件触发线程
- 定时触发线程
- 异步 http 请求线程

🔔 JS 为什么是单线程？

JS 作为浏览器脚本语言，主要用途是用来与用户交互和操作 DOM，这也就决定了它只能是单线程语言，否则就乱套了，虽然 HTML5 的 Web Worker 标准允许 JS 脚本创建多个线程，但这些子线程都是受主线程控制的，且不允许操作 DOM，所以 JS 的本质就是单线程语言

🔔 JS 引擎线程

JS 引擎线程就是 JS 的内核，负责处理 JS 脚本程序，运行代码

JS 引擎会一直等待任务队列中任务的到来，然后进行执行

JS 引擎线程与 GUI 渲染线程是互斥的，会阻塞 GUI 渲染线程

🔔 GUI 渲染线程

负责渲染浏览器界面，解析 HTML、CSS，构建 DOM 树和 CSSOM 树，实现布局和绘制

GUI 渲染线程与 JS 引擎线程是互斥的，当 JS 引擎执行时，GUI 渲染线程会被挂起，GUI 更新会被放入一个队列中，等 JS 引擎空闲时就会被弹出立即执行

### 🏷️ 事件循环（Event Loop）

JS 分为同步任务和异步任务

同步任务在主线程（JS 引擎线程）上执行，会形成一个执行栈

除了主线程，还有一个消息队列，在遇到不同的异步任务时，会触发相应的异步线程，将异步任务分配给相应的线程进行处理，主线程继续执行同步代码，相应的异步线程任务处理完后，将异步任务的事件回调放入消息队列

同时放入消息队列时，事件触发线程优先级大于定时触发线程

一旦同步任务执行完毕，JS 执行主线程空闲了，就会读取消息队列，将可执行的异步任务事件回调放入执行栈，开始执行

执行栈中又分为同步任务和异步任务，如此循环往复，就形成了事件循环

🔔 宏任务（macroTask）

执行栈中每一次执行的代码都可以被当作是一个宏任务，宏任务从头到尾被完整执行，中间不会执行其他

每一个宏任务执行完毕，下一个宏任务开始执行之前，GUI 渲染线程就会渲染一次页面（互斥原则）

常见的宏任务：
- setTimeout()
- setInterval()
- 主代码块
- requestAnimationFrame()

🔔 微任务（microTask）

当执行每一个宏任务时，JS 引擎会在当前执行栈中创建一个执行上下文，该执行上下文中维护了一个微任务队列，执行过程中遇到的微任务，会被放入微任务队列

代码执行完毕后，在退出当前执行栈之前，JS 引擎就会去读取微任务队列，如果不为空，就会依次执行该队列中的微任务，直到微任务队列为空，退出当前执行栈

每一个宏任务执行完毕，再执行该宏任务中的所有为任务，然后进行一次 GUI 渲染，再去执行下一个宏任务

常见的微任务：
- promise.then()
- catch
- finally
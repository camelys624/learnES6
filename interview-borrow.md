# 浏览器

## 事件机制

### 事件触发三阶段

事件触发由三个阶段

- `window` 往事件触发处传播,遇到注册的事件捕获事件会触发
- 传播到事件触发处时触发注册的事件
- 从事件触发处往 `window` 传播,遇到注册的冒泡事件会触发

时间触发一般来说会按照上面的顺序进行,但是也有特例,如果给一个目标节点同时注册冒泡和捕获事件,事件触发会按照注册的顺序执行.

```js
// 以下会先打印冒泡然后是捕获
node.addEventListener('click', event => {
  console.log('冒泡');
}, false)
node.addEventListener('click', event => {
  console.log('捕获');
}, true)
```

### 注册事件

通常我们使用 `addEventListener` 注册事件,该函数的第三个参数可以是布尔值,也可以是对象.对于布尔值 `useCapture` 参数来说,该参数默认值为 `false` . `useCapture` 决定了注册的是捕获事件还是冒泡事件.对于对象参数来说,可以使用以下几个属性

- `capture`,布尔值,和 `useCapture` 作用一样
- `once`,布尔值,值为 `true` 表示该回调只会调用一次,调用后会移除监听
- `passive`,布尔值,表示永远不会调用 `preventDefault`

一般来说,我们只希望事件只触发在目标上,这时候可以使用 `stopPropagation` 来阻止事件的进一步传播.通常我们认为 `stopPropagation` 是用来阻止事件冒泡的,其实该函数也可以阻止捕获事件.`stopImmediatePropagation` 同样也能实现阻止事件,但是还能阻止该事件目标执行别的注册事件.

```js
node.addEventListener('click', envent => {
  event.stopImmediatePropagation();
  console.log('冒泡');
}, false)
// 点击 node 只会执行上面的函数,该函数不会执行
node.addEventListener('click', event => {
  console.log('捕获');
}, true)
```

### 事件代理

如果一个节点中的子节点是动态生成的,那么子节点需要注册事件的话应该注册在父节点上

```html
<ul id="ul">
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
  <li>5</li>
</ul>
<script>
let ul = document,querySelector('##ul')
ul.addEventListener('click', event => {
  console.log(event.target);
})
</script>
```

事件代理的方式相对于直接给目标注册事件来说,有以下优点

- 节省内存
- 不需要给子节点注销事件

## 跨域

因为浏览器出于安全考虑,有同源策略.也就是说,如果协议\域名或者端口有一个不同就是跨域,Ajax 请求就会失败.

我们可以通过以下几种常见方法解决跨域的问题

### JSONP

JSONP 的原理很简单,就是利用 `<script>` 标签没有跨域限制的漏洞. `<script>` 标签指向一个需要访问的地址并提供一个回调函数来接收数据当需要通讯时.

```html
<script src="http://domain/api?param1=a&param2=b&callback=jsonp"></script>
<script>
    function jsonp(data) {
    	console.log(data)
	}
</script>
```

JSONP 使用简单且兼容性不错,但是只限于 `get` 请求.

在开发中可能会遇到多个 JSONP 请求的回调函数名是相同的,这时候就需要自己封装一个 JSONP, 以下是简单实现

```js
function jsonp(url, jsonpCallback, success) {
  let script = document.createElement('script')
  script.src = url
  script.async = true
  script.type = 'text/javascript'
  window[jsonpCallback] = function(data) {
    success && success(data)
  }
  document.body.appendChild(script)
}
jsonp('http://xxx', 'callback', function(value) {
  console.log(value)
})
```

### CORS

CORS 需要浏览器和后端同时支持.IE 8 和 IE 9 需要通过 `XDomainRequest` 来实现.

浏览器会自动进行 CORS 通信,实现 CORS 通信的关键是后端.只要后端实现了CORS,就实现了跨域.

服务端设置 `Access-Contro;-Allow-Origin` 就可以开启 CORS.该属性表示哪些域名可以访问资源,如果设置通配符则表示所有网站都可以访问资源.

### document.domian

该方式只能用于二级域名相同的情况下,比如 `a.test.com` 和 `b.test.com` 适用于该方式.

只需要给页面添加 `document.domain = 'test.com'` 表示二级域名都相同就可以实现跨域

### postMessage

这种方式通常用于获取嵌入页面总的第三方页面数据.一个页面发送消息,另一个页面判断来源并接收消息

```js
// 发送消息端
window.parent.postMessage('message', 'http://test.com')
// 接收消息端
var mc = new MessageChannel()
mc.addEventListener('message', event => {
  var origin = event.origin || event.originalEvent.origin
  if (origin === 'http://test.com') {
    console.log('验证通过');
  }
})
```

## Event loop

众所周知 JS 是门非阻塞单线程语言,因为在最初 JS 就是为了和浏览器交互而诞生的.如果 JS 是门多线程的语言的话,我们在多个线程中处理 DOM 就可能会发生问题 (一个线程中新增节点,另一个线程中删除节点),当然可以引入读写锁解决这个问题.

JS 在执行的过程中会产生执行环境,这些执行环境会被顺序的加入到执行栈中.如果遇到异步的代码,会被挂起并加入到 Task (有多种 task) 队列中.一旦执行栈为空,Event loop 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行,所以本质上来说 JS 中的异步还是同步行为.

```js
console.log('script start');
setTimeout(function () {
  console.log('setTimeout');
}, 0);
console.log('script end');
```

以上代码虽然 `setTimeout` 延时为 0,其实还是异步.这是因为 HTML5 标准规定这个函数第二个参数不得小于 4 毫秒,不足会自动增加.所以 `setTimeout` 还是会在 `script end`之后打印.

不同的任务源会被分配到不同的 Task 队列中,任务源可以分为 微任务 (microtask) 和 宏任务 (macrotask).在 ES6 规范中, microtask 称为 `jobs`,macrotask 称为 `task`.

```js
console.log('script start');
setTimeout(function () {
  console.log('setTimeout');
}, 0);
new Promise(resolve => {
  console.log('Promise');
  resolve()
}).then(function() {
  console.log('Promise1');
}).then(function() {
  console.log('promise2');
})

console.log('script end');
// script start => Promise => script end => promise1 => promise2 => setTimeout
```

以上代码虽然 `setTimeout` 写在 `Promise` 前面,但是因为 `Promise` 属于微任务而 `setTimeout` 属于宏任务,所以会有以上的打印.

微任务包括 `process.nextTick`, `promise`, `Object.observe`, `MutationObserver`

宏任务包括 `script`, `setTimeout`, `setInterval`, `setImmediate`, `I/O`, `UI rendering`

很多人有个误区,认为微任务快于宏任务,其实是错误的.因为宏任务包含了 `script`,浏览器会先执行一个宏任务,接下来有异步代码的话就先执行微任务.

所以正确的一次 Event loop 顺序是这样的

1. 执行同步代码,这属于宏任务
2. 执行栈为空,查询是否有微任务需要执行
3. 执行所有的微任务
4. 必要的话渲染 UI
5. 然后开始下一轮 Event loop,执行宏任务中的异步代码

通过上述的 Event loop 顺序可知,如果宏任务中的异步代码有大量的计算并且需要操作 DOM 的话,为了更快的 界面响应,我们可以把操作 DOM 放入微任务中.

### Node 中的 Event loop

Node 中的 Event loop 和浏览器中的不相同

Node 的 Event loop 分为 6 个阶段,它们会按照顺序反复运行

```js
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<──connections───     │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

### timer

timer 阶段会执行 `setTimeout` 和 `setInterval`

一个 `timer` 指定的时间并不是准确时间,而是达到这个时间后尽快执行回调,可能会因为系统正在执行别的事务而延迟.

下限的时间由一个范围: `[1, 2147483647]` ,如果设定的时间不再这个范围,将被设置为 1.

### I/O

I/O 阶段会执行除了 close 事件,定时器和 `setTimeout` 的回调

### idle,prepare

idle,prepare 阶段内部实现

### poll

poll 阶段很重要,这一阶段中,系统会做两件事情

1. 执行到点的定时器
2. 执行 poll 队列中的事件

并且当 poll 中没有定时器的情况下,会发现以下两件事情

- 如果 poll 队列不为空,会遍历回调队列同步执行,直到队列为空或者系统限制
- 如果 poll 队列为空,会有两件事发生
  + 如果有 `setImmediate` 需要执行,poll 阶段会停止并且进入到 check 阶段执行 `setImmediate`
  + 如果没有 `setImmediate` 需要执行,会等待回调被加入队列中并立即执行回调

如果由别的定时器需要被执行,会回到 timer 阶段执行回调

### check

check 阶段执行 `setImmediate`

### close callbacks

close callbacks 阶段执行 close 事件

并且在 Node 中,有些情况下的定时器执行顺序是随机的

```js
setTimeout(() => {
  console.log('setTimeout');
}, 0)
setImmediate(() => {
  console.log('setImmediate');
})
// 这里可能会输出 setTimeout, setImmediate
// 可能也会相反的输出,这取决于性能
// 因为可能进入 event loop 用了不到 1 毫秒,这时候会执行 setImmediate
// 否则会执行 setTimeout
```

当然在这种情况下,执行顺序都是相同的

```js
var fs = require('fs')
fs.readFile(__filename, () => {
  setTimeout(function () {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  })
})
// 因为 readFile 的回调在 poll 中执行
// 发现有 setImmediate ,所以会立即跳到 check 阶段执行回调
// 再去 timer 阶段执行 setTimeout
// 所以以上输出一定是 setImmediate,setTimeout
```

上面介绍的都是 macrotask 的执行情况,microtask 会在以上每个阶段完成后立即执行.

```js
setTimeout(function () {
  console.log('timer1');

  Promise.resolve().then(function() {
    console.log('promise1');
  })
}, 0);

setTimeout(function () {
  console.log('timer2');

  Promise.resolve().then(function() {
    console.log('promise2');
  })
}, 0);
// 以上代码在浏览器和 node 中打印情况是不同的
// 浏览器中一定打印 timer1, promise1, timer2, promise2
// node 中可能打印 timer1, timer2, promise1, promise2
// 也可能打印 timer1, promise1, timer2, promise2
```

Node 中的 `process.nextTick` 会先于其他 microtask 执行.

```js
setTimeout(function () {
  console.log('timer1');

  Promise.resolve().then(function() {
    console.log('promise1');
  })
}, 0);

process.nextTick(() => {
  console.log('nextTick');
})
// nextTick, timer1, promise1
```

## 存储

### cookie, localStorage, sessionStorage, indexDB

| 特性             | cookie                                  | localStorage            | sessionStorage | indexDB                 |
| ---------------- | --------------------------------------- | ----------------------- | -------------- | ----------------------- |
| 数据生命周期     | 一般由服务器生成,可以设置过期时间       | 除非被清理,否则一致存在 | 页面关闭就清理 | 除非被清理,否则一直存在 |
| 数据存储大小     | 4K                                      | 5M                      | 5M             | 无限                    |
| 与服务器端通信后 | 每次都携带在 header 中,对于请求性能影响 | 不参与                  | 不参与         | 不参与                  |

从上表可以看出, `cookie`已经不建议用于存储.如果没有大量数据存储需求的话,可以使用 `localStorage` 和 `sessionStorage`.对于不怎么改变的数据尽量使用 `localStorage` 存储,否则可以用 `sessionStorage` 存储.

对于 `cookie` ,我们还需要注意安全性.

| 属性      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| value     | 如果用于保存用户登录态,应该将该值加密,不能使用明文的用户标识 |
| http-only | 不能通过 JS 访问 Cookie,减少 XSS 攻击                        |
| secure    | 只能在协议为 HTTPS 的请求中携带                              |
| same-site | 规定浏览器不能在跨域请求中携带 Cookie,减少 XSS 攻击          |
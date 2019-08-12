# Promise 对象

## 1. Promise 的含义

Promise 是异步编程的一种解决方案，比传统的解决方案--回调函数和事件--更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了`Promise`对象。

所谓`Promise`，简单说就是一个容器，里面保存着未来才会结束的事件 (通常是一个异步操作) 的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以使用同样的方法进行处理。

`Promise`对象有以下两个特点。

1. 对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，人和其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这是就称为 resolved ( 已定型 )。如果改变已经发生了，你再对`promise`对象添加回调函数，也会立即得到这个结果。这与事件 ( Event ) 完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

有了`Promise`对象，就可以讲异步操作以同步的流程表达出来，避免了层层嵌套的回调函数。此外，`Promise`对象提供统一的接口，使得控制异步操作更加容易。

`Promise`也有一些缺点。首先，无法取消`Promise`，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，`Promise`内部抛出的错误，不会反应到外部。第三，当处于`pengding`状态时，无法得知当前进展到哪一阶段（刚开始还是即将完成）。

## 2. 基本用法

ES6 规定，`Promise`对象是一个构造函数，用来生成`Promise`实例。

下面代码创造了一个`Promise`实例。

```js
const promise = new Promise(function(resolve, reject) {
    // ...doSomething

    if (/* 异步操作成功 */) {
        resolve(value)
    } else {
        reject(error)
    }
})
```

`Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

`resolve`函数的作用是，讲`Promise`对象的状态从“未完成”变成“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject`函数的作用是，讲`Promise`对象的状态从“未完成”转变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```js
promise.then(function (res) {
    // success
}, function (error) {
    // failure
})
```

`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态`rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受`Promise`对象传出的值作为参数。

下面是一个`Promise`对象的简单例子。

```js
function timeout(ms) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, ms, 'done')
    })
}

timeout(100).then(value => {
    console.log(value)
})
```

上面代码中，`timeout`方法返回一个`Promise`实例，表示一段时间以后才会发生的结果。过了指定的事件 (`ms`参数) 以后，`Promise`实例的状态变为`resolved`，就会触发`then`方法绑定的回调函数。

Promise 新建后就会立即执行。

```js
let promise = new Promise((resolve, reject) => {
    console.log('Promise')
    resolve();
});

promise.then(() => {
    console.log('resolved.')
})

console.log('Hi!')

// Promise
// Hi
// resolved
```

上面代码中，Promise 新建后立即执行，所以首先输出的是`Promise`。然后，`then`方法指定的回调函数，将在当前脚本所有同步执行完才会执行，所以`resolved`最后输出。

下面是异步加载图片的例子。

```js
function loadImageAsnyc(url) {
    return new Promise((resolve, reject) => {
        const image = new Image();

        image.onload = function () {
            resolve(image);
        }

        image.onerror = function () {
            reject(new Error('Could not load image at ' + url))
        }

        image.src = url;
    })
}
```

上面代码中，使用`Promise`包装了一个图片加载的异步操作。如果加载成功，就调用`resolve`方法，否则就调用`reject`方法。

下面是一个用`Promise`对象实现的 Ajax 操作的例子。

```js
const getJSON = function (url) {
    const promise = new Promise(function (resolve, reject) {
        const handler = function () {
            if (this.readyState !== 4) {
                return;
            }
            if (this.status === 200) {
                resolve(this.response)
            } else {
                reject(new Error(this.ststusText))
            }
        }
        const client = new XMLHttpRequest();
        client.open('GET', url);
        client.onreadystatechange = handler;
        client.responseType = "json";
        client.setRequestHeader("Accept", "application/json");
        client.send();
    })

    return promise;
}

getJSON("/post.json").then(function (json) {
    console.log('Contents:' + json)
}, function (error) {
    console.log('出错了', error)
})
```

上面代码中，`getJSON`是对 XMLHttpRequest 对象的封装，用于发出一个针对 JSON 数据的 HTTP 请求，并且返回一个 `Promise` 对象。需要注意的是，在`getJSON`内部，`resolve`函数和`reject`函数调用时，都带有参数。

如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给回调函数。`reject`函数的参数通常是`Error`对象的实例，表示抛出的错误；`resolve`函数的参数除了正常的值以外，还可能是另一个 Promise 实例，比如下面这样。

```js
const p1 = new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
    setTimeout(() => resolve(p1), 1000)
})

p2.then(res => console.log(res)).catch(err => console.log(err))
// Error: fail
```

上面代码中，`p1`是一个 Promise，3 秒之后变为`rejected`。`p2`的状态在1 秒之后改变，`resolve`方法返回的是`p1`。由于`p2`返回的是另一个 Promise，导致`p2`自己的状态无效了，由`p1`的状态决定`p2`的状态。所以，后面的`then`语句都变成针对后者 (`p1`)。又过了两秒，`p1`变为`rejected`，导致触发`catch`方法指定的回调函数。

注意，调用`resolve`或`reject`并不会总结 Promise 的参数函数的执行。

```js
new Promise((resolve, reject) => {
    resolve(1);
    console.log(2)
}).then(r => {
    console.log(r)
})

// 2
// 1
```

上面代码中，调用`resolve(1)`以后，后面的`console.log(2)`还是会执行，并且会首先打印出来。这是因为立即 resolved 的 Promise 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。

一般来说，调用`resolve`或`reject`以后，Promise 的使命就完成了，后续操作应该放到`then`方法里面，而不应该直接写在`resolve`或`reject`的后面。所以，最好在它们前面加上`return`语句，这样就不会有意外。

```js
new Promise((resolve, reject) => {
    return resolve(1);
    // 后面的代码就不会执行
    console.log(2)
})
```

## 3. Promise.prototype.then()

Promise 实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数 (可选) 是`rejected`状态的回调函数。

`then`方法返回的是一个新的`Promise`实例 (注意，不是原来那个`Promise`实例)。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。

```js
getJSON("/post.json").then(function (json) {
    return json.post
}).then(function (post) {
    // doSomething...
})
```

上面的代码使用`then`方法，依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。

采用链式的`then`，可以指定一组按照次序调用的回调函数。这是，前一个回调函数，有可能返回的还是一个`Promise`对象 (即有异步操作)，这时，后一个回调函数有可能返回的还是一个`Promise`对象 (即有异步操作)，这时，后一个回调函数，就会等待该`Promise`对象的状态发生变化，才会被调用。

```js
getJSON("/post/1.json").then(
  post => getJSON(post.commentURL)
).then(
  comments => console.log("resolved: ", comments),
  err => console.log("rejected: ", err)
);
```

上面代码中，第一个`then`方法指定的回调函数，返回的是另一个`Promise`对象。这时，第二个`then`方法指定的回调函数，就会等待这个新的`Promise`对象状态发生变化。如果变为`resolved`，就调用第一个回调函数，如果状态变为`rejected`，就调用第二个回调函数。

## 4. Promise.prototype.catch()

## 5. Promise.prototype,finally()

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。

```js
promise.then(res => { ... })
.catch(err => { ... })
.finally(() => { ... })
```

上面代码中，不管`promise`最后的状态，在执行完`then`或`catch`指定的回调函数以后，都会执行`finally`方法指定的回调函数。

下面是一个例子，服务器使用 Promise 处理请求，然后使用`finally`方法关掉服务器。

```js
server.listen(port).then(function () {
    // ...
}).finally(server.stop)
```

`finally`方法的回调函数不接受人和参数，这意味着没有办法知道，前面的 Promise 状态到底是`fulfilled`还是`rejected`。这表明，`finally`方法里面的操作，应该是与状态无关的，不依赖于 Promise 的执行结果。

`finally`本质上是`then`方法的特例。

## 6. Promise.all()

`Promise.all`方法用于将多个 Promise 实例，包装称一个新的 Promise 实例。

```js
const p = Promise.all([p1, p2, p3]);
```

上面代码中，`promise.all`方法接受一个数组作为参数，`p1`、`p2`、`p3`都是 Promise 实例，如果不是，就会先调用下面讲到的`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。(`Promise.all`方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。)

`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

1. 只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才回变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。
2. 只要`p1`、`p2`、`p3`之中有一个被`reject`，`p`的状态就变成`rejected`，此时第一个`reject`的实例返回值，会传递给`p`的回调函数。

##　6. Promise.race()

`Promise.race`方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。

```js
const p = Promise.race([p1, p2, p3])
```

上面代码中，只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递个`p`的回调函数。

`Promise.race`方法的参数与`Promise.all`方法一样，如果不是 Promise 实例，就会先调用下面讲到的`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。

下面是一个例子，如果指定时间内没有获得结果，就将 Promise 的状态变为 `reject`，否则变为`resolve`。

```js
const p = Promise.race([
    fetch('/resource-that-may-take-a-while'),
    new Promise(function (resolve, reject) {
        setTimeout(() => reject(new Error('request timeout')), 5000)
    })
])

p.then(console.log).catch(console.error)
```

上面代码中，如果 5 秒之内`fetch`方法无返回结果，变量`p`的状态就会变成`rejected`，从而触发`catch`方法指定的回调函数。

## 8. Promise.resolve()

有时需要将现有对象转为 Promise 对象，`Promise.resolve`方法就起到这个作用。

```js
const jsPromise = Promise.resolve($.ajax('/whatever.json'))
```

上面代码将 jQuery 生成`deferred`对象，转为一个新的 Promise 对象。

`Promise.resolve`等价于下面的写法。

```js
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

`Promise.resolve`方法的参数分为四种情况。

**(1) 参数是一个** **Promise 实例**

如果参数是 Promise 实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。

**(2) 参数是一个`thenable`对象**

`thenable`对象指的是具有`then`方法的对象，比如下面这个对象。

```js
let thenable = {
    then: function (resolve, reject) {
        resolve(42)
    }
}
```

`Promise.resolve`方法会将这个对象转为 Promise 对象，然后就立即执行`thenable`对象的`then`方法。

```js
let thenable = {
    then: function (resolve, reject) {
        resolve(42)
    }
}

let p1 = Promise.resolve(thenable)
p1.then(function (value) {
    console.log(value)  // 42
})
```

**(3) 参数不是具有`then`方法的对象，或根本就不是对象**

如果参数是一个原始值，或者是一个不具有`then`方法的对象，则`Promise.resolve`方法返回一个新的 Promise 对象，状态为 `resolved`。

```js
const p = Promise.resolve('Hello')

p.then(function (s) {
    console.log(s)
})

// Hello
```

上面代码生成一个新的 Promise 对象的实例`p`。由于字符串`Hello`不属于异步操作 ( 判断方法是字符串对象不具有 then 方法 )，返回 Promise 实例的状态从一生成就是 `resolved`，所以回调函数会立即执行。`Promise.resolve`方法的参数，会同时传给回调函数。

**(4) 不带有任何参数**

`Promise.resolve()`方法允许调用时不带参数，直接返回一个`resolved`状态的 Promise 对象。

所以，如果希望得到一个 Promise 对象，比较方便的方法就是直接调用`Promise.resolve()`方法。

```js
const p = Promise.resolve();

p.then(() => {
    // ...
})
```

上面代码的变量`p`就是一个 Promise 对象。

需要注意的是，立即`resolve()`的 Promise 对象，是在本轮“事件循环”( event loop ) 的结束时执行，而不是在下一轮“事件循环”的开始时。

```js
setTimeout(function () {
    console.log('three')
}, 0);

Promise.resolve().then(() => {
    console.log('two')
})

console.log('one')

// one
// two
// three
```

上面代码中，`setTimeout(fn, 0)`在下一轮“事件循环”开始时执行，`Promise.resolve()`在本轮“事件循环”结束时执行，`console.log('one')`则是立即执行，因此最先输出。

## 9. Promise.reject()

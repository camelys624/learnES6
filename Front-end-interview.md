# JS

## 内置类型

js 中分为七种内置类型，七种内置类型又分为两大类型：**基本类型和对象 (Object) 。**

基本类型有六种：

`null`,`undefined`,`boolean`,`number`,`string`,`symbol`.

其中 JS  的数字类型是浮点类型的，没有整型。并且浮点类型基于 IEEE 754 标准实现，在使用中会遇到某些 BUG 。`NaN` 也属于 `number` 类型，并且 `NaN` 不属于自身。

对于基本类型来说，如果使用字面量的方式，那么这个变量只是个字面量，只有在必要的时候才会转换为对应的类型

```js
let a = 111 // 这只是字面量，不是 number 类型
a.toString()  // 使用的时候才会转换为对象类型
```

对象 (Object) 是引用类型，在使用过程中会遇到浅拷贝和深拷贝的问题。

```js
let a = { name: 'FE' }
let b = a
b.name = 'EF'
console.log(a.name) // ->'EF'
```

## Typeof

`Typeof` 对于基本类型，除了 `null` 都可以显式正确的类型

```js
typeof 1  // number
typeof '1'  // string
typeof undefined  // undefined
typeof true // boolean
typeof Symbol() // symbol
typeof b  // b 没有声明，但是还是会显示 undefined
```

`typeof` 对于对象，除了函数都会显示 `object`

```js
typeof [] // object
typeof {} // object
typeof console.log()  // function
```

对于 `null` 来说，虽然它是基本类型，但是会显示 `object`，这是存在很久了的bug

```js
typeof null   // object
```

PS: 为什么会出现这种情况？因为在 JS 的最初版本中，使用的是 32 位的系统，为了性能考虑使用地位存储了变量的基本类型信息， `000` 开头代表是对像，然而 `null` 表示为全零，所以将它错误的判断为 `object` 。虽然现在内部类型判断代码已经改变了，但是对于这个 Bug 却是一直流传下来。

如果我们想获取一个变量的正确类型，可以通过 `Object.prototype.toString.call(xxx)` 。这样我们就可以得到类似 `[object Type]` 的字符串。

```js
let a
// 我们也可以这样判断 undefined
# a === undefined
// 但是 undefined 不是保留字，能够在低版本浏览器被赋值
# let undefined = 1
// 这样判断就会出错
// 所以可以用下面的方式来判断，并且代码量更少
// 因为 void 后面随便跟上一个组成表达式
// 返回就是 undefined
a === void 0
```

**值得注意的是 void 后面随便跟一个表达式，返回的都是 undefined ，所以可以用来判断一个变量是否为 undefined**。

## 类型转换

### 转 boolean

在条件判断时，除了 `undefined`,`null`,`false`,`NaN`,`''`,'0','-0',其他所有值都转为 `true`,包括所有对象。

### 对象转基本类型

对象在转换基本类型时，首先会调用 `valueOf` 然后调用 `toString`。并且这两个方法都是可以重写的。

```js
let a = {
  valueOf() {
    return 0
  }
}
```

当然我们也可以重写 `Symbol.toPrimitive`，该方法在转基本类型时调用优先级最高。

```js
let a - {
  valueOf() {
    return 10;
  },
  toString() {
    return '1';
  },
  [Symbol.toPrimitive]() {
    return 2;
  }
}
1 + a // -> 3
'1' + a // -> '12'
```

### 四则运算符

只有当加法运算时，其中一方是字符串类型，就会把另一个也转为字符串类型。其他运算只要是其中一方是数字，那么另一方就转为数字。并且加法运算会触发三种类型转换：将值转换为原始值，转换为数字，转换为字符串。

```js
1 + '1' // '11'
2 * '2' // 4
[1, 2] + [2, 1] // '1,22,1'
// 具体实现如下：
// [1, 2].toString()  -> '1,2'
// [2, 1].toString()  -> '2,1'
// '1,2' + '2,1'  -> '1,22,1'
```

对于加号需要注意这个表达式 `'a' + + 'b'`

```js
'a' + + 'b' // -> NaN
// 因为 + 'b' -> NaN
// 可能在一些代码中看到过 + '1'  -> 1
// 这是快速将字符串转为数字
```

### `==` 操作符

![judge](./static/img/judge.png)

上图中的 `toPrimitive` 就是对象转基本类型。

这里分析一道题目 `[] == ![] // -> true`，下面是这个表达式为何为 `true` 的步骤

```js
// [] 转成 true，然后取反变成 false
![] == false
// 根据第 8 条得出
![] == ToNumber(false)
![] == 0
// 根据第 10 条得出
toPrimitive([]) == 0
// [].toString() -> ''
'' == 0
// 根据第 6 条得出
0 == 0  // -> true
```

### 比较运算符

1. 如果是对象，就通过 `toPrimitive` 转换对象
2. 如果是字符串，就通过 `unicode` 字符索引来比较

## 原型

![原型](./static/img/prototype.png)

每个函数都有 `prototype` 属性，除了 `Function.prototype.bind()`,该属性指向原型。

每个对象都有 `__proto__` 属性，指向了创建该对象的构造函数的原型。其实这个属性指向了 `[[prototype]]`,但是 `[[prototype]]` 是内部属性，我们并不能访问到，所以使用 `__proto__` 来访问。

对象可以通过 `__proto__` 来寻找不属于该对象的属性，`__proto__` 将对象链接起来组成了原型链。

## new

1. 新生成了一个对象
2. 链接到原型
3. 绑定 this
4. 返回新对象

在调用 `new` 的过程中会发生以上四件事情，我们也可以试着来自己实现一个 `new`

```js
function create() {
  // 创建一个空对象
  let obj = new Object()
  // 获得构造函数
  let Con = [].shift.call(arguments)
  // 链接到原型
  obj.__proto__ = Con.prototype
  // 绑定 this，执行构造函数
  let result = Con.apply(obj, arguments)
  // 确保 new 出来的是个对象
  return typeof result === 'object' ? result : obj
}
```

对于实例对象来说，都是通过 `new` 产生的，无论是 `function Foo()` 还是 `let a = { b: 1 }`。

对于创建一个对象来说，更推荐使用字面量的方式创建对象(无论是性能上还是可读性)。因为使用 `new Object()` 的方式创建对象需要通过作用域一层层找到 `Object` ，但是你使用字面量的方式就没有这个问题。

对于 `new` 来说，还需要注意下运算符优先级。

```js
function Foo() {
  return this;
}
Foo.getName = function () {
  console.log('1');
}
Foo.prototype.getName = function () {
  console.log('2');
}

new Foo.getName();  // -> 1
new Foo().getName();  // -> 2
```

![precedence](./static/img/precedence.png)

从上图可以看出，`new Foo()` 的优先级大于 `new Foo`,所以对于上述代码来说可以这样划分执行顺序

```js
new (Foo.getName());
(new Foo()).getName();
```

对于第一个函数来说，先执行了 `Foo.getName()`,所以结果为 1；对于后者来说，先执行 `new Foo()` 产生了一个实例，然后通过原型链找到了 `Foo` 上的 `getName` 函数，所以结果为 2.

## instanceof

`instanceof` 可以正确的判断对象的类型，因为内部机制是通过判断对象的原型链是不是能找到类型的 `prototype`。

我们也可以试着实现以下 `instanceof`

```js
function instanceof(left, right) {
  // 获得类型的原型
  let prototype = right.prototype
  // 获得对象的原型
  left = left.__proto__
  // 判断对象的类型是否等于类型的原型
  while (true) {
    if (left === null){
      return false
    }
    if (prototype === left) {
      return true
    }
    left = left.__proto__
  }
}
```

## this

`this` 是很多人会混淆的概念，但是其实它一点都不难，只需要记住几个规则就可以了。

```js
function foo () {
  console.log(this.a);
}
var a = 1;
foo()

var obj = {
  a: 2,
  foo: foo()
}

obj.foo()

// 以上两者情况 `this` 只依赖于调用函数前的对象，优先级是第二个情况大于第一个情况

// 以下情况是优先级最高的，'this' 只会绑定在 `c` 上，不会被任何方式修改 `this` 指向

var c = new foo()
c.a = 3
console.log(c.a);

// 还有种就是利用 call,apply,bind 改变 this，这个优先级仅次于 new
```

以上几种情况明白了，很多代码中的 `this` 应该就没什么问题了，下面我们看看箭头函数中的 `this`

```js
function a() {
  return () => {
    return () => {
      console.log(this);
    }
  }
}

console.log(a()()());
```

箭头函数其实是没有 `this` 的，这个函数中的 `this` 只取决于他外面的第一个不是箭头函数的函数的 `this`。在这个例子中，因为调用 `a` 符合前面代码中的第一个情况，所以 `this` 是 `window`。并且 `this` 一旦绑定了上下文，就不会被任何代码改变。

## 执行上下文

当执行 JS 代码时，会产生三种执行上下文

- 全局执行上下文
- 函数执行上下文
- eval 执行上下文

每个执行上下文中都有三个重要属性

- 变量对象 (VO),包含变量、函数声明和函数的形参，该属性只能在全局上下文中访问
- 作用域链 (JS采用词法作用域，也就是说变量的作用域实在定义时就决定了)
- this

```js
var a = 10
function foo(i) {
  var b = 20
}
foo()
```

对于上述代码，执行栈中有两个上下文：全局上下文和函数 `foo` 上下文

```js
stack = [
  globalContext,
  fooContext
]
```

对于全局上下文来说，VO大概是这样的

```js
globalContext.VO === globe
globalContext.VO = {
  a: undefined,
  foo: <Function>,
}
```

对于函数 `foo` 来说，VO不能访问，只能访问到活动对象 (AO)

```js
fooContext.VO === foo.AO
fooContext.AO = {
  i: undefined,
  b: undefined,
  arguments: <>
}
// arguments 是函数独有的对象 (箭头函数没有)
// 该对象是一个伪数组，有 `length` 属性且可以通过下标访问元素
// 该对象中的 'callee' 属性戴白哦函数本身
// 'caller' 属性代表函数的调用者
```

对于作用域链，可以把它理解成包含自身变量对象和上级变量对象的列表，通过 `[[Scope]]` 属性查找上级变量

```js
fooContext.[[Scope]] = [
  globalContext.VO
]
fooContext.Scope = fooContext.[[Scope]] + fooContext.VO
fooContext.Scope = [
  fooContext.VO,
  globalContext.VO
]
```

接下来让我们看一个老生常谈的例子， `var`

```js
b() // call b
console.log(a); //undefined

var a = 'Hello world'

function b() {
  console.log('call b');
}
```

这是因为函数和变量提升的原因。通常提升的解释是说将声明的代码移动到了顶部，这其实没有什么错误，便于大家理解。但是更准确的解释应该是：在生成执行上下文时，会有两个阶段。第一个阶段是创建的阶段 (具体步骤是创建 VO)，JS 解释器会找到需要提升的变量和函数，并且给它们提前在内存中开辟好空间，函数的话会将整个函数存入内存中，变量只声明并且赋值为 undefined,所以在第二阶段，也就是代码执行阶段，我们可以直接提前使用。

在提升的过程中，相同的函数会覆盖上一个函数，并且函数由于变量提升

```js
b() // call b second

function b() {
  console.log('call b first');
}

function b() {
  console.log('call b second');
}

var b = 'Hello world'
```

`var` 会产生很多错误，所以在 ES6 中引入了 `let`。`let` 不能在声明前使用，但是这并不是常说的 `let` 不会提升，`let` 提升了声明但是没有赋值，因为临时死区导致了并不能在声明前使用。

对于非匿名立即执行函数需要注意以下一点

```js
var foo = 1
(function foo() {
  foo = 10
  console.log(foo);
})()
// -> f foo() { foo = 10; console.log(foo) }
```

因为当 JS 解释器在遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，然后将函数名称作为这个对象的属性，因此函数内部才可以访问到 `foo`,但是这个值又是只读的，所以对它的赋值并不生效。所以打印的结果还是这个函数，并且外部的值也没有发生改变。

```js
specialObject = {};

Scope = specialObject + Scope;

foo = new FunctionExpression;
foo.[[Scope]] = Scope;
specialObject.foo = foo;  // {DontDelete}, {ReadOnly}

delete Scope[0];  // remove specialObject from the front of scope chain
```

## 闭包

闭包的定义很简单：函数 A 返回了一个函数 B，并且函数 B 中使用了函数 A 的变量，函数 B 就被称为闭包。

```js
function A() {
  let a = 1;
  function B() {
    console.log(a);
  }
  return B;
}
```

为什么函数 A 已经弹出调用栈了，为什么函数 B 还能引用到函数 A 中的变量。因为函数 A 中的变量这时候时存储在堆上的。现在的 JS 引擎可以通过逃逸分析辨别出哪些变量需要存储在堆上，哪些需要存储到栈上。

经典面试题，循环中使用闭包解决 `var` 定义函数的问题

```js
for (var i = 1; i <= 5; i++) {
  setTimeout( function timer() {
    console.log( i );
  }, i * 1000)
}
```

首先因为 `setTimeout` 是个异步函数，所以会先把循环全部执行完毕，这时候 `i` 就是 6 了，所以会输出一堆 6.

解决办法两种，第一种使用闭包

```js
for(var i = 1;i <= 5; i++) {
  (function (j){
    setTimeout(function timer() {
      console.log(j);
    },j * 1000)
  })(i)
}
```

第二种就是使用 `setTimeout` 的第三个参数

```js
for (var i = 1; i <= 5; i++) {
  setTimeout( function timer(j) {
    console.log(j);
  }, i * 1000, i)
}
```

第三种就是使用 `let` 定义 `i` 了

```js
for (let i = 1; i <= 5; i++) {
  setTimeout( function timer() {
    console.log( i );
  }, i * 1000)
}
```

因为对于 `let` 来说，它会创建一个块级作用域，相当于

```js
{ // 形成块级作用域
  let i = 0;
  {
    let ii = i
    setTimeout(function timer(){
      console.log(ii);
    },i * 1000);
  }
  i++
  {
    let ii = i
  }
  ....
}
```

## 深浅拷贝

```js
let a = {
  age: 1
}
let b = a
a.age = 2
console.log(b.age); // 2
```

从上述例子中我们可以发现，如果一个变量赋值一个对象，那么两者的值会是同一个引用，其中一方改变，另一方也会相应改变。

通常在开发中我们不希望出现这样的问题，我们可以使用浅拷贝来解决这个问题。

### 浅拷贝

首先可以通过 `Object.assign` 来解决这个问题。

```js
let a = {
  age: 1
}
let b = Object.assign({}, a)
a.age = 2
console.log(b.age); // 1
```

当然我们也可以通过展开运算符 (...) 来解决

```js
let a = {
  age: 1
}
let b = {...a}
a.age = 2
console.log(b.age); // 1
```

通常浅拷贝就能解决大部分问题了，但是当我们遇到如下情况就需要使用到深拷贝

```js
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = {...a}
a.jobs.first = 'native'
console.log(b.jobs.first);  // native
```

### 深拷贝

这个问题通常可以通过 `JSON.parse(JSON.stringify(object))` 来解决。

```js
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = JSON.parse(JSON.stringify(a))
a.jobs.first = 'native'
console.log(b.jobs.first);  // FE
```

但是该方法也是有局限性的：

- 会忽略 `undefined`
- 会忽略 `symbol`
- 不能序列化函数
- 不能解决循环引用的对象

```js
let obj = {
  a: 1,
  b: {
    c: 2,
    d: 3,
  },

}
obj.c = obj.b
obj.e = obj.a
obj.b.c = obj.c
obj.b.d = obj.b
obj.b.e = obj.b.c
let newObj = JSON.parse(JSON.stringify(obj))
console.log(newObj);
```

上述代码不能通过 `JSON.pase(....)` 方法进行深拷贝

![](./static/img/error.png)

遇到函数、 `undefined` 或者 `symbol` 的时候，该对象也不能正常地序列化

```js
let a = {
  age: undefined,
  sex: Symbol('male'),
  jobs: function() {},
  name: 'ykk'
}
let b = JSON.parse(JSON.stringify(a));
console.log(b); // -> {name: 'ykk'}
```

在上述情况中，该方法会忽略点函数和 `undefined` 。

但是在通常情况下，复杂数据都是可以序列化地，所以这个函数可以解决大部分问题，并且该函数时内置函数中处理深拷贝最快地。当然如果你的数据中含有以上三种情况下，可以使用[lodash 的深拷贝函数](https://lodash.com/docs##cloneDeep).

如果需要拷贝的对象含有内置类型并且不包含函数，可以使用 `MessageChannel`

```js
function structuralClone(obj) {
  return new Promise(resolve => {
    const {post1, port2} = new MessageChannel();
    port2.onmessage = ev => resolve(ev.data);
    port1.postMessage(obj);
  });
}
var obj = {a: 1, b: {
  c: b
}}

// 注意该方向是异步的
// 可以处理 undefined 和循环引用对象
(async () => {
  const clone = await structuralClone(obj)
})()
```

## 模块化

在有 Babel 的情况下，我们可以直接使用 ES6 的模块化

```js
// file a.js
export function a() {}
export function b() {}

// file b.js
export default function() {}

import {a, b} from './a.js'
import XXX from './b.js'
```

### CommonJS

`CommonJS` 是 Node 独有的规范，浏览器中使用就需要用到 `Browserify` 解析了。

```js
// a.js
module.exports = {
  a: 1
}

// or
exports.a = 1

// b.js
var module = require('./a.js')
module.a  // -> 1
```

在上述代码中， `module.exports` 和 `exports` 很容易混淆，让我们来看看内部实现

```js
var module = require('./a.js')
module.a
// 这里其实就是包装了一层立即执行函数，这样就不会污染全局变量了
// 重要的是 module 这里，module 是 Node 独有的一个变量
module.exports = {
  a: 1
}
// 基本实现
var module = {
  exports: {} // exports 就是个空对象
}

// 这个是为什么 exports 和 module.exports 用法相似的原因
var exports = module.exports
var load = function (module) {
  // 导出的东西
  var a = 1
  module.exports = a
  return module.exports
};
```

再来说说 `module.exports` 和 `exports`,用法其实是相似的，但是不能对 `export` **直接赋值**，不会有任何效果。

对于 `CommonJS` 和 ES6 中的模块化的两者区别是：

- 前者支持动态导入，也就是 `require(${path}/xx.js)`,后者目前不支持，但是已有提案
- 前者是同步导入，因为用于服务器，文件都在本地，同步导入即使卡住主线程影响也不大，而后者是异步导入，因为用于浏览器，需要下载文件，如果也采用同步导入会对渲染有很大影响
- 前者在导出时都是值拷贝，就算导出的值变了，导入的值也不会改变，所以如果想要更新值吗，必须重新导入一次。但是后者采用实时绑定的方式，导入导出的值都指向同一个内存地址，所以导入值会跟随导出值变化
- 后者会编译为 `require/exports` 来执行的

### AMD

AMD 是由 `RequireJS` 提出的

```js
// AMD
defined(['./a', './b'], function(a, b) {
  a.do()
  b.do()
})
defined(function(require, exports, module) {
  var a = require('./a')
  a.doSomething()
  var b = require('./b')
  b.doSomething()
})
```

## 防抖

在日常开发中经常遇到这种问题，在滚动事件中需要做个复杂计算或者实现一个按钮的防止二次点击操作。

这些需求都可以通过函数放抖动来实现。尤其实第一个需求，如果在频繁的事件回调中做复杂计算，很有可能导致页面卡顿，不如将多次计算合并为一次计算，只在一个精确点做操作。

PS：防抖和节流的作用都是防止函数多次调用。区别在于，假设一个用户一直触发这个函数，且每次触发函数的介个小于wait，防抖的情况下只会调用一次，而节流的情况会每隔一定时间 (参数wait) 调用函数。

我们先来看一下袖珍版的防抖理解一下防抖的实现：

```js
// func是用户传入需要防抖的函数
// wait是等待时间
const debounce = (func, wait = 50) => {
  // 缓存一个定时器id
  let timer = 0;
  // 这里返回的函数是每次用户实际调用的防抖函数
  // 如果已经设定过定时器了就清空上一次的定时器
  // 开始一个新的定时器，延迟执行用户传入的方法
  return function (...args) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => {
      func.apply(this, args)
    }, wait)
  }
}
// 不难看出如果用户调用该函数的间隔小于wait的情况下，上一次的时间还未到就被清除了，并不会执行函数
```

这是简单的防抖，但是有缺陷，这个防抖只能在最后调用。一般的防抖会有 immediate 选项，表示是否立即调用。这两者的区别，举个例子来说：

- 例如在搜索引擎问题的时候，我们当然是希望用户输入完最后一个字才调用查询接口，这个时候适用`延迟执行`的防抖函数，它总是在一连串 (间隔小于wait的) 函数触发之后调用。
- 例如用户给 interviewMap 点 star 的时候，我们希望用户点第一下的时候就去调用接口，并且成功之后改变 star 按钮的样子，用户就可以立马得到反馈是否 star 成功了，这种情况适用 `立即执行` 的防抖函数，它总是在第一次调用，并且下一次调用必须与前一次调用的时间间隔大于 wait 才会触发。

下面我们来实现一个带有立即执行选项的防抖函数

```js
// 这个是用来获取当前时间戳的
function now() {
  return +new Date()
}
/**
 * 防抖函数，返回函数连续调用时，空闲时间必须大于或等于 wait,func 才会执行
 *
 * @param {function} func     回调函数
 * @param {number}   wait     表示时间窗口的间隔
 * @param {boolean}  immediate 设置为 true 时，是否立即调用函数
 * @return {function}          返回客户端调用函数
 */
function debounce (func, wait = 50, immediate = true) {
  let timer, context, args

  // 延迟执行函数
  const later = () => setTimeout(() => {
    // 延迟函数执行完毕，清空缓存的定时器序号
  timer = null
    // 延迟执行的情况下，函数会在延迟函数中执行
    // 使用到之前缓存的参数和上下文
    if (!immediate) {
      func.apply(context, args)
      context = args = null
    }
  }, wait)

  // 这里返回的函数是每次实际调用的函数
  return function(...params) {
    // 如果没有创建延迟执行函数 (later) ，就创建一个
    if (!timer) {
      timer = later()
      // 如果是立即执行，调用函数
      // 否则缓存参数和调用上下文
      if (immediate) {
        func.apply(this, params)
      }else {
        context = this
        args = params
      }
    }else {
      // 如果已有延迟执行函数 (later),调用的时候清除原来的并重新设定一个
      // 这样做延迟函数会重新计时
      clearTimeout(timer)
      timer = later()
    }
  }
}
```

整体函数实现的不难，总结一下。

- 对于按钮防点击来说的实现：如果函数是立即执行的，就立即调用，如果函数是延迟执行的，就缓存上下文和参数，放到延迟函数中去执行。一旦我开始一个定时器，只要我定时器还在，你每次点击我都会重新计时。一旦你点累了，定时器时间到，定时器重置为 `null`，就可以再次点击了。
- 对于延时执行函数来说的实现：清除定时器ID，如果是延迟调用就调用函数

## 节流

防抖动和节流本质是不一样的。防抖动是将多次执行变为最后一次执行，节流是将多次执行变成每隔一段时间执行。

```js
/**
 * underscore 节流函数，返回函数连续调用时，func 执行频率限定为 次 / wait
 *
 * @param {function}  func      回调函数
 * @param {number}    wait      表示时间窗口的间隔
 * @param {object}    options   如果想忽略开始函数的调用，传入{leading: false}.
 *                              如果想忽略结尾函数的调用，传入{trailing: false}
 *                              两者不能共存，否则函数不能执行
 * @return {function}           返回客户调用函数
 */
_.throttle = function(func, wait, options) {
  var context, args, result;
  var timeout = null;
  // 之前的时间戳
  var previous = 0;
  // 如果 options 没传则设为空对象
  if (!options) options = {};
  // 定时器回调函数
  var later = function() {
    // 如果设置了 leading,就将 previous 设为 0
    // 用于下面函数的第一个 if 判断
    previous = options.leading === false ? 0 : _.now();
    // 置空一是为了防止内存泄漏，二是为了下面的定时器判断
    timeout = null;
    result = func.apply(context, args);
    if (!timeout) context = args = null;
  };
  return Function() {
    // 获取当前时间戳
    var now = _.now();
    // 首次进入前者肯定为 true
    // 如果需要第一次不执行函数
    // 就将上次时间戳设为当前的
    // 这样在接下来计算 remaining 的值时会大于 0
    if (!previous && options.leading === false) previous = now;
    // 计算剩余时间
    var remianing = wait - (now - previous);
    context = this;
    args = arguments;
    // 如果当前调用已经大于上次调用时间 + wait
    // 或者用户手动调了时间
    // 如果设置了 trailing,只会进入条件
    // 如果没有设置 leading,那么第一次会进入这个条件
    // 还有一点，你可能会觉得开启了定时器那么应该不会进入这个 if 条件了
    // 其实还是会进入的，因为定时器的延时
    // 并不是准确的时间，很可能你设置了 2 秒
    // 但是他需要 2.2 秒才触发，这时候就会进入这个条件
    if (remianing <= 0 || remianing > wait) {
      // 如果存在定时器就清理掉否则会调用二次回调
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
    }else if(!timeout && options.trailing !== false) {
      // 判断是否设置了定时器和 trailing
      // 没有的话就开启一个定时器
      // 并且不能同时设置 leading 和 trailing
      timeout = setTimeout(later, remianing);
    }
    return result;
  };
};
```

## 继承

在 ES5 中，我们可以使用如下方式解决继承的问题

```js
function Super() {}
Super.prototype.getNumber = function() {
  return 1
}

function Sub() {}
let s = new Sub()
Sub.prototype = Object.create(Super.prototype, {
  constructor: {
    value: Sub,
    enumerable: false,
    writable: true,
    configurable: true
  }
})
```

以上继承实现思路就是将子类的原型设置为父类的原型

在 ES6 中，我们可以通过过 `class` 语法轻松解决这个问题

```js
class MyDate extends Date {
  test() {
    return this.getTime()
  }
}
let myDate = new MyDate()
myDate.test()
```
但是 ES6 不是所有浏览器都兼容，所以我们需要使用 Babel 来编译这段代码。

如果你使用编译过的代码调用 `myDate.test()` 你就会惊奇的发现出现了报错

因为在 JS 底层有限制，如果不是由 `Date` 构造出来的实例的话，是不能调用 `Date` 里面的函数的。所以这也侧面说明了：**ES6 中的 `class` 继承与 ES5 中的一般继承写法是不同的。**

既然底层限制了实例必须由 `Date` 构造出来，那么我们可以改变下思路实现继承

```js
function MyDate () {

}
MyDate.prototype.test = function () {
  return this.getTime()
}
let d = new Date()
Object.setPrototypeOf(d, MyData.prototype)
Object.setPrototypeOf(MyData.prototype,Date.prototype)
```

以上继承实现思路：**先创建父类实例**=>改变实例原先的 `__proto__` 转而连接到子类的 `prototype` => 子类的 `prototype` 的 `__proto__` 改为父类的 `prototype`。

通过以上方法实现的继承就可以丸美解决 JS 底层的这个限制。

## call,apply,bind 区别

首先说下前两者的区别。

`call` 和 `apply` 都是为了解决改变 `this` 的指向。作用都是相同的，只是传参的方式不同。

除了第一个参数外，`call` 可以接受一个参数列表，`apply` 只接受一个参数数组。

```js
let a = {
  value: 1
}
function getValue(name, age) {
  console.log(name);
  console.log(age);
  console.log(this.value);
}
getValue.call(a, 'ykk', '23')
getValue.apply(a, ['ykk', '23'])
```

### 模拟实现 call 和 apply

可以从一下几点来考虑如何实现

- 不传入第一个参数，那么默认为 `window`
- 改变了 this 指向，让新的对象可以执行该函数。那么思路是否可以变成给新的对象添加一个函数，然后在执行完以后删除？

```js
Function.prototype.myCall = function (context) {
  var context = context || window
  // 给 context 添加一个属性
  // getValue.call(a, 'ykk', '23')  -> a.fn = getValue
  context.fn = this
  // 将 context 后面的参数取出来
  var args = [...arguments].slice(1)
  // getValue.call(a, 'ykk', '23')  -> a.fn('ykk', '23')
  var result = context.fn(...args)
  // 删除 fn
  delete context.fn
  return result
}
```

以上就是 `call` 的思路，`aplly` 的实现也类似

```js
Function.prototype.myApply = function (context) {
  var context = context || window
  context.fn = this

  var result
  // 需要判断是否存储第二个参数
  // 如果存在，就将第二个参数展开
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  }else {
    result = context.fn()
  }

  delete context.fn;
  return result
}
```

`bind` 和其他两个方法作用也是一致的，只是该方法会返回一个函数。并且我们可以通过过 `bind` 实现柯里化。

同样，也来模拟实现下 `bind`

```js
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  var _this = this
  var args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F(),所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

## Promise 实现

Promise 是 ES6 新增的语法，解决了回调地狱的问题。

可以把 Promise 看成一个状态机。初始是 `pending` 状态，可以通过函数 `resolve` 和 `reject`，将状态转变为 `resolved` 或者 `rejected` 状态，状态一旦改变就不能再次变化。

`then` 函数会返回一个 Promise 实例，并且该返回值是一个新的实例而不是之前的实例。因为 Promise 规范规定除了 `pending` 状态，其他状态是不可以改变的，如果返回的是一个相同实例的话，多个 `then` 调用就是去意义了。

对于 `then` 来说，本质上可以把它看成是 `flatMap`

```js
// 三种状态
const PENDING = "pengding";
const RESOLVED = "resolved";
const REJECTED = "rejected";
// promise 接收一个函数参数，该函数会立即执行
function MyPromise(fn) {
  let _this = this;
  _this.currentSate = PENDING;
  _this.value = undefined;
  // 用于保存 then 中的回调，只有当 Promise
  // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
  _this.resolvedCallbacks = [];
  _this.rejectedCallbacks = [];

  _this.resolve = function (value) {
    if (value instanceof MyPromise) {
      // 如果 value 是个 Promise，递归执行
      return value.then(_this.resolve, _this.reject)
    }
    setTimeout(function () {
      // 异步执行，保证执行顺序
      if (_this.currentSate === PENDING) {
        _this.currentSate = RESOLVED;
        _this.value = value;
        _this.resolvedCallbacks.forEach(cb => cb());
      }
    });
  };

  _this.reject = function (reason) {
    setTimeout(() => {
      if(_this.currentSate === PENDING) {
        _this.currentSate = REJECTED;
        _this.value = reason;
        _this.rejectedCallbacks.forEach(cb => cb())
      }
    });
  }
  // 用于解决一下问题
  // new Promise(() => throw Error('error'))
  try {
    fn(_this.resolve, _this.reject);
  }catch (e){
    _this.reject(e);
  }
}


```
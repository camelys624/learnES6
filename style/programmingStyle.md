# 编程风格
很多公司和组织公开了它们的风格规范，下面内容主要参考了[Airbnb](https://github.com/airbnb/javascript)公司的JavaScript风格规范。

### 1. 块级作用域

#### (1) [let取代var](../let&const/let和const命令.md)
ES6提出了两个新的变量声明的命令：`let`和`const`。其中，`let`完全可以取代`var`，因为两者语义相同。而且`let`没有副作用。

`var`命令存在变量提升效用，`let`命令没有这个问题。

``` js
'use strict'

if (true) {
  console.log(x); // RefernceError
  let x = 'hello';
}
```

如果使用`var`代替`let`，`console.log`那一行就不会报错，而是会输出`undefined`。

#### (2) 全局常量和线程安全
在`let`和`const`之间，建议优先使用`const`，尤其是在全局环境，不应该设置变量，只应设置常量。

`const`优于`let`有几个原因。一个是`const`可以提醒阅读程序的人，这个变量不应该改变；另一个是`const`比较符合函数是编程思想，运算不改变值，只是新建值，而且这样有利于将来的分布式运算；最后一个原因是JavaScript编译器会对`const`进行优化，所以多使用`const`有利于提高程序的运行效率，也就是说`let`和`const`的本质区别，其实是编译器内部的处理不同。

``` js
// bad
var a = 1, b = 2, c = 3;

// good
const a = 1;
const b = 2;
const c = 3;

// best
const [a, b, c] = [1, 2, 3];
```

`const`声明常量还有两个好处，意识阅读代码的人立刻会意识到不应该修改这个值，二是防止了无意间修改量值所导致的错误。

**所有的函数都应该设置为常量。**

长远来看，JavaScript可能会有多线程的实现，这时`let`表示的变量，只应出现在单线程运行的代码中，不能是多线程共享的，这样有利于保证线程安全。

------------------

### 2. 字符串
静态字符串一律使用单引号或反引号，不使用双引号。动态字符串使用反引号。

``` js
// bad
const a = "foobar";
const b = 'foo' + a + 'bar';

// acceptable
const c = `foobar`;

// good
const a = 'foobar';
const b = `foo${a}bar`;
```
-----------------

### 3. 解构赋值
使用数组成员对变量赋值时，优先使用解构赋值。

``` js
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

函数的参数如果是对象的成员，优先使用解构赋值。

``` js
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
}

// good
function getFullName(obj) {
  const { firstName, lastName } = obj;
}

// best
function getFullName({ firstName, lastName }) {

}
```

如果函数返回多个值，优先使用**对象的解构赋值**，而不是**数组的解构赋值**。这样便于以后添加返回值，以便更改返回值的顺序。

``` js
// bad
function processInput(input) {
  return [left, right, top, bottom];
}

// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, right } = processInput(input);
```

### 4. 对象
单行定义的对象，最后一个成员不以逗号结尾。多行定义的对象，最后一个成员以逗号结尾。

``` js
// bad
const a = { k1: v1, k2: v2, };
const b = {
  k1: v1,
  k2: v2
};

// good
const a = { k1: v1, k2: v2 };
const b = {
  k1: v1,
  k2: v2,
};
```

对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，要使用`object.assign`方法。

``` js
// bad
const a = {};
a.x = 3;

// if reshape unavoidable
const a = {};
Object.assign(a, { x: 3 });

// good
const a = { x: null };
a.x = 3;
```

如果对象的属性名是动态的，可以在创建对象的时候，使用属性表达式定义。

``` js
// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```

上面代码中，对象`obj`的最后一个属性名，需要计算得到，这时，最好采用属性表达式，在新建`obj`的时候，将该属性与其他属性定义在一起。这样一来，所有属性就在一个地方定义了。

另外，对象的属性和方法，尽量采用简洁表达法，这样易于描述和书写。

``` js
var ref = 'some value';

// bad
const atom = {
  ref: ref,
  value: 1,
  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  ref,
  value: 1,
  addValue(value) {
    return atom.value + value;
  },
};
```

---------------------

### 5. 数组
使用扩展运算符(...)拷贝数组。

``` js
// bad
const len = items.length;
const itemsCopy = [];
let i;
for (i = 0; i < len; i++){
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

**使用Array.from**方法，将类似数组的对象转为数组。

``` js
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```

--------------------

### 6. 函数
立即执行函数可以写成箭头函数的形式

``` js
(() => {
  return x * x;
})();
```

那些需要使用函数表达式的场合，尽量使用箭头函数代替，因为这样更简洁，而且绑定了this。

``` js
// bad
[1, 2, 3].map(function (x) {
  return x * x;
});

// good
[1, 2, 3].map((x) => {
  return x * x;
});

// best
[1, 2, 3].map(x => x * x);
```

箭头函数取代`Function.prototype.bind`,不应再用self/_this/that绑定this。

``` js
// bad
const self = this;
const boundMethod = function(...params) {
  return method.apply(self, params);
}

// acceptable
const boundMethod = method.bind(this);

// best
const boundMethod = (...params) => method.apply(this, params);
```

简单的、单行的、不会复用的函数，建议采用箭头函数。如果函数体较为复杂，行数较多，还是应该采用传统的函数写法。

所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数。

``` js
// bad
function divide(a, b, option = false){

}

// good
function divide(a, b, { option = false} = {}) {

}
```

不要在函数体内使用arguments变量，使用rest运算符(...)代替。因为rest运算符显式表明你想要获取参数，而且arguments是一个类似数组的对象，而rest运算符可以提供一个真正的数组。

``` js
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

使用默认值语法设置函数参数的默认值

``` js
// bad
function handleThings(opts) {
  opts = opts || {};
}

// good
function handleThings(opts = {}) {
  // ...
}
```

---------------------

### 7. Map结构
注意区分Object和Map，只有模拟现实世界的实体对象时，才使用Object。如果只是需要`key:value`的数据结构，使用Map结构。因为Map有内建的遍历机制。

``` js
let map = new Map(arr);

for (let key of map.keys()) {
  console.log(key);
}

for (let value of map.values()) {
  console.log(value);
}

for(let item of map.entries()) {
  console.log(item[0], item[1]);
}
```

-------------------

### 8. Class
总是用Class，取代需要prototype的操作。因为Class的写法更简洁，更易于理解。

``` js
// bad
function Queue(contents = []) {
  this._queue = [...contents];
}

Queue.prototype.pop = function() {
  const value = this._queue[0];
  this._queue.splice(0, 1);
  return value;
}

// good
class Queue {
  constructor(contents = []) {
    this._queue = [...contents];
  }
  pop() {
    const value = this._queue;
    this._queue.splice(0, 1);
    return value;
  }
}
```

使用`extends`实现继承，因为这样更简单，不会有破坏`instanceof`运算的危险。

``` js
// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
  Queue.apply(this, contents);
}

inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function() {
  return this._queue[0];
}

// good
class PeekableQueue extends Queue {
  peek() {
    return this._queue[0];
  }
}
```

----------------

### 9. 模块
首先，Module语法时JavaScript模块的标准写法，坚持使用这种写法。使用`import`取代`require`。

``` js
// bad
const moduleA = require('moduleA');
const func1 = moduleA.func1;
const func2 = moduleA.func2;

// good
import { func1, func2 } from 'moduleA';
```

使用`export`取代`module.exports`。

``` js
// commonJS的写法
var React = require('react');

var Breadcrumbs = React.creatClass({
  render() {
    return <nav />;
  }
});

module.exports = Breadcrumbs;

//ES6的写法
import React from 'react';

class Breadcrumbs extends React.Component {
  render() {
    return <nav />;
  }
};

export default Breadcrumbs;
```

如果模块只有**一个输出值**，就使用`export default`,如果模块有**多个输出值**，就不是使用`export default`，`export default`和普通的`export`不要同时使用。

不要在模块输入中使用通用符。因为这样可以确保你的模块之中，有一个默认输出(export default)。

``` js
// bad
import * as myObject from './importModule';

// good
import myObject from './importModule';
```

如果模块默认**输出一个函数**，函数名的首字母应该小写。

``` js
function makeStyleGuide() {

}

export default makeStyleGuide;
```

如果模块默认**输出一个对象**，对象名的首字母应该大写。

``` js
const StyleGuide = {
  es6: {

  }
};

export default StyleGuide
```

--------------

### 10. ESLint的使用
ESLint是一个语法规则和代码风格的检查工具，可以用来保证写出语法正确、风格统一的代码。

首先，安装ESLint。

``` command
npm i -g eslint
```

然后，安装Airbnb语法规则，以及import、a11y、react插件。

``` command
npm i -g eslint-config-airbnb
npm i -g eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react
```

最后，在项目的根目录下新建一个`.eslintrc`文件，配置ESLint。

``` json
{
  "extends": "eslint-config-airbnb"
}
```

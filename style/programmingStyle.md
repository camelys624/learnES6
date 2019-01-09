# 编程风格
很多公司和组织公开了它们的风格规范，下面内容主要参考了[Airbnb](https://github.com/airbnb/javascript)公司的JavaScript风格规范。

### 1. 块级作用域

#### (1) let取代var
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

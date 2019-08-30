# Set 和 Map 数据结构

## 1. Set

### 基本用法

ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。`Set`本身是一个构造函数，用来生成 Set 数据结构。

```js
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
    console.log(i)
}
// 2 3 5 4
```

上面代码通过`add()`方法向 Set 结构加入成员，结果表明 Set 结构不会添加重复的值。

`Set`函数可以接受一个数组 ( 或者具有 iterable 接口的其他数据结构 ) 作为参数，用来初始化。

```js
// 例一
const set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 例二
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size  // 5

// 例三
const set = new Set(document.querySelectorAll('div'));
set.size    //

// 类似于
const set = new Set();
document.querySelectorAll('div')
.forEach(div => set.add(div))
set.size
```

上面代码中，例一和例二都是 `Set` 函数接收数组作为参数，例三是接收类似数组的对象作为参数。

上面代码也展示了一种去除数组重复成员的方法。

```js
// 去除数组的重复成员
[...new Set(array)]
```

上面的方法也可以用于，去除字符串里面的重复字符。

```js
[...new Set('abbabbc')].join('')
// "abc"
```

向 Set 加入值的时候，不会发生类型转换，所以`5`和`"5"`是两个不同的值。Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于精确相等运算符（`===`），主要的区别是向 Set 加入值时认为`NaN`等于自身，而精确相等运算符认为`NaN`不等于自身。

```js
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set { NaN }
```

上面代码向 Set 实例添加了两次`NaN`，但是只会加入一个。这表明，在 Set nebula，两个`NaN`是相等的。

另外，两个对象总是不相等的。

```js
let set = new Set()

set.add({});
set.size    // 1

set.add({});
set.size    // 2
```

上面代码表示，由于两个空对象不相等，所以它们被视为两个值。

### Set 实例的属性和方法

Set 结构的实例有以下属性。

- `Set.prototype.constructor`: 构造函数，默认就是`Set`函数。
- `Set.prototypr.size`: 返回`Set`实例的成员总数。

Set 实例的方法分为两大类：操作方法 (用于操作数据)和遍历方法 (用于遍历成员)。下面先介绍四个操作方法。

- `Set.prototype.add(value)`: 添加某个值，返回 Set 结构本身。
- `Set.prototype.delete(value)`: 删除某个值返回一个布尔值，表示删除是否成功。
- `Set.prototype.has(value)`: 返回一个布尔值，表示该值是否为`Set`的成员。
- `Set.prototype.clear()`: 清除所有成员，没有返回值。

`Array.from`方法可以将 Set 结构转为数组。

### 遍历操作

Set 结构的实例有四个遍历方法，可以用于遍历成员。

- `Set.prototype.keys()`: 返回键名的遍历器
- `Set.prototype.values()`: 返回键值的遍历器
- `Set.prototype.entries()`: 返回键值对的遍历器
- `Set.prototype.forEach()`: 使用回调函数遍历每个成员

需要特别指出的是，`Set`的遍历顺序就是插入顺序。这个特性有时非常有用，比如**使用 Set 保存一个回调函数列表，调用时就能保证按照添加顺序调用。**

#### (1)`keys()`, `values()`, `entries()`

`keys`方法、`values`方法、`entries`方法返回的都是遍历器对象。由于 Set 结构没有键名，只有键值 (或者说键名和键值是同一个值)，所以`keys`方法和`values`方法的行为完全一致。

#### (2) `forEach()`

与数组一样

#### (3)遍历的应用

扩展运算符(`...`)内部使用`for...of`循环，所以也可以用于 Set 结构。

而且，数组的`map`和`filter`方法也可以间接用于 Set 了。

```js
let set = new Set([1, 2, 3]);
set = new ([...set].map(x => x * 2))
// 返回 Set 结构：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5])
set = new Set([...set].filter(x => (x % 2) == 0))
// 返回 Set 结构: {2, 4}
```

因此使用 Set 可以很容易地实现**并集(Union)、交集(Intersect)和差集(Difference)**。

```js
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)))
// set {2, 3}

// 差集
let defference = new Set([...a].filter(x => !b.has(x)))
// set {1}
```

## 2. WeakSet

### 含义

WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。

首先，WeakSet的成员**只能是对象**，而不能是其他类型的值。

```js
const ws = new WeakSet();
ws.add(1)
// TypeError: Invalid value used in weak set
ws.add(Symbol())
// TypeError: invalid value used in weak set
```

上面代码试图向 WeakSet 添加一个数值和 `Symbol`值，结果报错，因为 WeakSet 只能放置对象。

WeakSet 种的对象都是弱引用，即立即回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象不在引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在与 WeakSet 之中。

这是因为垃圾回收机制依赖引用计数，如果一个值的引用次数不为`0`，垃圾回收机制就不会释放这块内存。结束使用该值之后，有时会忘记取消引用，导致内存无法释放，今儿可能会引发内存泄露。WeakSet 里面的引用，都不计入垃圾回收机制，所以就不存在这个问题。因此，WeakSet适合临时存放一组对象，一级存放跟对象绑定的信息。只要这些对象在外部消失，它在 WeakSet 里面的引用就会自动消失。

由于上面这个特点，WeakSet 的成员是不适合引用的，因为它会随时消失。另外，由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此 ES6 规定 WeakSet 不可遍历。

### 语法

WeakSet 是一个构造函数，可以使用`new`命令，创建 WeakSet 数据结构。

WeakSet 结构有以下三个方法。

- `WeakSet.prototype.add(value)`: 添加一个新成员
- `WeakSet.prototype.delete(value)`: 删除一个成员
- `WeakSet.prototype.has(value)`: 返回一个布尔值，表示某个值是否还在其中

## 3. Map

### 含义和基本用法

JavaScript 的对象 (Object)，本质上是键值对的集合 (Hash 结构)，但是传统上只能用字符串当做键。这给它的使用带来了很大的限制。

```js
const data = {};
const element = document.getElementById('myDiv');

data[element] = 'metadata'
```

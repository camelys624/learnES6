# 数值的扩展

## 1. 二进制和八进制表示法

ES6 提供了二进制和八进制数值的新的写法，分别是前缀 `0b` (或`0B`) 和 `0o` (或`0O`) 表示。

```js
0b111110111 === 503 // true
0o767 === 503   // true
```

从 ES5 开始，在严格模式之中，八进制就不再允许使用前缀`0`表示，ES6进一步明确，要使用前缀`0o`表示。

```js
// 非严格模式
(function() {
    console.log(0o11 === 011);
})()    // -> true

// 严格模式
(function() {
    'use strict';
    console.log(0o11 === 011);
})()    // Uncaught SyntaxError: Octal literals are not allowed in strict mode.
```

如果要将 `0b` 和 `0o` 前缀的字符串数值转为十进制，要使用 `Number` 方法。

```js
Number('0b111') // 7
Number('0o10')  // 8
```

## 2. Number.isFinite(),Number.isNaN()

ES6 在 `Number` 对象上，新提供了 `Number.isFinite()` 和 `Number.isNaN()` 两个方法。

`Number.isFinite()` 用来检查一个数值是否为有限的 (finite),即不是 `Infinity`。

```js
Number.isFinite(15);    // true
Number.isFinite(0.8);   // true
Number.isFinite(NaN);   // true
Number.isFinite(Infinity);  // true
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15');  // false
Number.isFinite(true);  // false
```

注意，如果参数类型不是数值， `Number.isFinite` 一律返回 `false`。

`Number.isNaN()` 用来检查一个值是否为 `NaN`。

```js
Number.isNaN(NaN)   // true
Number.isNaN(15)    //false
Number.isNaN(9/NaN) // true
Number.isNaN('true' / 0)
// -> true
Number.isNaN('true'/'true')
// -> true
```

如果参数类型不是 `NaN`,`Number.isNaN`一律返回 `false`。

他们与传统的全局方法 `isFinite()` 和 `isNaN()` 的区别在于，传统方法先调用 `Number()` 将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，`Number.isFinite()` 对于非数值一律返回 `false`，`Number.isNaN()`只有对于 `NaN` 才返回 `true`，非 `NaN` 一律返回 `false`。

## 3. Number.parseInt(),Number.parseFloat()

ES6 将全局方法 `parseInt()` 和 `parseFloat()`，移植到 `Number` 对象上面，行为完全保持不变。

```js
// ES5 的写法
parseInt('12.32')   // 12
parseFloat('123.45#')   // 123.45

// ES6 的写法
Number.parseInt('12.34')    // 12
Number.parseFloat('123.45#')    // 123.45
```
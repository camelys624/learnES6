# let和const命令

### 1. let命令

#### 基本用法
ES6新增了`let`命令，用来声明变量。它的用法类似于`var`，但是所声明的变量，**只在`let`命令所在的代码块内有效**。

``` js
{
  let a = 10;
  var b = 1;
}

a // RederenceError: a is not defined.
b // 1
```

上面代码再代码块之中，分别用`let`和`var`声明了两个变量。然后在代码块之外调用这两个变量，结果`let`声明的变量报错，`var`声明的变量返回了正确的值。这表明，`let`声明的变量只在它所在的代码块有效。

`for`循环的计数器，就很适合使用`let`命令。

``` js
for (let i = 0; i < 10; i++) {
  // do something
}

console.log(i);
// 
```

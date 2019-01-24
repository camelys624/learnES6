### 1. 字符串的Unicode表示法
JavaScript允许采用`\uxxxx`形式表示一个字符，其中`xxxx`表示字符的Unicode码点。

``` js
"\u0061"
// "a"
```

但是这种表示法只限于码点在`\u0000~\uFFFF`之间的字符。超出这个范围的字符，必须用两个双字节的形式表示。

``` js
"\uD842\uDFB7"
// "𠮷"

"\u20BB7"
// "7"
```

ES6对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。

有了这种表示法之后，JavaScript共有6种方法表示一个字符。

``` js

'\z' === 'z'  // true
'\172' === 'z'  // true
'\x7A' === 'z'  // true
'\u007A' === 'z'  // true
'\u{7A}' === 'z'  // true
```

### 2. codePointAt()
JavaScript内部，字符以UTF-16的格式存储，每个字符固定为**2**个字节，对于那些需要**4**个字符存储的字符（Unicode码点大于`0xFFFF`的字符），JavaScript会认为它们是两个字符。

ES6提供了`codePointAt`方法，能够正确处理4个字节存储的字符，返回一个字符的码点。

``` js
let s = '𠮷a';

s.codePointAt(0); // 134071
s.codePointAt(1); // 57271

s.codePointAt(2); // 97
```

`codePointAt`方法返回的码点是十进制的，如果想要返回十六进制的值，可以用`toString`方法转换。

`codePointAt`方法是测试一个字符由两个字节还是由四个字节组成的最简单方法。

``` js
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a")  // false
```

### 3. String.fromCodePoint()
ES5提供`String.fromCharCode`方法，用于从码点返回对应字符，但是这个方法不能识别32位UTF-16字符(Unicode编号大于`0xFFFF`)。

``` js
String.fromCharCode(0x20BB7)
// "ஷ"
```

ES6提供`String.fromCodePoint`方法，可以识别大于`0xFFFF`的字符。在作用上，正好与`codePointAt`方法相反。

### 4. 字符串的遍历器接口
ES6为字符串添加了遍历器接口，使得字符串可以被`for...of`循环遍历。

``` js
for (let codePoint of 'foo') {
  console.log(codePoint)
}

// "f"
// "o"
// "o"
```

除了遍历字符串，这个遍历器最大的优点就是可以识别大于`0xFFFF`的码点，传统的`for`循环无法识别这样的码点。

### 5. mormalize()

### 6. includes(),startsWith(),endsWith()
传统上，JavaScript只有`indexof`方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6有提供了三种新方法。

- includes(): 返回布尔值，表示是否找到了参数字符串。
- startWith(): 返回布尔值，表示参数字符串是否在原字符串的头部。
- endsWith(): 返回布尔值，表示参数是否在原字符串的尾部。

``` js
let s = 'Hello world!';

s.startWith('Hello')  // true
s.endsWith('!') // true
s.includes('o') // true
```

这三个方法都支持第二个参数，表示开始搜索的位置。

``` js
let s = 'Hello world!';

s.startWith('world', 6) // true
s.endsWith('Hello', 5)  // true
s.includes('Hello', 6)  // false
```

上面代码表示，使用第二个参数`n`时，`endsWith`的行为与其他两个方法有所不同。它针对前`n`个字符，而其他两个方法针对从第`n`个位置直到字符串结束。

### 7. repeat()
`repeat`方法返回一个新字符串，表示将原字符串重复`n`次。

``` js
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0)  // ""
```

# Date

## 获取一个月有多少天

获取一个指定月份的天数，比如2019年6月20日

```js
const days = new Date(2019, 6, 0).getDate();
console.log(days) // 30
```

## 获取指定日期为周几

获取一个指定的日期为星期几，例如2019年6月20日

```js
const day = new Date('2019-6-20').getDay();
console.log(day)  // 4

// 0代表周日，日期中间的'-'也可以修改为'/'
```

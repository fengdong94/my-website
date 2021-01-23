---
id: trick
title: 奇技淫巧
---

## 交换变量

```js
// 结构赋值
let a = 1;
let b = 3;
[a, b] = [b, a];

// 求和
let a = 1;
let b = 3;
a = a + b;
b = a - b;
a = a - b;

// 创建新变量
```

## 数组去重

```js
[...new Set(arr)]
```

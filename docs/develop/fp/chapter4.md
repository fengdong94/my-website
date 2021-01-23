---
id: chapter4
title: 第 4 章：柯里化（curry）
---

**curry 的概念很简单：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。**

```jsx
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

curry 提供“预加载”函数的能力，通过传递一到两个参数调用函数，就能得到一个记住了这些参数的新函数：

```jsx
var curry = require('lodash').curry;

var match = curry(function(what, str) {
  return str.match(what);
});
var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var hasSpaces = match(/\s+/g);
hasSpaces("hello world");

var findSpaces = filter(hasSpaces);
findSpaces(["tori_spelling", "tori amos"]);
```

作用：

- 能够大量减少样板文件代码

    ```jsx
    var curry = require('lodash').curry;
    var map = curry(function(f, ary) {
      return ary.map(f);
    });

    var getChildren = function(x) {
      return x.childNodes;
    };
    var allTheChildren = map(getChildren);  // allTheChildren 可复用

    // 普通的 lodash map 需要传递两个参数
    var allTheChildren = function(elements) {
      return _.map(elements, getChildren);
    };
    ```

- 保留了数学的函数定义

    每传递一个参数调用函数，就返回一个新函数处理剩余的参数。这正是一个输入对应一个输出。
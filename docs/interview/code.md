---
id: code
title: 代码实现篇
---

## 防抖和节流

```js
function debounce(fn, delay) {
  let timer;

  return function() {
    // 保存函数调用时的上下文和参数，传递给 fn
    const context = this;
    const args = arguments;

    // 每次这个返回的函数被调用，就清除定时器，以保证不执行 fn
    clearTimeout(timer);

    timer = setTimeout(function() {
      fn.apply(context, args);
    }, delay);
  };
};

function throttle(fn, delay) {
  let timer;

  return function() {
    // 保存函数调用时的上下文和参数，传递给 fn
    const context = this;
    const args = arguments;

    if (!timer) {
      timer = setTimeout(function() {
        fn.apply(context, args);
        timer = null;
      }, delay);
    }
  };
};
```

## Promise.all

```js
Promise.all = function(promises) {
  return new Promise(function(resolve, reject) {
    // 数据格式校验
    if (!Array.isArray(promises)) {
      reject(new TypeError('arguments must be an array'));
    }

    let resolvedCounter = 0;
    const promiseNum = promises.length;
    const result = new Array(promiseNum);

    for (let i = 0; i < promiseNum; i++) {
      // Promise.resolve 统一处理，因为 promises 里可能有非 promise 值
      Promise.resolve(promises[i])
      .then(function(value) {
        resolvedCounter++;
        result[i] = value;

        if (resolvedCounter === promiseNum) {
          resolve(result);
        }
      }, function(reason) {
        reject(reason);
      });
    }
  });
};
```

## Promise

```js
function Promise() {

};
```

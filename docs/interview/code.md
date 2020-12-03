---
id: code
title: 手写代码篇
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

## 数组扁平化

```js
const flatten = (arr, res = []) => {
  // forEach 遍历数组会自动跳过空元素，for of 不会
  arr.forEach((item) => {
    if (Array.isArray(item)) {
      flatten(item, res);
    } else {
      res.push(item);
    }
  });

  return res;
}
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
function Promise(fn) {
  let data = undefined;
  let reason = undefined;
  let status = 'pending';
  const succCbs = [];
  const failCbs = [];

  this.then = function(fulfilled, rejected) {
    return new Promise(function(resolve, reject) {
      function succ(value) {
        const ret =
          (typeof fulfilled === 'function') ? fulfilled(value) || value;

        if (ret && typeof ret.then === 'function') { // 判断then中的返回的是否是promise对象，如果是注册then方法
          ret.then(function(value) {
            resolve(value);
          });
        } else {
          resolve(ret);
        }
      }

      function fail(reason) {
        reason =
          (typeof rejected === 'function') ? rejected(reason) || reason;
        reject(reason);
      }

      if (status === 'pending') {
        succCbs.push(succ);
        failCbs.push(fail);
      } else if (status === 'fulfilled') {
        succ(data);
      } else {
        fail(reason);
      }
    };
  }

  function resolve(value) {
    setTimeout(function() { // 加入延时，确保同步的resolve在then之后执行（这里是宏任务和正常promise有区别）
      status = 'fulfilled';
      data = value;
      succCbs.forEach((cb) => cb(value));
    }, 0);
  }

  function reject(value) {
    setTimeout(function() { // 加入延时，确保同步的resolve在then之后执行（这里是宏任务和正常promise有区别）
      status = 'rejected';
      reason = value;
      failCbs.forEach((cb) => cb(value));
    }, 0);
  }

  fn(resolve, reject);
};
```

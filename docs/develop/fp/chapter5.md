---
id: chapter5
title: 第 5 章：代码组合（compose）
---

### 什么是组合

```jsx
var compose = function(f,g) {
  return function(x) {
    return f(g(x));
  };
};
```

f 和 g 都是函数，x 是在它们之间通过“**管道**”传输的值。

在 compose 的定义中，g 将先于 f 执行，因此就创建了一个**从右到左的数据流**。这样做的**可读性**远远高于嵌套一大堆的函数调用。

### 结合律

```jsx
var associative = compose(f, compose(g, h)) == compose(compose(f, g), h);
```

结合律的一大好处是任何一个函数分组都可以被拆开来，然后再以它们自己的组合方式打包在一起。

```jsx
var loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

// 或
var last = compose(head, reverse);
var loudLastUpper = compose(exclaim, toUpperCase, last);

// 或
var last = compose(head, reverse);
var angry = compose(exclaim, toUpperCase);
var loudLastUpper = compose(angry, last);

// 更多变种...
```

### pointfree

函数无须提及将要操作的数据是什么样的。

pointfree 模式能够帮助我们**减少不必要的命名**，让代码保持简洁和通用。

```jsx
// 非 pointfree，因为提到了数据：word
var snakeCase = function (word) {
  return word.toLowerCase().replace(/\s+/ig, '_');
};

// pointfree
// 利用 curry，我们能够做到让每个函数都先接收数据，然后操作数据，最后再把数据传递到下一个函数那里去。
var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```

### debug

组合的一个常见错误是，在没有局部调用之前，就组合类似 map 这样接受两个参数的函数：

```jsx
// 错误做法：我们传给了 `angry` 一个数组，根本不知道最后传给 `map` 的是什么东西。
var latin = compose(map, angry, reverse);

latin(["frog", "eyes"]);
// error

// 正确做法：每个函数都接受一个实际参数。
var latin = compose(map(angry), reverse);

latin(["frog", "eyes"]);
// ["EYES!", "FROG!"])
```

利用不纯的 trace 函数来追踪代码的执行情况

```jsx
var trace = curry(function(tag, x){
  console.log(tag, x);
  return x;
});

var dasherize = compose(join('-'), toLower, split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');
// TypeError: Cannot read property 'apply' of undefined

var dasherize = compose(join('-'), toLower, trace("after split"), split(' '), replace(/\s{2,}/ig, ' '));
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]

// 啊！toLower 的参数是一个数组，所以需要先用 map 调用一下它
var dasherize = compose(join('-'), map(toLower), split(' '), replace(/\s{2,}/ig, ' '));
dasherize('The world is a vampire');
// 'the-world-is-a-vampire'
```

### 总结

组合像一系列管道那样**把不同的函数联系在一起，数据就可以也必须在其中流动**——毕竟纯函数就是输入对输出，所以打破这个链条就是不尊重输出，就会让我们的应用一无是处。

我们认为组合是高于其他所有原则的设计原则，这是因为**组合让我们的代码简单而富有可读性**。
---
id: chapter3
title: 第 3 章：纯函数的好处
---

### **纯函数**

**纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。**

```jsx
var xs = [1,2,3,4,5];

// 纯的
xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

// 不纯的
xs.splice(0,3);
//=> [1,2,3]

xs.splice(0,3);
//=> [4,5]

xs.splice(0,3);
//=> []

// 另一个例子
// 不纯的
var minimum = 21;

var checkAge = function(age) {
  return age >= minimum;
};

// 纯的
var checkAge = function(age) {
  var minimum = 21;
  return age >= minimum;
};
```

checkAge 的结果将取决于 minimum 这个可变变量的值。换句话说，它取决于系统状态（system state）；这一点令人沮丧，因为它引入了外部的环境，从而增加了**认知负荷（cognitive load）**。

这种依赖状态是影响系统复杂度的罪魁祸首。

也可以让 minimum 成为一个不可变（immutable）对象，这样就能保留纯粹性

```jsx
var immutableState = Object.freeze({
  minimum: 21
});
```

### 副作用

**副作用是在计算结果的过程中，系统状态的一种变化，或者与外部世界进行的可观察的交互。**

副作用可能包含，但不限于：

- 更改文件系统
- 往数据库插入记录
- 发送一个 http 请求
- 可变数据
- 打印/log
- 获取用户输入
- DOM 查询
- 访问系统状态

### **追求“纯”的理由**

在数学上函数是不同数值之间的特殊关系：每一个输入值返回且只返回一个输出值。

**纯函数就是数学上的函数，而且是函数式编程的全部。**

理由：

- **可缓存性（Cacheable）**

    纯函数总能够根据输入来做缓存。实现缓存的一种典型方式是 memoize 技术：

    ```jsx
    var memoize = function(f) {
      var cache = {};

      return function() {
        var arg_str = JSON.stringify(arguments);
        cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
        return cache[arg_str];
      };
    };

    var squareNumber  = memoize(function(x){ return x*x; });

    squareNumber(4);
    //=> 16

    squareNumber(4); // 从缓存中读取输入值为 4 的结果
    //=> 16

    squareNumber(5);
    //=> 25

    squareNumber(5); // 从缓存中读取输入值为 5 的结果
    //=> 25
    ```

    还可以通过延迟执行的方式把不纯的函数转换为纯函数：

    ```jsx
    var pureHttpCall = memoize(function(url, params){
      return function() { return $.getJSON(url, params); }
    });
    ```

- **可移植性／自文档化（Portable / Self-Documenting）**

    自文档化：参数能在最小程度上提供足够多的信息。

    可移植性：通过强迫“注入”依赖，或者把它们当作参数传递，应用更加灵活。

    在 JavaScript 的设定中，可移植性可以意味着把函数序列化（serializing）并通过 socket 发送。也意味着代码能够在 web workers 中运行。总之，可移植性是一个非常强大的特性。

    命令式编程中“典型”的方法和过程都深深地根植于它们所在的环境中，通过状态、依赖和有效作用（available effects）达成；纯函数与此相反，它与环境无关，只要我们愿意，可以在任何地方运行它。

    “面向对象语言的问题是，它们永远都要随身携带那些隐式的环境。你只需要一个香蕉，但却得到一个拿着香蕉的大猩猩...以及整个丛林”。

- **可测试性（Testable）**

    只需简单地给函数一个输入，然后断言输出就好了。

- **合理性（Reasonable）**

    纯函数最大的好处是引用透明性（referential transparency）。

    如果一段代码可以替换成它执行所得的结果，而且是在不改变整个程序行为的前提下替换的，那么我们就说这段代码是**引用透明**的。

    等式推导带来的分析代码的能力对重构和理解代码非常重要（🌰：重构海鸥程序）。

- **并行代码**

    纯函数根本不需要访问共享的内存，也不会因副作用而进入竞争态（race condition）。
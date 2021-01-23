---
id: scope-closure
title: 作用域和闭包
---

## 作用域是什么

**编译原理**

- 分词/词法分析(Tokenizing/Lexing)

    将字符串分解成词法单元(token)

- 解析/语法分析(Parsing)

    将词法单元流(数组)转换成抽象语法树(Abstract Syntax Tree，AST)

- 代码生成

    将AST转换为可执行代码(机器指令)

**引擎、编译器、作用域如何配合**

- 引擎从头到尾负责整个JS 程序的编译及执行过程
- 编译器负责语法分析及代码生成
- 作用域负责收集并维护由所有声明的标识符(变量)组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

变量的赋值操作会执行两个动作，首先编译器在**代码生成时**会在当前作用域中声明一个变量(声明过则忽略当前声明)，然后在**运行时**引擎会在作用域中查找该变量，如果能够找到就会对它赋值。

**LHS、RHS**

- LHS(Left Hand Side 等号在左边)：查找某个变量的值
- RHS(Right Hand Side)：找到变量的容器本身，从而可以对其赋值
- function foo(a) {}; 编译器可以在**代码生成的同时处理声明和值的定义**。因此，将函数声明理解成LHS查询和赋值的形式并不合适。

异常处理

- 如果 RHS 找不到所需的变量，引擎就会抛出 ReferenceError
- LHS 查询时，如果在全局作用域中也无法找到目标变量，全局作用域中就会创建一个变量，并将其返还给引擎。但是严格模式下也会报ReferenceError异常

## 词法作用域

编译的**词法分析阶段**基本能够知道全部标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它们进行查找。

JS 中有两个机制可以“欺骗”词法作用域: eval(..) 和 with

副作用：引擎无法在编译时对作用域查找进行优化，导致代码运行变慢

## 函数作用域和块作用域

为什么需要作用域：

- 遵循最小授权(最小暴露)原则：在软件设计中，应该最小限度地暴露必要内容
- 私有化、规避冲突

如何实现：

- 全局命名空间：在全局作用域中声明一个名字足够独特的变量作为命名空间
- 模块管理：使用模块管理器，任何库都无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将库的标识符显式地导入到另外一个特定的作用域中

### 函数作用域

**函数声明**

在 JS 中定义一个函数，该函数的关键字 function 在整个语句块首部，并且存在函数名称标识符的函数代码称为函数声明。  function foo() {};

函数声明名称标识符(foo)被绑定在所在作用域中

**函数表达式**

在 JS 中定义一个函数，该函数整体作为变量的赋值语句或者调用执行的语句而存在，该函数语句块称为函数表达式。

函数表达式名称标识符(foo)被绑定在自身的函数中，不会污染外部作用域

```jsx
var f = function foo() {
  console.log("这是函数表达式示例");
}
var fAnonymous = function() {
  console.log("这也是函数表达式示例");
}
(function() {
  console.log("这还是函数表达式示例");
}());
```

**匿名函数表达式**

优点：书写简单快捷

缺点：

- 匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难
- 如果没有函数名，当函数需要引用自身时只能使用已经过期的 arguments.callee 引用，比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑自身。
- 代码可读性下降

**立即执行函数表达式**

IIFE (Immediately Invoked Function Expression)

(function() { .. }())、(function() {...})() 两者并无区别

### 块作用域

**with**：块作用域的一种形式

**try/catch**：catch分句会创建一个块作用域，其中声明的变量仅在catch内部有效

**let**

es6 let 关键字可以将变量隐式地绑定到所在的任意作用域中(通常是{ .. }内部)

另一个块作用域非常有用的原因和**闭包及回收内存垃圾**的回收机制相关

```jsx
// click 事件的回调会形成 closure 导致模块内的其他变量不会被垃圾回收
// 但是在块中定义的内容可以销毁

function process(data) { ... }
{
  let someReallyBigData = { .. };
  process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );
btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
});
```

for 循环头部的 let 不仅将 i 绑定到了 for 循环的块中，事实上它将其重新绑定到了循环的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。

## 提升

- 每个作用域都会进行提升操作
- 只有声明本身会被提升，而赋值或其他运行逻辑会留在原地
- 函数声明(包括实际函数的隐含值)会被提升，但是函数表达式却不会被提升

**函数优先**

- 函数会首先被提升，然后才是变量
- 出现在**后面的函数声明可以覆盖前面的**
- 一个普通**块内部**的函数声明通常会被提升到**所在作用域的顶部**  (但是需要注意这个行为并不可靠，在 JS 未来的版本中有可能发生改变，因此应该尽可能避免在块内部声明函数)

## 作用域闭包

**当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。**

闭包场景：定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或者任何其他的异步(或者同步)任务中，只要使用了回调函数，实际上就是在使用闭包!

**循环和闭包**

```jsx
for (var i=1; i<=5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i*1000);
}
// 以每秒一次的频率输出五次 6

// 用IIFE闭包改进
for (var i=1; i<=5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j);
    }, j*1000);
  })(i);
}

// 块作用域
for (var i=1; i<=5; i++) {
  let j = i; // 是的，闭包的块作用域!
  setTimeout(function timer() {
    console.log(j);
  }, j*1000);
}

// 块作用域进化版
// for循环头部的let声明还会有一个特殊的行为。这个行为指出变量在循环过程中不止被声明一次
// 每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量
// 与var不同，外部作用域访问不到i
for (let i=1; i<=5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i*1000);
}
```

### **模块**

```jsx
// 模块：这种代码模式利用了闭包的强大威力
function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }
  function doAnother() {
    console.log(another.join("!"));
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
}

var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3

// 可以将模块函数转换成IIFE来实现单例模式
// 模块也是普通的函数，可以接受参数
```

大多数模块依赖加载器/管理器本质上都是将这种模块定义封装进一个友好的API

```jsx
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
 		}
		// 调用 apply 为了把 deps 作为数组传入（es6 可通过 ... 实现）
 		// apply 函数本身是为了确保this指向函数对象本身
    modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();

MyModules.define( "bar", [], function() {
	function hello(who) {
		return "Let me introduce: " + who;
	}
	return {
		hello: hello
	};
});

MyModules.define( "foo", ["bar"], function(bar) {
	var hungry = "hippo";
	function awesome() {
		console.log(bar.hello(hungry).toUpperCase());
	}
	return {
		awesome: awesome
	};
});

var bar = MyModules.get("bar");
var foo = MyModules.get("foo");

console.log(bar.hello("hippo")); // Let me introduce: hippo
foo.awesome(); // LET ME INTRODUCE: HIPPO
```

ES6 会将文件当作独立的模块来处理，每个模块都可以导入其他模块或特定的API成员，同样也可以导出自己的API成员。

模块文件中的内容会被当作包含在作用域闭包中一样来处理，就和前面介绍的函数闭包模块一样。

差异：

1. 相比之下，ES6 模块 API 更加稳定(API不会在运行时改变)。因此在编译期检查对导入模块的 API 成员的引用是否真实存在，如果 API 引用并不存在，编译器会在运行时抛出“早期”错误，而不会在运行期采用动态的解决方案。
2. 基于函数的模块并不是一个能被稳定识别的模式(编译器无法识别)，它们的 API 语义只有在运行时才会被考虑进来。因此可以在运行时修改一个模块的 API。
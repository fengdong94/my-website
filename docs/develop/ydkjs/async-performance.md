---
id: async-performance
title: 异步和性能
---

## 异步:现在与将来

### 分块的程序

把一段代码包装成一个函数，并指定它在响应某个事件(定时器、鼠标点 击、Ajax 响应等)时执行，你就是在代码中创建了一个将来执行的块，也由此在这个程序 中引入了异步机制。

异步控制台: console.* 不是 JavaScript 正式的一部分，而是由宿主环境添加到 JavaScript 中的。在某些条件下，某些浏览器的 console.log(..) 并不会把传入的内容立即输出。

在许多程序(不只是 JavaScript)中，I/O 是非常低速的阻塞部分。所以，浏览器在后台异步处理控制台 I/O 能够提高性能。(解决方法：使用断点；次优的方案是把对象序列化到一个字符串中，以强制执行一次“快照”，比如通过 JSON.stringify(..) )

### 事件循环

宿主环境提供了一种机制来处理程序中多个块的执行，且执行每块时调用 JavaScript 引擎，这种机制被称为事件循环。JavaScript 引擎本身并没有时间的概念，只是一个按需执行 JavaScript 任意代码片段的环境。事件调度(将代码块(函数)加入队列)总是由包含它的环境进行。(ES6 从本质上改变了在哪里管理事件循环)

setTimeout(..) 可能会在那个时刻运行，也可能在那之后运行，要根据事件队列的状态而定。

```jsx
// eventLoop是一个用作队列的数组
// (先进，先出)
var eventLoop = [ ];
var event;

// JS主线程的同步代码执行完毕后会不断检查事件队列
//“永远”执行
while (true) {
	// 一次tick
	if (eventLoop.length > 0) {
		// 拿到队列中的下一个事件
		event = eventLoop.shift();

		// 现在，执行下一个事件
		try {
			event();
		}
       catch (err) {
       	reportError(err);
       }
	}
}
```

### 并行线程

- 异步是关于现在和将来的时间间隙，而并行是关于能够同时发生的事情。
- 多线程并行如果不经过特殊处理，可能会得到出乎意料的、不确定的行为。比如修改共享的内存地址(修改同一变量)。
- 由于 JavaScript 的单线程特性，函数具有原子性--完整运行(run-to-completion)特性。
- 由于异步产生的函数执行顺序的不确定性就是通常所说的竞态条件(race condition)。

### 并发

- 并发是指两个或多个事件链随时间发展交替执行，以至于从更高的层次来看，就像是同时在运行(尽管在任意时刻只处理一个事件)。
- 宿主环境交叉将不同事件的响应函数扔到事件队列中。（例子：onscroll事件响应和Ajax响应并发）

**非交互**

如果‘进程’间没有相互影响的话，不确定性是完全可以接受的。

**交互**

如果并发的“进程”需要相互交流，通过作用域或 DOM 间接交互，就需要对它们的交互进行协调以避免竞态的出现。

**协作**

取到一个长期运行的“进程”，并将其分割成多个步骤或多批任务，使得其他并发“进程”有机会将自己的运算插入到事件循环队列中交替运行。

```jsx
var res = [];

// response(..)从Ajax调用中取得结果数组
function response(data) {
	// 一次处理1000个
	var chunk = data.splice( 0, 1000 );

	// 添加到已有的res组
	res = res.concat(
		// 创建一个新的数组把chunk中所有值加倍
		chunk.map( function(val){
			return val * 2;
		})
	);

	// 还有剩下的需要处理吗?
	if (data.length > 0) {
		// 异步调度下一次批处理
		setTimeout( function(){
			response( data );
		}, 0 );
	}
}

// ajax(..)是某个库中提供的某个Ajax函数
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

### 任务

- 在ES6中，有一个新的概念建立在事件循环队列之上，叫作任务队列(job queue)。
- 任务队列是挂在事件循环队列的每个tick之后的一个队列。在事件循环的每个tick中，可能出现的异步动作不会导致一个完整的新事件添加到事件循环队列中，而会在当前tick的任务队列末尾添加一个项目(一个任务)。
- Promise的异步特性是基于任务的。

### 语句顺序

JavaScript 引擎在编译代码之后可能会发现通过(安全地)重新安排这些语句的顺序有可能提高执行速度。代码编写的方式(从上到下的模式)和编译后执行的方式之间的联系非常脆弱。

## 回调

- 回调函数包裹或者说封装了程序的延续(continuation)。
- 顺序阻塞式的大脑计划行为无法很好地映射到面向回调的异步代码。

### 信任问题

控制反转(inversion of control)，把自己程序一部分的执行控制交给第三方。

1. 调用回调过早(在追踪之前)
2. 调用回调过晚(或没有调用)
3. 调用回调的次数太少或太多
4. 没有把所需的环境/参数成功传给你的回调函数
5. 吞掉可能出现的错误或异常

回调并没有为我们提供任何东西来支持这一点。我们不得不自己构建全部的机制，而且通常为每个异步回调重复这样的工作最后都成了负担。

可以发明一些特定逻辑来解决这些信任问题，但是其难度高于应有的水平，可能会产生更笨重、更难维护的代码，并且缺少足够的保护。

## Promise

### 什么是 Promise

- Promise是一种封装和组合未来值的易于复用的机制。
- 侦听的 Promise 决议“事件”严格说来并不算是事件(尽管它们实现目标的行为方式确实很像事件)，通过 then(..) 注册一个 "then" 回调。或者可能更精确地说，then(..) 注册 "fullfillment" 和/或 "rejection" 回调。
- new Promise(function(..){ .. })模式通常称为revealing constructor，传入的函数会立即执行。

### 具有then方法的鸭子类型

```jsx
if (
	p !== null &&
	(
		typeof p === "object" ||
		typeof p === "function"
	) &&
	typeof p.then === "function"
) {
	// 假定这是一个thenable!
}
else {
	// 不是thenable
}
```

### Promise 信任问题

**调用过早**

Promise不必担心这种问题，因为即使是立即完成的Promise(类似于 new Promise(function(resolve){ resolve(42); }))也无法被同步观察到。不再需要插入你自己的 setTimeout(..,0) hack。

**调用过晚**

- Promise 创建对象调用 resolve(..) 或 reject(..) 时，这个 Promise 的 then(..) 注册的观察回调就会被自动调度。可以确信，这些被调度的回调在下一个异步事件点上一定会被触发。
- Promise 里再resolve Promise会异步地展开，在异步任务队列中顺序靠后。

**回调未调用**

- 没有任何东西能阻止 Promise 向你通知它的决议(如果它决议了的话)。
- 如果 Promise 本身永远不被决议呢?即使这样，Promise 也提供了解决方案，其使用了一种称为竞态的高级抽象机制。

**调用次数过少或过多**

- Promise 只能被决议一次，所以任何通过 then(..) 注册的(每个)回调就只会被调用一次。

**未能传递参数/环境值**

- 如果使用多个参数调用 resovle(..) 或者 reject(..)，第一个参数之后的所有参数都会被默默忽略。

**吞掉错误或异常**

- Promise 甚至把 JavaScript 异常也变成了异步行为，进而极大降低了竞态条件出现的可能。

**是可信任的 Promise 吗**

Promise.resolve(..) 提供了可信任的 Promise 封装工具：

- 如果向 Promise.resolve(..) 传递一个非 Promise、非 thenable 的立即值，就会得到一个用这个值填充的promise。
- 如果向 Promise.resolve(..) 传递一个真正的Promise，就只会返回同一个promise。
- 如果向 Promise.resolve(..) 传递了一个非 Promise 的 thenable 值，前者就会试图展开这个值，而且展开过程会持续到提取出一个具体的非类 Promise 的最终值。

### 链式流

- 调用 Promise 的 then(..) 会自动创建一个新的 Promise 从调用返回。
- 在完成或拒绝处理函数内部，如果返回一个值或抛出一个异常，新返回的(可链接的) Promise 就相应地决议。
- 如果完成或拒绝处理函数返回一个 Promise，它将会被展开，这样一来，不管它的决议值是什么，都会成为当前 then(..) 返回的链接 Promise 的决议值。
- reject(..) 不会像 resolve(..) 一样进行展开。如果向 reject(..) 传入一个 Promise/thenable 值，它会把这个值原封不动地设置为拒绝理由。

### 错误处理

- try..catch 无法跨异步操作工作，需要一些额外的环境支持(生成器)。
- 多级 error-first 回调交织在一起，再加上这些无所不在的 if 检查语句，都不可避免地导致了回调地狱的风险。
- Promise 没有采用流行的 error-first 回调设计风格，而是使用了分离回调(split-callback)风格。

**绝望的陷阱**

- 为了避免丢失被忽略和抛弃的 Promise 错误，一些开发者表示，Promise 链的一个最佳实践就是最后总以一个 catch(..) 结束。
- 任何 Promise 链的最后一步，不管是什么，总是存在着在未被查看的 Promise 中出现未捕获错误的可能性。

**处理未捕获的情况**

- 定义一个某个时长的定时器，比如3秒钟，在拒绝的时刻启动。如果 Promise 被拒绝，而在定时器触发之前都没有错误处理函数被注册，那它就会假定你不会注册处理函数，进而就是未被捕获错误。
- 更常见的一种看法是:Promsie 应该添加一个 done(..) 函数。

### Promise 模式

**Promise.all([ .. ])**

- 列表中的每个值都会通过 Promise. resolve(..) 过滤。
- 如果数组是空的，主 Promise 就会立即完成。

**Promise.race([ .. ])**

- 如果传入了一个空数组，主 race([..]) Promise 永远不会决议，而不是立即决议。

**all([ .. ]) 和 race([ .. ]) 的变体**

- none([ .. ]) any([ .. ]) first([ .. ]) last([ .. ])

**并发迭代**

- Promise.map

### Promise 局限性

**顺序错误处理**

- 没有为 Promise 链序列的中间步骤保留的引用。

**单一值**

- 有了‘解构’后，将重复样板代码量保持在了最小。

**单决议**

- Promise 只能被决议一次(完成或拒绝)。
- 很多异步的情况适合另一种模式——一种类似于事件和/或数据流的模式。如果不在 Promise 之上构建显著的抽象，Promise 肯定完全无法支持多值决议处理。

**惯性**

- 把需要回调的函数封装为支持 Promise 的函数。多数库都提供了这样的支持，解决 Promise 这个特定的限制都不需要太多代价(可对比回调地狱给我们带来的痛苦!)。

**无法取消的 Promise**

- 单独的 Promise 不应该可取消，但是取消一个可序列是合理的，因为你不会像对待 Promise 那样把序列作为一个单独的不变值来传送。

**Promise 性能**

- Promise 使所有一切都成为异步的了，即有一些立即(同步)完成的步骤仍然会延迟到任务的下一步。
- Promise 稍慢一些，但是作为交换，你得到的是大量内建的可信任性、对 Zalgo 的避免以及可组合性。

## 生成器

生成器是一类特殊的函数，可以一次或多次启动和停止，并不一定非得要完成。

**输入和输出**

- 生成器函数可以接受参数(即输入)，也能够返回值(即输出)。
- 调用生成器函数会创建一个迭代器对象。
- 第一个 next(参数会被忽略) 总是启动一个生成器，并运行到第一个 yield 处。第二个 next 调用完成第一个被暂停的 yield 表达式，第三个 next 调用完成第二个 yield，以此类推。
- 消息是双向传递的——yield.. 作为一个表达式可以发出消息响应 next(..) 调用，next(..) 也可以向暂停的 yield 表达式发送值。
- 如果生成器中没有 return 的话——在生成器中和在普通函数中一样，return 不是必需的——总有一个假定的/隐式的return;(也就是return undefined;)，它会在默认情况下响应对应的 next(..) 调用。

**多个迭代器**

- 每次构建一个迭代器，实际上就隐式构建了生成器的一个实例，通过这个迭代器来控制的是这个生成器实例。

### 生成器产生值

**生产者与迭代器**

迭代器是一个定义良好的接口，用于从一个生产者一步步得到一系列值。

```jsx
// 迭代器 something
var something = (function(){
	var nextVal;
	return {
		// for..of循环需要
		[Symbol.iterator]: function(){ return this; },
		// 标准迭代器接口方法
		next: function(){
			if (nextVal === undefined) {
				nextVal = 1;
			}
			else {
	 			nextVal = (3 * nextVal) + 6;
			}
			return { done:false, value:nextVal };
		}
	};
})();
something.next().value; // 1
something.next().value; // 9
something.next().value; // 33
something.next().value; // 105

// ES6还新增了一个 for..of 循环，这意味着可以通过原生循环语法自动迭代标准迭代器:
// for..of 循环在每次迭代中自动调用 next()，它不会向 next() 传入任何值，
// 并且会在接收到 done:true 之后自动停止。
for (var v of something) {
	console.log( v );
	// 不要死循环!
	if (v > 500) {
		break;
	}
}
// 1 9 33 105 321 969
```

许多 JavaScript 的内建数据结构(从ES6开始)，比如array，也有默认的迭代器。

**iterable**

- 从一个iterable中提取迭代器的方法是：iterable必须支持一个函数，其名称是专门的ES6符号值 Symbol.iterator。调用这个函数时，它会返回一个迭代器。

**生成器迭代器**

- 生成器本身并不是iterable，尽管非常类似——当你执行一个生成器，就得到了一个迭代器。
- 生成器的迭代器也有一个 Symbol.iterator 函数，基本上这个函数做的就是 return this。

```jsx
function *something() {
	var nextVal;
	while (true) {
		if (nextVal === undefined) {
			nextVal = 1;
		}
		else {
			nextVal = (3 * nextVal) + 6;
		}
		yield nextVal;
	}
}
// 生成器会在每个 yield 处暂停，函数 *something() 的状态(作用域)会被保持，
// 即意味着不需要闭包在调用之间保持变量状态。
```

- for..of 循环的“异常结束”(也就是“提前终止”)，通常由 break、return 或者未捕获异常引起，会向生成器的迭代器发送一个信号使其终止(防止挂起)。
- 如果在生成器内有 try..finally 语句，它将总是运行，即使生成器已经外部结束。
- 调用 it.return(..) 之后，它会立即终止生成器，finally 语句仍然会运行。

### 异步迭代生成器

```jsx
function foo(x,y) {
	ajax("http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
				// 向*main()抛出一个错误
				it.throw( err );
			}
			else {
				// 用收到的data恢复*main()
				it.next( data );
			}
		}
	);
}
function *main() {
	try {
		var text = yield foo( 11, 31 ); // foo(11,31)，它没有返回值(即返回undefined)
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
var it = main();
// 这里启动!
it.next();
```

同步错误处理

```jsx
// 可以通过 throw(..) 抛入生成器一个错误，基本上也就是给生成器一个处理它的机会;
// 如果没有处理的话，迭代器代码就必须处理。
function *main() {
	var x = yield "Hello World";
	// 永远不会到达这里
	console.log( x );
}
var it = main();
it.next();
try {
	// *main()会处理这个错误吗?看看吧!
	it.throw( "Oops" );
}
catch (err) {
	// 不行，没有处理!
	console.error( err ); // Oops
}
```

### 生成器 + Promise

**支持 Promise 的 Generator Runner**

```jsx
function run(gen) {
	var args = [].slice.call( arguments, 1), it;
	// 在当前上下文中初始化生成器
	it = gen.apply( this, args );
	// 返回一个promise用于生成器完成
	return Promise.resolve()
		.then( function handleNext(value){
			// 对下一个yield出的值运行
			var next = it.next( value );
			return (function handleResult(next){
				// 生成器运行完毕了吗?
				if (next.done) {
					return next.value;
				}
				// 否则继续运行
				else {
					return Promise.resolve( next.value )
						.then(
						// 成功就恢复异步循环，把决议的值发回生成器
						handleNext,
						// 如果value是被拒绝的 promise，
						// 就把错误传回生成器进行出错处理
						function handleErr(err) {
							return Promise.resolve(
								it.throw( err )
							)
							.then( handleResult );
						}
					);
				}
			})(next);
	});
}

// => ES7 async/await
```

**生成器中的 Promise 并发**

```jsx
// r1、r2依次执行
var r1 = yield request( "http://some.url.1" );
var r2 = yield request( "http://some.url.2" );

// 让两个请求"并行"
var p1 = request( "http://some.url.1" );
var p2 = request( "http://some.url.2" );
// 等待两个promise都决议
var r1 = yield p1;
var r2 = yield p2;

// 让两个请求"并行"，并等待两个promise都决议
var results = yield Promise.all([
	request( "http://some.url.1" ),
	request( "http://some.url.2" )
]);
var [r1,r2] = results;

var r3 = yield request( "http://some.url.3/?v=" + r1 + "," + r2 );
```

- 隐藏的 Promise：把 Promise 逻辑隐藏在一个只从生成器代码中调用的函数内部。这样更简洁也更清晰。

### 生成器委托

- yield * __
- 可以 yield 委托到任意 iterable，yield *[1,2,3] 会消耗数组值 [1,2,3] 的默认迭代器。

**为什么用委托**

yield 委托的主要目的是代码组织，以达到与普通函数调用的对称。

**消息委托**

yield 委托也用于双向消息传递。错误和异常也是双向传递的。

**异步委托**

yield *foo() 代替 yield run(foo)

### 生成器并发

通信顺序进程(Communicating Sequential Processes，CSP)

### 形实转换程序(thunk)

JavaScript 中的 thunk 是指一个用于调用另外一个函数的函数，没有任何参数。

```jsx
function foo(x,y) {
	return x + y;
}
function fooThunk() {
	return foo( 3, 4 );
}
// 将来
console.log( fooThunk() ); // 7

// 异步thunk
function foo(x,y,cb) {
	setTimeout( function(){
		cb( x + y );
	}, 1000 );
}
function fooThunk(cb) {
	foo( 3, 4, cb );
}
// 将来
fooThunk( function(sum){
	console.log( sum ); // 7
} );

function thunkify(fn) {
	var args = [].slice.call( arguments, 1 );
	return function(cb) {
		// 假定原始foo(..)函数原型需要的回调放在最后的位置(callback-last 风格)
		args.push( cb );
		return fn.apply( null, args );
	};
}
var fooThunk = thunkify( foo, 3, 4 );
// 将来
fooThunk( function(sum) {
	console.log( sum );      // 7
} );

// 典型的方法并不是 thunkify(..) 构造 thunk 本身
// 而是 thunkify(..) 工具产生一个生成 thunk 的函数 thunkory
function thunkify(fn) {
	return function() {
		var args = [].slice.call( arguments );
		return function(cb) {
			args.push( cb );
			return fn.apply( null, args );
		};
	};
}
var fooThunkory = thunkify( foo );
var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );
// 将来
fooThunk1( function(sum) {
	// 可以是 error-first 风格的回调
    console.log( sum ); // 7
} );
fooThunk2( function(sum) {
	// 可以是 error-first 风格的回调
    console.log( sum ); // 11
} );
```

- promise和thunk都可以被看作是对一个值的请求，回答可能是异步的。Promise 要比裸 thunk 功能更强、更值得信任。
- Promise 封装工具生成 promisory，而 promisory 则接着产生 Promise。与thunkory 和 thunk 是完全对称的。

### ES6 之前的生成器

- 手动转换：可以通过函数闭包模拟。
- 自动转换：[http://facebook.github.io/regenerator/](http://facebook.github.io/regenerator/)

## 程序性能（宏观）

### Web Worker

- 这是浏览器(即宿主环境)的功能，实际上和 JavaScript 语言本身几乎没什么关系。也就是说，JavaScript 当前并没有任何支持多线程执行的功能。

```jsx
var w1 = new Worker( "http://some.url.1/mycoolworker.js" );
// 这个 URL 应该指向一个 JavaScript 文件的位置，这个文件将被加载到一个 Worker 中。
// 然后浏览器启动一个独立的线程，让这个文件在这个线程中作为独立的程序运行。
```

- Worker 之间以及它们和主程序之间，不会共享任何作用域或资源，那会把所有多线程编程的噩梦带到前端领域，而是通过一个基本的事件消息机制相互联系。
- 如果浏览器中有两个或多个页面(或同一页上的多个 tab !)试图从同一个文件 URL 创建 Worker，那么最终得到的实际上是完全独立的 Worker。

**Worker 环境**

- 在 Worker 内部无法访问主程序的任何资源。但是可以执行网络操作(Ajax、WebSockets)以及设定定时器。还有，Worker 可以访问几个重要的全局变量和功能的本地复本，包括 navigator、location、JSON 和 applicationCache。
- Web Worker 通常应用于以下方面：
    1. 处理密集型数学计算
    2. 大数据集排序
    3. 数据处理(压缩、音频分析、图像处理等)
    4. 高流量网络通信

**数据传递**

- 如果要传递一个对象，可以使用结构化克隆算法。
- 还有一个更好的选择，特别是对于大数据集而言，就是使用 Transferable 对象。

**共享 Worker**

整个站点或 app 的所有页面实例都可以共享的中心 Worker -- SharedWorker。通过端口(port)来得知消息来自于哪个程序。

### SIMD

单指令多数据(SIMD)是一种数据并行(data parallelism)方式，与 Web Worker 的任务并行(task parallelism)相对，因为这里的重点实际上不再是把程序逻辑分成并行的块，而是并行处理数据的多个位。

## 性能测试与调优（微观）

- 与其打造你自己的统计有效的性能测试逻辑，不如直接使用 Benchmark.js 库，它已经为你实现了这些。
- 从尽可能多的环境中得到尽可能多的测试结果以消除硬件 / 设备的偏差，这一点很重要。[jsPerf.com](http://jsperf.com/) 是很好的网站，用于众包性能测试运行。

JavaScript 代码只是对引擎要做什么的提示和建议，而不是逐字逐句的要求，对于具体语法细节的很多执着都没有必要，引擎会自动优化。

```jsx
var x = [ .. ];
// 选择1
for (var i=0; i < x.length; i++) { .. }
// 选择2
for (var i=0, len = x.length; i < len; i++) { .. }

// 在某些像 v8 这样的引擎中，可以看到(http://mrale.ph/blog/2014/12/24/array-length-caching.html)，
// 预先缓存长度而不是让引擎为你做这件事情，会使性能稍微下降一点。
```

- 我们应该关注优化的大局，而不是担心这些微观性能的细微差别。
- “非关键路径上的优化是万恶之源。”所以，关键是确定你的代码是否在关键路径上——如果在的话，就应该优化!
- Number(..) 也是一个函数调用。从行为角度说，它和一元运算符 + 选择是完全一样的，但实际上它可能更慢一些，要求更多的执行函数的机制。当然，也可能 JavaScript 引擎意识到了行为上的相同性，会帮你把 Number(..) 在线化(即 +x)! 沉迷于 +x 与 x | 0 的对比在绝大多数情况下都是浪费时间。这是一个微观性能问题，是一个你不应该让其影响程序可读性的问题。

### 尾调用优化(Tail Call Optimization，TCO)

- 调用一个新的函数需要额外的一块预留内存来管理调用栈，称为栈帧。
- 如果支持 TCO 的引擎能够意识到 foo(y+1) 调用位于尾部，这意味着 bar(..) 基本上已经完成了，那么在调用 foo(..) 时，它就不需要创建一个新的栈帧，而是可以重用已有的 bar(..) 的栈帧。这样不仅速度更快，也更节省内存。
- 尾调用优化是 ES6 要求的一种优化方法。它使 JavaScript 中原本不可能的一些递归模式变得实际。对递归算法来说，引擎不再需要限制栈深度。
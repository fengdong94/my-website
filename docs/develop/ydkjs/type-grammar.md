---
id: type-grammar
title: 类型和语法
---

## 类型

对语言引擎和开发人员来说，类型是值的内部特征，它定义了值的行为，以使其区别于其他值。

### 内置类型

```jsx
typeof undefined === "undefined";
typeof true === "boolean";
typeof 42 === "number";
typeof "42" === "string";
typeof { life: 42 }  === "object";
typeof Symbol() === "symbol";

// 函数是“可调用对象”，它有一个内部属性 [[Call]]，该属性使其可以被调用
// 函数对象的 length 属性是其声明的参数的个数
typeof function a(){ /* .. */ } === "function";

typeof [1,2,3] === "object";

// JS的一个bug
typeof null === "object";
// 需要使用复合条件来检测 null 值的类型
var a = null;
(!a && typeof a === "object");
```

### 值和类型

JavaScript 中的变量是没有类型的，只有值才有。变量可以随时持有任何类型的值。

```jsx
// typeof 运算符总是会返回一个字符串
typeof typeof 42; // "string"
```

对于 undeclared(或者 not defined)变量，typeof 照样返回 "undefined"，并没有报错，这是因为typeof有一个特殊的安全防范机制。

- 通过 typeof 的安全防范机制(阻止报错)来检查 undeclared 变量，有时是个不错的办法。
- 与 undeclared 变量不同，访问不存在的对象属性(甚至是在全局对象 window 上)不会产生 ReferenceError 错误。

## 值

### 数组

- 使用 delete 运算符可以将单元从数组中删除，但是数组的 length 属性并不会发生变化。
- “稀疏”数组(sparse array)
- 有时需要将类数组(一组通过数字索引的值)转换为真正的数组，这一般通过数组工具函数(如 indexOf(..)、concat(..)、forEach(..) 等)来实现。

```jsx
// arguments从 ES6 开始已废止
function foo() {
	var arr = Array.prototype.slice.call( arguments );

	// 用 ES6 中的内置工具函数 Array.from(..) 也能实现同样的功能
	var arr = Array.from( arguments );

	arr.push( "bam" );
	console.log( arr );
}
foo( "bar", "baz" ); // ["bar","baz","bam"]
```

### 字符串

- 字符串和数组很相似，都有 length 属性以及 indexOf(..)(从 ES5 开始数组支持此方法)和 concat(..) 方法。
- JavaScript 中字符串是不可变的，而数组是可变的。字符串不可变是指字符串的成员函数不会改变其原始值，而是创建并返回一个新的字符串。
- 字符串可以通过call、apply借用数组的非变更方法(join、map...)。
- 无法“借用”数组的可变更成员函数(reverse...)，先将字符串转换为数组(split)，待处理完后再将结果转换回字符串(join)。

### 数字

JavaScript 中的数字类型是基于 IEEE 754 标准来实现的，该标准通常也被称为“浮点数”。JavaScript 使用的是“双精度”格式(即 64 位二进制)。

**数字的语法**

- toExponential(): 显示格式"5e+10"
- toFixed(): 指定小数部分的显示位数
- toPrecision(): 指定有效数位的显示位数

**较小的数值**

```jsx
0.1 + 0.2 === 0.3; // false

// 解决方案: 设置一个误差范围值(机器精度 machine epsilon)
// ES6 之前版本polyfill
if (!Number.EPSILON) {
	Number.EPSILON = Math.pow(2,-52);
}

function numbersCloseEnoughToEqual(n1,n2) {
	return Math.abs( n1 - n2 ) < Number.EPSILON;
}
var a = 0.1 + 0.2;
var b = 0.3;
numbersCloseEnoughToEqual( a, b );  // true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 );  // false
```

**整数的安全范围**

- Number.MIN_SAFE_INTEGER
- Number.MAX_SAFE_INTEGER

**整数检测**

- Number.isInteger(..)

```jsx
// polyfill before ES6
if (!Number.isInteger) {
	Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0;
	};
}
```

- Number.isSafeInteger(..)

### 特殊数值

**不是值的值**

- null 指空值(empty value)
- undefined 指没有值(missing value)
- null 是一个特殊关键字，不是标识符，我们不能将其当作变量来使用和赋值。然而 undefined 却是一个标识符，可以被当作变量来使用和赋值。

**undefined**

- 在非严格模式下，我们可以为全局标识符 undefined 赋值，严格模式TypeError。
- 按惯例用void 0来获得undefined（使用void true或其他 void 表达式也可以）。

**特殊的数字**

如果数学运算的操作数不是数字类型(或者无法解析为常规的十进制或十六进制数字)， 就无法返回一个有效的数字，这种情况下返回值为 NaN (“无效数值”“失败数值”或者“坏数值”).

```jsx
var a = 2 / "foo";      // NaN
typeof a === "number";  // true
a == NaN;   // false
a === NaN;  // false
// 唯一一个自反的值
NaN != NaN;   // true
```

```jsx
var a = 2 / "foo";
var b = "foo";
window.isNaN( a ); // true
window.isNaN( b ); // true JS的又一个bug😒

// ES6可以使用工具函数 Number.isNaN(..)
// polyfill before ES6
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return (
			typeof n === "number" && window.isNaN( n )
		);
	};
}

// 更简单的方法，利用 NaN 是JS中唯一一个不等于自身的值这个特点
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return n !== n;
	};
}
```

- 无穷数 Number.NEGATIVE_ INFINITY; Number.POSITIVE_ INFINITY
- 零值 0 -0

**特殊等式**

- ES6 中新加入了一个工具方法 [Object.is](http://object.is/)(..) 来判断两个值是否绝对相等，效率较低，主要用来处理那些特殊的相等比较。

### 值和引用

- 简单值(即标量基本类型值，scalar primitive)总是通过值复制的方式来赋值/传递，包括 null、undefined、字符串、数字、布尔和 ES6 中的 symbol。
- 复合值(compound value)——对象(包括数组和封装对象)和函数，则总是通过引用复制的方式来赋值 / 传递。
- JavaScript 中的引用和其他语言中的引用 / 指针不同，它们不能指向别的变量 / 引用，只能指向值。

## 原生函数

常用的原生函数有: String()、Number()、Boolean()、Array()、Object()、Function()、RegExp()、Date()、Error()、Symbol()。

通过构造函数(如new String("abc"))创建出来的是封装了基本类型值(如"abc")的封装对象。

### 内部属性 [[Class]]

所有 typeof 返回值为 "object" 的对象(如数组)都包含一个内部属性 [[Class]]（一个内部的分类）。

```jsx
Object.prototype.toString.call( [1,2,3] );
// "[object Array]"
Object.prototype.toString.call( /regex-literal/i );
// "[object RegExp]"
```

一般情况下，对象的内部 [[Class]] 属性和创建该对象的内建原生构造函数相对应。

```jsx
Object.prototype.toString.call( "abc" );
// "[object String]"
// 基本类型值被封装对象自动包装，所以它们的内部 [[Class]] 属性值为 "String"
```

### 封装对象包装

基本类型值没有 .length 和 .toString() 这样的属性和方法，需要通过封装对象才能访问，此时 JavaScript 会自动为基本类型值包装一个封装对象。

一般情况下，我们不需要直接使用封装对象。最好的办法是让 JavaScript 引擎自己决定什么时候应该使用封装对象。

new String( a ) 等价于 Object( a )

### 拆封

```jsx
var a = new String( "abc" );
a.valueOf(); // "abc"
```

在需要用到封装对象中的基本类型值的地方会发生隐式拆封。

### 原生函数作为构造函数

**Array(..)**

- 构造函数 Array(..) 不要求必须带 new 关键字。不带时，它会被自动补上。
- Array 构造函数只带一个数字参数的时候，该参数会被作为数组的预设长度(length)，而非只充当数组中的一个元素。
- 虽然Array.apply( null, { length: 3 } )在创建undefined值的数组时有些奇怪和繁琐，但是其结果远比 Array(3) 更准确可靠。

总之，永远不要创建和使用空单元数组。

**Object(..)、Function(..) 和 RegExp(..)**

- 除非万不得已，否则尽量不要使用 Object(..)/Function(..)/RegExp(..)。
- 在实际情况中没有必要使用new Object()来创建对象，因为这样就无法像常量形式那样一次设定多个属性，而必须逐一设定。
- 构造函数 Function 只在极少数情况下很有用，比如动态定义函数参数和函数体的时候。
- 强烈建议使用常量形式(如 /^a*b+/g)来定义正则表达式，这样不仅语法简单，执行效率也更高，因为 JavaScript 引擎在代码执行前会对它们进行预编译和缓存。与前面的构造函数不同，RegExp(..) 有时还是很有用的，比如动态定义正则表达式。

```jsx
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );
var matches = someText.match( namePattern );
```

**Date(..) 和 Error(..)**

Date(..) 和 Error(..) 的用处要大很多，因为没有对应的常量形式来作为它们的替代。

**Symbol(..)**

符号是具有唯一性的特殊值(并非绝对)，用它来命名对象属性不容易导致重名。

```jsx
var mysym = Symbol( "my own symbol" );
mysym;						// Symbol(my own symbol)
mysym.toString();		// "Symbol(my own symbol)"
typeof mysym;  			// "symbol"

var a = { };
// 主要用于命名私有或特殊属性
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

**原生原型**

Function.prototype 是一个函数，RegExp.prototype 是一个正则表达式，而 Array. prototype 是一个数组。

## 强制类型转换

### 值类型转换

将值从一种类型转换为另一种类型通常称为类型转换(type casting)，这是显式的情况;隐式的情况称为强制类型转换(coercion)。

强制类型转换总是返回标量基本类型值。

### 抽象值操作

**ToString**

```jsx
null.toString() === 'null';
undefined.toString() === 'undefined';
true.toString() === 'true';
// 数字的字符串化则遵循通用规则
```

- 如果是对象，进行ToPrimitive抽象操作。
- 普通对象 (valueOf()不返回基本类型值，使用Object.protoType.toString) toString()返回内部属性 [[Class]] 的值，如 "[object Object]"。
- 数组的默认 toString() 方法经过了重新定义，将所有单元字符串化以后再用 "," 连接起来。
- 对大多数简单值来说，JSON 字符串化(stringify)和 toString() 的效果基本相同。
- 安全的 JSON 值：能够呈现为有效 JSON 格式的值。
- 不安全的 JSON 值：undefined、function、symbol和包含循环引用的对象都不符合 JSON结构标准。
- 如果对象中定义了 toJSON() 方法，JSON 字符串化时会首先调用该方法，然后用它的返回值来进行序列化。

**ToNumber**

```jsx
true.toNumber() === 1;
false.toNumber() === 0;
undefined.toNumber() === NaN;
null.toNumber() === 0;
// ToNumber 对字符串的处理基本遵循数字常量的相关规则/语法。
// 处理失败时返回NaN(处理数字常量失败时会产生语法错误)。
```

- 如果是对象，ToPrimitive 抽象操作检查该值是否有 valueOf() 方法。如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就使用 toString() 的返回值(如果存在)来进行强制类型转换。如果 valueOf() 和 toString() 均不返回基本类型值，会产生 TypeError 错误。

**ToBoolean**

JavaScript 中的值可以分为以下两类：

1. 可以被强制类型转换为 false 的值
2. 其他(被强制类型转换为 true 的值)
- 假值(falsy value)：undefined、null、false、+0、-0 和 NaN、""
- 假值对象(falsy object)：document.all
- 真值(truthy value)：假值列表之外的值

### 显式强制类型转换

**字符串和数字之间的显式转换**

String(..) 和 Number(..)：前面没有 new 关键字，并不创建封装对象。遵循toString 和 toNumber。

日期显式转换为数字

```jsx
var timestamp = +new Date();
var timestamp = new Date().getTime();
// 返回结果为 Unix 时间戳，以微秒为单位
// (从 1970 年 1 月 1 日 00:00:00 UTC 到当前时间)
```

**显式解析数字字符串**

```jsx
var a = "42";
var b = "42px";

Number( a );    // 42
Number( b );    // NaN
// 解析
parseInt( a );  // 42
parseInt( b );  // 42

es5之前 parseInt('xx', 10) es5开始默认转换为十进制数。
```

**显式转换为布尔值**

Boolean()、!!

### 隐式强制类型转换

**字符串和数字之间的隐式强制类型转换**

如果 + 的其中一个操作数是字符串(或者通过ToPrimitive可以得到字符串)，则执行字符串拼接；否则执行数字加法。

```jsx
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

// ToPrimitive抽象操作
a + "";         // "42"
//  String(a) 直接调用 ToString()
String( a );    // "4"
```

/ * 运算符会将操作数强制转换成数字类型(或者通过ToPrimitive得到字符串再将其转换成数字类型)。

**隐式强制类型转换为布尔值**

- if (..)语句中的条件判断表达式
- for ( .. ; .. ; .. )语句中的条件判断表达式(第二个)
- while (..) 和 do..while(..) 循环中的条件判断表达式
- ? :中的条件判断表达式
- 逻辑运算符 ||(逻辑或)和 &&(逻辑与)左边的操作数(作为条件判断表达式)

**|| 和 &&**

|| 和 && 首先会对第一个操作数(如果其不是布尔值就先进行 ToBoolean 强制类型转换)执行条件判断。

对于 || 来说，如果条件判断结果为 true 就返回第一个操作数的值，如果为 false 就返回第二个操作数的值。

&& 则相反，如果条件判断结果为 true 就返回第二个操作数的值，如果为 false 就返回第一个操作数的值。

**符号的强制类型转换**

符号不能够被强制类型转换为数字(显式和隐式都会产生错误)，但可以被强制类型转换为布尔值(显式和隐式结果都是 true)。

### 宽松相等和严格相等

== 允许在相等比较中进行强制类型转换，而 === 不允许

**相等比较操作的性能**

仅仅是微秒级的差别而已，不用在乎性能。

**抽象相等**

==

- 如果 Type(x) 是数字，Type(y) 是字符串，则返回 x == ToNumber(y) 的结果。
- 如果 Type(x) 是字符串，Type(y) 是数字，则返回 ToNumber(x) == y 的结果。
- 如果 Type(x) 是布尔类型，则返回 ToNumber(x) == y 的结果。
- 如果 Type(y) 是布尔类型，则返回 x == ToNumber(y) 的结果。
- 在 == 中 null 和 undefined 相等。
- 如果 Type(x) 是字符串或数字，Type(y) 是对象，则返回 x == ToPrimitive(y) 的结果。
- 如果 Type(x) 是对象，Type(y) 是字符串或数字，则返回 ToPromitive(x) == y 的结果。

**比较少见的情况**

- a == 2 && a == 3 可以为true，只需要让a.valueOf()每次调用都产生副作用。
- 隐式强制类型转换在部分情况下确实很危险，这时为了安全起见就要使用 ===

### 抽象关系比较

- 比较双方首先调用 ToPrimitive，如果结果出现非字符串，就根据 ToNumber 规则将双方强制类型转换为数字来进行比较。
- 如果比较双方都是字符串，则按字母顺序来进行比较。
- 为了保证安全，应该对关系比较中的值进行显式强制类型转换。

## 语法

词法(syntax)、语法(grammar)，很多时候二者是同一个意思。

### 语句和表达式

**语句的结果值**

语句都有一个结果值，在代码中是没有办法获得这个结果值的，除了 eval()、do{..}

**表达式的副作用**

- 最常见的有副作用(也可能没有)的表达式是函数调用。
- ++ 在前面时，它的副作用(递增)产生在表达式返回结果值之前，而 a++ 的副作用则产生在之后。

```jsx
var a = 42;
a++;    // 42
a;      // 43
++a;    // 44
a;      // 44

var a = 42, b;
// a++, a 中第二个表达式 a 在 a++ 之后执行，结果为 43，并被赋值给 b
b = ( a++, a );
a;  // 43
b;  // 43
```

- 如果操作成功，delete 返回 true，否则返回 false。其副作用是属性被从对象中删除(或者单元从 array 中删除)。
- a = 42中的=运算符看起来没有副作用，实际上它的结果值是42，它的副作用是将42赋值给 a。

```jsx
var a, b, c;
a = b = c = 42;

function vowels(str) {
	var matches;
	if (str) {
		// 提取所有元音字母
		matches = str.match( /[aeiou]/g );
		if (matches) {
			return matches;
		}
	}
}
// 可以利用赋值语句的副作用将两个 if 语句合二为一
function vowels(str) {
	var matches;
	// 提取所有元音字母
	if (str && (matches = str.match( /[aeiou]/g ))) {
		return matches;
	}
}
```

**上下文规则**

- 大括号：对象常量，标签(带标签的循环跳转一个更大的用处在于，和break __一起使用可以实现从内层循环跳转到外层循环)。
- {"a":42} 作为 JSON 值没有任何问题，但是在作为代码执行时会产生错误(貌似现在已经没问题)，因为它会被当作一个带有非法标签的语句块来执行。foo({"a":42}) 就没有这个问题，因为 {"a":42} 在这里是一个传递给 foo(..) 的对象常量。所以准确地说，JSON-P 能将 JSON 转换为合法的 JavaScript 语法。
- 代码块

```jsx
// {} 出现在 + 运算符表达式中，因此它被当作一个值(空对象)来处理
// [] 会被强制类型转换为 ""
[] + {}; // "[object Object]"

// {} 被当作一个独立的空代码块(不执行任何操作)。代码块结尾不需要分号，所以这里不存在语法上的问题。
//  + [] 将 [] 显式强制类型转换为 0
{} + []; // 0
```

- 对象解构
- else if 和可选代码块：
    1. if 和 else 只包含单条语句的时候可以省略代码块的 { }。
    2. else if 不符合编码规范，else 中是一个单独的 if 语句(省略代码块的 { })。

### 运算符优先级

**短路**

对 && 和 || 来说，如果从左边的操作数能够得出结果，就可以忽略右边的操作数。我们将这种现象称为“短路”(即执行最短路径)。

**更强的绑定**

&& > || > ? :

**关联**

- 右关联不是指从右往左执行，而是指从右往左组合
- ? : 、 = 是右关联，&& 、|| 是左关联

**释疑**

如果 ( ) 有助于提高代码可读性，就使用 ( )。

### 自动分号

- 自动分号插入(Automatic Semicolon Insertion，ASI)只在换行符处起作用，而不会在代码行的中间插入分号。
- 不建议省略分号。

### 错误

- 运行时错误(TypeError、ReferenceError、SyntaxError 等)
- 编译阶段发现的代码错误叫作“早期错误”(early error)，语法错误是早期错误的一种。不过由于没有 GrammarError 类型，一些浏览器选择用 SyntaxError 来代替。
- TDZ(Temporal Dead Zone，暂时性死区)。

### 函数参数

对 ES6 中的参数默认值而言，参数被省略或被赋值为 undefined 效果都一样，都是取该参数的默认值。

### try..finally

finally 中的代码总是会在 try(即使return了) 之后执行，如果有 catch 的话则在 catch 之后执行。

如果 finally 中抛出异常(无论是有意还是无意)，函数就会在此处终止。如果此前 try 中 已经有 return 设置了返回值，则该值会被丢弃。

finally 中的 return 会覆盖 try 和 catch 中 return 的返回值。

### switch

case 表达式的匹配算法与 === 相同；特殊处理 case a == 10: case + 表达式
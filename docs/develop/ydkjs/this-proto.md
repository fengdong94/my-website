---
id: this-proto
title: this 和对象原型
---

## 关于this

为什么要用this：

this 提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将API设计得更加简洁并且易于复用。随着你的使用模式越来越复杂，显式传递上下文对象会让代码变得越来越混乱。

this 在任何情况下都不指向函数的词法作用域。在 JS 内部，作用域确实和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过 JS 代码访问，它存在于 JS 引擎内部。

this到底是什么：

**this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式**。

当一个函数被调用时，会创建一个执行上下文，包含函数在哪里被调用(调用栈)、函数的调用方法、传入的参数等信息。this 就是上下文的其中一个属性，会在函数执行的过程中用到。

## this全面解析

### 绑定规则

**默认绑定**

独立函数调用时会使用默认绑定，this 指向全局对象。严格模式下会绑定到 undefined。

**隐式绑定**

当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。

```jsx
// 调用位置会使用obj上下文来引用函数
function foo() {
	console.log( this.a );
}
var obj = {
	a: 2,
	foo: foo,
};
obj.foo(); // 2

// 隐式丢失
var bar = obj.foo;
bar();  // 此时this指向全局

// 参数传递其实就是一种隐式赋值，结果和上述隐式丢失例子一样。
```

**显式绑定**

call 和 apply: 在某个对象上强制调用函数

如果你传入了一个原始值(字符串类型、布尔类型或者数字类型)来当作 this 的绑定对象，这个原始值会被转换成它的对象形式(也就是new String(..)、new Boolean(..)或者 new Number(..))。这通常被称为“装箱”。

硬绑定: 显式绑定的一个变种

```jsx
// 硬绑定可以解决丢失绑定问题
function foo() {
	console.log(this.a);
}
var obj = { a:2 };

var bar = function() {
	foo.call(obj);
};
bar(); // 2
setTimeout( bar, 100 ); // 2

// 硬绑定的bar不可能再修改它的this
bar.call( window ); // 2
```

```jsx
// 硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值
function foo(something) {
	console.log(this.a, something);
	return this.a + something;
}
var obj = { a:2 };
var bar = function() {
	return foo.apply(obj, arguments);
};
var b = bar( 3 ); // 2 3
console.log( b ); // 5

// 另一种使用方法是创建一个可以重复使用的辅助函数
// 简单的辅助绑定函数(内置的bind更复杂)
function bind(fn, obj) {
	return function() {
		return fn.apply(obj, arguments);
	};
}
var obj = { a: 2 };
var bar = bind(foo, obj);
var b = bar(3); // 2 3
console.log(b); // 5

// Function.prototype.bind 会返回一个硬编码的新函数
// 它会把参数设置为 this 并调用原始函数
```

API调用的“上下文”

```jsx
function foo(el) {
	console.log(el, this.id);
}
var obj = { id: "awesome" };
// 调用 foo(..) 时把this绑定到obj
[1, 2, 3].forEach(foo, obj); // 1 awesome 2 awesome 3 awesome
```

**new绑定**

使用 new 来调用函数，会自动执行下面的操作：

1. 构造一个全新的对象
2. 这个新对象会被执行[[原型]]连接
3. 这个新对象会绑定到函数调用的 this
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象

### 优先级

**默认绑定的优先级最低**

**显示绑定的优先级比隐式绑定更高**

```jsx
function foo() {
	console.log( this.a );
}
var obj1 = {
	a: 2,
	foo: foo
};
var obj2 = {
	a: 3,
	foo: foo
};
obj1.foo(); // 2
obj2.foo(); // 3
obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

**new 绑定比隐式绑定优先级高**

```jsx
function foo(something) {
	this.a = something;
}
var obj1 = { foo: foo };
var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );

console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

**new绑定比显式绑定优先级高**

```jsx
// new和call/apply无法一起使用，因此无法通过new foo.call(obj1)来直接进行测试，
// 但是我们可以使用硬绑定来测试new绑定和显式绑定的优先级。

function foo(something) {
	this.a = something;
}
var obj1 = {};
var bar = foo.bind( obj1 );

bar( 2 );
console.log( obj1.a ); // 2

// 内置的bind会判断硬绑定函数是否是被new调用，
// 如果是的话就会使用新创建的this替换硬绑定的this
var baz = new bar(3);
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

在 new 中使用硬绑定函数进行“柯里化”

```jsx
// 部分应用，是“柯里化”的一种
function foo(p1,p2) {
	this.val = p1 + p2;
}
// 之所以使用 null 是因为在本例中我们并不关心硬绑定的 this 是什么
// 反正使用 new 时 this 会被修改
// bind可入多个参数
var bar = foo.bind( null, "p1" );
var baz = new bar( "p2" );
baz.val; // p1p2
```

### 绑定例外

**被忽略的this**

把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，这些值在调用时会被忽略，实际应用的是默认绑定规则

什么情况下你会传入 null ？

```jsx
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// 把数组“展开”成参数，es6中可用...
foo.apply( null, [2, 3] ); // a:2, b:3

// 使用 bind(..) 进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

然而，总是使用 null 来忽略 this 绑定可能产生一些副作用。如果某个函数确实使用了 this (比如第三方库中的一个函数)，那么会修改全局对象。

**更安全的 this**

```jsx
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// Object.create(null) 不会创建 Object.prototype 这个委托
// 所以它比{}“更空”
var ø = Object.create( null );

// 把数组展开成参数
foo.apply( ø, [2, 3] ); // a:2, b:3

// 使用bind(..)进行柯里化
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

**间接引用**

```jsx
function foo() {
	console.log( this.a );
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
// p.foo = o.foo 的返回值是目标函数的引用
(p.foo = o.foo)(); // 2
```

**软绑定：**给默认绑定指定一个全局对象和 undefined 以外的值

```jsx
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this;
		// 捕获所有 curried 参数
		var curried = [].slice.call( arguments, 1 );
		var bound = function() {
			return fn.apply(
				(!this || this === (window || global)) ? obj : this,
				curried.concat.apply( curried, arguments )
			);
		};
	  bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}

function foo() {
	console.log("name: " + this.name);
}
var obj = { name: "obj" },
	obj2 = { name: "obj2" },
	obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );
fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2

fooOBJ.call( obj3 ); // name: obj3

setTimeout( obj2.foo, 10 ); // name: obj
```

**箭头函数不使用 this 的四种标准规则，而是根据外层作用域来决定 this**

```jsx
function foo() {
	setTimeout(() => {
		// 这里的 this 在此法上继承自 foo()
		console.log( this.a );
	},100);
}
var obj = { a:2 };
foo.call( obj ); // 2

同-----------

function foo() {
	var self = this; // lexical capture of this
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}
var obj = { a: 2 };
foo.call( obj ); // 2
```

## 对象

构造形式和文字(声明)形式生成的对象是一样的。唯一的区别是，在文字声明中你可以添加多个键值对，但是在构造形式中你必须逐个添加属性。

简单基本类型(string、boolean、number、null、symbol 和 undefined)本身并不是对象。对 null 执行typeof null 时会返回字符串"object"，实际上，null本身是基本类型。

JS 中还有一些对象子类型，通常被称为内置对象：String、Number、Boolean、Object、Function、Array、Date、RegExp、Error

```jsx
// 字面量，string基本类型
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false

// String，object子类型
var strObject = new String( "I am a string" );
typeof strObject; // "object"
strObject instanceof String; // true

// 检查 sub-type 对象
Object.prototype.toString.call( strObject ); // [object String]
```

在必要时语言会自动把字符串字面量转换成一个String对象，number同理。

### 内容

.a “属性访问”；["a"]“键访问”

如果使用string(字面量)以外的其他值作为属性名，那它首先会被转换为一个字符串

es6新增的 Symbol 类型也可以作为属性名

ES6 增加了可计算属性名，可以在文字形式中使用[]包裹一个表达式来当作属性名

在其他语言中，属于对象(也被称为“类”)的函数通常被称为“方法”，在 JS 中函数永远不会“属于”一个对象，所以把对象内部引用的函数称为“方法”似乎有点不妥。

**数组**

数组和普通的对象都根据其对应的行为和用途进行了优化，所以最好只用对象来存储键/值对，只用数组来存储数值下标/值对。

```jsx
// 如果你试图向数组添加一个属性，但是属性名“看起来”像一个数字
// 那它会变成一个数值下标(因此会修改数组的内容而不是添加一个属性)
var myArray = [ "foo", 42, "bar" ];
myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```

**复制对象**

对于 JSON 安全(也就是说可以被序列化为一个 JSON 字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象)的对象来说，有一种巧妙的复制方法:

```jsx
// 不适用于有函数属性的对象
var newObj = JSON.parse(JSON.stringify(someObj));
```

ES6 定义了 Object.assign(..) 方法来实现浅复制，它会遍历一个或多个源对象的所有可枚举的自有键并把它们复制(使用=操作符赋值)到目标对象，最后返回目标对象。源对象属性的一些特性(比如 writable)不会被复制到目标对象。

**属性描述符**

```jsx
Object.defineProperty(myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
});
```

- writable: 为 false 时对于属性值的修改会静默失败，严格模式下 TypeError
- Configurable: 不管是不是处于严格模式，尝试修改一个不可配置的属性描述符都会TypeError，delete 一个不可配置的属性会静默失败。
- Enumerable: 可枚举性

**不变性**

所有的方法创建的都是浅不变性

1. 对象常量: 结合 writable:false 和 configurable:false 就可以创建一个真正的常量属性(不可修改、 重定义或者删除)
2. 禁止扩展: Object.preventExtensions(..)，创建新属性会静默失败，严格模式下 TypeError
3. 密封: Object.seal(..) = Object.preventExtensions(..) + configurable:false
4. 冻结: Object.freeze(..) = Object.seal(..) + writable:false; 这个方法是你可以应用在对象上的级别最高的不可变性

**[[Get]]、[[Put]]**

myObject.a 在 myObject 上实际上是实现了[[Get]]操作(有点像函数调用:[Get])

如果已经存在这个属性，[[Put]] 算法大致会检查下面这些内容:

1. 属性是否是访问描述符? 如果是并且存在setter就调用setter。
2. 属性的数据描述符中 writable 是否是 false ? 如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
3. 如果都不是，将该值设置为属性的值。

如果对象中不存在这个属性，[[Put]] 操作会更加复杂。

**Getter 和 Setter**

当你给一个属性定义getter、setter或者两者都有时，这个属性会被定义为“**访问描述符**”(和“数据描述符”相对)。

对于访问描述符来说，JS 会忽略它们的 value 和 writable 特性，取而代之的是 set 和 get (还有configurable 和 enumerable)特性。

```jsx
var myObject = {
	// 给 a 定义一个 getter
	get a() {
		return this._a_;
	},
	// 给 a 定义一个 setter
	set a(val) {
		this._a_ = val * 2;
	}
};
myObject.a = 2;
myObject.a; // 4
```

**存在性**

```jsx
var myObject = { a:2 };

// in 操作符会检查属性是否在对象及其 [[Prototype]] 原型链中
("a" in myObject); // true
("b" in myObject); // false

// hasOwnProperty(..) 只会检查属性是否在 myObject 对象中
myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```

枚举

1. for..in
2. myObject.propertyIsEnumerable( "a" ) 会检查给定的属性名是否直接存在于对象中(而不是在原型链上)并且满足 enumerable: true
3. Object.keys(..) 会返回一个数组，包含所有可枚举属性。Object.keys(..) 和 Object.getOwnPropertyNames(..) 都只会查找对象直接包含的属性

(目前)并没有内置的方法可以获取 in 操作符使用的属性列表(对象本身的属性以及 [[Prototype]] 链中的所有属性）。只能递归...

### 遍历

- forEach(..) 会遍历数组中的所有值并忽略回调函数的返回值。every(..) 会一直运行直到回调函数返回false(或者“假”值)，some(..)会一直运行直到回调函数返回true(或者 “真”值)
- 遍历对象属性时的顺序是不确定的，在不同的 JS 引擎中可能不一样
- ES6 增加了一种用来遍历数组的 for..of 循环语法(如果对象本身定义了迭代器的话也可以遍历对象)

```jsx
var myObject = { a: 2, b: 3 };
Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
});

// 手动遍历 myObject
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// 用 for..of 遍历 myObject
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

## 混合对象“类”

面向类的设计模式: **实例化**(instantiation)、**继承**(inheritance)和(相对)**多态**(polymorphism)。

### 类理论

类/继承描述了一种代码的组织结构形式——一种在软件中对真实世界中问题领域的建模方法

面向对象编程强调的是数据和操作数据的行为本质上是互相关联的，因此好的设计就是把数据以及和它相关的行为打包(或者说**封装**)起来

例子：可以应用在字符串上的行为(计算长度、添加数据、搜索，等等)都被设计成 String 类的方法。所有字符串都是 String 类的一个实例，包含字符数据和我们可以应用在数据上的函数。

**多态**：这个概念是说父类的通用行为可以被子类用更特殊的行为重写。实际上，相对多态性允许我们从重写行为中引用基础行为。类理论强烈建议父类和子类使用相同的方法名来表示特定的行为，从而让子类重写父类。但是在 JS 代码中这样做会降低代码的可读性和健壮性。

有些语言(比如 Java)并不会给你选择的机会，类并不是可选的——万物皆是类。其他语言(比如 C/C++ 或者 PHP)会提供**过程化**和**面向类**这两种语法，开发者可以选择其中一种风格或者混用两种风格。

在近似类的表象之下，JS 的机制其实和类完全不同。语法糖和(广泛使用的) JavaScript “类”库试图掩盖这个现实，但是**其他语言中的类和 JS 中的“类”并不一样**。

**类的机制**

类通过复制操作被实例化为对象形式

类实例是由一个特殊的类方法构造的，这个方法名通常和类名相同，被称为**构造函数**。这个方法的任务就是初始化实例需要的所有信息

```jsx
class CoolGuy {
	specialTrick = nothing
	CoolGuy( trick ) {
		specialTrick = trick
	}
	showOff() {
		output( "Here's my trick: ", specialTrick )
	}
}

Joe = new CoolGuy( "jumping rope" )
Joe.showOff() // 这是我的绝技:跳绳
```

**类的继承**

子类会包含父类行为的**原始副本**，但是也可以**重写**所有继承的行为甚至**定义新行为**

```jsx
class Vehicle {
	engines = 1
	// 点火
 	ignition() {
		output( "Turning on my engine." );
	}
 	drive() {
		ignition();
		output( "Steering and moving forward!" )
	}
}

class Car inherits Vehicle {
	wheels = 4
	drive() {
		// super == inherited
		// 相对多态
		inherited:drive()
		output( "Rolling on all ", wheels, " wheels!" )
	}
}

// 快艇
class SpeedBoat inherits Vehicle {
	engines = 2
	ignition() {
		output( "Turning on my ", engines, " engines." )
	}
	pilot() {
		inherited:drive()
		output( "Speeding through the water with ease!" )
	}
}
```

- Car 重写了继承自父类的 drive() 方法，但是之后 Car 调用了 inherited:drive() 方法， 这表明 Car 可以引用继承来的原始 drive() 方法。快艇的 pilot() 方法同样引用了原始 drive() 方法。这个技术被称为多态或者虚拟多态。在本例中，更恰当的说法是相对多态(相对引用 “查找上一层”)。
- 多态并不表示子类和父类有关联，子类得到的只是父类的一份副本。类的继承其实就是复制。

JS 本身并**不提供“多重继承”**功能

### 混入

在继承或者实例化时，JS 的对象机制并不会自动执行复制行为。简单来说， JS 中只有对象，并不存在可以被实例化的“类”。一个对象并不会被复制到其他对象，它们会被关联起来。JS 开发者想出了一个方法来**模拟类的复制行为**，这个方法就是**混入**。

**显式混入**

```jsx
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// 只会在不存在的情况下复制
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}
	return targetObj;
}

var Vehicle = {
	engines: 1,
	ignition: function() {
		console.log( "Turning on my engine." );
	},
	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,
	drive: function() {
		// 显式多态
		Vehicle.drive.call( this );
		console.log("Rolling on all " + this.wheels + " wheels!");
	}
});
```

- JS (在ES6之前)并没有相对多态的机制
- 由于两个对象引用的是同一个函数，因此这种复制(或者说混入)实际上并不能完全模拟面向类的语言中的复制
- 如果你向目标对象中显式混入超过一个对象，就可以部分模仿多重继承行为，但是仍没有直接的方式来处理函数和属性的同名问题

寄生继承：显式混入模式的一种变体被称为“寄生继承”，它既是显式的又是隐式的。

```jsx
//“传统的 JavaScript 类”Vehicle
function Vehicle() {
	this.engines = 1;
}
Vehicle.prototype.ignition = function() {
	console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
	this.ignition();
	console.log( "Steering and moving forward!" );
};

//“寄生类”Car
function Car() {
	// 首先，car 是一个 Vehicle
	var car = new Vehicle();

	// 接着我们对 car 进行定制
	car.wheels = 4;

	// 保存到 Vehicle::drive() 的特殊引用
	var vehDrive = car.drive;

	// 重写 Vehicle::drive()
	car.drive = function() {
		vehDrive.call( this );
		console.log(
			"Rolling on all " + this.wheels + " wheels!"
		);
	}

	return car;
}

var myCar = new Car();
myCar.drive(); // 发动引擎。 // 手握方向盘! // 全速前进!
```

**隐式混入**

```jsx
var Something = {
	cool: function() {
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
	cool: function() {
		// 隐式把 Something 混入 Another
		Something.cool.call( this );
	}
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1(count不是共享状态)
```

### 小结

- 类是一种设计模式。许多语言提供了对于面向类软件设计的原生语法。JS 也有类似的语法，但是和其他语言中的类完全不同。
- 类意味着复制。传统的类被实例化时，它的行为会被复制到实例中。类被继承时，行为也会被复制到子类中。
- 多态(在继承链的不同层次名称相同但是功能不同的函数)看起来似乎是从子类引用父类，但是本质上引用的其实是复制的结果。
- JS 并不会(像类那样)自动创建对象的副本。
- 混入模式(无论显式还是隐式)可以用来模拟类的复制行为，但是通常会产生丑陋并且脆弱的语法，比如显式伪多态(OtherObj.methodName.call(this, ...))，这会让代码更加难懂并且难以维护。
- 此外，显式混入实际上无法完全模拟类的复制行为，因为对象(和函数!别忘了函数也是对象)只能复制引用，无法复制被引用的对象或者函数本身。忽视这一点会导致许多问题。
- 总地来说，在 JavaScript 中模拟类是得不偿失的，虽然能解决当前的问题，但是可能会埋下更多的隐患。

## 原型

### [[Prototype]]

JS 中的对象有一个特殊的 **[[Prototype]] 内置属性**，其实就是对于其他对象的引用。几乎所有的对象在创建时 [[Prototype]] 属性都会被赋予一个非空的值。

所有普通的 [[Prototype]] 链最终都会指向内置的 Object.prototype

**属性设置和屏蔽**

myObject 中包含的 foo 属性会屏蔽原型链上层的所有 foo 属性，因为 myObject.foo 总是会选择原型链中最底层的 foo 属性。

foo 不直接存在于 myObject 中而是存在于原型链上层时 myObject.foo = "bar" 会出现的三种情况:

1. 如果在[[Prototype]]链上层存在名为foo的普通数据访问属性并且没有被标记为只读(writable:false)，那就会直接在 myObject 中添加一个名为 foo 的新属性，它是屏蔽属性。
2. 如果在[[Prototype]]链上层存在foo，但是它被标记为只读，那么无法修改已有属性或者在 myObject 上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
3. 如果在[[Prototype]]链上层存在foo并且它是一个setter，那就一定会调用这个 setter。foo 不会被添加到(或者说屏蔽于)myObject，也不会重新定义 foo 这个 setter。

如果你希望在第二种和第三种情况下也屏蔽 foo，那就不能使用 = 操作符来赋值，而是使用 Object.defineProperty(..)。

### “类”

JS 和面向类的语言不同，它并没有类来作为对象的抽象模式或者说蓝图。

JS 中只有对象。是少有的可以不通过类，直接创建对象的语言。

在 JS 中，类无法描述对象的行为，(因为根本就不存在类!) 对象直接定义自己的行为。

**“类”函数**

“类似类”的行为利用了函数的一种特殊特性: **所有的函数默认都会拥有一个名为 prototype 的公有并且不可枚举的属性**，它会指向另一个对象，这个对象通常被称为 Foo 的原型，因为我们通过名为 Foo.prototype 的属性引用来访问它。

new Foo() 会生成一个新对象，这个新对象的内部链接[[Prototype]]关联的是 Foo.prototype 对象。new Foo() 这个函数调用实际上并没有直接创建关联，这个关联只是一个意外的副作用。new Foo() 只是间接完成了我们的目标: 一个关联到其他对象的新对象。 

Object.create(..)可以更直接地做到这一点。

**[[Prototype]]关联机制通常被称为原型继承**，它常常被视为**动态语言版本的类继承**。

**“构造函数”**

```jsx
// 对于 JavaScript 引擎来说首字母大写没有任何意义，大写假装自己是“面向类”
function Foo() { // ... }
Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true

// Foo.prototype 默认有一个公有并且不可枚举的属性 constructor
// 这个属性引用的是对象关联的函数
// 实际上 a 本身并没有 .constructor 属性
// 而且，虽然 a.constructor 确实指向 Foo 函数，但是这个属性并不是表示 a 由 Foo“构造”。
```

new 会劫持所有普通函数并用构造对象的形式来调用它:

```jsx
function NothingSpecial() {
	console.log( "Don't mind me!" );
}
var a = new NothingSpecial();  // "Don't mind me!"
a; // {}

// 函数不是构造函数，但是当且仅当使用 new 时，函数调用会变成“构造函数调用”。
```

下面这段代码展示了另外两种“面向类”的技巧:

1. this.name = name 给每个对象(也就是 a 和 b，this绑定)都添加了 name 属性，有点像类实例封装的数据值。
2. Foo.prototype.myName = ... ，它会给 Foo.prototype 对象添加一个属性(函数)。

```jsx
function Foo(name) {
	this.name = name;
}
Foo.prototype.myName = function() {
	return this.name;
};
var a = new Foo( "a" );
var b = new Foo( "b" );
a.myName(); // "a"
b.myName(); // "b"
```

### (原型)继承

```jsx
// 原型风格的“类继承”
function Foo(name) {
	this.name = name;
}
Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// 我们创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype
Bar.prototype = Object.create( Foo.prototype );

// 注意!现在没有 Bar.prototype.constructor 了
// 如果你需要这个属性的话可能需要手动修复一下它

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );
a.myName(); // "a"
a.myLabel(); // "obj a"
```

Object.create(..) 会凭空创建一个“新”对象并把新对象内部的 [[Prototype]] 关联到你指定的对象

```jsx
// 引用，Bar.prototype 添加属性时会添加到 Foo.prototype
Bar.prototype = Foo.prototype;

// 基本上满足你的需求，但是可能会产生一些副作用
// (修改状态、注册到其他对象、给 this 添加数据属性，等等)
Bar.prototype = new Foo();

// ES6之前需要抛弃默认的 Bar.prototype (可读性更高)
Bar.ptototype = Object.create( Foo.prototype );

// ES6开始可以直接修改现有的Bar.prototype
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

检查“类”关系(内省、反射)

1. a instanceof Foo; instanceof 操作符的左操作数是一个普通的对象，右操作数是一个函数。instanceof 回答的问题是: 在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 的对象?
2. b.isPrototypeOf( c ); b 是否出现在 c 的 [[Prototype]] 链中?
3. Object.getPrototypeOf( a ); 可以直接获取一个对象的 [[Prototype]] 链。Object.getPrototypeOf( a ) === Foo.prototype;
4. 绝大多数浏览器也支持一种非标准的方法来访问内部 [[Prototype]] 属性，a.proto === Foo.prototype;

### 对象关联

如果在对象上没有找到需要的属性或者方法引用，引擎就会继续在 [[Prototype]] 关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的 [[Prototype]]，以此类推。这一系列对象的链接被称为“**原型链**”。

**创建关联**

Object.create(..) 会创建一个新对象(bar)并把它关联到我们指定的对象(foo)，这样我们就可以充分发挥 [[Prototype]] 机制的威力(委托)并且避免不必要的麻烦(比如使用 new 的构造函数调用会生成 .prototype 和 .constructor 引用)。

Object.create(null) 会创建一个拥有空/null [[Prototype]] 链接的对象，这个对象无法进行委托。这些特殊的空 [[Prototype]] 对象通常被称作“**字典**”，它们完全不会受到原型链的干扰，因此非常适合用来存储数据。

polyfill (before es5)

```jsx
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}

// Object.create(..) 的第二个参数指定了需要添加到新对象中的属性名以及这些属性的属性描述符。
// 因为 ES5 之前的版本无法模拟属性操作符，所以 polyfill 代码无法实现这个附加功能。
```

**内部委托**

```jsx
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};
var myObject = Object.create( anotherObject );
myObject.cool(); // "cool!"

// 内部委托比起直接委托可以让 API 接口设计更加清晰
var myObject = Object.create( anotherObject );
myObject.doCool = function() {
	this.cool(); // 内部委托!
};
myObject.doCool(); // "cool!"
```

## 行为委托

### 面向委托的设计

**类理论**

```jsx
// 面向类(面向对象)

class Task {
	id;

	// 构造函数 Task()
	Task(ID) { id = ID; }
	outputTask() { output( id ); }
}

class XYZ inherits Task {
	label;

	// 构造函数 XYZ()
	XYZ(ID,Label) { super( ID ); label = Label; }
	outputTask() { super(); output( label ); }
}

class ABC inherits Task {
	// ...
}
```

**委托理论**

```jsx
// “对象关联”编码风格，行为委托设计模式

Task = {
	setID: function(ID) { this.id = ID; },
	outputID: function() { console.log( this.id ); }
};

// 让XYZ委托Task
XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
	this.setID( ID );
	this.label = Label;
};
XYZ.outputTaskDetails = function() {
	this.outputID();
	console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

对象并不是按照父类到子类的关系垂直组织的，而是通过任意方向的委托关联并排组织的。

对象关联风格的代码还有一些不同之处：

1. 通常来说，在 [[Prototype]] 委托中最好把状态保存在委托者(XYZ、ABC)而不是委托目标(Task)上
2. 在类设计模式中，我们故意让父类(Task)和子类(XYZ)中都有outputTask方法，这样就可以利用重写(多态)的优势。在委托行为中则恰好相反: 我们会**尽量避免在 [[Prototype]] 链的不同级别中使用相同的命名**，否则就需要使用笨拙并且脆弱的语法来消除引用歧义(call apply)。
3. 由于调用位置触发了 this 的隐式绑定规则，因此虽然 setID(..) 方法在 Task 中，运行时 this 仍然会绑定到 XYZ，这正是我们想要的。

禁止互相委托

**比较思维模型**

```jsx
function Foo(who) {
	this.me = who;
}
Foo.prototype.identify = function() {
	return "I am " + this.me;
};

function Bar(who) {
	Foo.call( this, who );
}
Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```

```jsx
// 只是把对象关联起来，并不需要那些既复杂又令人困惑的模仿类的行为(构造函数、原型以及new)。

Foo = {
	init: function(who) {
		this.me = who;
	},
	identify: function() {
		return "I am " + this.me;
	}
};
Bar = Object.create( Foo );

Bar.speak = function() {
	alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );

b1.speak();
b2.speak();
```

### 类与对象

**控件“类”**

```jsx
// 不使用任何“类”辅助库或者语法的情况下，使用纯 JavaScript 实现类风格

// 父类
function Widget(width,height) {
	this.width = width || 50;
	this.height = height || 50;
	this.$elem = null;
}
Widget.prototype.render = function($where){
	if (this.$elem) {
		this.$elem.css( {
			width: this.width + "px",
			height: this.height + "px"
		}).appendTo( $where );
	}
};

// 子类
function Button(width,height,label) {
	// 调用“super”构造函数
	Widget.call( this, width, height );
	this.label = label || "Default";
	this.$elem = $( "<button>" ).text( this.label );
}

// 让 Button“继承”Widget
Button.prototype = Object.create( Widget.prototype );

// 重写 render(..)
Button.prototype.render = function($where) {
	//“super”调用
	Widget.prototype.render.call( this, $where );
	this.$elem.click( this.onClick.bind( this ) );
};

Button.prototype.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
	var $body = $( document.body );
	var btn1 = new Button( 125, 30, "Hello" );
	var btn2 = new Button( 150, 40, "World" );

	btn1.render( $body );
	btn2.render( $body );
});
```

```jsx
// class 仍然是通过 [[Prototype]] 机制实现的

class Widget {
	constructor(width,height) {
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	}
	render($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
}

class Button extends Widget {
	constructor(width,height,label) {
		super( width, height ); // 相当于 Widget.prototype.constructor.call(this)
		this.label = label || "Default";
		this.$elem = $( "<button>" ).text( this.label );
	}
	render($where) {
		super( $where );
		this.$elem.click( this.onClick.bind( this ) );
	}
	onClick(evt) {
		console.log( "Button '" + this.label + "' clicked!" );
	}
}

$( document ).ready( function(){
	var $body = $( document.body );
	var btn1 = new Button( 125, 30, "Hello" );
	var btn2 = new Button( 150, 40, "World" );

	btn1.render( $body );
	btn2.render( $body );
} );
```

- super()只能用在子类的构造函数之中
- super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类
- ES6 规定，在子类普通方法中通过 super 调用父类的方法时，方法内部的 this 指向当前的子类实例
- 由于 this 指向子类实例，所以如果通过 super 对某个属性赋值，这时 super 就是this；而当读取super.x 的时候，读的是 A.prototype.x

**委托控件对象**

```jsx
var Widget = {
	init: function(width,height){
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	},
	insert: function($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
};

var Button = Object.create( Widget );

Button.setup = function(width,height,label){
	// 委托调用
	this.init( width, height );
	this.label = label || "Default";

	this.$elem = $( "<button>" ).text( this.label );
};

Button.build = function($where) {
	// 委托调用
	this.insert( $where );
	this.$elem.click( this.onClick.bind( this ) );
};

Button.onClick = function(evt) {
	console.log( "Button '" + this.label + "' clicked!" );
};

$( document ).ready( function(){
	var $body = $( document.body );
	// 对象关联可以更好地支持关注分离(separation of concerns)原则
	// 创建和初始化并不需要合并为一个步骤
	var btn1 = Object.create( Button );
	btn1.setup( 125, 30, "Hello" );

	var btn2 = Object.create( Button );
	btn2.setup( 150, 40, "World" );

	btn1.build( $body );
	btn2.build( $body );
} );
```

**更简洁的设计**

对象关联除了能让代码看起来更简洁(并且更具扩展性)外还可以通过行为委托模式简化代码结构

**更好的语法**

```jsx
class Foo {
	// 不需要function关键字
	methodName() { /* .. */ }
}

var LoginController = {
	errors: [],
	getUser() { //妈妈再也不用担心代码里有function了!
		// ...
	},
	getPassword() {
		// ...
	}
	// ...
};

// 使用更好的对象字面形式语法和简洁方法
var AuthController = {
 	errors: [],
 	// 匿名函数，无法递归
 	checkAuth() {
		// ...
	},
 	server(url,data) {
 		// ...
	}
	// ...
};
// 现在把 AuthController 关联到 LoginController Object.setPrototypeOf( AuthController, LoginController );
```

**内省**

自省就是检查实例的类型

鸭子类型(ES6 的 Promise 就是典型的“鸭子类型”)

```jsx
if (a1.something) {
	a1.something();
}
```

对象关联比类风格的代码更加简洁，内省时不需要使用 instanceof 判断 Foo.prototype 或者繁琐的 Foo.prototype.isPrototypeOf(..)，只需要使用 isPrototypeOf; getPrototypeOf 判断“你是我的原型吗?”。
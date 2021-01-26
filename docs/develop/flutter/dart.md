---
id: dart
title: Dart 语言基础
---

## 基础语法与类型变量：Dart是如何表示信息的？

- 在 Dart 中，所有类型都是对象类型，都继承自顶层类型 Object，因此一切变量都是对象，数字、布尔值、函数和 null 也概莫能外；
- 未初始化变量的值都是 null；
- 为变量指定类型，这样编辑器和编译器都能更好地理解你的意图。

Dart 内置了一些基本类型，如 **num、bool、String、List 和 Map**

- num 只有两种子类：int、double

    ```js
    int x = 1;
    int hex = 0xEEADBEEF;
    double y = 1.1;
    double exponents = 1.13e5;
    int roundY = y.round();
    ```

- 只有两个对象具有 bool 类型：true 和 false
不能使用 if(nonbooleanValue)
- String

    ```js
    var s = 'cat';
    var s1 = 'this is a uppercased string: ${s.toUpperCase()}';
    var s2 = 'Hello' + ' ' + 'World!' ;
    // 多行字符串
    var s3 = """This is a
    multi-line string.""";
    ```

- List

    ```js
    var arr1 = ["Tom", "Andy", "Jack"];
    var arr2 = List.of([1,2,3]);
    arr2.add(499);
    arr2.forEach((v) => print('${v}'));

    var arr1 = <String>['Tom', 'Andy', 'Jack'];
    var arr2 = new List<int>.of([1,2,3]);
    arr2.add(499);
    arr2.forEach((v) => print('${v}'));
    print(arr2 is List<int>); // true
    ```

- Map

    ```js
    var map1 = {"name": "Tom", 'sex': 'male'}; 
    var map2 = new Map();
    map2['name'] = 'Tom';
    map2['sex'] = 'male';
    map2.forEach((k,v) => print('${k}: ${v}'));

    var map1 = <String, String>{'name': 'Tom','sex': 'male',};
    var map2 = new Map<String, String>();
    map2['name'] = 'Tom';
    map2['sex'] = 'male';
    map2.forEach((k,v) => print('${k}: ${v}')); 
    print(map2 is Map<String, String>); // true
    ```

- const 表示变量在编译期间即能确定的值
- final 则不太一样，用它定义的变量可以在运行时确定值，而一旦确定后就不可再变

## 函数、类与运算符：Dart是如何处理信息的？

如果函数体只有一行表达式，可以像 JS 那样用箭头函数来简化这个函数

```js
bool isZero(int number) {
	return number == 0;
}
bool isZero(int number) => number == 0;
```

Dart 认为重载会导致混乱，因此从设计之初就不支持重载，而是提供了**可选命名参数**和**可选参数**

```js
// 重载：在相同的声明域中的函数名相同的，而参数表不同的，
// 即通过函数的参数表而唯一标识并且来区分函数的一种特殊的函数。
int Max （int，int）； // 返回两个整数的最大值；
int Max （const vector <int> &）； // 返回vector容器中的最大值；
int Max （const matrix &）； // 返回matrix引用的最大值；

// Dart 可选命名参数的用法，在定义函数的时候给参数加上 {}
void enable1Flags({bool bold, bool hidden}) => print("$bold , $hidden");
// 定义可选命名参数时增加默认值
void enable2Flags({bool bold = true, bool hidden = false}) => print("$bold ,$hidden");

// 可忽略的参数在函数定义时用[]符号指定
void enable3Flags(bool bold, [bool hidden]) => print("$bold ,$hidden");
// 定义可忽略参数时增加默认值
void enable4Flags(bool bold, [bool hidden = false]) => print("$bold ,$hidden");

//可选命名参数函数调用
enable1Flags(bold: true, hidden: false); //true, false
enable1Flags(bold: true); //true, null
enable2Flags(bold: false); //false, false
//可忽略参数函数调用
enable3Flags(true, false); //true, false
enable3Flags(true,); //true, null
enable4Flags(true); //true, false
enable4Flags(true,true); // true, true
```

Dart 是如何定义和使用类的

```js
class Point {
  num x, y;
  static num factor = 0;
  //语法糖，等同于在函数体内：this.x = x;this.y = y;
  Point(this.x,this.y);
  void printInfo() => print('($x, $y)');
  static void printZValue() => print('$factor');
}

var p = new Point(100,200); // new 关键字可以省略
p.printInfo();  // 输出(100, 200);
Point.factor = 10;
Point.printZValue(); // 输出10
```

与 C++ 类似，Dart 支持**初始化列表**。在构造函数的函数体真正执行之前，你还有机会给实例变量赋值，甚至重定向至另一个构造函数。

```js
// Point 类中有两个构造函数 Point.bottom 与 Point，
// 其中：Point.bottom 将其成员变量的初始化重定向到了 Point 中，
// 而 Point 则在初始化列表中为 z 赋上了默认值 0。
class Point {
  num x, y, z;
  Point(this.x, this.y) : z = 0; // 初始化变量z
  Point.bottom(num x) : this(x, 0); // 重定向构造函数
  void printInfo() => print('($x,$y,$z)');
}

var p = Point.bottom(100);
p.printInfo(); // 输出(100,0,0)
```

在面向对象的编程语言中，将其他类的变量与方法纳入本类中进行复用的方式一般有两种：**继承父类和接口实现**。

- 继承父类意味着，子类由父类派生，会自动获取父类的成员变量和方法实现，子类可以根据需要覆写构造函数及父类方法；
- 接口实现则意味着，子类获取到的仅仅是接口的成员变量符号和方法符号，需要重新实现成员变量，以及方法的声明和初始化，否则编译器会报错。

```js
class Point {
  num x = 0, y = 0;
  void printInfo() => print('($x,$y)');
}

//Vector继承自Point
class Vector extends Point{
  num z = 0;
  @override
  void printInfo() => print('($x,$y,$z)'); //覆写了printInfo实现
}

//Coordinate是对Point的接口实现
class Coordinate implements Point {
  num x = 0, y = 0; //成员变量需要重新声明
  void printInfo() => print('($x,$y)'); //成员函数需要重新声明实现
}

var xxx = Vector(); 
xxx
  ..x = 1
  ..y = 2
  ..z = 3; //级联运算符，等同于xxx.x=1; xxx.y=2;xxx.z=3;
xxx.printInfo(); //输出(1,2,3)

var yyy = Coordinate();
yyy
  ..x = 1
  ..y = 2; //级联运算符，等同于yyy.x=1; yyy.y=2;
yyy.printInfo(); //输出(1,2)
print (yyy is Point); //true
print(yyy is Coordinate); //true
```

Dart 还提供了另一种机制来实现类的复用，即“**混入**”（**Mixin**）

混入鼓励代码重用，可以被视为具有实现方法的接口。这样一来，不仅可以解决 Dart 缺少对多重继承的支持问题，还能够**避免由于多重继承可能导致的歧义**（菱形问题）。

```js
// 通过混入，一个类里可以以非继承的方式使用其他类中的变量与方法
class Coordinate with Point {
}

var yyy = Coordinate();
print (yyy is Point); //true
print(yyy is Coordinate); //true
```

Dart 多了几个额外的运算符，用于简化处理**变量实例缺失**（即 null）的情况

- ?. 运算符：假设 Point 类有 printInfo() 方法，p 是 Point 的一个可能为 null 的实例。那么，p 调用成员方法的安全代码，可以简化为 p?.printInfo() ，表示 p 为 null 的时候跳过，避免抛出异常。
- ??= 运算符：如果 a 为 null，则给 a 赋值 value，否则跳过。这种用默认值兜底的赋值语句在 Dart 中我们可以用 a ??= value 表示。
- ?? 运算符：如果 a 不为 null，返回 a 的值，否则返回 b。在 Java 或者 C++ 中，我们需要通过三元表达式 (a != null)? a : b 来实现这种情况。而在 Dart 中，这类代码可以简化为 a ?? b。

在 Dart 中，一切都是对象，就连运算符也是对象成员函数的一部分。Dart 提供了类似 C++ 的运算符覆写机制：

```js
class Vector {
  num x, y;
  Vector(this.x, this.y);
  // 自定义相加运算符，实现向量相加
  Vector operator +(Vector v) =>  Vector(x + v.x, y + v.y);
  // 覆写相等运算符，判断向量相等
  bool operator == (dynamic v) => x == v.x && y == v.y;
}

final x = Vector(3, 3);
final y = Vector(2, 2);
final z = Vector(1, 1);
print(x == (y + z)); //  输出true
```

## 综合案例：掌握Dart核心特性

不使用任何 Dart 语法特性的情况下，一个有着基本功能的购物车程序

```js
//定义商品Item类
class Item {
  double price;
  String name;
  Item(name, price) {
    this.name = name;
    this.price = price;
  }
}

//定义购物车类
class ShoppingCart {
  String name;
  DateTime date;
  String code;
  List<Item> bookings;

  price() {
    double sum = 0.0;
    for(var i in bookings) {
      sum += i.price;
    }
    return sum;
  }

  ShoppingCart(name, code) {
    this.name = name;
    this.code = code;
    this.date = DateTime.now();
  }

  getInfo() {
    return '购物车信息:' +
          '\n-----------------------------' +
          '\n用户名: ' + name+ 
          '\n优惠码: ' + code + 
          '\n总价: ' + price().toString() +
          '\n日期: ' + date.toString() +
          '\n-----------------------------';
  }
}

void main() {
  ShoppingCart sc = ShoppingCart('张三', '123456');
  sc.bookings = [Item('苹果',10.0), Item('鸭梨',20.0)];
  print(sc.getInfo());
}
```

类抽象改造：利用语法糖以及初始化列表，简化赋值过程，从而直接省去构造函数的函数体

```js
class Item {
  double price;
  String name;
  Item(this.name, this.price);
}

class ShoppingCart {
  String name;
  DateTime date;
  String code;
  List<Item> bookings;
  price() {...}
  //删掉了构造函数函数体
  ShoppingCart(this.name, this.code) : date = DateTime.now();
...
}
```

考虑到 name 属性与 price 属性（方法）的名称与类型完全一致，可以抽象出一个新的基类 Meta，用于存放 price 属性与 name 属性。

```js
class Meta {
  double price;
  String name;
  Meta(this.name, this.price);
}
class Item extends Meta{
  Item(name, price) : super(name, price);
}

class ShoppingCart extends Meta{
  DateTime date;
  String code;
  List<Item> bookings;
  
  double get price {...}
  ShoppingCart(name, this.code) : date = DateTime.now(),super(name,0);
  getInfo() {...}
}
```

方法改造

```js
class Item extends Meta{
  ...
  //重载了+运算符，合并商品为套餐商品
  Item operator+(Item item) => Item(name + item.name, price + item.price); 
}

class ShoppingCart extends Meta{
  ...
  //把迭代求和改写为归纳合并
  double get price => bookings.reduce((value, element) => value + element).price;
  ...
  getInfo() {...}
}
```

```js
getInfo () => '''
购物车信息:
-----------------------------
  用户名: $name
  优惠码: $code
  总价: $price
  Date: $date
-----------------------------
''';
```

对象初始化方式的优化

```js
class ShoppingCart extends Meta{
  ...
  //默认初始化方法，转发到withCode里
  ShoppingCart({name}) : this.withCode(name:name, code:null);
  //withCode初始化方法，使用语法糖和初始化列表进行赋值，并调用父类初始化方法
  ShoppingCart.withCode({name, this.code}) : date = DateTime.now(), super(name,0);

  //??运算符表示为code不为null，则用原值，否则使用默认值"没有"
  getInfo () => '''
购物车信息:
-----------------------------
  用户名: $name
  优惠码: ${code??"没有"}
  总价: $price
  Date: $date
-----------------------------
''';
}

void main() {
  ShoppingCart sc = ShoppingCart.withCode(name:'张三', code:'123456');
  sc.bookings = [Item('苹果',10.0), Item('鸭梨',20.0)];
  print(sc.getInfo());

  ShoppingCart sc2 = ShoppingCart(name:'李四');
  sc2.bookings = [Item('香蕉',15.0), Item('西瓜',40.0)];
  print(sc2.getInfo());
}
```

增加 PrintHelper 类，Dart 不支持多继承，使用 Mixin

```js
abstract class PrintHelper {
  printInfo() => print(getInfo());
  getInfo();
}

class ShoppingCart extends Meta with PrintHelper{
...
}
```

使用级联操作符避免创建临时变量，让代码看起来更流畅

```js
void main() {
  ShoppingCart.withCode(name:'张三', code:'123456')
  ..bookings = [Item('苹果',10.0), Item('鸭梨',20.0)]
  ..printInfo();

  ShoppingCart(name:'李四')
  ..bookings = [Item('香蕉',15.0), Item('西瓜',40.0)]
  ..printInfo();
}
```

最终代码

```js
class Meta {
  double price;
  String name;
  //成员变量初始化语法糖
  Meta(this.name, this.price);
}

class Item extends Meta{
  Item(name, price) : super(name, price);
  //重载+运算符，将商品对象合并为套餐商品
  Item operator+(Item item) => Item(name + item.name, price + item.price); 
}

abstract class PrintHelper {
  printInfo() => print(getInfo());
  getInfo();
}

//with表示以非继承的方式复用了另一个类的成员变量及函数
class ShoppingCart extends Meta with PrintHelper{
  DateTime date;
  String code;
  List<Item> bookings;
  //以归纳合并方式求和
  double get price => bookings.reduce((value, element) => value + element).price;
  //默认初始化函数，转发至withCode函数
  ShoppingCart({name}) : this.withCode(name:name, code:null);
  //withCode初始化方法，使用语法糖和初始化列表进行赋值，并调用父类初始化方法
  ShoppingCart.withCode({name, this.code}) : date = DateTime.now(), super(name,0);

  //??运算符表示为code不为null，则用原值，否则使用默认值"没有"
  @override
  getInfo() => '''
购物车信息:
-----------------------------
  用户名: $name
  优惠码: ${code??"没有"}
  总价: $price
  Date: $date
-----------------------------
''';
}

void main() {
  ShoppingCart.withCode(name:'张三', code:'123456')
  ..bookings = [Item('苹果',10.0), Item('鸭梨',20.0)]
  ..printInfo();

  ShoppingCart(name:'李四')
  ..bookings = [Item('香蕉',15.0), Item('西瓜',40.0)]
  ..printInfo();
}
```
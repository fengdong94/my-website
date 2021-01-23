---
id: chapter8
title: 第 8 章：特百惠
---

注：特百惠是美国家居用品品牌，代表产品是塑料容器。

书写函数式的程序即通过管道把数据在一系列纯函数间传递。这些程序就是声明式的行为规范。

但是，控制流（control flow）、异常处理（error handling）、异步操作（asynchronous actions）和状态（state）呢？还有更棘手的作用（effects）呢？本章将对上述这些抽象概念赖以建立的基础作一番探究。

### 强大的容器

```jsx
var Container = function(x) {
  this.__value = x;
}

Container.of = function(x) { return new Container(x); };

Container.of(3)
//=> Container(3)

Container.of("hotdogs")
//=> Container("hotdogs")
```

### 第一个 functor

一旦容器里有了值，不管这个值是什么，我们就需要一种方法来让别的函数能够操作它。

```jsx
// (a -> b) -> Container a -> Container b
Container.prototype.map = function(f){
  return Container.of(f(this.__value))
}

Container.of("flamethrowers").map(function(s){ return s.toUpperCase() })
//=> Container("FLAMETHROWERS")

Container.of("bombs").map(concat(' away')).map(_.prop('length'))
//=> Container(10)
```

Container 里的值传递给 map 函数操作之后再把它放回 Container。这样就能连续地调用 map，运行任何我们想运行的函数。那它不就是个组合（composition）么！

- functor 是实现了 map 函数并遵守一些特定规则的容器类型。

### 薛定谔的 Maybe

还有另外一种 functor，那就是实现了 map 函数的类似容器的数据类型，这种 functor 在**调用 map 的时候能够提供非常有用的行为**。

```jsx
// 空值判断 防止报错
var Maybe = function(x) {
  this.__value = x;
}
Maybe.of = function(x) {
  return new Maybe(x);
}
Maybe.prototype.isNothing = function() {
  return (this.__value === null || this.__value === undefined);
}
Maybe.prototype.map = function(f) {
  return this.isNothing() ? Maybe.of(null) : Maybe.of(f(this.__value));
}

Maybe.of("Malkovich Malkovich").map(match(/a/ig));
//=> Maybe(['a', 'a'])
Maybe.of(null).map(match(/a/ig));
//=> Maybe(null)
Maybe.of({name: "Boris"}).map(_.prop("age")).map(add(10));
//=> Maybe(null)
Maybe.of({name: "Dinah", age: 14}).map(_.prop("age")).map(add(10));
//=> Maybe(24)
```

这种点记法（dot notation syntax）已经足够函数式了，如果我们更想保持一种 pointfree 的风格。map 完全有能力以 curry 函数的方式来“代理”任何 functor：

```jsx
// map :: Functor f => (a -> b) -> f a -> f b
// **Functor f => 告诉我们 f 必须是一个 functor**

var map = curry(function(f, any_functor_at_all) {
  return any_functor_at_all.map(f);
});
```

### 用例

Maybe 最常用在那些可能会无法成功返回结果的函数中

```jsx
//  safeHead :: [a] -> Maybe(a)
var safeHead = function(xs) {
  return Maybe.of(xs[0]);
};

var streetName = compose(map(_.prop('street')), safeHead, _.prop('addresses'));

streetName({addresses: []});
// Maybe(null)

streetName({addresses: [{street: "Shady Ln.", number: 4201}]});
// Maybe("Shady Ln.")
```

有时候函数可以明确返回一个 Maybe(null) 来表明失败

```jsx
//  withdraw :: Number -> Account -> Maybe(Account)
var withdraw = curry(function(amount, account) {
  return account.balance >= amount ?
    Maybe.of({balance: account.balance - amount}) :
    Maybe.of(null);
});

//  finishTransaction :: Account -> String
var finishTransaction = compose(remainingBalance, updateLedger); // <- 假定这两个函数已经在别处定义好了

//  getTwenty :: Account -> Maybe(String)
var getTwenty = compose(map(finishTransaction), withdraw(20));

getTwenty({ balance: 200.00});
// Maybe("Your balance is $180.00")

getTwenty({ balance: 10.00});
// Maybe(null)
```

如果 withdraw 失败了，map 就会切断后续代码的执行，因为它根本就不会运行传递给它的函数，即 finishTransaction。这正是预期的效果：如果取款失败，我们并不想更新或者显示账户余额。

### 释放容器里的值

如果我们想返回一个自定义的值然后还能继续执行后面的代码的话，是可以做到的；要达到这一目的，可以借助一个帮助函数 maybe：

```jsx
//  maybe :: b -> (a -> b) -> Maybe a -> b
var maybe = curry(function(x, f, m) {
  return m.isNothing() ? x : f(m.__value);
});

//  getTwenty :: Account -> String
var getTwenty = compose(
  maybe("You're broke!", finishTransaction), withdraw(20)
);

getTwenty({ balance: 200.00});
// "Your balance is $180.00"

getTwenty({ balance: 10.00});
// "You're broke!"
```

maybe 使我们得以避免普通 map 那种**命令式的 if/else 语句**：if(x !== null) { return f(x) }。

### “纯”错误处理

```jsx
var Left = function(x) {
  this.__value = x;
}

Left.of = function(x) {
  return new Left(x);
}

Left.prototype.map = function(f) {
  return this;
}

var Right = function(x) {
  this.__value = x;
}

Right.of = function(x) {
  return new Right(x);
}

Right.prototype.map = function(f) {
  return Right.of(f(this.__value));
}
```

```jsx
var moment = require('moment');

//  getAge :: Date -> User -> Either(String, Number)
var getAge = curry(function(now, user) {
  var birthdate = moment(user.birthdate, 'YYYY-MM-DD');
  if(!birthdate.isValid()) return Left.of("Birth date could not be parsed");
  return Right.of(now.diff(birthdate, 'years'));
});

getAge(moment(), {birthdate: '2005-12-12'});
// Right(9)

getAge(moment(), {birthdate: 'balloons!'});
// Left("Birth date could not be parsed")
```

就像 Maybe(null)，当返回一个 Left 的时候就直接让程序短路。跟 Maybe(null) 不同的是，现在我们对程序为何脱离原先轨道至少有了一点头绪。

```jsx
//  fortune :: Number -> String
var fortune  = compose(concat("If you survive, you will be "), add(1));

//  zoltar :: User -> Either(String, _)
var zoltar = compose(map(console.log), map(fortune), getAge(moment()));

zoltar({birthdate: '2005-12-12'});
// "If you survive, you will be 10"
// Right(undefined)

zoltar({birthdate: 'balloons!'});
// Left("Birth date could not be parsed")
```

就像 Maybe 可以有个 maybe 一样，Either 也可以有一个 either。两者的用法类似，但 either 接受两个函数（而不是一个）和一个静态值为参数。这两个函数的返回值类型一致：

```jsx
//  either :: (a -> c) -> (b -> c) -> Either a b -> c
var either = curry(function(f, g, e) {
  switch(e.constructor) {
    case Left: return f(e.__value);
    case Right: return g(e.__value);
  }
});

// _ 表示一个应该忽略的值
// id 就是简单地复制了 Left 里的错误消息
//  zoltar :: User -> _
var zoltar = compose(console.log, either(id, fortune), getAge(moment()));

zoltar({birthdate: '2005-12-12'});
// "If you survive, you will be 10"
// undefined

zoltar({birthdate: 'balloons!'});
// "Birth date could not be parsed"
// undefined
```

### 王老先生有作用...

通过把副作用包裹在另一个函数里的方式把它变得看起来像一个纯函数。

```jsx
//  getFromStorage :: String -> (_ -> String)
var getFromStorage = function(key) {
  return function() {
    return localStorage[key];
  }
}
```

然而，这并没有多大的用处，要是能有办法进到这个容器里面，拿到它藏在那儿的东西就好了...办法是有的，请看 IO：

```jsx
var IO = function(f) {
  this.__value = f;
}
IO.of = function(x) {
  return new IO(function() {
    return x;
  });
}
IO.prototype.map = function(f) {
  return new IO(_.compose(f, this.__value));
}

//  io_window_ :: IO Window
var io_window = new IO(function(){ return window; });

io_window.map(function(win){ return win.innerWidth });
// IO(1430)

io_window.map(_.prop('location')).map(_.prop('href')).map(split('/'));
// IO(["http:", "", "localhost:8000", "blog", "posts"])

//  $ :: String -> IO [DOM]
var $ = function(selector) {
  return new IO(function(){ return document.querySelectorAll(selector); });
}

$('#myDiv').map(head).map(function(div){ return div.innerHTML; });
// IO('I am some inner html')
```

IO 把非纯执行动作（impure action）捕获到包裹函数里，目的是延迟执行这个非纯动作。

对 IO 调用 map 已经积累了太多不纯的操作，有没有可能我们只运行 IO 却不让不纯的操作弄脏双手？答案是可以的，只要把责任推到**调用者**身上就行了：

```jsx
////// 纯代码库: lib/params.js ///////

//  url :: IO String
var url = new IO(function() { return window.location.href; });

//  toPairs =  String -> [[String]]
var toPairs = compose(map(split('=')), split('&'));

//  params :: String -> [[String]]
var params = compose(toPairs, last, split('?'));

//  findParam :: String -> IO Maybe [String]
var findParam = function(key) {
  return map(compose(Maybe.of, filter(compose(eq(key), head)), params), url);
};

////// 非纯调用代码: main.js ///////

// 调用 __value() 来运行它！
findParam("searchTerm").__value();
// Maybe(['searchTerm', 'wafflehouse'])
```

lib/params.js 把 url 包裹在一个 IO 里，然后把这头野兽传给了调用者；一双手保持的非常干净。

IO 的 __value 并不是它包含的值，__value 是手榴弹的弹栓，只应该被调用者以最公开的方式拉动。为了提醒用户它的变化无常，我们把它重命名为 unsafePerformIO 看看：

```jsx
var IO = function(f) {
  this.unsafePerformIO = f;
}

IO.prototype.map = function(f) {
  return new IO(_.compose(f, this.unsafePerformIO));
}

// 直白
findParam("searchTerm").unsafePerformIO()
```

### 异步任务

```jsx
// Node readfile example:
//=======================

var fs = require('fs');

//  readFile :: String -> Task(Error, JSON)
var readFile = function(filename) {
  return new Task(function(reject, result) {
    fs.readFile(filename, 'utf-8', function(err, data) {
      err ? reject(err) : result(data);
    });
  });
};

readFile("metamorphosis").map(split('\n')).map(head);
// Task("One morning, as Gregor Samsa was waking up from anxious dreams, he discovered that
// in bed he had been changed into a monstrous verminous bug.")

// jQuery getJSON example:
//========================

//  getJSON :: String -> {} -> Task(Error, JSON)
var getJSON = curry(function(url, params) {
  return new Task(function(reject, result) {
    $.getJSON(url, params, result).fail(reject);
  });
});

getJSON('/video', {id: 10}).map(_.prop('title'));
// Task("Family Matters ep 15")

// 传入普通的实际值也没问题
Task.of(3).map(function(three){ return three + 1 });
// Task(4)
```

map 就是 then，Task 就是一个 promise。

调用 fork 方法才能运行 Task，这种机制与 unsafePerformIO 类似，但是 fork 不会阻塞主线程。

```jsx
// Pure application
//=====================
// blogTemplate :: String

//  blogPage :: Posts -> HTML
var blogPage = Handlebars.compile(blogTemplate);

//  renderPage :: Posts -> HTML
var renderPage = compose(blogPage, sortBy('date'));

//  blog :: Params -> Task(Error, HTML)
var blog = compose(map(renderPage), getJSON('/posts'));

// Impure calling code
//=====================
blog({}).fork(
  function(error){ $("#error").html(error.message); },
  function(page){ $("#main").html(page); }
);

$('#spinner').show();
```

就算是有了 Task，IO 和 Either 这两个 functor 也照样能派上用场。

```jsx
// Postgres.connect :: Url -> IO DbConnection
// runQuery :: DbConnection -> ResultSet
// readFile :: String -> Task Error String

// Pure application
//=====================

//  dbUrl :: Config -> Either Error Url
var dbUrl = function(c) {
  return (c.uname && c.pass && c.host && c.db)
    ? Right.of("db:pg://"+c.uname+":"+c.pass+"@"+c.host+"5432/"+c.db)
    : Left.of(Error("Invalid config!"));
}

//  connectDb :: Config -> Either Error (IO DbConnection)
var connectDb = compose(map(Postgres.connect), dbUrl);

//  getConfig :: Filename -> Task Error (Either Error (IO DbConnection))
var getConfig = compose(map(compose(connectDB, JSON.parse)), readFile);

// Impure calling code
//=====================
getConfig("db.json").fork(
  logErr("couldn't read file"), either(console.log, map(runQuery))
);
```

### 一点理论

```jsx
// 同一律 identity
map(id) === id;

var idLaw1 = map(id);
var idLaw2 = id;

idLaw1(Container.of(2));
//=> Container(2)

idLaw2(Container.of(2));
//=> Container(2)

// 组合律 composition
compose(map(f), map(g)) === map(compose(f, g));

var compLaw1 = compose(map(concat(" world")), map(concat(" cruel")));
var compLaw2 = map(compose(concat(" world"), concat(" cruel")));

compLaw1(Container.of("Goodbye"));
//=> Container('Goodbye cruel world')

compLaw2(Container.of("Goodbye"));
//=> Container('Goodbye cruel world')

// 交换律
//  topRoute :: String -> Maybe(String)
var topRoute = compose(Maybe.of, reverse);

//  bottomRoute :: String -> Maybe(String)
var bottomRoute = compose(map(reverse), Maybe.of);

topRoute("hi");
// Maybe("ih")

bottomRoute("hi");
// Maybe("ih")
```

functor 也能嵌套使用：

```jsx
var nested = Task.of([Right.of("pillows"), Left.of("no sleep for you")]);

map(map(map(toUpperCase)), nested);
// Task([Right("PILLOWS"), Left("no sleep for you")])
```

如果不想嵌套，可以组合 functor

```jsx
var Compose = function(f_g_x){
  this.getCompose = f_g_x;
}

Compose.prototype.map = function(f){
  return new Compose(map(map(f), this.getCompose));
}

var tmd = Task.of(Maybe.of("Rock over London"))

var ctmd = new Compose(tmd);

map(concat(", rock on, Chicago"), ctmd);
// Compose(Task(Maybe("Rock over London, rock on, Chicago")))

ctmd.getCompose;
// Task(Maybe("Rock over London, rock on, Chicago"))
```
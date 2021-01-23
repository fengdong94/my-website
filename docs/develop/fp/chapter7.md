---
id: chapter7
title: 第 7 章：Hindley-Milner 类型签名
---

```jsx
//  strLength :: String -> Number
var strLength = function(s){
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs){
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
//  match :: Regex -> (String -> [String])
var match = curry(function(reg, s){
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
//  replace :: Regex -> (String -> (String -> String))
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

```jsx
// 变量
// map :: (a -> b) -> [a] -> [b]
var map = curry(function(f, xs){
  return xs.map(f);
});

// map 接受两个参数，第一个是从任意类型 a 到任意类型 b 的函数；
// 第二个是一个数组，元素是任意类型的 a；map 最后返回的是一个类型 b 的数组。

//  reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs){
  return xs.reduce(f, x);
});

// **第一个参数是个函数，这个函数接受一个 b 和一个 a 并返回一个 b
// 签名中的第二个和第三个参数就是 b 和元素为 a 的数组**
```

### 类型约束

```jsx
// sort :: Ord a => [a] -> [a]
// a 必须要实现 Ord 接口

// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
// 这个例子中有两个约束：Eq 和 Show。它们保证了我们可以检查不同的 a 是否相等，
// 并在有不相等的情况下打印出其中的差异。
```
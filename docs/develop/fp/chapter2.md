---
id: chapter2
title: 第 2 章：一等公民的函数
---

函数并没什么特殊的，你可以像对待任何其他数据类型一样对待它们——把它们存在数组里，当作参数传递，赋值给变量...等等。

### 不要对函数添加额外的包裹

```java
// 太傻了
const getServerStuff = callback => ajaxCall(json => callback(json));

// 这行
ajaxCall(json => callback(json));

// 等价于这行
ajaxCall(callback);

// 那么，重构下 getServerStuff
const getServerStuff = callback => ajaxCall(callback);

// ...就等于
const getServerStuff = ajaxCall // <-- 看，没有括号哦
```

果一个函数被不必要地包裹起来了，而且发生了改动，那么包裹它的那个函数也要做相应的变更。

```jsx
httpGet('/post/2', json => renderPost(json));

// 把整个应用里的所有 httpGet 调用都改成这样，可以传递 err 参数。
httpGet('/post/2', (json, err) => renderPost(json, err));

// 写成一等公民函数的形式，要做的改动将会少得多：
httpGet('/post/2', renderPost);  // renderPost 将会在 httpGet 中调用，想要多少参数都行
```

### 正确地为参数命名

```jsx
// 只针对当前的博客
const validArticles = articles =>
  articles.filter(article => article !== null && article !== undefined),

// 对未来的项目更友好
const compact = xs => xs.filter(x => x !== null && x !== undefined);
```

### 小心 this 值

```jsx
var fs = require('fs');

// 太可怕了
fs.readFile('freaky_friday.txt', Db.save);

// 好一点点
fs.readFile('freaky_friday.txt', Db.save.bind(Db));
```

this 就像一块**脏尿布**，尽可能地避免使用它，因为在函数式编程中根本用不到它。

然而，在使用其他的类库时，你却不得不向这个疯狂的世界低头。

也有人反驳说 this 能提高执行速度。**函数式编程不必对速度吹毛求疵**。
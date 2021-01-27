---
id: recursive
title: 递归
---

### 递归需要满足的三个条件

1. 一个问题的解可以分解为几个子问题的解
2. 这个问题与分解之后的子问题，除了数据规模不同，求解思路完全一样
3. 存在递归终止条件

### 如何编写递归代码？

写递归代码最关键的是写出**递推公式**，找到**终止条件**

假如这里有 n 个台阶，每次你可以跨 1 个台阶或者 2 个台阶，请问走这 n 个台阶有多少种走法？

可以根据第一步的走法把所有走法分为两类，第一类是第一步走了 1 个台阶，另一类是第一步走了 2 个台阶。

**递推公式**

```jsx
f(n) = f(n-1)+f(n-2)
```

**终止条件**

n=2 时，f(2)=f(1)+f(0)。如果递归终止条件只有一个 f(1)=1，那 f(2) 就无法求解了。所以还要有 f(0)=1，表示走 0 个台阶有一种走法，不过这样子看起来就不符合正常的逻辑思维了。

所以，我们可以把 f(2)=2 作为一种终止条件，表示走 2 个台阶，有两种走法，一步走完或者分两步来走。

所以，递归终止条件就是 f(1)=1，f(2)=2。

```jsx
int f(int n) {
  if (n == 1) return 1;
  if (n == 2) return 2;
  return f(n-1) + f(n-2);
}
```

当我们看到递归时，我们总想把递归平铺展开，脑子里就会循环，一层一层往下调，然后再一层一层返回，试图想搞清楚计算机每一步都是怎么执行的，这样就很容易被绕进去。

如果一个问题 A 可以分解为若干子问题 B、C、D，你可以**假设子问题 B、C、D 已经解决**，在此基础上思考如何解决问题 A。而且，你**只需要思考问题 A 与子问题 B、C、D 两层之间的关系即可**，不需要一层一层往下思考子问题与子子问题，子子问题与子子子问题之间的关系。**屏蔽掉递归细节**，这样子理解起来就简单多了。

### 递归代码要警惕堆栈溢出

我们可以通过在代码中限制递归调用的最大深度的方式来解决这个问题。

```jsx
// 全局变量，表示递归的深度。
int depth = 0;

int f(int n) {
  ++depth；
  if (depth > 1000) throw exception;
  
  if (n == 1) return 1;
  return f(n-1) + 1;
}
```

但这种做法并不能完全解决问题，因为最大允许的递归深度跟当前线程剩余的栈空间大小有关，事先无法计算。

### 递归代码要警惕重复计算

上面递归代码的例子，如果我们把整个递归过程分解一下的话，那就是这样的：

![imgs/e7e778994e90265344f6ac9da39e01bf.jpg](imgs/e7e778994e90265344f6ac9da39e01bf.jpg)

为了避免重复计算，我们可以通过一个数据结构（比如散列表）来保存已经求解过的 f(k)。当递归调用到 f(k) 时，先看下是否已经求解过了。如果是，则直接从散列表中取值返回：

```jsx
public int f(int n) {
  if (n == 1) return 1;
  if (n == 2) return 2;
  
  // hasSolvedList可以理解成一个Map，key是n，value是f(n)
  if (hasSolvedList.containsKey(n)) {
    return hasSolvedList.get(n);
  }
  
  int ret = f(n-1) + f(n-2);
  hasSolvedList.put(n, ret);
  return ret;
}
```

在时间效率上，递归代码里多了很多函数调用，当这些函数调用的数量较大时，就会积聚成一个可观的时间成本。在空间复杂度上，因为递归调用一次就会在内存栈中保存一次现场数据，所以在分析递归代码空间复杂度时，需要额外考虑这部分的开销。

### 怎么将递归代码改写为非递归代码？

```jsx
int f(int n) {
  if (n == 1) return 1;
  return f(n-1) + 1;
}

int f(int n) {
  int ret = 1;
  for (int i = 2; i <= n; ++i) {
    ret = ret + 1;
  }
  return ret;
}
```

```jsx
int f(int n) {
  if (n == 1) return 1;
  if (n == 2) return 2;
  
  int ret = 0;
  int pre = 2;
  int prepre = 1;
  for (int i = 3; i <= n; ++i) {
    ret = pre + prepre;
    prepre = pre;
    pre = ret;
  }
  return ret;
}
```

理论上，所有的递归代码都可以改为这种**迭代循环**的非递归写法。

但是这种思路实际上是将递归改为了“手动”递归，本质并没有变，而且也并没有解决前面讲到的某些问题，徒增了实现的复杂度。

### 如何用三行代码找到“最终推荐人”？

```jsx
long findRootReferrerId(long actorId) {
  Long referrerId = select referrer_id from [table] where actor_id = actorId;
  if (referrerId == null) return actorId;
  return findRootReferrerId(referrerId);
}
```
---
id: algorithm
title: 手撕算法篇
---

## 获取二叉树最大深度

```js
interface Node<T> { val: T, left: Node<T>, right: Node<T> }

const getDepth = (root) => {
  if (root === null) {
    return 0;
  } else {
    const leftDepth = getDepth(root.left);
    const rightDepth = getDepth(root.right);

    const childDepth = leftDepth > rightDepth ? leftDepth : rightDepth;

    return childDepth + 1; // 根节点不为空高度至少为1
  }
};
```

## 反转单向链表

```js
interface listNode<T> { val: T, next: listNode<T> }

const reverseList = (head) => {
  // 打印链表内容
  const printList = (node) => {
    let p = node;
    while(p) {
      console.log(p.val);
      p = p.next;
    }
  };

  // 反转函数
  const reverse = (node) => {
    let p = node;
    let pPre = null;
    let pNext;

    while(p) {
      pNext = p.next;
      p.next = pPre;
      pPre = p;
      p = pNext;
    }

    return pPre;
  };

  printList(reverse(head));
};
```

## 排序

### 快排

## 树形结构遍历

### 深度优先

### 广度优先

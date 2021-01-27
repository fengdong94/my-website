---
id: binary-search
title: 二分查找
---

二分查找针对的是一个**有序的数据集合**，查找思想有点类似分治思想。每次都通过跟区间的中间元素对比，将待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为 0。

**对数时间复杂度O(logn)**

二分查找底层依赖的是数组，除了数据本身之外，不需要额外存储其他信息，是**最省内存空间的查找方式**。

### 二分查找的递归与非递归实现

```jsx
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;

  while (low <= high) {
    int mid = (low + high) / 2;
    if (a[mid] == value) {
      return mid;
    } else if (a[mid] < value) {
      low = mid + 1;
    } else {
      high = mid - 1;
    }
  }

  return -1;
}
```

容易出错的 3 个地方：

- **循环退出条件**

    是 low<=high，而不是 low<high。

- **mid 的取值**

    mid=(low+high)/2 这种写法是有问题的。因为如果 low 和 high 比较大的话，两者之和就有可能会溢出。改进的方法是将 mid 的计算方式写成 low+(high-low)/2。更进一步，如果要将性能优化到极致的话，我们可以将这里的除以 2 操作转化成位运算 low+((high-low)>>1)。

- **low 和 high 的更新**

    low=mid+1，high=mid-1。如果直接写成 low=mid 或者 high=mid，就可能会发生死循环。比如，当 high=3，low=3 时，如果 a[3]不等于 value，就会导致一直循环不退出。

```jsx
// 二分查找的递归实现
public int bsearch(int[] a, int n, int val) {
  return bsearchInternally(a, 0, n - 1, val);
}

private int bsearchInternally(int[] a, int low, int high, int value) {
  if (low > high) return -1;

  int mid =  low + ((high - low) >> 1);
  if (a[mid] == value) {
    return mid;
  } else if (a[mid] < value) {
    return bsearchInternally(a, mid+1, high, value);
  } else {
    return bsearchInternally(a, low, mid-1, value);
  }
}
```

### 二分查找应用场景的局限性

- **二分查找依赖的是顺序表结构，简单点说就是数组。**

    二分查找算法需要按照下标随机访问元素。如果数据使用链表存储，二分查找的时间复杂就会变得很高。

- **二分查找针对的是有序数据。**

    如果数据没有序，需要先排序。

    如果针对的是一组静态的数据，没有频繁地插入、删除，我们可以进行一次排序，多次二分查找。这样排序的成本可被均摊，二分查找的边际成本就会比较低。

    如果数据集合有频繁的插入和删除操作，要想用二分查找，要么每次插入、删除操作之后保证数据仍然有序，要么在每次二分查找之前都先进行排序。

    所以，二分查找只能用在**插入、删除操作不频繁，一次排序多次查找的场景**中。针对动态变化的数据集合，二分查找将不再适用（二叉树）。

- **数据量太小不适合二分查找**

    如果要处理的数据量很小，完全没有必要用二分查找，顺序遍历就足够了。

    如果数据之间的比较操作非常耗时，不管数据量大小，都推荐使用二分查找。比如，数组中存储的都是长度超过 300 的字符串，如此长的两个字符串之间比对大小，就会非常耗时。需要尽可能地减少比较次数。

- **数据量太大也不适合二分查找**

    二分查找的底层需要依赖数组这种数据结构，而数组为了支持随机访问的特性，要求**内存空间连续**，对内存的要求比较苛刻。

### 变体一：查找第一个值等于给定值的元素

```jsx
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] > value) {
      high = mid - 1;
    } else if (a[mid] < value) {
      low = mid + 1;
    } else {
			// 关键
      if ((mid == 0) || (a[mid - 1] != value)) return mid;
      else high = mid - 1;
    }
  }
  return -1;
}
```

### 变体二：查找最后一个值等于给定值的元素

```jsx
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] > value) {
      high = mid - 1;
    } else if (a[mid] < value) {
      low = mid + 1;
    } else {
      if ((mid == n - 1) || (a[mid + 1] != value)) return mid;
      else low = mid + 1;
    }
  }
  return -1;
}
```

### 变体三：查找第一个大于等于给定值的元素

```jsx
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] >= value) {
      if ((mid == 0) || (a[mid - 1] < value)) return mid;
      else high = mid - 1;
    } else {
      low = mid + 1;
    }
  }
  return -1;
}
```

### 变体四：查找最后一个小于等于给定值的元素

```jsx
public int bsearch7(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] > value) {
      high = mid - 1;
    } else {
      if ((mid == n - 1) || (a[mid + 1] > value)) return mid;
      else low = mid + 1;
    }
  }
  return -1;
}
```

### 内容小结

凡是用二分查找能解决的，绝大部分我们更倾向于用散列表或者二叉查找树。即便是二分查找在内存使用上更节省，但是毕竟内存如此紧缺的情况并不多。

实际上，“值等于给定值”的二分查找确实不怎么会被用到，二分查找更适合用在“近似”查找问题，在这类问题上，二分查找的优势更加明显。比如这几种变体问题，用其他数据结构，比如散列表、二叉树，就比较难实现了。
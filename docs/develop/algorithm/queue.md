---
id: queue
title: 队列
---

### 如何理解“队列”？

- 先进者先出
- 队列只支持两个基本操作：**入队 enqueue()**，放一个数据到队列尾部；**出队 dequeue()**，从队列头部取一个元素。
- 队列跟栈一样，也是一种操作受限的线性表数据结构。

### 顺序队列和链式队列

用数组实现的队列叫作顺序队列，用链表实现的队列叫作链式队列。

```jsx
// 用数组实现的队列
public class ArrayQueue {
  // 数组：items，数组大小：n
  private String[] items;
  private int n = 0;
  // head表示队头下标，tail表示队尾下标
  private int head = 0;
  private int tail = 0;

  // 申请一个大小为capacity的数组
  public ArrayQueue(int capacity) {
    items = new String[capacity];
    n = capacity;
  }

  // 入队
  public boolean enqueue(String item) {
    // 如果tail == n 表示队列已经满了
    if (tail == n) return false;
    items[tail] = item;
    ++tail;
    return true;
  }

  // 出队
  public String dequeue() {
    // 如果head == tail 表示队列为空
    if (head == tail) return null;
    // 为了让其他语言的同学看的更加明确，把--操作放到单独一行来写了
    String ret = items[head];
    ++head;
    return ret;
  }
}
```

当 a、b、c、d 依次入队之后，队列中的 head 指针指向下标为 0 的位置，tail 指针指向下标为 4 的位置。

![imgs/5c0ec42eb797e8a7d48c9dbe89dc93cb.jpg](imgs/5c0ec42eb797e8a7d48c9dbe89dc93cb.jpg)

调用两次出队操作之后，队列中 head 指针指向下标为 2 的位置，tail 指针仍然指向下标为 4 的位置。

![imgs/dea27f2c505dd8d0b6b86e262d03430d.jpg](imgs/dea27f2c505dd8d0b6b86e262d03430d.jpg)

随着不停地进行入队、出队操作，head 和 tail 都会持续往后移动。当 tail 移动到最右边，即使数组中还有空闲空间，也无法继续往队列中添加数据了。

解决：出队函数 dequeue() 保持不变，在入队时如果没有空闲空间了，则集中触发一次数据的搬移操作。

```jsx
// 入队操作，将item放入队尾
  public boolean enqueue(String item) {
    // tail == n表示队列末尾没有空间了
    if (tail == n) {
      // tail ==n && head==0，表示整个队列都占满了
      if (head == 0) return false;
      // 数据搬移
      for (int i = head; i < tail; ++i) {
        items[i-head] = items[i];
      }
      // 搬移完之后重新更新head和tail
      tail -= head;
      head = 0;
    }
    
    items[tail] = item;
    ++tail;
    return true;
  }
```

出队操作的时间复杂度仍然是 O(1)，入队使用均摊法计算也为 O(1)。

**基于链表**的队列实现方法

![imgs/c916fe2212f8f543ddf539296444d393.jpg](imgs/c916fe2212f8f543ddf539296444d393.jpg)

### 循环队列

能避免 tail==n 时的数据搬移操作。

![imgs/58ba37bb4102b87d66dffe7148b0f990.jpg](imgs/58ba37bb4102b87d66dffe7148b0f990.jpg)

![imgs/71a41effb54ccea9dd463bde1b6abe80.jpg](imgs/71a41effb54ccea9dd463bde1b6abe80.jpg)

![imgs/3d81a44f8c42b3ceee55605f9aeedcec.jpg](imgs/3d81a44f8c42b3ceee55605f9aeedcec.jpg)

- 队列为空： head == tail
- 队满：(tail+1)%n=head

当队列满时，图中的 tail 指向的位置实际上是没有存储数据的。所以，循环队列会浪费一个数组的存储空间。

```jsx
public class CircularQueue {
  // 数组：items，数组大小：n
  private String[] items;
  private int n = 0;
  // head表示队头下标，tail表示队尾下标
  private int head = 0;
  private int tail = 0;

  // 申请一个大小为capacity的数组
  public CircularQueue(int capacity) {
    items = new String[capacity];
    n = capacity;
  }

  // 入队
  public boolean enqueue(String item) {
    // 队列满了
    if ((tail + 1) % n == head) return false;
    items[tail] = item;
    tail = (tail + 1) % n;
    return true;
  }

  // 出队
  public String dequeue() {
    // 如果head == tail 表示队列为空
    if (head == tail) return null;
    String ret = items[head];
    head = (head + 1) % n;
    return ret;
  }
}
```

### 阻塞队列和并发队列

阻塞队列就是在队列为空的时候，从队头取数据会被阻塞。因为此时还没有数据可取，直到队列中有了数据才能返回；如果队列已经满了，那么插入数据的操作就会被阻塞，直到队列中有空闲位置后再插入数据，然后再返回。

![imgs/5ef3326181907dea0964f612890185eb.jpg](imgs/5ef3326181907dea0964f612890185eb.jpg)

可以使用阻塞队列，轻松实现一个“生产者 - 消费者模型”！

这种基于阻塞队列实现的“生产者 - 消费者模型”，可以有效地协调生产和消费的速度。当“生产者”生产数据的速度过快，“消费者”来不及消费时，存储数据的队列很快就会满了。这个时候，生产者就阻塞等待，直到“消费者”消费了数据，“生产者”才会被唤醒继续“生产”。

基于阻塞队列，我们还可以通过协调“生产者”和“消费者”的个数，来提高数据的处理效率。比如前面的例子，我们可以多配置几个“消费者”，来应对一个“生产者”。

![imgs/9f539cc0f1edc20e7fa6559193898067.jpg](imgs/9f539cc0f1edc20e7fa6559193898067.jpg)

阻塞队列在多线程情况下，会有多个线程同时操作队列，这个时候就会存在线程安全问题。

线程安全的队列我们叫作**并发队列**。

最简单直接的实现方式是直接在 enqueue()、dequeue() 方法上加锁，但是锁粒度大并发度会比较低，同一时刻仅允许一个存或者取操作。

实际上，基于数组的循环队列，利用 CAS 原子操作，可以实现非常高效的并发队列。这也是循环队列比链式队列应用更加广泛的原因。

### 队列在线程池等有限资源池中的应用

线程池没有空闲线程时，新的任务请求线程资源时，线程池该如何处理？

一般有两种处理策略：

- 非阻塞的处理方式，直接拒绝任务请求；
- 阻塞的处理方式，将请求排队，等到有空闲线程时，取出排队的请求继续处理。那如何存储排队的请求呢？（队列）

基于链表的实现方式，可以实现一个支持**无限排队**的无界队列（unbounded queue），但是可能会导致过多的请求排队等待，请求处理的响应时间过长。所以，针对响应时间比较敏感的系统，基于链表实现的无限排队的线程池是不合适的。

而基于数组实现的有界队列（bounded queue），队列的大小有限，所以线程池中排队的请求超过队列大小时，接下来的请求就会被拒绝，这种方式对响应时间敏感的系统来说，就相对更加合理。

**对于大部分资源有限的场景，当没有空闲资源时，基本上都可以通过“队列”这种数据结构来实现请求排队。**
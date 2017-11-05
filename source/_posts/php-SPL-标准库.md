---
title: php SPL 标准库
date: 2017-11-03 15:57:39
tags: 设计模式
category: php
---

PHP标准库（SPL）

1.  SplStack, SplQueue, SqlHeap, SqlFixedArray等数据结构
2.  ArrayIterator, AppendIterator, Countable, ArrayObject
3.  SPL提供的函数

 SPL常用数据结构

# 入栈

```php
$stack = new SplStack();
$stack->push("data1<br/>");
$stack->push("data2<br/>");

echo $stack->pop();
echo $stack->pop();
```

输出结果：先进后出
	data2
	data1

# 队列

```php
$queue = new SplQueue();
$queue->enqueue("data1<br/>");
$queue->enqueue("data2<br/>");

echo $queue->dequeue();
echo $queue->dequeue();
```

输出结果：先进先出
data1
data2

# 堆

```php
// 堆
$heap = new SplMinHeap();
$heap->insert("data1<br/>");
$heap->insert("data2<br/>");

echo $heap->extract();
echo $heap->extract();
```

输出结果：

data1
data2

#  固定长度的数组

```php
// 固定长度的数组
$array = new SplFixedArray(10);
$array[1] = 'num 1';
$array[5] = 'num 5';

var_dump($array);
```

输出结果：

object(SplFixedArray)[4]
  public 0 => null
  public 1 => string 'num 1' (length=5)
  public 2 => null
  public 3 => null
  public 4 => null
  public 5 => string 'num 5' (length=5)
  public 6 => null
  public 7 => null
  public 8 => null
  public 9 => null

  thinkphp5 中模型基类实现了SPL ArrayAccess接口,使其对象也可进行数组操作.

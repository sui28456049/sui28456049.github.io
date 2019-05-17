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
  
  
# spl接口
  预定义接口 
  * （迭代器）接口
  * 聚合式迭代器
  * 数组式访问
  * 序列化
  * Closure
  * 生成器
  
## Iterator 迭代器

### 什么是迭代器?

迭代器有时又称光标（cursor）是程式设计的软件设计模式，可在容器物件（container，例如list或vector）上遍访的接口，设计人员无需关心容器物件的内容。

各种语言实作Iterator的方式皆不尽同，有些面向对象语言像Java, C#, Python, Delphi都已将Iterator的特性内建语言当中，完美的跟语言整合，我们称之隐式迭代器（implicit iterator），但像是C++语言本身就没有Iterator的特色，但STL仍利用template实作了功能强大的iterator。

### 迭代器的使用范围

使用返回迭代器的包或库时（如PHP5中的SPL迭代器）
无法在一次的调用获取容器的所有元素时
要处理数量巨大的无素时（数据库中的表以GB计的数据）

### php迭代器的实现

PHP内置的 迭代器还真多。迭代器到底来做什么的呢，其实就是用来遍历一个对象内部数据并且获得想要的结构。迭代器它可以控制foreach 语句的循环结构，通过适当的拓展可以获得指定情况下遍历的结果。就例如，PHP提供搜索，递归，聚集，XMl等各类型的迭代器。种类繁多，功能齐全，比如要处理个XML，用个simplexmliterator 就立马见效。

基础的几个迭代器的接口：

Traversable : 这个是最初始的接口，他提供了控制foreach 语句的内置功能。所谓内置，就是只由PHP核心器去控制，外部无法调用。也就是说这个接口是无法被外部应用所应用。

Iterator ： 这个接口继承了Traversable ，并且包含了控制foreach 的五个重要的函数 : rewind() , valid(), key(),current(),next() ，它实现了foreach每一步的动作。

iteratoraggregate : 这个接口以继承了Traversable, 它最大的功能是可以将实现Iterator的类移除在外，自身不用去实现那五个方法便可实现控制foreach的目的。

### 调用顺序

```php
<?php
class myIterator implements Iterator {
    private $position = 0;
    private $array = array(
        "firstelement",
        "secondelement",
        "lastelement",
    );

    public function __construct() {
        $this->position = 0;
    }

    function rewind() {
        var_dump(__METHOD__);
        $this->position = 0;
    }

    function current() {
        var_dump(__METHOD__);
        return $this->array[$this->position];
    }

    function key() {
        var_dump(__METHOD__);
        return $this->position;
    }

    function next() {
        var_dump(__METHOD__);
        ++$this->position;
    }

    function valid() {
        var_dump(__METHOD__);
        return isset($this->array[$this->position]);
    }
}

$it = new myIterator;

foreach ($it as $key => $value)
{
    var_dump($key,$value);
    echo "\n";
}


string(18) "myIterator::rewind"
string(17) "myIterator::valid"
string(19) "myIterator::current"
string(15) "myIterator::key"
int(0)
string(12) "firstelement"

string(16) "myIterator::next"
string(17) "myIterator::valid"
string(19) "myIterator::current"
string(15) "myIterator::key"
int(1)
string(13) "secondelement"

string(16) "myIterator::next"
string(17) "myIterator::valid"
string(19) "myIterator::current"
string(15) "myIterator::key"
int(2)
string(11) "lastelement"

string(16) "myIterator::next"
string(17) "myIterator::valid"
```
当一个实现了Iterator接口的对象，被foreach遍历时，会自动调用这些方法。调用的循序是：
rewind() -> valid() -> current() -> key() -> next()

### 传入数组遍历

```php
class myIterator implements Iterator {
    private $position = 0;
    public $array = [];

    public function __construct(array $arr) {
        $this->array = $arr;
    }

    function rewind() {
        $this->position = 0;
    }

    function current() {
        return $this->array[$this->position];
    }

    function key() {
        return $this->position;
    }

    function next() {
        ++$this->position;
    }

    function valid() {
        return isset($this->array[$this->position]);
    }
}


$info = ['one','two','three','four'];

$arr = new myIterator($info);

foreach ($arr as $k => $v) {
    echo "{$k} ----- {$v}".PHP_EOL;
}
```

### while方式遍历

```php
class myIterator implements Iterator {
    private $position = 0;
    public $array = [];

    public function __construct(array $arr) {
        $this->array = $arr;
    }

    function rewind() {
        $this->position = 0;
    }

    function current() {
        return $this->array[$this->position];
    }

    function key() {
        return $this->position;
    }

    function next() {
        ++$this->position;
    }

    function valid() {
        return isset($this->array[$this->position]);
    }
}


$info = ['one','two','three','four'];

$arr = new myIterator($info);

// 返回到迭代器的第一个元素
$arr->rewind();

while($arr->valid())
{
    $key = $arr->key();
    $value = $arr->current();
    echo "{$key}-------{$value}".PHP_EOL;
    $arr->next();
}
```

### iteratoraggregate 接口

创建外部迭代器的接口。

可以将实现Iterator的类移除在外，自身不用去实现那五个方法便可实现控制foreach的目的。

eg1:

```php
class myData implements IteratorAggregate {
    public $property1 = "Public property one";
    public $property2 = "Public property two";
    public $property3 = "Public property three";

    public function __construct() {
        $this->property4 = "last property";
    }

    public function getIterator() {
        return new ArrayIterator($this);
    }
}

$obj = new myData;

foreach($obj as $key => $value) {
    var_dump($key, $value);
    echo "\n";
}
```


```php
class Collection implements IteratorAggregate
{
    private $items = [];

    public function __construct($items = [])
    {
        $this->items = $items;
    }

    public function getIterator()
    {
        foreach ($this->items as $key => $val)
        {
            yield $key => $val;
        }
    }
}

$data = [ 'A', 'B', 'C', 'D' ];
$collection = new Collection($data);

foreach ($collection as $key => $val) {
    echo sprintf("[%s] => %s\n", $key, $val);
}
```
## ArrayAccess（数组式访问）接口 

提供像访问数组一样访问对象的能力的接口。

```php
class obj implements arrayaccess {
    private $container = array();
    public function __construct(array $arr)
    {
        $this->container = $arr;
    }

    public function offsetSet($offset, $value)
    {
        // TODO: Implement offsetSet() method.
        if(is_null($offset))
        {
            $this->container[] = $value;
        }else{
            $this->container[$offset] = $value;
        }
    }
    public function offsetGet($offset)
    {
        // TODO: Implement offsetGet() method.
        return isset($this->container[$offset])? $this->container[$offset] : null;
    }
    public function offsetExists($offset)
    {
        // TODO: Implement offsetExists() method.
        return isset($this->container[$offset]);
    }
    public function offsetUnset($offset)
    {
        // TODO: Implement offsetUnset() method.
        unset($this->container[$offset]);
    }

}

$info = array(
    "one"   => 1,
    "two"   => 2,
    "three" => 3,
);

$obj = new obj($info);

var_dump($obj['two']);
```


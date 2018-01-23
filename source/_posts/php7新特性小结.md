---
title: php7新特性小结
date: 2018-01-23 09:57:02
tags: php
category: php
---

# 标量类型与返回值类型声明

标量类型声明 有两种模式: `强制 (默认)` 和 `严格模式`。

## 强制模式

```
// 强制模式 
function sum(int ...$ints) 
{ 
   return array_sum($ints); 
} 

print(sum(2, '3', 4.1)); 
```

## 严格模式

```php
// 严格模式 
declare(strict_types=1); 

function sum(int ...$ints) 
{ 
   return array_sum($ints); 
} 

print(sum(2, '3', 4.1)); 
```

## 返回类型声明

```php
function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
```

# null合并运算符

日常使用中存在大量同时使用三元表达式和 isset()的情况， 我们添加了null合并运算符 (??) 这个语法糖。如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。

原来

```php
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';
```

新写法

```php
$username = $_GET['user'] ?? 'nobody';
```

# 太空船操作符（组合比较符）

太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。

```php
// 整数
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
```

# 常量数组

```php
define('ANIMALS', ['dog','cat','pig','bird']);

print_r(ANIMALS);
```

# 匿名类

现在支持通过`new class` 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。

```php
# 定义一个接口,后续实现
interface Logger
{
	public function log(string $msg);
}

class Application
{
	private $logger;

	public function getLogger(): Logger
	{
		return $this->logger;
	}

	public function setLogger(Logger $logger)
	{
		$this->logger = $logger;
	}
}

$app = new Application;

$app->setLogger(new class implements Logger{
	public function log(string $msg)
	{
		  print($msg);
	}
});

$app->getLogger()->log('这是一条日志信息');
```

# 过滤 unserialize()

PHP 7 增加了可以为 unserialize() 提供过滤的特性，可以防止非法数据进行代码注入，提供了更安全的反序列化数据。

```php
class MyClass1 {  
   public $obj1prop;    
} 
class MyClass2 { 
   public $obj2prop; 
} 


$obj1 = new MyClass1(); 
$obj1->obj1prop = 1; 
$obj2 = new MyClass2(); 
$obj2->obj2prop = 2; 

$serializedObj1 = serialize($obj1); 
$serializedObj2 = serialize($obj2); 

// 默认行为是接收所有类 
// 第二个参数可以忽略 
// 如果 allowed_classes 设置为 false, unserialize 会将所有对象转换为 __PHP_Incomplete_Class 对象 
$data = unserialize($serializedObj1 , ["allowed_classes" => true]); 

// 转换所有对象到 __PHP_Incomplete_Class 对象，除了 MyClass1 和 MyClass2 
$data2 = unserialize($serializedObj2 , ["allowed_classes" => ["MyClass1", "MyClass2"]]);

print($data->obj1prop); 
print(PHP_EOL); 
print($data2->obj2prop); 
```

# 整数除法函数 intdiv()

```php
# 新加的函数 intdiv() 用来进行 整数的除法运算。
var_dump(intdiv(10, 3));  # 3
```
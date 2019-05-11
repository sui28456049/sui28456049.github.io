---
title: php数组复习
date: 2019-05-08 15:27:20
tags: php
category: php
---

php 依赖于强大的数组,封装好的,不需要自己写(自己写的性能不好,并且有时还不好写),有必要复习一下.好记性不如烂笔头,自己亲自敲一下.只写实例,参数看官网.

https://www.php.net/manual/zh/ref.array.php

# array_change_key_case

 将数组中的所有键名修改为全大写或小写

 ```php
 $input_arr = ['Fist' =>1, 'Two' => 2];
 print_r(array_change_key_case($input_arr, 'CASE_UPPER'))
 ```

* 多维数据递归键名大小写

```
function array_change_key_case_recursive($arr)
{
    return array_map(function($item) {
        if(is_array($item))
            $item = array_change_key_case_recursive($item);
        return $item;
    }, array_change_key_case($arr));
}
print_r(array_change_key_case_recursive(['FIRST'=>["TWO"=>["Three"=>'FOUR']]]));

结果:
Array
(
    [first] => Array
        (
            [two] => Array
                (
                    [three] => FOUR
                )

        )

)
```

# array_chunk

将一个数组分割成多个数组，其中每个数组的单元数目由 size 决定。最后一个数组的单元数目可能会少于 size 个。

分成多少列数组示例

```php
$arr= ['dog','pig','cat','orange','apple','banana'];

function array_chunk_columns($input, $num, $preserve_keys = FALSE) {
    $count = count($input) ;
    if($count)
        $input = array_chunk($input, ceil($count/$num), $preserve_keys) ;
    $input = array_pad($input, $num, array());
    return $input ;
}

print_r(array_chunk($arr,2));
print_r(str_repeat('***',10));
print_r(array_chunk_columns($arr,2));
```
# array_column

array_column — 返回数组中指定的一列

```php
$records = array(
    array(
        'id' => 2135,
        'first_name' => 'John',
        'last_name' => 'Doe',
    ),
    array(
        'id' => 3245,
        'first_name' => 'Sally',
        'last_name' => 'Smith',
    ),
    array(
        'id' => 5342,
        'first_name' => 'Jane',
        'last_name' => 'Jones',
    ),
    array(
        'id' => 5623,
        'first_name' => 'Peter',
        'last_name' => 'Doe',
    )
);

$last_names = array_column($records, 'last_name', 'id');
print_r($last_names);



Array
(
    [2135] => Doe
    [3245] => Smith
    [5342] => Jones
    [5623] => Doe
)
```

# array_combine

 创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值
 
 ```php
 $a = array('green', 'red', 'yellow');
 $b = array('avocado', 'apple', 'banana');
 $c = array_combine($a, $b);
 
 print_r($c);
 
 
 Array
 (
     [green]  => avocado
     [red]    => apple
     [yellow] => banana
 )
 ```
 
 * 以数组的 value 值 分组
 
 ```php
 $info = [
     '李世民' => '唐代',
     '秦琼'  => '唐代',
     '朱元璋' => '明朝'
 ];
 
 // 以 value 值 分组
 function flipAndGroup($input) :array
 {
     $outArr = [];
     array_walk($input, function($value, $key)use(&$outArr){
         if(!isset($outArr[$value]) || !is_array($outArr[$value]))
         {
             $outArr[$value] = [];
         }
         $outArr[$value][] = $key;
     });
     return $outArr;
 }
 
 print_r(flipAndGroup($info));
 
 Array
 (
     [唐代] => Array
         (
             [0] => 李世民
             [1] => 秦琼
         )
 
     [明朝] => Array
         (
             [0] => 朱元璋
         )
 
 )

 ```
 
# array_count_values

统计数组中所有的值

array_count_values() 返回一个数组： 数组的键是 array 里单元的值； 数组的值是 array 单元的值出现的次数。

```php
$list = [
  ['id' => 1, 'userId' => 5],
  ['id' => 2, 'userId' => 5],
  ['id' => 3, 'userId' => 6],
];
$userId = 5;

echo array_count_values(array_column($list, 'userId'))[$userId]; // outputs: 2
```

# array_diff_assoc

带索引检查计算数组的差集
array_diff_assoc() 返回一个数组，该数组包括了所有在 array1 中但是不在任何其它参数数组中的值。注意和 array_diff() 不同的是键名也用于比较。

# array_fill_keys

使用指定的键和值填充数组

```php
$keys = array('foo', 5, 10, 'bar');
$a = array_fill_keys($keys, 'banana');
print_r($a);


Array
(
    [foo] => banana
    [5] => banana
    [10] => banana
    [bar] => banana
)
```

# array_fill

用给定的值填充数组

```php
$a = array_fill(5, 6, 'banana');
$b = array_fill(-2, 4, 'pear');
print_r($a);
print_r($b);

Array
(
    [5]  => banana
    [6]  => banana
    [7]  => banana
    [8]  => banana
    [9]  => banana
    [10] => banana
)
Array
(
    [-2] => pear
    [0] => pear
    [1] => pear
    [2] => pear
)
```

# array_filter

用回调函数过滤数组中的单元

```php
$arr = ['a' => 1, 'b' => 2, 'c' => 3, 'd' => 4];

var_dump(array_filter($arr, function($k) {
    return $k == 'b';
}, ARRAY_FILTER_USE_KEY));

var_dump(array_filter($arr, function($v, $k) {
    return $k == 'b' || $v == 4;
}, ARRAY_FILTER_USE_BOTH));

array(1) {
  ["b"]=>
  int(2)
}
array(2) {
  ["b"]=>
  int(2)
  ["d"]=>
  int(4)
}
```

# array_flip

交换数组中的键和值

```php
$input = ["随"=>"某人",'李'=>"寻欢",'陈'=>'独秀']
print_r(array_flip($input));


Array
(
    [某人] => 随
    [寻欢] => 李
    [独秀] => 陈
)
```

# array_key_exists

检查数组里是否有指定的键名或索引

```php
$search_array = array('first' => 1, 'second' => 4);
if (array_key_exists('first', $search_array)) {
    echo "The 'first' element is in the array";
}
```

# array_key_first

```php
$array = ['a' => 1, 'b' => 2, 'c' => 3];

$firstKey = array_key_first($array);

var_dump($firstKey);
```

# array_key_last

# array_keys 

返回数组中部分的或所有的键名

```php
$array = array(0 => 100, "color" => "red");
print_r(array_keys($array));

$array = array("blue", "red", "green", "blue", "blue");
print_r(array_keys($array, "blue"));

$array = array("color" => array("blue", "red", "green"),
               "size"  => array("small", "medium", "large"));
print_r(array_keys($array));
```

# array_map

为数组的每个元素应用回调函数

```php
$func = function($value) {
    return $value * 2;
};

print_r(array_map($func, range(1, 5)))
```
# array_merge

```php
$array1 = array("color" => "red", 2, 4);
$array2 = array("a", "b", "color" => "green", "shape" => "trapezoid", 4);
$result = array_merge($array1, $array2);
print_r($result);


Array
(
    [color] => green
    [0] => 2
    [1] => 4
    [2] => a
    [3] => b
    [shape] => trapezoid
    [4] => 4
)
```

# array_merge_recursive

递归地合并一个或多个数组

```php
$ar1 = array("color" => array("favorite" => "red"), 5);
$ar2 = array(10, "color" => array("favorite" => "green", "blue"));
$result = array_merge_recursive($ar1, $ar2);
print_r($result);


Array
(
    [color] => Array
        (
            [favorite] => Array
                (
                    [0] => red
                    [1] => green
                )

            [0] => blue
        )

    [0] => 5
    [1] => 10
)
```

# array_pad

以指定长度将一个值填充进数组

```php

$arr = ['11111','22222','33333'];

print_r(array_pad($arr,6,'数组填充为 6 个值是这个字段'));

Array
(
    [0] => 11111
    [1] => 22222
    [2] => 33333
    [3] => 数组填充为 6 个值是这个字段
    [4] => 数组填充为 6 个值是这个字段
    [5] => 数组填充为 6 个值是这个字段
)
```

# array_rand

从数组中随机取出一个或多个单元,注意,这个返回的 数组的键

```php
$input = array("one", "two", "three", "four", "five");
$rand_keys = array_rand($input, 2);
echo $input[$rand_keys[0]] . "\n";
echo $input[$rand_keys[1]] . "\n";
```

# array_reduce

用回调函数迭代地将数组简化为单一的值

```php
function sum($carry, $item)
{
    $carry += $item;
    return $carry;
}

function product($carry, $item)
{
    $carry *= $item;
    return $carry;
}

$a = array(1, 2, 3, 4, 5);
$x = array();

var_dump(array_reduce($a, "sum")); // int(15)
var_dump(array_reduce($a, "product", 10)); // int(1200), because: 10*1*2*3*4*5
var_dump(array_reduce($x, "sum", "No data to reduce")); // string(17) "No data to reduce"
```
# array_replace

使用传递的数组替换第一个数组的元素

```php
$arr = ['111','222','333'];
$replace = ['哈哈','哈哈1','哈哈 2','哈哈 3'];

print_r(array_replace($arr,$replace));


Array
(
    [0] => 哈哈
    [1] => 哈哈1
    [2] => 哈哈 2
    [3] => 哈哈 3
)
```

# array_search

在数组中搜索给定的值，如果成功则返回首个相应的键名

```php
$array = array(0 => 'blue', 1 => 'red', 2 => 'green', 3 => 'red');

$key = array_search('green', $array); // $key = 2;
$key = array_search('red', $array);   // $key = 1;
```

# array_slice

从数组中取出一段

```php
$input = array("a", "b", "c", "d", "e");

$output = array_slice($input, 2);      // returns "c", "d", and "e"
$output = array_slice($input, -2, 1);  // returns "d"
$output = array_slice($input, 0, 3);   // returns "a", "b", and "c"

// note the differences in the array keys
print_r(array_slice($input, 2, -1));
print_r(array_slice($input, 2, -1, true));


Array
(
    [0] => c
    [1] => d
)
Array
(
    [2] => c
    [3] => d
)
```

# array_splice

去掉数组中的某一部分并用其它值取代

```php
$input = array("red", "green", "blue", "yellow");
array_splice($input, 2);
// $input is now array("red", "green")

$input = array("red", "green", "blue", "yellow");
array_splice($input, 1, -1);
// $input is now array("red", "yellow")

$input = array("red", "green", "blue", "yellow");
array_splice($input, 1, count($input), "orange");
// $input is now array("red", "orange")

$input = array("red", "green", "blue", "yellow");
array_splice($input, -1, 1, array("black", "maroon"));
// $input is now array("red", "green",
//          "blue", "black", "maroon")

$input = array("red", "green", "blue", "yellow");
array_splice($input, 3, 0, "purple");
// $input is now array("red", "green",
//          "blue", "purple", "yellow");
```
# array_unique

移除数组中重复的值

```php
$input = array("a" => "green", "red", "b" => "green", "blue", "red");
$result = array_unique($input);
print_r($result);
```

# array_values

返回数组中所有的值

# array_walk

 使用用户自定义函数对数组中的每个元素做回调处理

 # array_walk_recursive

 对数组中的每个成员递归地应用用户函数

 ```php
$sweet = array('a' => 'apple', 'b' => 'banana');
$fruits = array('sweet' => $sweet, 'sour' => 'lemon');

function test_print($item, $key)
{
    echo "$key holds $item\n";
}

array_walk_recursive($fruits, 'test_print');

a holds apple
b holds banana
sour holds lemon
 ```
 注意上例中的键 'sweet' 并没有显示出来。任何其值为 array 的键都不会被传递到回调函数中去。

 # arsort

 对数组进行逆向排序并保持索引关系

 ```php
$fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");
arsort($fruits);
foreach ($fruits as $key => $val) {
    echo "$key = $val\n";
}
 ```

 # asort

 对数组进行排序并保持索引关系

 ```php
 $fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");
asort($fruits);
foreach ($fruits as $key => $val) {
    echo "$key = $val\n";
}


c = apple
b = banana
d = lemon
a = orange
 ```

 # compact

  建立一个数组，包括变量名和它们的值

  ```php
$city  = "San Francisco";
$state = "CA";
$event = "SIGGRAPH";

$location_vars = array("city", "state");

$result = compact("event", "nothing_here", $location_vars);
print_r($result);
  ```
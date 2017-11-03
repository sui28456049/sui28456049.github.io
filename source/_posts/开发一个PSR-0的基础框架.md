---
title: 开发一个PSR-0的基础框架
date: 2017-11-03 20:29:19
tags: 设计模式
---

PSR-0规范

```c
PSR-0规范

命名空间必须与绝对路径保持一致
类名首字母必须大写
除入口文件之外， 其他“.php”文件必须只有1个类
```

# 创建目录结构

├─framework             框架目录
│  ├─index.php          入口文件
│  ├─App                应用文件夹
|  |--Controller\Home\Index.php (此处简易,控制器文件)
│  ├─Think              类库文件夹
│  │  └─Object.php                 公共类
│  │  └─Database.php                类库文件
│  │  └─Loder.php              ......加载文件

# 创建公共类库文件(Object.php)

```php
// 公共类
class Object
{
    static function test()
    {
        echo __METHOD__;
    }
}
```

# 创建控制器(Index.php)

```php
namespace App\Controller\Home;

// 控制器
class Index
{
    static function test()
    {
        echo __METHOD__;
    }
}
```

# 创建自动加载类文件(Loder.php)
```php
namespace Think;

class Loder
{
    static function autoload($class)
    {
        // sui($class);
        require BASEDIR . '/' . str_replace('\\', '/', $class) . '.php';
    }
}
```

# 入口文件(index.php)

```php
// 入口文件
define('BASEDIR', __DIR__);
include BASEDIR . '/Think/Loder.php';
spl_autoload_register('\\Think\\Loder::autoload');

App\Controller\Home\Index::test();
echo "<br />";
Think\Object::test();
```

输出结果:
```php
App\Controller\Home\Index::test
Think\Object::test
```

# PHP常用魔术方法

1.  __get , __set //当给一个对象不存在的变量赋值时，自动调用 __set() 方法，读取一个不存在的对象的值时，自动调用 __get() 方法。

2. 	__call , __callStatic // 访问对象中1个不存在的方法，自动执行魔术方法 __call，如果访问的是静态方法，自动执行魔术方法 __callStatc 。

3.	__toString // 对象本身是不能直接输出的，当输出一个对象时，对象会自定回调访问 __toString() 魔术方法。
4.	__invoke // 当把一个对象当做函数一样使用，会自动调用魔术方法：__invoke()



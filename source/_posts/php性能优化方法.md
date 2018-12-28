---
title: php性能优化方法
date: 2018-12-28 08:52:37
tags: php
category: php
---

# 开启opcache

## Opcode???

Opcache 的前生是 Optimizer+ ，它是PHP的官方公司 Zend 开发的一款闭源但可以免费使用的 PHP 优化加速组件。 Optimizer+ 将PHP代码预编译生成的脚本文件 Opcode 缓存在共享内存中供以后反复使用，从而避免了从磁盘读取代码再次编译的时间消耗。同时，它还应用了一些代码优化模式，使得代码执行更快。从而加速PHP的执行。

Optimizer+ 于 2013年3月中旬改名为 Opcache。并且在 PHP License 下开源: 
`https://github.com/zendtech/ZendOptimizerPlus`

## Opcache的生命周期

正常的php代码的执行过程如下：

![opcode](/uploads/20181228_01.jpg)

```
request请求（nginx,apache,cli等）-->Zend引擎读取.php文件-->扫描其词典和表达式 -->解析文件-->创建要执行的计算机代码(称为Opcode)-->最后执行Opcode--> response 返回
```

每一次请求PHP脚本都会执行一遍以上步骤，如果PHP源代码没有变化，那么Opcode也不会变化，显然没有必要每次都重行生成Opcode，结合在Web中无所不在的缓存机制，我们可以把Opcode缓存下来，以后直接访问缓存的Opcode岂不是更快，启用Opcode缓存之后的流程图如下所示：

![opcode](/uploads/20181228_02.jpg)

```
request请求（nginx,apache,cli等）-->Zend引擎读取.php文件-->读取Opcode-->执行Opcode--> response 返回
```

## Opcache的安装

php5.5 以后默认安装,需要手动去开启

```
zend_extension=opcache.so
[opcache]
opcache.enable=1  #允许在web环境使用，默认是开启的。
opcache.enable_cli=1 #运行在cli环境使用
```
重启 fpm 和 nginx


## 推荐的 php.ini 中 Opcache的配置

```
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.fast_shutdown=1 ;(在PHP 7.2.0中被移除，会自动开启)
opcache.enable_cli=1
```

## opcache的配置说明

```
opcache.enable=1 (default "1")
;OPcache打开/关闭开关。当设置为Off或者0时，会关闭Opcache, 代码没有被优化和缓存。
opcache.enable_cli=1 (default "0")
;CLI环境下，PHP启用OPcache。这主要是为了测试和调试。从 PHP 7.1.2 开始，默认启用。
opcache.memory_consumption=128 (default "64")
;OPcache共享内存存储大小。用于存储预编译的opcode（以MB为单位）。
opcache.interned_strings_buffer=8 (default "4")
;这是一个很有用的选项，但是似乎完全没有文档说明。PHP使用了一种叫做字符串驻留（string interning）的技术来改善性能。例如，如果你在代码中使用了1000次字符串“foobar”，在PHP内部只会在第一使用这个字符串的时候分配一个不可变的内存区域来存储这个字符串，其他的999次使用都会直接指向这个内存区域。这个选项则会把这个特性提升一个层次——默认情况下这个不可变的内存区域只会存在于单个php-fpm的进程中，如果设置了这个选项，那么它将会在所有的php-fpm进程中共享。在比较大的应用中，这可以非常有效地节约内存，提高应用的性能。
这个选项的值是以兆字节（megabytes）作为单位，如果把它设置为16，则表示16MB，默认是4MB，这是一个比较低的值。
opcache.max_accelerated_files (default "2000")
;这个选项用于控制内存中最多可以缓存多少个PHP文件。这个选项必须得设置得足够大，大于你的项目中的所有PHP文件的总和。
设置值取值范围最小值是 200，最大值在 PHP 5.5.6 之前是 100000，PHP 5.5.6 及之后是 1000000。也就是说在200到1000000之间。
你可以运行“find . -type f -print | grep php | wc -l”这个命令来快速计算你的代码库中的PHP文件数。
opcache.max_wasted_percentage (default "5")
;计划重新启动之前，“浪费”内存的最大百分比。
opcache.use_cwd (default "1")
;如果启用，OPcache将在哈希表的脚本键之后附加改脚本的工作目录， 以避免同名脚本冲突的问题。禁用此选项可以提高性能，但是可能会导致应用崩溃
opcache.validate_timestamps (default "1")
;如果启用（设置为1），OPcache会在opcache.revalidate_freq设置的秒数去检测文件的时间戳（timestamp）检查脚本是否更新。
如果这个选项被禁用（设置为0），opcache.revalidate_freq会被忽略，PHP文件永远不会被检查。这意味着如果你修改了你的代码，然后你把它更新到服务器上，再在浏览器上请求更新的代码对应的功能，你会看不到更新的效果，你必须使用 `opcache_reset()` 或者 `opcache_invalidate()` 函数来手动重置 OPcache。或者重重你的web服务器或者php-fpm 来使文件系统更改生效。
我强烈建议你在生产环境中设置为0，why？因为当你在更新服务器代码的时候，如果代码较多，更新操作是有些延迟的，在这个延迟的过程中必然出现老代码和新代码混合的情况，这个时候对用户请求的处理必然存在不确定性。最后，等所有的代码更新完毕后，再平滑重启PHP和web服务器。
opcache.revalidate_freq (default "2")
;这个选项用于设置缓存的过期时间（单位是秒），当这个时间达到后，opcache会检查你的代码是否改变，如果改变了PHP会重新编译它，生成新的opcode，并且更新缓存。值为“0”表示每次请求都会检查你的PHP代码是否更新（这意味着会增加很多次stat系统调用，译注：stat系统调用是读取文件的状态，这里主要是获取最近修改时间，这个系统调用会发生磁盘I/O，所以必然会消耗一些CPU时间，当然系统调用本身也会消耗一些CPU时间）。可以在开发环境中把它设置为0，生产环境下不用管。
如果 `opcache.validate_timestamps` 配置指令设置为禁用（设置为0），那么此设置项将会被忽略。
opcache.revalidate_path (default "0")
;在include_path优化中启用或禁用文件搜索
如果被禁用，并且找到了使用的缓存文件相同的include_path，该文件不被再次搜索。因此，如果一个文件与include_path中的其他地方相同的名称出现将不会被发现。如果此优化对此有效，请启用此指令你的应用程序，这个指令的默认值是禁用的，这意味着该优化是活跃的。
```

## 验证opcache是否生效

先把检查时间设置为10秒，也就是说10秒内，不会去检查文件是否有更新，直接用缓存中的opcode

```php
zend_extension=opcache.so
[opcache]
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=10
opcache.fast_shutdown=1
opcache.enable_cli=1
```

把opcache.validate_timestamps=0设置为0，则表示，永远不去检查文件是否更新。

## 重置缓存

var_dump(opcache_reset());  //bool(true)

重置所有的opcache缓存。

## opcache图形化管理工具

 https://github.com/rlerdorf/opcache-status

# HugePage
 参考博客: http://www.laruence.com/2015/10/02/3069.html

 ```
 opcache.huge_code_pages=1
 ```

 # Opcache file cache

 ```
 opcache.file_cache=/tmp

 # PGO
 ```


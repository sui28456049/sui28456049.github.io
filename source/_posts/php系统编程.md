---
title: php系统编程
date: 2017-12-22 18:57:40
tags: php
category: php
---

> PHP的进程控制支持实现了Unix方式的进程创建, 程序执行, 信号处理以及进程的中断。 进程控制不能被应用在Web服务器环境，当其被用于Web服务环境时可能会带来意外的结果。

新建一个pcntl.php 文件

```php
$pid = pcntl_fork();

if($pid == -1)
{
	//错误处理:创建子进程失败返回-1

	die('cloud not fork');

}elseif ($pid) {

	//父进程会得到子进程号，所以这里是父进程执行的逻辑

	pcntl_wait($status); //等待子进程中断,防止子进程成为僵尸进程
	echo '||||||||||||||'; echo "父进程 $pid";
	
}else{
	 //子进程得到的$pid为0, 所以这里是子进程执行的逻辑。
	echo "子进程-------$pid";
	sleep(100);
}
```

查看进程情况

```bash
ps -ef|grep pcntl
```

```
root     16931  4875  1 08:55 pts/1    00:00:00 php pcntl.php
root     16932 16931  0 08:55 pts/1    00:00:00 php pcntl.php
root     16934 14333  0 08:55 pts/2    00:00:00 grep --color=auto pcntl
```

# PHP程序守护进程化

## 使用 supervisor

## nohub

```bash
nohup php test.php > log.txt &
```

单独执行 php test.php，当按下ctrl+c时就会中断程序执行，会kill当前进程以及子进程。

php test.php &，这样执行程序虽然也是转为后台运行，实际上是依赖终端的，当用户退出终端时进程就会被杀掉

## 

```php
function daemonize()
{
    $pid = pcntl_fork();

    if ($pid == -1)
    {
    die("fork(1) failed!\n");
    }
    elseif ($pid > 0)
    {
    //让由用户启动的进程退出
    exit(0);
    }

    //建立一个有别于终端的新session以脱离终端
    posix_setsid();

    $pid = pcntl_fork();

    if ($pid == -1)
    {
        die("fork(2) failed!\n");
    }
    elseif ($pid > 0)
    {
        //父进程退出, 剩下子进程成为最终的独立进程
        exit(0);
    }
    # echo '哈哈哈,没人管我了~~~';
}

daemonize();
sleep(150);
```

PHP程序需要转为后台运行时，只需要调用一次封装好的函数daemonize()即可

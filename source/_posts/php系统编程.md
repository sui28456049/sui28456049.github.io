---
title: php系统编程
date: 2017-12-22 18:57:40
tags: php
category: php
---

> PHP的进程控制支持实现了Unix方式的进程创建, 程序执行, 信号处理以及进程的中断。 进程控制不能被应用在Web服务器环境，当其被用于Web服务环境时可能会带来意外的结果。



#  PHP进程



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

##  孤儿进程

```php
//模拟 父进程先退出,子进程变成孤儿进程,被init1 收养

echo "本程序的进程pid:".getmypid().PHP_EOL;

	$pid = pcntl_fork();
	if($pid == 0)
	{
		//子进程
		echo "子进程pid为:".getmypid().PHP_EOL;
	    system("ps -ef|grep fork");
        sleep(10);
        system("ps -ef|grep fork");
		
	}else{
		//父进程
		echo "父进程pid为:".getmypid().PHP_EOL;
		sleep(3);
		exit(0);		
	}

```

## 僵尸进程

```php
//模拟 子进程先退出,子进程变成僵尸进程

echo "本程序的进程pid:".getmypid().PHP_EOL;

	$pid = pcntl_fork();
	if($pid == 0)
	{
		//子进程
		echo "子进程pid为:".getmypid().PHP_EOL;
		 system("ps -f");
		 exit(0);
	}else{
		//父进程
		echo "父进程pid为:".getmypid().PHP_EOL;
		# pcntl_wait($status);
		sleep(30);
		echo PHP_EOL;
		system("ps axu|grep Z");
	
	}
```



# PHP程序守护进程化

## 使用 supervisor

### 安装

```bash
 sudo apt-get install supervisor
```

### 命令

Supervisor 有两个可执行程序 – supervisord 和 supervisorctl:

`supervisord` 是后台管理服务器, 用来依据配置文件的策略管理后台守护进程, 它会随系统自动启动
`supervisorctl` 用于管理员向后台管理程序发送 启动/重启/停止 等指令;
它们之间的关系就相当于 Apache 的 httpd 和 apachectl.

### 配置文件

主配置文件 的路径位于 /etc/supervisor/supervisord.conf, 主配置文件中的末尾两行文本:

```
[include]
files = /etc/supervisor/conf.d/*.conf


指明了 Supervisor 会去 /etc/supervisor/conf.d/ 目录下查找以 .conf 结尾的子配置文件, 也就是说, 我们只需在 /etc/supervisor/conf.d/ 目录下为每个后台守护应用新建一个配置文件即可.

子配置文件示例

[program:php]
command = /usr/bin/php /home/sui/test.php
```
### 运行

注意: 每次 修改主配置文件 或 增改子配置文件 都需要执行 supervisorctl update 使新配置生效

```bash
    # 控制所有进程
    sudo supervisorctl start all
    sudo supervisorctl stop all
    sudo supervisorctl restart all

    # 定向控制指定进程
    sudo supervisorctl stop php
    sudo supervisorctl start php
    sudo supervisorctl restart php
```

## nohup

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
}

daemonize();
sleep(150);
```

改进版:

```php
Class Daemon{  
      
    /** 
     * 初始化一个守护进程 
     * @throws Exception 
     */  
    public function init(){  
        //创建一个子进程  
        $pid = pcntl_fork();  
          
        if ($pid == -1) {  
            throw new Exception('fork子进程失败');  
        } elseif ($pid > 0) {  
            //父进程退出,子进程变成孤儿进程被1号进程收养，进程脱离终端  
            exit(0);  
        }  
          
        //创建一个新的会话，脱离终端控制，更改子进程为组长进程  
        $sid = posix_setsid();  
        if ($sid == -1) {  
            throw new Exception('setsid fail');  
        }  
          
        //修改当前进程的工作目录，由于子进程会继承父进程的工作目录，修改工作目录以释放对父进程工作目录的占用。  
        chdir('/');  
          
        /** 
         * 通过上一步，我们创建了一个新的会话组长，进程组长，且脱离了终端，但是会话组长可以申请重新打开一个终端，为了避免 
         * 这种情况，我们再次创建一个子进程，并退出当前进程，这样运行的进程就不再是会话组长。 
         */  
        $pid = pcntl_fork();  
        if ($pid == -1) {  
            throw new Exception('fork子进程失败');  
        } elseif ($pid > 0) {  
            //再一次退出父进程，子进程成为最终的守护进程  
            exit(0);  
        }  
        //由于守护进程用不到标准输入输出，关闭标准输入，输出，错误输出描述符  
        fclose(STDIN);  
        fclose(STDOUT);  
        fclose(STDERR);  
    }  
}  
  
$daemon = new Daemon();  
$daemon->init();  

//处理业务代码  
while(true) 
{  
    file_put_contents('/home/sui/test/log.txt', time().PHP_EOL, FILE_APPEND);  
    sleep(5);  
}  
```

PHP程序需要转为后台运行时，只需要调用一次封装好的函数daemonize()即可

# 后台建立简单服务器

```php
/* 设置不显示任何错误 */
// error_reporting(0);

/* 脚本超时为无限 */
set_time_limit(0);

/* 开始固定清除 */
ob_implicit_flush();

/* 本机的IP和需要开放的端口 */
$address = '127.0.0.1';
$port = 10000;

/* 产生一个Socket */
if (($sock = socket_create(AF_INET, SOCK_STREAM, SOL_TCP)) < 0) {
    echo "socket_create() failed: reason: " . socket_strerror($sock) . "\n";
}

/* 把IP地址端口进行绑定 */
if (($ret = socket_bind($sock, $address, $port)) < 0) {
    echo "socket_bind() failed: reason: " . socket_strerror($ret) . "\n";
}

/* 监听Socket连接 */
if (($ret = socket_listen($sock, 5)) < 0) {
    echo "socket_listen() failed: reason: " . socket_strerror($ret) . "\n";
}

/* 永远循环监接受用户连接 */
do {
    if (($msgsock = socket_accept($sock)) < 0) {
        echo "socket_accept() failed: reason: " . socket_strerror($msgsock) . "\n";
        break;
    }
    /* 发送提示信息给连接上来的用户 */
    $msg = "==========================================\r\n" .
 " Welcome to the PHP Test Server. \r\n\r\n".
        " To quit, type 'quit'. \r\n" .
 " To shut down the server type 'shutdown'.\r\n" .
 " To get help message type 'help'.\r\n" .
 "==========================================\r\n" .
 "php> ";
    socket_write($msgsock, $msg, strlen($msg));

    do {
        if (false === ($buf = socket_read($msgsock, 2048, PHP_NORMAL_READ))) {
            echo "socket_read() failed: reason: " . socket_strerror($ret) . "\n";
            break 2;
        }
        if (!$buf = trim($buf)) {
            continue;
        }
  /* 客户端输入quit命令时候关闭客户端连接 */
        if ($buf == 'quit') {
            break;
        }
  /* 客户端输入shutdown命令时候服务端和客户端都关闭 */
        if ($buf == 'shutdown') {
            socket_close($msgsock);
            break 2;
        }
  /* 客户端输入help命令时候输出帮助信息 */
  if ($buf == 'help') {
   $msg = " PHP Server Help Message \r\n\r\n".
   " To quit, type 'quit'. \r\n" .
   " To shut down the server type 'shutdown'.\r\n" .
   " To get help message type 'help'.\r\n" .
   "php> ";
    socket_write($msgsock, $msg, strlen($msg));
    continue;
  }
  /* 客户端输入命令不存在时提示信息 */
        $talkback = "PHP: unknow command '$buf'.\r\nphp> ";
        socket_write($msgsock, $talkback, strlen($talkback));
        echo "$buf\n";
    } while (true);
    socket_close($msgsock);
} while (true);

/* 关闭Socket连接 */
socket_close($sock);
```

测试

```bash
telnet 127.0.0.1 10000
```

## swoole_process::daemon
使当前进程蜕变为一个守护进程。

~~~
bool swoole_process::daemon(bool $nochdir = true, bool $noclose = true);
~~~

* $nochdir，为true表示不要切换当前目录到根目录。
* $noclose，为true表示不要关闭标准输入输出文件描述符。

* 此函数在1.7.5-stable版本后可用
* 1.9.1或更高版本修改了默认值，现在默认nochir和noclose均为true
* 蜕变为守护进程时，该进程的PID将重新fork，可以使用getmypid()来获取当前的PID


使用PHP代码来实现daemon

~~~
function daemonize()
{
    $pid = pcntl_fork();
    if ($pid == -1){
        die("fork(1) failed!\n");
	}
    elseif ($pid > 0){
        //让由用户启动的进程退出
        exit(0);
	}

    //建立一个有别于终端的新session以脱离终端
    posix_setsid();

    $pid = pcntl_fork();
    if ($pid == -1){
        die("fork(2) failed!\n");
    }elseif ($pid > 0){
        //父进程退出, 剩下子进程成为最终的独立进程
        exit(0);
    }
}

daemonize();
~~~
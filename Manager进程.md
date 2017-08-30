# Manager进程
swoole中worker/task进程都是由Manager进程Fork并管理的。

* 子进程结束运行时，manager进程负责回收此子进程，避免成为僵尸进程。并创建新的子进程
* 服务器关闭时，manager进程将发送信号给所有子进程，通知子进程关闭服务
* 服务器reload时，manager进程会逐个关闭/重启子进程

>为什么不是Master进程呢，主要原因是Master进程是多线程的，不能安全的执行fork操作。
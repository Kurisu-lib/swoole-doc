# Swoole的实现
swoole使用纯C编写，不依赖其他第三方库。

* swoole并没有用libevent，所以不需要安装libevent
* swoole并不依赖php的stream/sockets/pcntl/posix/sysvmsg等扩展

**socket部分**

swoole使用底层的socket系统调用。参见 sys/socket.h

**IO事件循环**
* 主进程的事件循环使用select/poll，因为主线程中的文件描述符只有几个，使用select/poll即可
* reactor线程/worker进程中使用epoll/kqueue
* task进程没有事件循环，进程会循环阻塞读取管道

>有很多人使用strace -p去查看swoole主进程只能看到poll系统调用。正确的查看方法是strace -f -p

**多进程/多线程**
多进程使用fork()系统调用
多线程使用pthread线程库
**EventFd**
Swoole中使用了eventfd作为线程/进程间消息通知的机制。

**Timerfd**
Swoole使用timerfd来实现定时器

**SIgnalfd**
swoole中使用了signalfd来实现对信号的屏蔽和处理。可以有效地避免线程/进程被信号打断，系统调用restart的问题。在主进程中reactor线程不会接受任何信号。
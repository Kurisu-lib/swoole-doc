## 异步毫秒定时器 
swoole_server中已经提供了定时器的API，如果是在客户端程序中，也想使用毫秒定时器。可以用swoole提供的swoole_timer模块。

swoole_timer与PHP本身的pcntl_alarm是不同的。pcntl_alarm是基于时钟信号 + PHP tick函数实现,有4个缺陷：

* 最大仅支持到秒，而swoole_timer可以到毫秒级别
* 不支持同时设定多个定时器程序
* pcntl_alarm依赖declare(ticks = 1)性能很差
* 无法用于异步IO，只支持同步方式
swoole_timer是基于timerfd+epoll实现的异步毫秒定时器，可完美的运行在EventLoop中，与swoole_client/swoole_event等模块可以无缝结合。
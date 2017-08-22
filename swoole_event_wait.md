## swoole_event_wait

~~~
void swoole_event_wait(void);
~~~

PHP5.4之前的版本没有在ZendAPI中加入注册shutdown函数。所以swoole无法在脚本结尾处自动进行事件轮询。所以低于5.4的版本，需要在你的PHP脚本结尾处加swoole_event_wait函数。使脚本开始进行事件轮询。

* 5.4或更高版本不需要加此函数
* SwooleServer下也不需要加
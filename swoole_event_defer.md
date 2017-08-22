## swoole_event_defer

在下一个事件循环开始时执行函数。

~~~
swoole_event_defer(mixed $callback_function);
~~~
swoole_event_defer函数会在当前EventLoop的事件循环结束、下一次事件循环启动时响应

$callback_function 时间到期后所执行的函数，必须是可以调用的。回调函数不接受任何参数
可以使用匿名函数的use语法传递参数到回调函数中
使用示例
~~~
swoole_event_defer(function(){
    echo "After EventLoop\n";
});
~~~

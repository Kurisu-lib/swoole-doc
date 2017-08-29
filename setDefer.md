# setDefer
setDefer()

~~~
bool setDefer([bool $is_defer = true]);
~~~
* $is_defer：bool值，为true时，表明该Client要延迟收包，为false时，表明该Client非延迟收包，默认值为true
* 返回值：设置成功返回true，否则返回false。只有一种情况会返回false，当设置defer(true)并发包后，尚未recv()收包，就设置defer(false)，此时返回false。
* 如果需要进行延迟收包，需要在发包之前调用
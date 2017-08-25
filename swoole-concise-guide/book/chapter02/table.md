# Table
swoole_table一个基于共享内存和锁实现的超高性能，并发数据结构。用于解决多进程/多线程数据共享和同步加锁问题。

>最新版本已移除lock和unlock方法，请使用Swoole\Lock来实现数据同步

## swoole_table的优势
* 性能强悍，单线程每秒可读写200万次
* 无需加锁，swoole_table内置行锁自选锁，所有操作均是多线程/多进程安全。用户层完全不需要考虑数据同步问题。
* 支持多进程，swoole_table可以用于多进程之间共享数据
>swoole_table使用行锁，而不是全局锁，仅当2个进程在同一CPU时间，并发读取同一条数据才会进行发生抢锁
>swoole_table在1.7.5版本后可用

## 遍历Table
swoole_table类实现了迭代器和Countable接口，可以使用foreach进行遍历，使用count计算当前行数。

>遍历Table 依赖pcre 如果发现无法遍历table,检查机器是否安装pcre-devel

~~~
foreach($table as $row)
{
    var_dump($row);
}
echo count($table);
~~~
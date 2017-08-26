# Table
swoole_table一个基于共享内存和锁实现的超高性能，并发数据结构。用于解决多进程/多线程数据共享和同步加锁问题。

>最新版本已移除lock和unlock方法，请使用Swoole\Lock来实现数据同步

## swoole_table的优势
* 性能强悍，单线程每秒可读写200万次
* 无需加锁，swoole_table内置行锁自选锁，所有操作均是多线程/多进程安全。用户层完全不需要考虑数据同步问题。
* 支持多进程，swoole_table可以用于多进程之间共享数据

>swoole_table使用行锁，而不是全局锁，仅当2个进程在同一CPU时间，并发读取同一条数据才会进行发生抢锁
>swoole_table在1.7.5版本后可用

# 常量列表

swoole_table 目前只支持以下3种类型，个人感觉有点儿类似于表字段类型。
* swoole_table::TYPE_INT 整形字段  
* swoole_table::TYPE_FLOAT 浮点字段
* swoole_table::TYPE_STRING 字符串字段

# 函数列表

* [__construct](#__construct)
* [column](#column)
* [create](#create)
* [set](#set)
* [get](#get)
* [del](#del)
* [lock](#lock)
* [unlock](#unlock)

## __construct
**功能描述**：内存表构造函数，创建内存表对象对象。  
**函数原型**：  
```php
swoole_table->__construct(int $size)
```
**返回**：一个swoole_table对象。  
**参数说明**：  

| 参数        | 说明   |  
|  --------  |  -------- |  
| int size |   表格的最大行数   |  
**说明**：  
* 创建对象后会创建一个Mutex锁。  
* $table->lock()/$table->unlock() 在这之后即可使用。   

> swoole_table基于行锁，所以单次set/get/del在多线程/多进程的环境下是安全的  
set/get/del是原子操作，用户代码中不需要担心数据加锁和同步的问题  

**样例**：  
```php
$table = new swoole_table(1024);
```

## column
**功能描述**：给内存表增加一列  
**函数原型**：  
```php
bool swoole_table->column(string $name, int $type, $size);
```

**参数说明**：  

| 参数        | 说明   |  
|  --------  |  -------- |  
| string name  |  新增的字段名称  |  
| int type  |  swoole_table支持的3种字段类型,详情见[常量列表](#%E5%B8%B8%E9%87%8F%E5%88%97%E8%A1%A8) |  
| int size |  这是根据type的不同占用的字节数 |  

**说明**：

> swoole_table::TYPE_INT默认为4个字节，可以设置1，2，4，8一共4种长度  
swoole_table::TYPE_STRING设置后，set操作不能设置的值不能超过此长度  
swoole_table::TYPE_FLOAT会占用8个字节的内存  

**样例**：  
```php
$table->column('id', swoole_table::TYPE_INT, 4); 
```

## create
**功能描述**：创建内存表。  
**函数原型**：
```php
swoole_table->create()
```
**说明**：  
创建好表的结构后，执行create后创建表。  

> swoole_table使用共享内存来保存数据，在创建子进程前，务必要执行swoole_table->create()  
swoole_server中使用swoole_table，swoole_table->create() 必须在swoole_server->start()前执行

**样例**：  
```php
$table = new swoole_table(1024);
$table->column('id', swoole_table::TYPE_INT, 4);       //1,2,4,8
$table->column('name', swoole_table::TYPE_STRING, 64);
$table->column('num', swoole_table::TYPE_FLOAT);
$table->create();

$worker = new swoole_process('child1', false, false);
$worker->start();
```

## set
**功能描述**：设置行的数据，(swoole_table使用key-value的方式来访问数据，个人感觉key类似于db表记录的id，value则是整条记录的所有列的值。)  
**函数原型**：  
```php
swoole_table->set(string $key, array $value)
```

**参数说明**：  

| 参数        | 说明   |  
|  --------  |  -------- |  
| string key  |  数据的key，相同的$key对应同一行数据，如果set同一个key，会覆盖上一次的数据 |  
| array $value  |  必须是一个数组，value中的键名必须与字段定义的$name完全相同 |  

**说明**：  

> swoole_table->set() 可以设置全部字段的值，也可以只修改部分字段  
swoole_table->set() 未设置前，该行数据的所有字段均为空

## get
**功能描述**：获取一行数据。
**函数原型**：
```php
array swoole_table->get($key);
```

**返回**：  
* 如果找到则返回array 数组。
* 如果$key不存在，将返回false。

## del

**功能描述**：删除一行数据。
**函数原型**：
```php
bool swoole_table->del(string $key)
```

**返回**：  
* $key对应的数据不存在，将返回false。
* 成功删除返回true

## lock

**功能描述**：锁定整个表。
**函数原型**：
```php
swoole_table->lock()
```

**使用场景**：当多个进程同时要操作一个事务性操作时，一定要加锁，将整个表锁定。操作完成后释放锁。  
**说明**：  
* lock() 是互斥锁，所以只能保护lock/unlock中间的代码是安全的。lock/unlock之外的操作是不能保护的。
* set/get/del操作不使用互斥锁，所以lock之后无法阻止其他进程调用这些函数。

**注意**：

> lock/unlock必须成对出现，否则会发生死锁，这里务必要小心  
lock/unlock之间不应该加入太多操作，避免锁的粒度太大影响程序性能  
lock/unlock之间的代码，应当try/catch避免抛出异常导致跳过unlock发生死锁

## unlock

**功能描述**：释放锁。
**函数原型**：
```php
swoole_table->unlock()
```
## 整体使用案例
```php
$table = new swoole_table(1024);
$table->column('id', swoole_table::TYPE_INT, 4);       //1,2,4,8
$table->column('name', swoole_table::TYPE_STRING, 64);
$table->column('num', swoole_table::TYPE_FLOAT);
$table->create();

$table->set('tianfenghan@qq.com', array('id' => 145, 'name' => 'rango', 'num' => 3.1415));
$table->set('350749960@qq.com', array('id' => 358, 'name' => "Rango1234", 'num' => 3.1415));
$table->set('hello@qq.com', array('id' => 189, 'name' => 'rango3', 'num' => 3.1415));

$data = $table->get('350749960@qq.com');
$table->del('hello@qq.com');
$table->lock();
/**
事务性处理。
**/
$table->unlock();

count($table);  // 获得有多少条记录。
```

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
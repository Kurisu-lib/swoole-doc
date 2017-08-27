# swoole_server函数列表
---

[TOC]

## **swoole_server::__construct**
**功能描述**：创建一个swoole_server资源对象

**函数原型**：
```php
// 类成员函数
public function swoole_server::__construct(string $host, int $port, int $mode = SWOOLE_PROCESS,
    int $sock_type = SWOOLE_SOCK_TCP);
// 公共函数
function swoole_server_create(string $host, int $port, int $mode = SWOOLE_PROCESS,
    int $sock_type = SWOOLE_SOCK_TCP);
```
**返回**：一个swoole_server对象

**参数说明**：

| 参数        | 说明   |
|  --------  |  -------- |
| string host | 监听的IP地址 |
| int port |   监听的端口号   |
| int mode |   运行模式   |
| int sock_type |   指定的socket类型   |

**说明**：
host、port、socket_type的详细说明见[swoole_server::addlistener](#swoole_serveraddlistener)。<br>
mode指定了swoole_server的运行模式，共有如下三种：<br>

| mode        | 类型   | 说明 |
|  --------  |  -------- | -------- |
| SWOOLE_BASE | Base模式 | 传统的异步非阻塞Server。在Reactor内直接回调PHP的函数。如果回调函数中有阻塞操作会导致Server退化为同步模式。worker_num参数对与BASE模式仍然有效，swoole会启动多个Reactor进程 |
| SWOOLE_THREAD | 线程模式（已废弃）| 多线程Worker模式，Reactor线程来处理网络事件轮询，读取数据。得到的请求交给Worker线程去处理。多线程模式比进程模式轻量一些，而且线程之间可以共享堆栈和资源。 访问共享内存时会有同步问题，需要使用Swoole提供的锁机制来保护数据。 |
| SWOOLE_PROCESS | 进程模式（默认） | Swoole提供了完善的进程管理、内存保护机制。 在业务逻辑非常复杂的情况下，也可以长期稳定运行，适合业务逻辑非常复杂的场景。 |

**样例**:<br>
```php
$serv = new swoole_server("127.0.0.1" , 8888 , SWOOLE_PROCESS , SWOOLE_SOCK_TCP);
```

## **swoole_server::set**
**功能描述**：设置swoole_server运行时的各项参数<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::set(array $setting);
// 公共函数
function swoole_server_set(swoole_server $server, array $setting);
```
**返回**：无<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| array setting | 配置选项数组，采用key-value形式 |

**说明**：<br>
该函数必须在[swoole_server::start](#swoole_serverstart)函数调用前调用。<br>
全部swoole_server的配置参数[点此查看](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/01.%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9.md)<br>
**样例**:<br>
```php
$serv->set(
    array(
        'worker_num' => 8,
        'max_request' => 10000,
        'max_conn' => 100000,
        'dispatch_mode' => 2,
        'debug_mode'=> 1，
        'daemonize' => false,
    )
);
```

## **swoole_server::on**
**功能描述**：绑定swoole_server的相关回调函数<br>
**函数原型**：<br>
```php
// 类成员函数
public function bool swoole_server->on(string $event, mixed $callback);
```
**返回**：设置成功返回true，否则返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| string event | 回调的名称（大小写不敏感） |
| mixed callback | 回调的PHP函数，可以是函数名的字符串，类静态方法，对象方法数组，匿名函数 |

**说明**：<br>
该函数必须在[swoole_server::start](#swoole_serverstart)函数调用前调用。<br>
此方法与[swoole_server::handler](#swoole_serverhandler)功能相同，作用是与swoole_client风格保持一致。<br>
[swoole_server::on](#swoole_serveron)中事件名称字符串不要加on。<br>
全部的回调函数列表[点此查看](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md)<br>
**样例**:<br>
```php
$serv->on('connect', function ($serv, $fd){
    echo "Client:Connect.\n";
});
$serv->on('receive', array( $myclass, 'onReceive' ) ); // onReceive是myclass的成员函数
```

## **swoole_server::addlistener**
**功能描述**：给swoole_server增加一个监听的地址和端口<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::addlistener(string $host, int $port, $type = SWOOLE_SOCK_TCP);
// 公共函数
function swoole_server_addlisten(swoole_server $serv, string $host, int $port, $type = SWOOLE_SOCK_TCP);
```
**返回**：无<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| string host | 监听的IP地址 |
| int port |   监听的端口号   |
| int sock_type |   指定的socket类型   |

**说明**：
swoole支持如下socket类型:<br>

| sock_type        | 说明   |
|  --------  |  -------- |
| SWOOLE_TCP/SWOOLE_SOCK_TCP | TCP IPv4 Socket |
| SWOOLE_TCP6/SWOOLE_SOCK_TCP6 | TCP IPv6 Socket |
| SWOOLE_UDP/SWOOLE_SOCK_UDP | UDP IPv4 Socket |
| SWOOLE_UDP6/SWOOLE_SOCK_UDP6 | UDP IPv4 Socket |
| SWOOLE_UNIX_DGRAM | Unix UDP Socket |
| SWOOLE_UNIX_STREAM | Unix TCP Socket |

> Unix Socket仅在1.7.1+后可用，此模式下**host**参数必须填写可访问的文件路径，**port**参数忽略<br>
Unix Socket模式下，客户端**fd**将不再是数字，而是一个文件路径的字符串<br>
SWOOLE_TCP等是1.7.0+后提供的简写方式，与1.7.0前的SWOOLE_SOCK_TCP是等同的<br>

**样例**:<br>
```php
$serv->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP);
$serv->addlistener("192.168.1.100", 9503, SWOOLE_SOCK_TCP);
$serv->addlistener("0.0.0.0", 9504, SWOOLE_SOCK_UDP);
$serv->addlistener("/var/run/myserv.sock", 0, SWOOLE_UNIX_STREAM);

swoole_server_addlisten($serv, "127.0.0.1", 9502, SWOOLE_SOCK_TCP);
```

## **swoole_server::handler**
**功能描述**：设置Server的事件回调函数<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::handler(string $event_name, mixed $event_callback_function);
// 公共函数
function swoole_server_handler(swoole_server $serv, string $event_name, mixed $event_callback_function);
```
**返回**：设置成功返回true，否则返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| string event_name | 回调的名称（大小写不敏感） |
| mixed event_callback_function | 回调的PHP函数，可以是函数名的字符串，类静态方法，对象方法数组，匿名函数 |

**说明**：
该函数必须在[swoole_server::start](#swoole_server::start)函数调用前调用。<br>
事件名称字符串要加on。<br>
全部的回调函数列表[点此查看](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md)<br>
> onConnect/onClose/onReceive这3个回调函数必须设置。其他事件回调函数可选<br>
如果设定了timer定时器，onTimer事件回调函数也必须设置<br>
如果启用了task_worker，onTask/onFinish事件回调函数必须设置<br>

**样例**:
```php
$serv->handler('onStart', 'my_onStart');
$serv->handler('onStart', array($this, 'my_onStart'));
$serv->handler('onStart', 'myClass::onStart');
```

## **swoole_server::start**
**功能描述**：启动server，开始监听所有TCP/UDP端口<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::start()
```
**返回**：启动成功返回true，否则返回false<br>
**参数说明**：无<br>
**说明**：<br>
启动成功后会创建worker_num+2个进程：Master进程+Manager进程+worker_num 个 Worker进程。<br>
另外。启用task_worker会增加task_worker_num个Worker进程<br>
三种进程的说明如下：

| 进程类型        | 说明   |
|  --------  |  -------- |
| Master进程 | 主进程内有多个Reactor线程，基于epoll/kqueue进行网络事件轮询。收到数据后转发到Worker进程去处理 |
| Manager进程 | 对所有Worker进程进行管理，Worker进程生命周期结束或者发生异常时自动回收，并创建新的Worker进程 |
| Worker进程 | 对收到的数据进行处理，包括协议解析和响应请求 |

**样例**:
```php
$serv->start();
```

## **swoole_server::reload**
**功能描述**：重启所有worker进程。<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::reload()
```
**返回**：调用成功返回true，否则返回false<br>
**参数说明**：无<br>
**说明**：<br>
调用后会向Manager发送一个SIGUSR1信号，平滑重启全部的Worker进程（所谓平滑重启，是指重启动作会在Worker处理完正在执行的任务后发生，并不会中断正在运行的任务。）<br>

小技巧：在[onWorkerStart](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#3onworkerstart)回调中require相应的php文件，当这些文件被修改后，只需要通过SIGUSR1信号即可实现服务器热更新。<br>

1.7.7版本增加了仅重启task_worker的功能。只需向服务器发送SIGUSR2即可<br>
**样例**:
```php
$serv->reload();
```

## **swoole_server::shutdown**
**功能描述**：关闭服务器。<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::shutdown()
```
**返回**：调用成功返回true，否则返回false<br>
**参数说明**：无<br>
**说明**：<br>
此函数可以用在worker进程内，平滑关闭全部的Worker进程。<br>
也可向Master进程发送SIGTERM信号关闭服务器。<br>
**样例**:
```php
$serv->shutdown();
```

## **swoole_server::tick**
**功能描述**：设置一个固定间隔的定时器<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server->tick(int $ms, callable $callback, mixed $user_param);
// 公共函数
function swoole_timer_tick(int $ms, callable $callback, mixed $user_param);
```
**返回**：设置成功返回true，否则返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int interval | 定时器的时间间隔，单位为毫秒ms |
| callback $callback | 定时器的回调函数，单位为毫秒ms |

**说明**：<br>
swoole定时器的最小颗粒是1毫秒，支持多个不同间隔的定时器。<br>
注意不能存在2个相同间隔时间的定时器。<br>
使用多个定时器时，其他定时器必须为最小定时器时间间隔的整数倍。<br>

该函数只能在onWorkerStart/onConnect/onReceive/onClose回调函数中调用。<br>
增加定时器后需要为Server设置[onTimer](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#8ontimer)回调函数，否则Server将无法启动。<br>

**样例**:
```php
$serv->addtimer(1000);              //1s
swoole_server_addtimer($serv,20);   //20ms
```

## **swoole_server::deltimer**
**功能描述**：删除指定的定时器。<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::deltimer(int $interval);
```

**返回**：无<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int interval | 定时器的时间间隔，单位为毫秒ms |

**说明**：<br>
删除间隔为interval的定时器

**样例**:
```php
$serv->deltimer(1000);
```

## **swoole_server::after**
**功能描述**：在指定的时间后执行函数<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::after(int $after_time_ms, mixed $callback_function, mixed params);
// 公共函数
function swoole_timer_after(int $after_time_ms, mixed $callback_function, mixed params);
```
**返回**：无<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int after_time_ms | 指定时间，单位为毫秒ms |
| mixed callback_function | 指定的回调函数 |
| mixed params | 指定的回调函数的参数 |

**说明**：<br>
创建一个一次性定时器，在指定的after_time_ms时间后调用callback_funciton回调函数，执行完成后就会销毁。<br>
callback_function函数的参数为指定的params<br>
需要swoole-1.7.7以上版本。<br>

**样例**:
```php
$serv->after(1000, function( $params ){
    echo "Do something\n";
} , "data" );
```

## **swoole_server::close**
**功能描述**：关闭客户端连接<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::close(int $fd, int $from_id = 0);
```
**返回**：关闭成功返回true，失败返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int fd | 指定关闭的fd |
| int from_id | fd来自于哪个Reactor（swoole-1.6以后已废弃该参数） |

**说明**：<br>
调用close关闭连接后，连接并不会马上关闭，因此不要在close之后立即写清理逻辑，而是应该在[onClose](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#5onclose)回调中处理<br>

**样例**:
```php
$serv->close( $fd );
```

## **swoole_server::send**
**功能描述**：向客户端发送数据<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::send(int $fd, string $data, int $from_id = 0);
```
**返回**：发送成功返回true，失败返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int fd | 指定发送的fd |
| string data | 发送的数据 |
| int from_id | fd来自于哪个Reactor |

**说明**：<br>

- data，发送的数据，最大不得超过2M。send操作具有原子性，多个进程同时调用send向同一个连接发送数据，不会发生数据混杂。
- 如果要发送超过2M的数据，建议将数据写入临时文件，然后通过sendfile接口直接发送文件
- UDP协议，send会直接在worker进程内发送数据包
- 向UDP客户端发送数据，必须要传入from_id
- 发送成功会返回true，如果连接已被关闭或发送失败会返回false
- swoole-1.6以上版本不需要from_id
- UDP协议使用fd保存客户端IP，from_id保存from_fd和port
- UDP协议如果是在onReceive后立即向客户端发送数据，可以不传from_id

**样例**:
```php
$serv->send( $fd , "Hello World");
```

## **swoole_server::sendfile**
**功能描述**：发送文件到TCP客户端连接<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::sendfile(int $fd, string $filename);
```
**返回**：发送成功返回true，失败返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int fd | 指定发送的fd |
| string filename | 发送的文件名 |

**说明**：<br>
sendfile函数调用OS提供的sendfile系统调用，由操作系统直接读取文件并写入socket。sendfile只有2次内存拷贝，使用此函数可以降低发送大量文件时操作系统的CPU和内存占用。<br>

**样例**:
```php
$serv->sendfile($fd, __DIR__.'/test.jpg');
```

## **swoole_server::connection_info**
**功能描述**：获取连接的信息<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::connection_info(int $fd, int $from_id = 0);
```
**返回**：如果fd存在，返回一个数组，连接不存在或已关闭返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int fd | 指定发送的fd |
| int from_id | 来自于哪个Reactor |

**说明**：<br>
UDP socket调用该参数时必须传入from_id.<br>
返回的结果如下：

| 名称        | 说明   |
|  --------  |  -------- |
| int from_id | 来自于哪个Reactor |
| int from_fd | 来自哪个Server Socket |
| int from_port | 来自哪个Server端口 |
| int remote_port | 客户端连接的端口 |
| string remote_ip | 客户端连接的ip |
| int connect_time | 连接到Server的时间，单位秒 |
| int last_time | 最后一次发送数据的时间，单位秒 |

sendfile函数调用OS提供的sendfile系统调用，由操作系统直接读取文件并写入socket。sendfile只有2次内存拷贝，使用此函数可以降低发送大量文件时操作系统的CPU和内存占用。<br>

**样例**:
```php
$fdinfo = $serv->connection_info($fd);
$udp_client = $serv->connection_info($fd, $from_id);
```

## **swoole_server::connection_list**
**功能描述**：遍历当前Server的全部客户端连接<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::connection_list(int $start_fd = 0, int $pagesize = 10);
```
**返回**：调用成功将返回一个数字索引数组，元素是取到的fd。数组会按从小到大排序。最后一个fd作为新的start_fd再次尝试获取。<br>
调用失败返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| int start_fd | 起始fd |
| int pagesize | 每页取多少条，最大不得超过100. |

**说明**：<br>
connection_list仅可用于TCP，UDP服务器需要自行保存客户端信息<br>

**样例**:
```php
$start_fd = 0;
while(true)
{
    $conn_list = $serv->connection_list($start_fd, 10);
    if($conn_list===false)
    {
        echo "finish\n";
        break;
    }
    $start_fd = end($conn_list);
    var_dump($conn_list);
    foreach($conn_list as $fd)
    {
        $serv->send($fd, "broadcast");
    }
}
```

## **swoole_server::stats**
**功能描述**：获取当前Server的活动TCP连接数，启动时间，accpet/close的总次数等信息。<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server->stats();
```
**返回**：结果数组<br>
**参数说明**：无<br>
**说明**：<br>
stats方法在1.7.5+后可用<br>

| 名称        | 说明   |
|  --------  |  -------- |
| int start_time | 启动时间 |
| int connection_num | 当前的连接数 |
| int accept_count | accept总次数 |
| int close_count | close连接的总数 |

**样例**:
```php
$status = $serv->stats();
```

## **swoole_server::task**
**功能描述**：投递一个异步任务到task_worker池中<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::task(string $data, int $dst_worker_id = -1);
```
**返回**：调用成功返回task_worker_id，失败返回false<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| string data | task数据 |
| int dst_worker_id | 指定投递给哪个task进程，默认随机投递 |

**说明**：<br>
此功能用于将慢速的任务异步地去执行，调用task函数会立即返回，Worker进程可以继续处理新的请求。<br>
函数会返回一个`$task_id`数字，表示此任务的ID<br>
任务完成后，可以通过return(低于swoole-1.7.2版本使用[finish](#swoole_serverfinish)函数)将结果通过[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)回调返回给Worker进程。<br>
发送的data必须为字符串，如果是数组或对象，请在业务代码中进行serialize处理，并在[onTask](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#6ontask)/[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)中进行unserialize。<br>
data可以为二进制数据，最大长度为**8K**(超过8K可以使用临时文件共享)。字符串可以使用gzip进行压缩。<br>
使用task必须为Server设置[onTask](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#6ontask)和[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)回调,并指定[task_worker_num](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/01.%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9.md#6task_worker_num)配置参数。


**样例**:
```php
$task_id = $serv->task("some data");
```

## **swoole_server::taskwait**
**功能描述**：投递一个同步任务到task_worker池中<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::taskwait(string $task_data, float $timeout = 0.5, int $dst_worker_id = -1);
```
**返回**：task执行的结果<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| string data | task数据 |
| float timeout | 超时时间 |
| int dst_worker_id | 指定投递给哪个task进程，默认随机投递 |

**说明**：<br>
taskwait与task方法作用相同，用于投递一个异步的任务到task进程池去执行。与task不同的是taskwait是阻塞等待的，直到任务完成或者超时返回。<br>
任务完成后，可以通过return直接返回结果<br>

**样例**:
```php
$task_id = $serv->taskwait("some data"， 30);
```
## **swoole_server->taskWaitMulti** 
**功能描述**：并发执行多个Task
**函数原型**：<br>

~~~
array swoole_server->taskWaitMulti(array $tasks, double $timeout = 0.5);
~~~
* $tasks 必须为数字索引数组，不支持关联索引数组，底层会遍历$tasks将任务逐个投递到Task进程
* $timeout 为浮点型，单位为秒，默认为0.5
* 任务完成或超时，返回结果数组。结果数组中每个任务结果的顺序与$tasks对应，如：$tasks[2]对应的结果为$result[2]
* 某个任务执行超时不会影响其他任务，返回的结果数据中将不包含超时的任务

>taskWaitMulti接口在1.8.8或更高版本可用
>最大并发任务不得超过1024

**使用实例**
```php
$tasks[] = mt_rand(1000, 9999); //任务1
$tasks[] = mt_rand(1000, 9999); //任务2
$tasks[] = mt_rand(1000, 9999); //任务3
var_dump($tasks);

//等待所有Task结果返回，超时为10s
$results = $serv->taskWaitMulti($tasks, 10.0);

if (!isset($results[0])) {
    echo "任务1执行超时了\n";
}
if (isset($results[1])) {
    echo "任务2的执行结果为{$results[1]}\n";
}
if (isset($results[2])) {
    echo "任务3的执行结果为{$results[2]}\n";
}
```
## **swoole_server->taskCo** 
**功能描述**：并发执行Task并进行协程调度。仅用于2.0版本。
**函数原型**：<br>
~~~
function swoole_server->taskCo(array $tasks, float $timeout = 0.5) : array;
~~~
* $tasks任务列表，必须为数组。底层会遍历数组，将每个元素作为task投递到Task进程池
* $timeout超时时间，默认为0.5秒，当规定的时间内任务没有全部完成，立即中止并返回结果
* 任务完成或超时，返回结果数组。结果数组中每个任务结果的顺序与$tasks对应，如：$tasks[2]对应的结果为$result[2]
* 某个任务执行失败或超时，对应的结果数组项为false，如：$tasks[2]失败了，那么$result[2]的值为false

>最大并发任务不得超过1024
>taskCo在2.0.9或更高版本可用

**调度过程**
$tasks列表中的每个任务会随机投递到一个Task工作进程，投递完毕后，yield让出当前协程，并设置一个$timeout秒的定时器
在onFinish中收集对应的任务结果，保存到结果数组中。判断是否所有任务都返回了结果，如果为否，继续等待。如果为是，进行resume恢复对应协程的运行，并清除超时定时器
在规定的时间内任务没有全部完成，定时器先触发，底层清除等待状态。将未完成的任务结果标记为false，立即resume对应协程
**使用示例**
~~~
<?php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num' => 1,
    'task_worker_num' => 2,
]);

$server->on('Task', function (swoole_server $serv, $task_id, $worker_id, $data) {
    echo "#{$serv->worker_id}\tonTask: worker_id={$worker_id}, task_id=$task_id\n";
    if ($serv->worker_id == 1) {
        sleep(1);
    }
    return $data;
});

$server->on('Request', function ($request, $response) use ($server)
{
    $tasks[0] = "hello world";
    $tasks[1] = ['data' => 1234, 'code' => 200];
    $result = $server->taskCo($tasks, 0.5);
    $response->end('Test End, Result: '.var_export($result, true));
});

$server->start();
~~~

## **swoole_server::finish**
**功能描述**：传递Task结果数据给worker进程<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::finish(string $task_data);
```
**返回**：无<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| string data | 结果数据 |

**说明**：<br>
使用swoole_server::finish函数必须为Server设置[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)回调函数。此函数只可用于Task Worker进程的[onTask](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#6ontask)回调中<br>
swoole_server::finish是可选的。如果Worker进程不关心任务执行的结果，可以不调用此函数<br>
此函数在swoole-1.7.2以上版本已废弃，使用return代替。<br>
**样例**:
```php
$serv->finish("result data");
```

## **swoole_server::heartbeat**
**功能描述**：进行心跳检测<br>
**函数原型**：<br>
```php
// 类成员函数
public function swoole_server::heartbeat(bool $if_close_connection = true);
```
**返回**：无<br>
**参数说明**：<br>

| 参数        | 说明   |
|  --------  |  -------- |
| bool if_close_connection | 是否关闭超时的连接，默认为true |

**说明**：<br>
该函数会主动发起一次检测，遍历全部连接，根据设置的[heartbeat_check_interval](server/set.md)和[heartbeat_idle_time](server/set.md)参数，找到那些处于idle闲置状态的连接<br>
swoole默认会直接关闭这些连接，heartbeat会返回这些连接的fd<br>
如果if_close_connection为false，则heartbeat会返回这些idle连接的fd，但不会关闭这些连接<br>
if_close_connection参数 在swoole-1.7.4以上版本可用<br>
**样例**:
```php
$serv->heartbeat();
```
## **swoole_server->getSocket**
**功能描述**：调用此方法可以得到底层的socket句柄，返回的对象为sockets资源句柄。

>此方法需要依赖PHP的sockets扩展，并且编译swoole时需要开启--enable-sockets选项

使用socket_set_option函数可以设置更底层的一些socket参数。

~~~
$socket = $server->getSocket();
if (!socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1)) {
    echo 'Unable to set option on socket: '. socket_strerror(socket_last_error()) . PHP_EOL;
}
~~~
**监听端口**
使用listen方法增加的端口，可以使用Swoole\Server\Port对象提供的getSocket方法。

~~~
$port = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
$socket = $port->getSocket();
~~~

**支持组播**
使用socket_set_option设置MCAST_JOIN_GROUP参数可以将Socket加入组播，监听网络组播数据包。
~~~
$server = new swoole_server('0.0.0.0', 9905, SWOOLE_BASE, SWOOLE_SOCK_UDP);
$server->set(['worker_num' => 1]);
$socket = $server->getSocket();

$ret = socket_set_option(
    $socket,
    IPPROTO_IP,
    MCAST_JOIN_GROUP,
    array('group' => '224.10.20.30', 'interface' => 'eth0')
);

if ($ret === false)
{
    throw new RuntimeException('Unable to join multicast group');
}

$server->on('Packet', function (swoole_server $serv, $data, $addr)
{
    $serv->sendto($addr['address'], $addr['port'], "Swoole: $data");
    var_dump( $addr, strlen($data));
});

$server->start();
~~~
* group 表示组播地址
* interface 表示网络接口的名称，可以为数字或字符串，如eth0、wlan0

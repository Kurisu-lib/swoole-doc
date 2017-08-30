# Reactor线程
Swoole的主进程是一个多线程的程序。其中有一组很重要的线程，称之为Reactor线程。它就是真正处理TCP连接，收发数据的线程。

Swoole的主线程在Accept新的连接后，会将这个连接分配给一个固定的Reactor线程，并由这个线程负责监听此socket。在socket可读时读取数据，并进行协议解析，将请求投递到Worker进程。在socket可写时将数据发送给TCP客户端。

>分配的计算方式是fd % serv->reactor_num

**TCP和UDP的差异**
* TCP客户端，Worker进程处理完请求后，调用$server->send会将数据发给Reactor线程，由Reactor线程再发给客户端
* UDP客户端，Worker进程处理完请求后，调用$server->sendto会直接发给客户端，无需经过Reactor线程
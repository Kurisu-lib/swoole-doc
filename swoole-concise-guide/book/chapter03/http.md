# HttpServer 
swoole-1.7.7增加了内置Http服务器的支持，通过几行代码即可写出一个异步非阻塞多进程的Http服务器。

~~~
$http = new swoole_http_server("127.0.0.1", 9501);
$http->on('request', function ($request, $response) {
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});
$http->start();
~~~
>swoole_http_server对Http协议的支持并不完整，建议仅作为应用服务器。并且在前端增加Nginx作为代理

通过使用apache bench工具进行压力测试，在Inter Core-I5 4核 + 8G内存的普通PC机器上，swoole_http_server可以达到近11万QPS。远远超过php-fpm，golang自带http服务器，node.js自带http服务器。性能几乎接近与Nginx的静态文件处理。

~~~
ab -c 200 -n 200000 -k http://127.0.0.1:9501
~~~

## 使用Http2协议
* 需要依赖nghttp2库，下载nghttp2后编译安装
* 使用Http2协议必须开启openssl
* 需要高版本openssl必须支持TLS1.2、ALPN、NPN
* ./configure --enable-openssl --enable-http2
* 设置http服务器的open_http2_protocol为true

~~~
$serv = new swoole_http_server("127.0.0.1", 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$serv->set([
    'ssl_cert_file' => $ssl_dir . '/ssl.crt',
    'ssl_key_file' => $ssl_dir . '/ssl.key',
    'open_http2_protocol' => true,
]);
~~~

## nginx+swoole配置
~~~
server {
    root /data/wwwroot/;
    server_name local.swoole.com;

    location / {
        proxy_http_version 1.1;
        proxy_set_header Connection "keep-alive";
        proxy_set_header X-Real-IP $remote_addr;
        if (!-e $request_filename) {
             proxy_pass http://127.0.0.1:9501;
        }
    }
}
~~~
>在swoole中通过读取$request->header['x-real-ip']来获取客户端的真实IP
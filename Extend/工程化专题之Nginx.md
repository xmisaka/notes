## nginx常用命令  
- 启动nginx  ./sbin/nginx  
- 停止nginx ./sbin/nginx -s stop    ./sbin/nginx -s quit  
- 重载配置  ./sbin/nginx -s reload(平滑重启)  service nginx reload   
- 重载指定配置文件 ./sbin/nginx -c /usr/local/nginx/conf/nginx.conf  
- 查看nginx版本 ./sbin/nginx -v  
- 检查配置文件是否正确 ./sbin/nginx -t  
- 显示帮助信息 ./sbin/nginx -h  

## nginx状态码  
499：服务端处理时间过长，客户端主动关闭了连接。

## nginx是如何实现高并发的  
一个主进程，多个工作进程，每个工作进程可以处理多个请求  
每进来一个request，会有一个worker进程去处理。但不是全程的处理，处理到可能发生阻塞的地方，比如向上游（后端）服务器转发request，并等待请求返回。那么，这个处理的worker继续处理其他请求，而一旦上游服务器返回了，就会触发这个事件，worker才会来接手，这个request才会接着往下走。  
由于web server的工作性质决定了每个request的大部份生命都是在网络传输中，实际上花费在server机器上的时间片不多。这是几个进程就解决高并发的秘密所在。即@skoo所说的webserver刚好属于网络io密集型应用，不算是计算密集型。

## nginx功能  
- 作为http server(代替apache，对PHP需要FastCGI处理器支持)  
- 反向代理服务器  
- 实现负载均衡  
- 虚拟主机  
- FastCGI：Nginx本身不支持PHP等语言，但是它可以通过FastCGI来将请求扔给某些语言或框架处理

## 502错误可能原因  
- FastCGI进程是否已经启动
- FastCGI worker进程数是否不够
- FastCGI执行时间过长
    - fastcgi_connect_timeout 300;
    - fastcgi_send_timeout 300;
    - fastcgi_read_timeout 300;
- FastCGI Buffer不够
    - nginx和apache一样，有前端缓冲限制，可以调整缓冲参数
    - fastcgi_buffer_size 32k;
    - fastcgi_buffers 8 32k;
- Proxy Buffer不够  
如果你用了Proxying，调整  
    - proxy_buffer_size   16k;
    - proxy_buffers    4 16k;
- php脚本执行时间过长
将php-fpm.conf的<value name="request_terminate_timeout">0s</value>的0s改成一个时间

## nignx配置
- worker_processes  8;     工作进程个数
- worker_connections  65535;  每个工作进程能并发处理（发起）的最大连接数（包含所有连接数）
- error_log         /data/logs/nginx/error.log;  错误日志打印地址
- access_log      /data/logs/nginx/access.log  进入日志打印地址
- log_format  main  '$remote_addr"$request" ''$status $upstream_addr "$request_time"'; 进入日志格式

- fastcgi_connect_timeout=300; #连接到后端fastcgi超时时间
- fastcgi_send_timeout=300; #向fastcgi请求超时时间(这个指定值已经完成两次握手后向fastcgi传送请求的超时时间)
- fastcgi_rend_timeout=300; #接收fastcgi应答超时时间，同理也是2次握手后
- fastcgi_buffer_size=64k; #读取fastcgi应答第一部分需要多大缓冲区，该值表示使用1个64kb的缓冲区读取应答第一部分(应答头),可以设置为fastcgi_buffers选项缓冲区大小
- fastcgi_buffers 4 64k;#指定本地需要多少和多大的缓冲区来缓冲fastcgi应答请求，假设一个php或java脚本所产生页面大小为256kb,那么会为其分配4个64kb的缓冲来缓存
- fastcgi_cache TEST;#开启fastcgi缓存并为其指定为TEST名称，降低cpu负载,防止502错误发生

- listen       80;                                            监听端口
- server_name  rrc.test.jiedaibao.com;       允许域名
- root  /data/release/rrc/web;                    项目根目录
- index  index.php index.html index.htm;  访问根文件

## nginx和apache的区别
- 轻量级，同样起web 服务，比apache 占用更少的内存及资源 
- 抗并发，nginx 处理请求是异步非阻塞的，而apache 则是阻塞型的，在高并发下nginx 能保持低资源低消耗高性能
- 高度模块化的设计，编写模块相对简单 
- 最核心的区别在于apache是同步多进程模型，一个连接对应一个进程；nginx是异步的，多个连接（万级别）可以对应一个进程 

## fastcgi与cgi的区别
cgi:
web服务器会根据请求的内容，然后会fork一个新进程来运行外部c程序（或perl脚本...）， 这个进程会把处理完的数据返回给web服务器，最后web服务器把内容发送给用户，刚才fork的进程也随之退出。 如果下次用户还请求改动态脚本，那么web服务器又再次fork一个新进程，周而复始的进行。
fastcgi:
web服务器收到一个请求时，他不会重新fork一个进程（因为这个进程在web服务器启动时就开启了，而且不会退出），web服务器直接把内容传递给这个进程（进程间通信，但fastcgi使用了别的方式，tcp方式通信），这个进程收到请求后进行处理，把结果返回给web服务器，最后自己接着等待下一个请求的到来，而不是退出。 

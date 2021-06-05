[toc]

# 高性能反向代理器Nginx

Nginx是俄罗斯人编写的一款高性能的HTTP和反向代理服务器，在高连接并发的情况下，它能够支持高达50000个并发连接数的响应，但是内存、CPU等系统资源消耗却很低，运行很稳定。

## 1、Nginx安装

（1）解压安装包

```shell
tar -xvzf nginx-1.18.0.tar.gz
```

（2）配置安装目录

```shell
./configure --prefix=/usr/soft/nginx
```

安装过程中可能会出现缺少pcre、openssl等依赖的问题，此时需要用到yum去安装这些依赖，安装完依赖之后再次配置安装目录

```shell
yum install -y gcc pcre pcre-devel openssl openssl-devel gd gd-devel
```

（3）编译安装

```shell
make && make install
```

（4）启动Nginx

进入安装目录的sbin目录下

执行 -c表示指定配置文件

```
./nginx -c /usr/soft/nginx/conf/nginx.conf
```

启动之后可在浏览器直接输入ip访问nginx首页

![image-20200726230202561](.\typora-user-images\image-20200726230202561.png)

（5）停止 Nginx

可用kill命令

```
kill -QUIT  进程号
kil -TERM  进程号
```

也可用nginx的脚本

```
./nginx -s stop  停止
./nginx -s quit   退出
./nginx -s reload  重新加载nginx.conf
```



## 2、Nginx核心配置

主要包括三部分：Main（全局设置）、Event、Http（主机设置）

```shell
user nobody nobody;#主模块指令，指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。
#Main
worker_processes  1; #需开启的进程数
#Event，设定工作模式以及连接数上限
events {
    use epoll; #Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中。对于Linux系统，epoll工作模式是首选。跟netty的是一样的
    worker_connections  1024; #每个进程的最大连接数
}
#Http
http {
    include       mime.types;#主模块指令，实现对配置文件所包含的文件的设定，可以减少主配置文件的复杂度
    default_type  application/octet-stream;#如果没有include的类型，则采用默认类型
    server {
        listen       80; #监听端口
        server_name  localhost; #主机名
        location / { #匹配url
            root   html; #转至html目录
            index  index.html index.htm; #目录下的文件
        }
        error_page   500 502 503 504  /50x.html; #定制错误码返回页面
        location = /50x.html {
            root   html;
        }
    }
}
```

### （1）日志配置

日志配置有两个参数，分别是 log_format和access_log。

log_format：用来设置日志格式

access_log：日志记录

```shell
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
access_log  logs/access.log  main;
```



格式：

logo声明  路径及文件名 日志标识

例如 ：

```
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
```



```shell
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;

    server {
        listen       80;
        server_name  localhost;
	    access_log  logs/host.access.log  main;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }   
     }
}     
```



### （2）location语法配置

格式：

```
location [~|=|^~|~*] /uri {
	 
}
```

匹配规则 ：

=：精准匹配，优先级最高

```shell
location =/index  {
}
```

~：开头表示区分大小写的正则匹配

^~：开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）	。以xx开头

~*：开头表示不区分大小写的正则匹配 

`!~`和`!~*`分别为区分大小写不匹配及不区分大小写不匹配 的正则

/： 一般匹配，任何请求都会匹配到。普通匹配的优先级要高于正则匹配，如果存在多个相同的前缀的一般匹配，那么最终会按照最大长度来做匹配

```shell
location /index  {
}
```



![image-20200726233928462](.\typora-user-images\image-20200726233928462.png)





### （3）Rewrite的使用

Rewrite通过ngx_http_rewrite_module模块支持url重写、支持if判断，但不支持else
rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向
rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用



常用指令
If 空格 (条件) {设定条件进行重写}
条件的语法：
1.“=” 来判断相等，用于字符比较
2.“~” 用正则来匹配（表示区分大小写），“~*” 不区分大小写
3.“-f -d -e” 来判断是否为文件、目录、是否存在

**return指令**

语法：return code;

停止处理并返回指定状态码给客户端。

if ($request_uri ~ *\.sh ){

 return 403

}

***\*set指令\****

set variable value; 

定义一个变量并复制，值可以是文本、变量或者文本变量混合体

***\*rewrite\*******\*指令\****

语法：rewrite regex replacement [flag]{last / break/ redirect 返回临时302/ permant  返回永久302}

last: 停止处理后续的rewrite指令集、 然后对当前重写的uri在rewrite指令集上重新查找

break; 停止处理后续的rewrite指令集 ,并不会重新查找

例如：将符合/images/([a-z]{3})/(.*)\.(png|jpg)正则的重写到/mic路径下，其中$2是（.\*），$3是.(png|jpg)，try_files尝试在硬盘中寻找这个文件，如果有，则直接返回，如果没有，就转到/image404.html路径下。

```shell
location / {
    rewrite '^/images/([a-z]{3})/(.*)\.(png|jpg)$'  /mic?file=$2.$3;
    set $image_file $2;
    set $image_type $3;
}

location /mic {
    root html;
    try_files /$arg_file /image404.html;
}

location /image404.html {
    return 404 "image not found exception";
}
```



如上配置对于： /images/ttt/test.png 会重写到/mic?file=test.png, 于是匹配到 location /mic ; 通过try_files获取存在的文件进行返回。最后由于文件不存在所以直接返回404错误



**rewrite匹配规则**

表面看rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。很多情况下rewrite也会写在location里，它们的执行顺序是：

· 执行server块的rewrite指令

· 执行location匹配

· 执行选定的location中的rewrite指令

如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回500 Internal Server Error错误



### （4）浏览器本地缓存配置及动静分离

语法： expires 60s|m|h|d

操作步骤

· 在html目录下创建一个images文件，在该文件中放一张图片

· 修改index.html, 增加<img src=”图片”/>

· 修改nginx.conf配置。配置两个location实现动静分离，并且在静态文件中增加expires的缓存期限

```shell
location / {
                root html;
                index index.html index.hml;
}
location ~  \.(jpg)$ {
                root html/images;
                index a.jpg;
                expires 5m;
}
```

### （5）Gzip压缩策略

浏览器请求 -> 告诉服务端当前浏览器可以支持压缩类型->服务端会把内容根据浏览器所支持的压缩策略去进行压缩返回

->浏览器拿到数据以后解码；  常见的压缩方式：gzip、deflate 、sdch



Gzip on|off 是否开启gzip压缩

Gzip_buffers 4 16k #设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。4 16k代表以16k为单位，安装原始数据大小以16k为单位的4倍申请内存。

Gzip_comp_level[1-9] 压缩级别， 级别越高，压缩越小，但是会占用CPU资源

Gzip_disable #正则匹配UA 表示什么样的浏览器不进行gzip

Gzip_min_length #开始压缩的最小长度（小于多少就不做压缩）

Gzip_http_version 1.0|1.1 表示开始压缩的http协议版本

Gzip_proxied  （nginx 做前端代理时启用该选项，表示无论后端服务器的headers头返回什么信息，都无条件启用压缩）

Gzip_type text/pliain,application/xml  对那些类型的文件做压缩 （conf/mime.conf）

Gzip_vary on|off  是否传输gzip压缩标识



```shell
gzip on;
gzip_buffers 4 16k;
gzip_comp_level 7;
gzip_min_length 500;
gzip_types text/css application/javascript text/xml;
```



**注意点**

\1. 图片、mp3这样的二进制文件，没必要做压缩处理，因为这类文件压缩比很小，压缩过程会耗费CPU资源

\2. 太小的文件没必要压缩，因为压缩以后会增加一些头信息，反而导致文件变大

\3. Nginx默认只对text/html进行压缩 ，如果要对html之外的内容进行压缩传输，我们需要手动来配置





### （6）Nginx反向代理

Proxy_pass

通过反向代理把请求转发到百度

Proxy_pass 既可以是ip地址，也可以是域名，同时还可以指定端口

Proxy_pass指定的地址携带了URI，看我们前面的配置【/s】，那么这里的URI将会替换请求URI中匹配location参数部分；如上代码将会访问到http://www.baidu.com/s

interface_version

 proxy_set_header：设置请求头参数



### （7）负载均衡

**upstream**是Nginx的HTTP Upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡

### Upstream常用参数介绍

**语法：**server address [parameters]

其中关键字server必选。
address也必选，可以是主机名、域名、ip或unix socket，也可以指定端口号。
parameters是可选参数，可以是如下参数：

​	***\*down\****：表示当前server已停用

​	***\*backup\****：表示当前server是备用服务器，只有其它非backup后端服务器都挂掉了或者很忙才会分配到请求

​	***\*weight\****：表示当前server负载权重，权重越大被请求几率越大。默认是1

***\*max_fails\****和***\*fail_timeout\****一般会关联使用，如果某台server在fail_timeout时间内出现了max_fails次连接失败，那么Nginx会认为其已经挂掉了，从而在fail_timeout时间内不再去请求它，fail_timeout默认是10s，max_fails默认是1，即默认情况是只要发生错误就认为服务器挂掉了，如果将max_fails设置为0，则表示取消这项检查。

```shell
upstream myserver{
          server 192.168.197.100:80;
          server 192.168.197.110:80 weight=5;
     }
```





## 为什么localtion配置不起效？





## 3、Nginx的进程模型

### **（1）Master进程**

充当整个进程组与用户的交互接口，同时对进程进行监护。它不需要处理网络事件，不负责业务的执行，只会通过管理work进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。
主要是用来管理worker进程
1、接收来自外界的信号 （前面提到的 kill -HUP 信号等）

我们要控制nginx，只需要通过kill向master进程发送信号就行了。比如kill -HUP pid，则是告诉nginx，从容地重启nginx，我们一般用这个信号来重启nginx，或重新加载配置，因为是从容地重启，因此服务是不中断的。master进程在接收到HUP信号后是怎么做的呢？首先master进程在接到信号后，会先重新加载配置文件，然后再启动新的worker进程，并向所有老的worker进程发送信号，告诉他们可以光荣退休了。新的worker在启动后，就开始接收新的请求，而老的worker在收到来自master的信号后，就不再接收新的请求，并且在当前进程中的所有未处理完的请求处理完成后，再退出

2、向各个worker进程发送信号

3、监控worker进程的运行状态

4、当worker进程退出后（异常情况下），会自动重新启动新的worker进程

### **（2）Work进程**

主要是完成具体的任务逻辑。它的主要关注点是客户端和后端真实服务器之间的数据可读、可写等I/O交互事件
各个worker进程之间是对等且相互独立的，他们同等竞争来自客户端的请求，一个请求只可能在一个worker进程中处理，worker进程个数一般设置为cpu核数

```shell
worker_processes  1; #需开启的进程数
```

master进程先建好需要listen的socket后，然后再fork出多个woker进程，这样每个work进程都可以去accept这个socket。当一个client连接到来时，所有accept的work进程都会受到通知，但只有一个进程可以accept成功，其它的则会accept失败



## 4、Nginx配置https的请求

· https基于SSL/TLS这个协议；

· 非对称加密、对称加密、 hash算法

· crt的证书->返回给浏览器





### ***\*创建证书\****

· 创建服务器私钥

openssl genrsa -des3 -out server.key 1024

· 创建签名请求的证书（csr）; csr核心内容是一个公钥

openssl req -new -key server.key -out server.csr

· 去除使用私钥是的口令验证

cp server.key server.key.org

openssl rsa -in server.key.org -out server.key

· 标记证书使用私钥和csr

openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

x509是一种证书格式

server.crt就是我们需要的证书



## 5、Nginx+KeepAlive

**keepalived – >VRRP(虚拟路由器冗余协议)**

VRRP全称 Virtual Router Redundancy Protocol，即 虚拟路由冗余协议。可以认为它是实现路由器高可用的容错协议，即将N台提供相同功能的路由器组成一个路由器组(Router Group)，这个组里面有一个master和多个backup，但在外界看来就像一台一样，构成虚拟路由器，拥有一个虚拟IP（vip，也就是路由器所在局域网内其他机器的默认路由），占有这个IP的master实际负责ARP相应和转发IP数据包，组中的其它路由器作为备份的角色处于待命状态。master会发组播消息，当backup在超时时间内收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master，保证路由器的高可用。

**安装keepalived**

```
1.tar -zxvf keepalived-2.1.5.tar.gz
2../configure --prefix=/usr/soft/keepalived --sysconf=/etc
3.缺少依赖：yum install gcc ; yum install openssl-devel；yum -y install libnl libnl-devel  
4.编译安装 make && make install
5.cd到解压的包 /usr/soft/keepalived-2.1.5
7.cp /usr/soft/keepalived-2.1.5/keepalived/etc/init.d/keepalived      /etc/init.d
8.cp /usr/soft/keepalived-2.1.5/keepalived/etc/sysconfig/keepalived /etc/sysconfig/keepalived
9.cp /usr/soft/keepalived/sbin/keepalived /usr/sbin/keepalived
10.mkdir /etc/keepalived
11.cp /usr/soft/keepalived-2.1.5/keepalived/etc/keepalived/keepalived.conf/etc/keepalived/keepalived.conf
12.添加到系统服务
	a)chkconfig --add keepalived
	b)chkconfig keepalived on
	c)Service keepalived start
```



vim  /etc/keepalived/keepalived.conf

```shell
global_defs {
   router_id LVS_DEVEL //标识本节点的字条串,通常为hostname
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER  //如果是备用，则为BACKUP
    interface eno16777736 //使用ifconfig获取
    virtual_router_id 51  
    priority 100  //优先级
    advert_int 1 //MASTER与BACKUP节点间同步检查的时间间隔，单位为秒
    authentication { //验证类型和验证密码
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.197.140  //虚拟ip
    }
}

virtual_server 192.168.197.140 80 { //监听端口
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.197.100 80 { //真实ip及端口
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout 3
            delay_before_retry 3
            nb_get_retry 3
        }
    }
}
```



之后就看可以访问虚拟ip
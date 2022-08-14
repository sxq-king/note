# Nginx是什么
Nginx(engine x)是一个高性能的HTTP和反向代理服务器，特点是占有内存小，并发能力强，事实上nginx的并发能力确实是在同类型的网页服务器中表现较好。

Nginx专为性能优化而开发，性能是其最终要的考量，实现上非常注重效率能经受高负载的考验，有报告表名能支持高达50000个并发连接数

## 反向代理

代理的是服务器端，隐藏了服务器端。


## 负载均衡
单个服务器解决不了的问题，增加服务器的数量，然后将请求分发到各个服务器。将原先请求集中到单个服务器的情况改为到多个服务器上，将负载分发到不同的服务器，这就是我们说的负载均衡。

### 分配策略
 - 轮询(默认)
 > 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除
 - weight
 > weight代表权重，默认为1，权重越高分配的客户端越多
 - ip_hash
 > 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
 - fair(第三方)

 > 按后端服务器的响应时间来分配请求，响应时间短的优先

## 动静分离

为了加快网站解析速度，可以把动态页面和静态页面分别用不同的服务器来解析，加快解析速度，降低原来单个服务器的压力



## Nginx安装
### 依赖安装
在linux下安装nginx，首先要安装gcc-c++编译器。然后再安装nginx依赖的pcre和zlib包。最后安装nginx即可。
```
- 1. 先安装gcc-c++编译器
    yum install gcc-c++
    yum install -y openssl openssl-devel

- 2. 安装pcre依赖
    yum install -y pcre pcre-devel
    可以使用一下命令查看版本号
    pcre-config --version

- 3. 安装zlib包
    yum install -v zlib zlib-devel

```
```
- 1. 将nginx安装在/usr/local/nginx目录下
    mkdir /usr/local/nginx
    cd /usr/local/nginx
- 2. 下载nginx包
    wget https://nginx.org/download/nginx-1.20.1.tar.gz
    建议进入官网下载当前稳定版
- 3. 解压nginx并进入nginx的目录
    tar -zxvf nginx-1.20.1.tar.gz
    cd nginx-1.20.1
- 4. 使用nginx默认配置
    ./configure
- 5. 编译安装
    make && make install
- 6. 进入nginx的sbin目录
    cd /usr/local/nginx/sbin
- 7. 运行nginx，在sbin目录中有可执行文件nginx
    ./nginx
- 8. 查看是否启动成功
    ps -ef | grep nginx
- 9. 在网页上输入自己的ip就可出现欢迎页说明成功
    ifconfig
```

### 开放端口
```
- 查看开放的端口号
    firewall-cmd --list-all
    
- 设置开放的端口号
    firewall-cmd --add-service= http =permanent
    sudo firewall-cmd --add-port=80/tcp -permanent
- 重启防火墙
    firewall-cmd -reload
```
## 常用命令
使用nginx命令时先要进入到nginx的目录
```
/usr/local/nginx/sbin

常用：
查看进程
ps -ef | grep nginx
```

```
- 1.查看nginx版本号
    ./nginx -v
- 2.启动nginx
    ./nginx
- 3.关闭nginx
    ./nginx -s stop
- 4.重新加载nginx
    当修改了nginx的配置文件nginx.conf，不想重启，就用这个
    ./nginx -s reload
```


## 配置文件解析
### 配置文件位置
```
/usr/local/nginx/conf/nginx.conf
```
### 第一块--全局块
```
- 例子
    worker_processes 1;
    这是Nginx服务器并发处理服务的关键配置**，worker_processes的值越大，可以支持的并发处理量也越多**，但是会受到硬件，软件等设备的制约。
```
> **从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令**，主要包括配置运行nginx服务器的用户(组)，允许生成的worker process 数，进程PID存放路径，日志存放路径和类型以及配置文件的引入等。

### 第二块--events块
```
- 例子
    events {
    worker_connections  1024;
    }
    表示work process支持的最大连接数为1024
```
> events块设计的指令主要影响nginx服务器与用户的网络连接，常用的配置包括是否开启对多work process下的网络连接进行序列化，是否允许同时接受欧多个网络连接，选取那种事件驱动模型来处理连接请求，每个work process可以同时支持的最大连接数等。**这部分配置对Nginx的性能影响较大，在实际中要小心配置**。

### 第三块--http块
```
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```
> 这是Nginx服务器配置中最常用的部分，代理缓存和日志等绝大多数功能和第三方模块的配置都在这里。**http块也可以包括http全局块、server块**。

- http全局块
http全局块配置的指令包括文件引入，MIME-TYPE定义，日志自定义，连接超时时间，单链接请求数上线等。


- server块
这块和虚拟主机有关，虚拟主机对用户来说和一台硬件主机是完全一样的

    每个http块可以包括多个server块，而每个server块就相当于一个虚拟主机
  
    每个server块也分为全局server块，以及可以同时包括多个location块
    - 全局server块
    > 常见配置虚拟机的监听配置和本虚拟主机的名称或IP配置 
    - location块
    > 主要作用是基于Nginx服务器接受到的请求字符串，对虚拟主机之外的字符串进行匹配，对特定的请求进行处理。地址定向，数据缓存和应答控制等功能，还有一些第三方模块的配置。

## 配置案例
### 实现打开浏览器输入一个网址，显示的是tomcat的主页面

- 1 在linux中安装jdk环境，并安装tomcat


```
- 安装jdk
yum search java|grep jdk    //查看jdk版本
yum install -y java-1.8.0-openjdk       //版本自己选
- 安装tomcat
    先创建目录 mkdir /usr/local/tomcat
    进入目录    cd /usr/local/tomcat
    下载tomcat wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.54/bin/apache-tomcat-9.0.54.tar.gz
    解压tomcat tar -zxvf apache-tomcat-9.0.54.tar.gz
    进入tomcat的bin目录 cd /usr/local/tomcat/apache-tomcat-9.0.54/bin
    运行startup.sh文件  ./startup.sh
    查看是否运行成功日志 先进入logs文件夹 cd ..,cd logs
    查看日志 tail -f catalina.out
```
如果开启防火墙的话还要开放端口号
```
查看已经开放的端口号
firewall-cmd --list-all
开放8080端口
firewall-cmd --add-port=8080/tcp --permanent
重启防火墙
firewall-cmd --reload
```
> 这里分为主机（windows），服务器（虚拟机），nginx
- 一 修改本机hosts文件，加入假域名

```
在hosts文件中加入
192.168.255.129 www.baidu.com
```

- 二 在nginx进行请求转发(反向代理配置 )

```
修改nginx.conf文件，将server块的server_name的内容改为虚拟机192.168.255.129


在location /{}块中添加 proxy_pass http://127.0.0.1:8080(将原本的80端口切换到8080)
```








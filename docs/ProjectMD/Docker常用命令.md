## 帮助启动类命令
- 启动： systemctl start docker
- 停止:  systemctl stop docker
- 重启:  systemctl restart docker
- 查看Docker状态: systemctl status docker
- 开机启动: systemctl enable docker
- 查看docker概要信息: docker info
- 查看docker总体帮助文档: docker --help
- 查看docker命令帮助文档: docker 具体命令 --help

## 镜像命令
- docker images
> 作用 列出本主机上的镜像 
可选参数: 
-a:列出本地所有镜像(包括历史镜像)；
-q：只显示镜像ID

- docker search 某个xxx镜像的名字
> 作用 在仓库里找xxx镜像
可选参数:
--limit N:只列出N个镜像，默认
25个

- docker pull 某个xxx镜像的名字:TAG
> 作用 下载镜像
TAG就是版本
没有TAG就默认是最新版

- docker rmi 某个xxx镜像的名字ID
> 作用 删除镜像
docker rmi -f 镜像ID;删除单个
docker rmi -f 镜像名1:TAG 镜像名2:TAG ;删除多个
docker rmi -f $(docker images -qa);删除全部

了解一个东西，虚悬镜像,就是仓库名和标签都是<none>的镜像。


## 容器命令
新建+启动容器
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

 OPTIONS说明（常用）：有些是一个减号，有些是两个减号
 
--name="容器新名字"       为容器指定一个名称；
-d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；
 
-i：以交互模式运行容器，通常与 -t 同时使用；
-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；
也即启动交互式容器(前台有伪终端，等待交互)；
 
-P: 随机端口映射，大写P
-p: 指定端口映射，小写p

docker run -it centos /bin/bash 

参数说明：
-i: 交互式操作。
-t: 终端。
centos : centos 镜像。
/bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
要退出终端，直接输入 exit:

```
列出当前所有正在运行的容器
```
docker ps [OPTIONS]

OPTIONS说明（常用）：
-a :列出当前所有正在运行的容器+历史上运行过的
-l :显示最近创建的容器。
-n：显示最近n个创建的容器。
-q :静默模式，只显示容器编号。

```
退出容器
```
exit
    run进入容器后,exit退出。容器停止
ctrl+p+q
    run进入容器后，ctrl+p+q直接退出。容器不停止
```

启动已经停止运行的容器
```
docker start 容器ID或者容器名
```
重启容器
```
docker restart 容器ID或者容器名
```
停止容器
```
docker stop 容器ID或容器名
```
强制停止容器
```
docker kill 容器名或者ID
```
删除已停止的容器
```
docker rm 容器ID

一次删除多个容器实例
docker rm -f $(docker ps -a -q);
docker ps -a -q | xargs docker rm;
```
启动守护式容器(后台服务器)
```
docker run -d 容器名

启动redis为例
docker run -it redis:6.0.8//前台交互式启动
docker run -d redis:6.0.8//后台守护式启动
```

查看容器日志
```
docker logs 容器ID
```
查看容器内运行的进程
```
docker top 容器ID
```
查看容器内步细节
```
docker inspect 容器ID
```

进入正在运行的容器并以命令行交互
```
docker ps 找到要进入的容器ID
docker exec -it 容器ID
 /bin/bash或其他命令行
 
 docker attach 容器ID /bin/bash
 ：这个不会创建新进程，退出后容器也会销毁，不常用。
```
 从容器拷贝文件到主机上
 ```
 容器---> 主机
 docker cp 容器ID:容器内路径 目的主机路径
 ```
 导入和导出容器
 ```
 export导出容器内的内容留作为tar归档文件[对应import命令]
 import从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]
 
 docker export 容器ID > 文件名.tar
 
 cat 文件名.tar | docker import - 镜像用户/镜像名：镜像版本号
 


 ```

## 安装相关

### redis

```java
docker run -p 6379:6379 \
    --name my_redis \
    --privileged=true \
    -v /app/redis/redis.conf:/etc/redis/redis.conf \
    -v /app/redis/data:/data \
    -d redis redis-server /etc/redis/redis.conf

docker run --name my_redis \
    -p 6379:6379 \
    -v /app/redis/data:/data \
    -v /app/redis/redis.conf:/etc/redis/redis.conf \
    -d redis redis-server /etc/redis/redis.conf

```

### mysql

```java
docker run -d -p 3306:3306 \
    --privileged=true \
    -v /app/mysql/log:/var/log/mysql \
    -v /app/mysql/data:/var/lib/mysql \
    -v /app/mysql/conf:/etc/mysql/conf.d \
    -e MYSQL_ROOT_PASSWORD=密码 \
    --name mysql mysql:5.7
```

### minio

```java
docker run --name minio \
-p 9000:9000 \  
-p 9090:9090 \
-d --restart=always \
-e "MINIO_ROOT_USER=admin" \
-e "MINIO_ROOT_PASSWORD=996957240.sxq" \
-v /app/minio/data:/data \
-v /app/minio/config:/root/.minio \
minio/minio server /data \
--console-address '0.0.0.0:9090'
```

### mongodb

```java
#docker以指定配置文件启动 (-v 宿主机目录:容器内目录)
docker run -d -p 27017:27017 -v /app/mongodb/data/db:/data/db -v /app/mongodb/conf:/data/conf -v /app/mongodb/log:/data/log --name mongo mongo:4.4
```

配置文件mongodb/conf
>  ​	#端口
> ​	port=27017
> ​	#数据库文件存放目录
> ​	dbpath=/data/mongo/data
> ​	#日志文件存放路径
> ​	logpath=/data/mongo/log
> ​	#使用追加方式写日志
> ​	logappend=true
> ​	#以守护线程的方式运行，创建服务器进程
> ​	fork=true
> ​	#最大同时连接数
> ​	maxConns=100
> ​	#不启用验证
> ​	#noauth=true
> ​	#每次写入会记录一条操作日志
> ​	journal=true
> ​	#存储引擎有mmapv1、wiredTiger、mongorocks
> ​	storageEngine=wiredTiger
> ​	#访问IP
> ​	bind_ip=0.0.0.0
> ​	#用户验证
> ​	#auth=true

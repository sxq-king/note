## 1.2.1 NoSQL概述
NoSQL(NOSQL = Not Only SQL),不仅仅是sql。泛指非关系型数据库。NOSQL不依赖业务逻辑方式存储，而是用简单的key-value模式存储，因此大大提高了数据库的扩展能力。
- 不遵循SQL标准
- 不支持ACID
- 远超SQL的性能

### 1.2.2 NoSQL适用场景
- 数据库高并发的读写
- 对数据高可扩展性的
- 海量数据的读写

### 1.2.3 NoSQL不适用场景
- 需要事务支持
- 基于sql的结构化查询存储，处理复杂的关系，需要即席查询
- (用不着sql的和用了sql也不行的情况，考虑用Nosql)

### 1.2.4 Memcache
- 很早出现的Nosql数据库
- 数据都在内存中，一般不持久化
- 支持简单的key-value模式，支持类型单一
- 一般是最为缓存数据库辅助持久化的数据库

### 1.2.5 Redis
- 几乎覆盖了Memcache的绝大部分功能
- 数据都在内存中，支持持久化，主要用作备份恢复
- 除了支持简单的key-value模式，还支持多种数据结构的存储，比如list，sethash，zset等。
- 一般是作为缓存数据库辅助持久化的数据库


### 1.2.6 MongoDB

- 高性能、开源、模式自由的文档型数据库
- 数据都在内存中，如果内存不足，把不常用的数据保存到硬盘
- 虽然是key-value模式，但是对value提供了丰富的查询功能
- 支持二进制数据机大型对象
- 可以支持数据的特点替代RDBMS，成为独立的数据库。或者配合RDBMS，存储特定的数据。
## 2.1 Redis概述

- Redis是一个开源的key-value存储系统
- 和Memcache类似，他支持存储的value类型相对更多，包括string，list，set，zset(sorted set--有序集合)和hash。
- 这些数据类型都支持push/pop、add/remove及取交集并集及更丰富的操作，而且这些操作都是原子性的
- 在此基础上，Redis支持各种不同方式的排序
- 与memecache一样，为了保证效率，数据都是缓存在内存中。
- 区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。
- 并且再此基础上实现了master-slave(主从)同步


## 2.2 Redis安装
> Windows上有手就行吧。
```
查看redis是否启动
ps -ef|grep redis

启动后台redis服务
./redis-server /etc/redis.conf

查看6379端口号是否被占用
lsof -i :6379

查看防火墙是否关闭
systemctl status firewalld.service

关闭防火墙
systemctl stop firewalld.service
```
## 3常用key操作


keys * 查看当前库所有key

exists key 判断某个key是否存在 存在返回1否则返回0

type key 查看你的key是什么类型

del key 删除指定的key数据

unlink key 根据value选择非阻塞删除
> 仅将keys从keyspace原数据中删除，真正的删除会在后续异步操作

expire key 10 10秒钟：为给定的key设置过期时间

ttl key 查看还有多少秒过期 —1表示永不过期，—2表示已过期

select 命令切换数据库

dbsize 查看当前数据库的key数量

flushdb 清空当前库

flushall 通杀所有库

# 五大常用数据类型
## String
String是redis最基本的数据类型，你可以理解为Memcached一摸一样的类型，一个key对应一个value。

String类型是二进制安全的。意味着Redis可以包含任何数据。比如jpg图片或者序列化的对象。

String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M

---

### 常用命令
set key value

get key 得到key的值，查询对应键值

append key value 将给定的value追加到原值的末尾

strlen key 获得值的长度

setnx key value 只有key不存在时设置key的值

incr key 将key中存储的数字值增1，只能对数字值操作，如果为空，新值增1

decr key 将key中存储的数字值减1，只能对数字值操作，如果为空，新增值为-1

incrby/decrby key 步长 将key中存储的数字值增减。自定义步长

incr key操作具有原子性，不会被线程调度机制打断的操作

mset key1 value1 key2 value2 同时设置一个或者多个key-value对

mget key1 key2 
同时获取一个或者多个value值

msetnx key1 value1 key2 value2
同时设置一个或者多个key-value对，当且仅当所有给定key都不存在




## List
> 单键多值
Redis列表是简单的字符串列表，按照插入顺序排序，你可以添加一个元素的列表的头部或尾部
底层实际是一个双向链表，对两段的操作性能很高，通过索引下表的操作中间节点性能会较差
### 常用命令

lpush/rpush key value1 value2...(从左边/从右边插入一个或者多个值)

lpop/rpop key 从左边或者右边吐出一个值。        值在键在，值光键亡

rpoplpush key1 key2 从key1列表吐出一个值，插到key2列表左边

lrange key start stop 按照索引下表获得元素(从左到右)

lindex key 索引值 按照索引获取对应索引的值

llen key 获取列表的长度

linsert key before value newvalue 在value的后面插入newvalue值

lrem key n value 从左边删除n个value (从左到右)

lset key index value 将列表key下标为index 的值替换成value


### 数据结构

List的数据结构使用的快速列表quickList

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也就是压缩列表

他将所有的元素都紧挨着一起存储，分配的是一块连续的内存

当数据量较多的时候才会改成quicklist



## Set
> Redis的Set是string类型的无序集合。他的底层其实是一个value为null的hash表，所以添加，删除，查找的复杂度都是O(1)
### 常用命令
sadd key value1 value2...
> 将一个或者多个member元素加入到集合key中，已经存在的member元素将被忽略

smembers key 
> 取出该集合的所有值

sismember key value 
> 判断集合 key 是否为含有该value值，有1，没有0

scard key
> 返回该集合的元素个数

srem key value1 value2 .. 
> 删除集合中的某个元素

spop key 
> 随机从该集合中吐出一个值

srandmember key n
> 随机从该集合中去除n个值。不会从集合中删除

smove source destination value 
> 把集合中的一个值从一个集合移动到另一个集合

sinter key1 key2 
> 返回两个集的交集元素

sunion key1 key2 
> 返回量个集合的并集元素

sdiff key1 key2 
> 返回两个集合的差集元素(key1中的，不包含key2中的)

### 数据结构

Set数据结构是dict字典，字典是用哈希表实现

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都只想同一个对象。Redis的set结构也是一样，他的内部也是使用hash结构，所有的value都指向同一个内部值。

## Hash 

> Redis hash 是一个键值对集合
Redis hash 是一个string类型的field和value的映射表，hash特别适用于存储对象。
类似Java中的Map<String,Object>


### 常用命令

hset key field value 
> 给key集合中的 filed键赋值 value

hget key1 field
> 从key1集合field去除value

hmset key1 field1 value1 field value2。。。
> 批量设置hash的值

hexists key1 field
> 查看哈希表key中，给定域field知否存在

hkeys key 
> 列出该hash 集合中的所有field

hvals  key 
> 列出该hash集合的所有的value

hincrby key field increment 
> 为哈希表key中的域field的值加上增量1 -1

hsetnx key field value 
> 将哈希表key中的field的只值设置为value，当且仅当域field不存在

### 数据结构

Hash类型对应的数据结构是两种，ziplist(压缩列表),hashtable(哈希表)。当field-value长度较短且个数较少是，使用ziplist，否则使用hashtable。

## Zset
Redis**有序**集合zset与普通集合set非常相似，是一个**没有重复的元素**字符串集合

不同之处是有序集合的每个成员都关联了一个评分(score)，这个评分(score)被用来从最低分到最高分的方式排序集合中的成员。**集合的成员是唯一的，但是评分是可以重复的。**

因为元素是有序的，所以你也可以很快的根据评分(score)或者次序(position)俩获取一个范围的元素

访问有序集合中间元素的也是非常快的，因此你能够使用有序集合作为一个乜有重复成员的智能列表。 

### 常用命令
zadd key score1 value1 score2 value2
> 将一个或多个member元素及其score值加入到有序集key当中

zrange key start stop [WITHSCORES]
> 返回有序集key中，下表在start stop之间的元素  带WITHSCORES，可以让分数一起和值返回到结果集

zrangebyscore key minmax [withscores][limit offset count]
> 返回有序集合key中，所有score值介于min和max之间（包括min或max）的成员。有序集成员按score值递增(从小到大)次序排列

zrevrangebyscore key maxmin [withscores][limit offset count]
> 同上，改为从大到小排列

zincrby key increment value
> 为元素的score加上增量

zrem key value 
> 删除该集合下，指定值的元素

zcount key min max 
> 统计该集合下，分数区间内的元素个数

zrank key value 
> 返回该值 在集合中的排名，从0开始

### 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面他等价于java的数据结构Map<String,Double>，可以给每一个缘故value赋予一个权重score，另一方面他有类似与TreeSet，内部的元素会按照权重score进行排序，可以的到每个元素的名次，还可以通过score的范围来获取元素的列表


zset底层使用了两个数据结构

(1)hash,hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的socre值

(2)跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表

## 配置文件
TODO

## 发布订阅
### 什么是发布和订阅

Redis发布订阅(pub/sub)是一种消息通讯模式：发送者(pub)发送消息，订阅者(sub)接受消息

Redis客户端可以订阅任意数量的频道

### 发布订阅命令行实现
1. 打开一个客户端订阅channel1
```
subscribe channel1
```
2. 打开另一个客户端给channel1发布消息hello
```
publish channel1 hello
```
3. 打开第一个客户端可以看到发送的消息

注意：发布的消息没有持久化，如果在订阅的客户端收不到hello，只能收到订阅后发布的消息

## 新数据类型
### 1、Bitmaps

Redis提供了Bitmaps这个"数据类型"可以实现对位的操作

(1) Bitmaps本身不是一种数据类型，实际上它就是字符串，但是他可以对字符串的位进行操作

(2)Bitmaps单独提供了一套命令，所以在Redis中使用Bitmaps和使用字符串的方法不太相同。可以把Bitmaps想象成一个以位为单位的数组，数据的每个单元只能存储0和1，数组的下表在Bitmaps中叫偏移量

#### 命令
- setbit key offset value
> 设置Bitmaps中某个便宜量的值(0或1)

offset偏移量从0开始

- getbit
- bitcount key [start end]
> 统计字符串从start字节到end字节比特值为1的数量
- bitop and(or/not/xor) destkey [key...]
> bitop是一个符合操作，他可以做多个Bitmaps的and(交集)、or(并集)、not(非)、xor(异或)操作并将结果保存在destkey中

### 2、HyperLogLog
常用于
- 统计网站的pv页面访问量，可以使用Redis的incr，incrby轻松实现
- 快速计算基数(不重复元素)
- 
Redis HyperLogLog是用来做基数统计的算法，HyperLogLog的优点是，在数据元素的数量或者体积非常非常大时，计算基数所需要的空间是固定的，并且是很小的。

#### 命令
1、pfadd key element [element...]
> 添加指定元素到HyperLogLog

2、pfcount key [key]
> 计算HLL的近似基数，可以计算多个HLL，比如用HLL存储每天的UV，计算一周的UV可以使用7天的UV合并计算即可

3、pfmerge destkey sourcekey [sourcekey...]
> 讲一个或多个HLL合并狗的结果存储在另一个HLL中，比如每月活跃用户可以使用每天的活跃用户量来合并计算可得

### 3、Geospatial
简介
> Redis3.2中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上街市经纬度。reids基于该类型，提供了经纬度设置，查询，范围查询距离查询，经纬度Hash等常见操作。

**有效的经度在-180到180度之间**

**有效的纬度在-85.05112878度到95.05112878度**

#### 命令
1、geoadd key longitude latitude member [longitude latitude member...]
> 添加地理位置 (经度，纬度，名称)

2、geopos key member[memeber...]
> 获得指定地区的坐标值

3、geodist key member1 member2[m|km|mi|ft]
> 获取两位置之间的直线距离  mi表示单位为英里    ft表示单位为英尺。默认使用米作为单位

4、georadius key longitude latitude radius m|km|ft|mi 
> 以给定的经纬度为中心，找出某一半径内的元素

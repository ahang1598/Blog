

[查询手册](https://redis.io/commands)

## NoSQL

### 概述

NoSQL（Not-Only SQL）：泛指非关系型的数据库，作为关系型数据库的补充。 

MySQL 支持 ACID 特性，保证可靠性和持久性，读取性能不高，因此需要缓存的来减缓数据库的访问压力。

作用：应对基于海量用户和海量数据前提下的数据处理问题

特征：

* 可扩容，可伸缩，SQL 数据关系过于复杂，Nosql 不存关系，只存数据
* 大数据量下高性能，数据不存取在磁盘 IO，存取在内存
* 灵活的数据模型，设计了一些数据存储格式，能保证效率上的提高
* 高可用，集群

常见的 Nosql：Redis、memcache、HBase、MongoDB

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/电商场景解决方案.png)



参考视频：https://www.bilibili.com/video/BV1CJ411m7Gc

参考视频：https://www.bilibili.com/video/BV1Rv41177Af



***



### Redis

Redis (REmote DIctionary Server) ：用 C 语言开发的一个开源的高性能键值对（key-value）数据库。

特征：

* 数据间没有必然的关联关系，**不存关系，只存数据**
* 数据**存储在内存**，存取速度快，解决了磁盘 IO 速度慢的问题
* 内部采用**单线程**机制进行工作
* 高性能，官方测试数据，50个并发执行100000 个请求,读的速度是110000 次/s,写的速度是81000次/s
* 多数据类型支持
  * 字符串类型：string（String）
  * 列表类型：list（LinkedList）
  * 散列类型：hash（HashMap）
  * 集合类型：set（HashSet）
  * 有序集合类型：zset/sorted_set（TreeSet）
* 支持持久化，可以进行数据灾难恢复

应用：

* 为热点数据加速查询（主要场景），如热点商品、热点新闻、热点资讯、推广类等高访问量信息等

* 即时信息查询，如排行榜、网站访问统计、公交到站信息、在线人数（聊天室、网站）、设备信号等

* 时效性信息控制，如验证码控制、投票控制等

* 分布式数据共享，如分布式集群架构中的 session 分离
* 消息队列



***



### 安装启动

#### CentOS

1. 下载 Redis

   下载安装包：

   ```sh
   wget https://download.redis.io/releases/redis-6.2.6.tar.gz
   ```

   解压安装包：

   ```sh
   tar xzf redis-6.2.6.tar.gz
   ```

   编译（在解压的目录中执行）：

   ```sh
   cd redis-6.2.6
   make
   ```

   安装（在解压的目录中执行）：

   ```sh
   make install
   ```

2. 启动服务端` ./src/redis-server ./redis.conf`

   启动客户端`src/redis-cli`

   测试是否成功`ping` 回应`PONG`

   关闭连接`quit`

   切换数据库`select index`

   

3. Redis相关

   redis-server，服务器启动命令 客户端启动命令

   redis-cli，redis核心配置文件

   redis.conf，RDB文件检查工具（快照持久化文件）

   redis-check-dump，AOF文件修复工具

   redis-check-aof



***



#### Ubuntu

安装：

* Redis 5.0 被包含在默认的 Ubuntu 20.04 软件源中

  ```sh
  sudo apt update
  sudo apt install redis-server
  ```

* 检查Redis状态

  ```sh
  sudo systemctl status redis-server
  ```

启动：

* 启动服务器——参数启动

  ```sh
  redis-server [--port port]
  #redis-server --port 6379
  ```

* 启动服务器——配置文件启动

  ```sh
  redis-server config_file_name
  #redis-server /etc/redis/conf/redis-6397.conf
  ```

* 启动客户端：

  ```sh
  redis-cli [-h host] [-p port]
  #redis-cli -h 192.168.2.185 -p 6397
  ```

  注意：服务器启动指定端口使用的是--port，客户端启动指定端口使用的是-p



***



### 基本配置

#### 系统目录

1. 创建文件结构

   创建配置文件存储目录

   ```sh
   mkdir conf
   ```

   创建服务器文件存储目录（包含日志、数据、临时配置文件等）

   ```sh
   mkdir data
   ```

2. 创建配置文件副本放入 conf 目录，Ubuntu系统配置文件 redis.conf 在目录 `/etc/redis` 中

   ```sh
   cat redis.conf | grep -v "#" | grep -v "^$" -> /conf/redis-6379.conf
   ```

   去除配置文件的注释和空格，输出到新的文件，命令方式采用 redis-port.conf



***



#### 服务器

* 设置服务器以守护进程的方式运行，关闭后服务器控制台中将打印服务器运行信息（同日志内容相同）：

  ```sh
  daemonize yes|no
  ```

* 绑定主机地址，绑定本地IP地址，否则SSH无法访问：

  ```sh
  bind ip
  ```

* 设置服务器端口：

  ```sh
  port port
  ```

* 设置服务器文件保存地址：

  ```sh
  dir path
  ```

* 设置数据库的数量：

  ```sh
  databases 16
  ```

* 多服务器快捷配置：

  导入并加载指定配置文件信息，用于快速创建 redis 公共配置较多的 redis 实例配置文件，便于维护

  ```sh
  include /path/conf_name.conf
  ```

  

***



#### 客户端

* 服务器允许客户端连接最大数量，默认0，表示无限制，当客户端连接到达上限后，Redis会拒绝新的连接：

  ```sh
  maxclients count
  ```

* 客户端闲置等待最大时长，达到最大值后关闭对应连接，如需关闭该功能，设置为 0：

  ```sh
  timeout seconds
  ```



***



#### 日志配置

* 设置服务器以指定日志记录级别：

  ```sh
  loglevel debug|verbose|notice|warning
  ```

* 日志记录文件名

  ```sh
  logfile filename
  ```

注意：日志级别开发期设置为 verbose 即可，生产环境中配置为 notice，简化日志输出量，降低写日志IO的频度



**配置文件：**

```sh
bind 192.168.2.185
port 6379
#timeout 0
daemonize no
logfile /etc/redis/data/redis-6379.log
dir /etc/redis/data
dbfilename "dump-6379.rdb"
```





***



## 体系结构

### 存储对象

Redis 使用对象来表示数据库中的键和值，当在 Redis 数据库中新创建一个键值对时至少会创建两个对象，一个对象用作键值对的键（键对象），另一个对象用作键值对的值（值对象）

Redis 中对象由一个 redisObject 结构表示，该结构中和保存数据有关的三个属性分别是 type、 encoding、ptr：

```c
typedef struct redisObiect{
	//类型
	unsigned type:4;
	//编码
	unsigned encoding:4;
	//指向底层数据结构的指针
	void *ptr;
}
```

Redis 中主要数据结构有：简单动态字符串（SDS）、双端链表、字典、压缩列表、整数集合、跳跃表

Redis 并没有直接使用数据结构来实现键值对数据库，而是基于这些数据结构创建了一个对象系统，包含字符串对象、列表对象、哈希对象、集合对象和有序集合对象这五种类型的对象，而每种对象又通过不同的编码映射到不同的底层数据结构

Redis 自身是一个 Map，其中所有的数据都是采用 key : value 的形式存储，**键对象都是字符串对象**，而值对象有五种基本类型和三种高级类型对象

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-对象模型.png)





***



### 线程模型

Redis 基于 Reactor 模式开发了网络事件处理器，这个处理器被称为文件事件处理器（file event handler），这个文件事件处理器是单线程的，所以 Redis 叫做单线程的模型

文件事件处理器以单线程方式运行，但是使用 I/O 多路复用程序来监听多个套接字，既实现了高性能的网络通信模型，又很好地与 Redis 服务器中其他同样以单线程方式运行的模块进行对接，保持了 Redis 单线程设计的简单性

工作原理：

* 文件事件处理器使用 I/O 多路复用（multiplexing）程序来同时监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器

* 当被监听的套接字准备好执行连接应答 (accept)、读取 (read)、写入 (write)、关闭 (close) 等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器会将处理请求放入**单线程的执行队列**中，等待调用套接字关联好的事件处理器来处理事件

**Redis 单线程也能高效的原因**：

* 纯内存操作
* 核心是基于非阻塞的 IO 多路复用机制，单线程可以高效处理多个请求
* 底层使用 C 语言实现，C 语言实现的程序距离操作系统更近，执行速度相对会更快
* 单线程同时也**避免了多线程的上下文频繁切换问题**，预防了多线程可能产生的竞争问题



****



### 多线程

Redis6.0 引入多线程主要是为了提高网络 IO 读写性能，因为这是 Redis 的一个性能瓶颈（Redis 的瓶颈主要受限于内存和网络），多线程只是用来**处理网络数据的读写和协议解析**， 执行命令仍然是单线程顺序执行，因此不需要担心线程安全问题。

Redis6.0 的多线程默认是禁用的，只使用主线程。如需开启需要修改 redis 配置文件 `redis.conf` ：

```sh
io-threads-do-reads yesCopy to clipboardErrorCopied
```

开启多线程后，还需要设置线程数，否则是不生效的，同样需要修改 redis 配置文件 :

```sh
io-threads 4 #官网建议4核的机器建议设置为2或3个线程，8核的建议设置为6个线程
```

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-多线程.png )

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-多线程.png)

参考文章：https://mp.weixin.qq.com/s/dqmiR0ECf4lB6Y2OyK-dyA



***





## 基本指令

### 操作指令

读写数据：

* 设置 key，value 数据：

  ```sh
  set key value
  #set name seazean
  ```

* 根据 key 查询对应的 value，如果**不存在，返回空（nil）**：

  ```sh
  get key
  #get name
  ```

帮助信息：

* 获取命令帮助文档

  ```sh
  help [command]
  #help set
  ```

* 获取组中所有命令信息名称

  ```sh
  help [@group-name]
  #help @string
  ```

退出服务

* 退出客户端：

  ```sh
  quit
  exit
  ```

* 退出客户端服务器快捷键：

  ```sh
  Ctrl+C
  ```

  

***



### key 指令

key 是一个字符串，通过 key 获取 redis 中保存的数据

* 基本操作

  ```sh
  del key						#删除指定key
  unlink key   				#非阻塞删除key，真正的删除会在后续异步操作
  exists key					#获取key是否存在
  type key					#获取key的类型
  sort key [ASC/DESC]			#对key中数据排序，默认对数字排序，并不更改集合中的数据位置，只是查询
  sort key alpha				#对key中字母排序
  rename key newkey			#改名
  renamenx key newkey			#改名
  ```

* 时效性控制

  ```sh
  expire key seconds			#为指定key设置有效期，单位为秒
  pexpire key milliseconds	#为指定key设置有效期，单位为毫秒
  expireat key timestamp		#为指定key设置有效期，单位为时间戳
  pexpireat key mil-timestamp	#为指定key设置有效期，单位为毫秒时间戳
  
  ttl key						#获取key的有效时间，每次获取会自动变化(减小)，类似于倒计时，
  							#-1代表永久性，-2代表不存在/失效
  pttl key					#获取key的有效时间，单位是毫秒，每次获取会自动变化(减小)
  persist key					#切换key从时效性转换为永久性
  ```

* 查询模式

  ```sh
  keys pattern				#查询key
  ```

  查询模式规则：*匹配任意数量的任意符号；?配合一个任意符号；[]匹配一个指定符号

  ```sh
  keys *						#查询所有key
  keys aa*					#查询所有以aa开头
  keys *bb					#查询所有以bb结尾
  keys ??cc					#查询所有前面两个字符任意，后面以cc结尾 
  keys user:?					#查询所有以user:开头，最后一个字符任意
  keys u[st]er:1				#查询所有以u开头，以er:1结尾，中间包含一个字母，s或t
  ```

  



***



### DB 指令

Redis 在使用过程中，随着操作数据量的增加，会出现大量的数据以及对应的 key，数据不区分种类、类别混在一起，容易引起重复或者冲突，所以 Redis 为每个服务提供 16 个数据库，编码 0-15，每个数据库之间相互独立，**共用 **Redis 内存，不区分大小

* 基本操作

  ```sh
  select index	#切换数据库，index从0-15取值
  ping			#测试数据库是否连接正常，返回PONG
  echo message	#控制台输出信息
  ```

* 扩展操作

  ```sh
  move key db		#数据移动到指定数据库，db是数据库编号
  dbsize			#获取当前数据库的数据总量，即key的个数
  flushdb			#清除当前数据库的所有数据
  flushall		#清除所有数据
  ```




****



### 通信指令

Redis 发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

Redis 客户端可以订阅任意数量的频道

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-发布订阅.png)

操作命令：

1. 打开一个客户端订阅 channel1：`SUBSCRIBE channel1`
2. 打开另一个客户端，给 channel1发布消息 hello：`publish channel1 hello`
3. 第一个客户端可以看到发送的消息

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-发布订阅指令操作.png )

注意：发布的消息没有持久化，所以订阅的客户端只能收到订阅后发布的消息



****



### ACL 指令

Redis ACL 是 Access Control List（访问控制列表）的缩写，该功能允许根据可以执行的命令和可以访问的键来限制某些连接

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-ACL指令.png)

* acl cat：查看添加权限指令类别
* acl whoami：查看当前用户

* acl setuser username on >password ~cached:* +get：设置有用户名、密码、ACL 权限（只能 get）





***



## 数据类型

### string

#### 简介

存储的数据：单个数据，最简单的数据存储类型，也是最常用的数据存储类型，实质上是存一个字符串，string 类型是二进制安全的，意味着 Redis 的 string 可以包含任何数据，比如图片或者序列化的对象

存储数据的格式：一个存储空间保存一个数据，每一个空间中只能保存一个字符串信息

存储内容：通常使用字符串，如果字符串以整数的形式展示，可以作为数字操作使用

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-string结构图.png )

Redis 所有操作都是**原子性**的，采用**单线程**机制，命令是单个顺序执行，无需考虑并发带来影响，原子性就是有一个失败则都失败



***



#### 操作

指令操作：

* 数据操作：

  ```sh
  set key value			#添加/修改数据添加/修改数据
  del key					#删除数据
  setnx key value			#判定性添加数据，键值为空则设添加
  mset k1 v1 k2 v2...		#添加/修改多个数据，m：Multiple
  append key value		#追加信息到原始信息后部（如果原始信息存在就追加，否则新建）
  ```

* 查询操作

  ```sh
  get key					#获取数据
  mget key1 key2...		#获取多个数据
  strlen key				#获取数据字符个数（字符串长度）
  ```

* 设置数值数据增加/减少指定范围的值

  ```sh
  incr key					#key++
  incrby key increment		#key+increment
  incrbyfloat key increment	#对小数操作
  decr key					#key--
  decrby key increment		#key-increment
  ```

* 设置数据具有指定的生命周期

  ```sh
  setex key seconds value  		#设置key-value存活时间，seconds单位是秒
  psetex key milliseconds value	#毫秒级
  ```

```sh
127.0.0.1:6379> set string haha  
OK
127.0.0.1:6379> get string
"haha"
127.0.0.1:6379> del string
(integer) 1
127.0.0.1:6379> get string
(nil)
127.0.0.1:6379> set cnt 1
OK
127.0.0.1:6379> incr cnt 
(integer) 2
127.0.0.1:6379> get cnt
"2"
127.0.0.1:6379> incrby cnt 10
(integer) 12
127.0.0.1:6379> get cnt
"12"
127.0.0.1:6379> decr cnt
(integer) 11
127.0.0.1:6379> decrby cnt 5
(integer) 6
127.0.0.1:6379> get cnt
"6"
```



注意事项：

1. 数据操作不成功的反馈与数据正常操作之间的差异

   * 表示运行结果是否成功
     (integer) 0  → false                 失败

     (integer) 1  → true                  成功

   * 表示运行结果值
     (integer) 3  → 3                        3个

     (integer) 1  → 1                         1个

2. 数据未获取到时，对应的数据为（nil），等同于null

3. **数据最大存储量**：512MB

4. string 在 redis 内部存储默认就是一个字符串，当遇到增减类操作 incr，decr 时**会转成数值型**进行计算

5. 按数值进行操作的数据，如果原始数据不能转成数值，或超越了redis 数值上限范围，将报错
   9223372036854775807（java 中 Long 型数据最大值，Long.MAX_VALUE）

6. redis 可用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性


单数据和多数据的选择：

* 单数据执行 3 条指令的过程：3 次发送 + 3 次处理 + 3次返回
* 多数据执行 1 条指令的过程：1 次发送 + 3 次处理 + 1次返回（发送和返回的事件略高于单数据）

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/string单数据与多数据操作.png )





***



#### 应用

主页高频访问信息显示控制，例如新浪微博大 V 主页显示粉丝数与微博数量

* 在 Redis 中为大 V 用户设定用户信息，以用户主键和属性值作为 key，后台设定定时刷新策略

  ```sh
  set user:id:3506728370:fans 12210947
  set user:id:3506728370:blogs 6164
  set user:id:3506728370:focuses 83
  ```

* 使用 JSON 格式保存数据

  ```sh
  set user:id:3506728370 '{"fans":12210947,"blogs":6164,"focuses":83}'
  ```

* key的设置约定：表名 : 主键名 : 主键值 : 字段名

  | 表名  | 主键名 | 主键值    | 字段名 |
  | ----- | ------ | --------- | ------ |
  | order | id     | 29437595  | name   |
  | equip | id     | 390472345 | type   |
  | news  | id     | 202004150 | title  |



***



#### 实现

Redis 字符串对象底层的数据结构实现主要是 int 和简单动态字符串 SDS，是可以修改的字符串，内部结构实现上类似于 Java 的 ArrayList，采用**预分配冗余空间**的方式来减少内存的频繁分配

```c
struct sdshdr{
     //记录buf数组中已使用字节的数量
     //等于 SDS 保存字符串的长度
     int len;
     //记录 buf 数组中未使用字节的数量
     int free;
     //字节数组，用于保存字符串
     char buf[];
}
```

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-string数据结构.png)

内部为当前字符串实际分配的空间 capacity 一般要高于实际字符串长度 len，当字符串长度小于 1M 时，扩容都是双倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间，需要注意的是字符串最大长度为 512M



详解请参考文章：https://www.cnblogs.com/hunternet/p/9957913.html



***



### hash

#### 简介

数据存储需求：对一系列存储的数据进行编组，方便管理，典型应用存储对象信息

数据存储结构：一个存储空间保存多个键值对数据

hash 类型：底层使用**哈希表**结构实现数据存储

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/hash结构图.png )

Redis 中的 hash 类似于 Java 中的  `Map<String, Map<Object,object>>`，左边是 key，右边是值，中间叫 field 字段，本质上 **hash 存了一个 key-value 的存储空间**

hash 是指的一个数据类型，并不是一个数据

* 如果 field 数量较少，存储结构优化为**压缩列表结构**（有序）
* 如果 field 数量较多，存储结构使用 HashMap 结构（无序）



***



#### 操作

指令操作：

* 数据操作

  ```sh
  hset key field value		#添加/修改数据
  hdel key field1 [field2]	#删除数据，[]代表可选
  hsetnx key field value		#设置field的值，如果该field存在则不做任何操作
  hmset key f1 v1 f2 v2...	#添加/修改多个数据
  ```

* 查询操作

  ```sh
  hget key field				#获取指定field对应数据
  hgetall key					#获取指定key所有数据
  hmget key field1 field2...	#获取多个数据
  hexists key field			#获取哈希表中是否存在指定的字段
  hlen key					#获取哈希表中字段的数量
  ```

* 获取哈希表中所有的字段名或字段值

  ```sh
  hkeys key					#获取所有的field	
  hvals key					#获取所有的value
  ```

* 设置指定字段的数值数据增加指定范围的值

  ```sh
  hincrby key field increment		#指定字段的数值数据增加指定的值，increment为负数则减少
  hincrbyfloat key field increment#操作小数
  ```

```sh
127.0.0.1:6379> hset hash-key k1 v1
(integer) 1
127.0.0.1:6379> hset hash-key k2 v2
(integer) 1
127.0.0.1:6379> hset hash-key k1 v1
(integer) 0
127.0.0.1:6379> hset hash-key k1 v2
(integer) 0
127.0.0.1:6379> sadd myset a
(integer) 0
127.0.0.1:6379> hgetall hash-key
1) "k1"
2) "v2"
3) "k2"
4) "v2"
127.0.0.1:6379> hget hash-key k1
"v2"
127.0.0.1:6379> hget hash-key k2
"v2"
127.0.0.1:6379> hdel hash-key k1
(integer) 1
127.0.0.1:6379> hgetall hash-key
1) "k2"
2) "v2"
```



注意事项

1. hash 类型中 value 只能存储字符串，不允许存储其他数据类型，不存在嵌套现象，如果数据未获取到，对应的值为（nil）
2. 每个 hash 可以存储 2^32 - 1 个键值对
3. hash 类型和对象的数据存储形式相似，并且可以灵活添加删除对象属性。但 hash 设计初衷不是为了存储大量对象而设计的，不可滥用，不可将 hash 作为对象列表使用
4. hgetall 操作可以获取全部属性，如果内部 field 过多，遍历整体数据效率就很会低，有可能成为数据访问瓶颈



***



#### 应用

```sh
user:id:3506728370 → {"name":"春晚","fans":12210862,"blogs":83}
```

对于以上数据，使用单条去存的话，存的条数会很多。但如果用 json 格式，存一条数据就够了。

假如现在粉丝数量发生了变化，要把整个值都改变，但是用单条存就不存在这个问题，只需要改其中一个就可以

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/hash应用场景结构图.png )

可以实现购物车的功能，key 对应着每个用户，存储空间存储购物车的信息

**当前设计是否加速了购物车的呈现?**

当前仅仅是将数据存储到了redis中，并没有起到加速的作用，商品信息还需要二次查询数据库

- 每条购物车中的商品记录保存成两条hash
- hash1专用于保存购买数量
  命名格式：商品id:nums
  保存数据：数值
- hash2专用于保存购物车中显示的信息，包含文字描述，图片地址，所属商家信息等
  命名格式：商品id:info
  保存数据：json

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/hash应用.png )

***



#### 实现

##### 底层结构

哈希类型的内部编码有两种：ziplist（压缩列表）、hashtable（哈希表、字典）

当存储的数据量比较小的情况下，Redis 才使用压缩列表来实现字典类型，具体需要满足两个条件：

- 当键值对个数小于 hash-max-ziplist-entries 配置（默认 512 个）
- 所有键值都小于 hash-max-ziplist-value 配置（默认 64 字节）

ziplist 使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比 hashtable 更加优秀，当 ziplist 无法满足哈希类型时，Redis 会使用 hashtable 作为哈希的内部实现，因为此时 ziplist 的读写效率会下降，而 hashtable 的读写时间复杂度为 O(1)



****



##### 压缩列表

压缩列表（ziplist）是列表和哈希的底层实现之一，压缩列表用来紧凑数据存储，节省内存，有序：

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-压缩列表数据结构.png )

压缩列表是由一系列特殊编码的连续内存块组成的顺序型（sequential）数据结枃，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值



***



##### 哈希表

Redis 字典使用散列表为底层实现，一个散列表里面有多个散列表节点，每个散列表节点就保存了字典中的一个键值对，发生哈希冲突采用链表法解决，存储无序

* 为了避免散列表性能的下降，当装载因子大于 1 的时候，Redis 会触发扩容，将散列表扩大为原来大小的 2 倍左右
* 当数据动态减少之后，为了节省内存，当装载因子小于 0.1 的时候，Redis 就会触发缩容，缩小为字典中数据个数的 50 % 左右



***



### list

#### 简介

数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分

数据存储结构：一个存储空间保存多个数据，且通过数据可以体现进入顺序，允许重复元素

list 类型：保存多个数据，底层使用**双向链表**存储结构实现，类似于 LinkedList

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/list结构图.png )

如果两端都能存取数据的话，这就是双端队列，如果只能从一端进一端出，这个模型叫栈



***



#### 操作

指令操作：

* 数据操作

  ```sh
  lpush key value1 [value2]...#从左边添加/修改数据
  rpush key value1 [value2]...#从右边添加/修改数据
  lpop key					#从左边获取并移除第一个数据，类似于出栈/出队
  rpop key					#从右边获取并移除第一个数据
  lrem key count value		#删除指定数据，count=2删除2个，该value可能有多个(重复数据)
  ```

* 查询操作

  ```sh
  lrange key start stop		#从左边遍历数据并指定开始和结束索引，0是第一个索引，-1是终索引
  lindex key index			#获取指定索引数据，没有则为nil，没有索引越界
  llen key					#list中数据长度/个数
  ```

* 规定时间内获取并移除数据

  ```sh
  b							#代表阻塞
  blpop key1 [key2] timeout	#在指定时间内获取指定key(可以多个)的数据，超时则为(nil)
  							#可以从其他客户端写数据，当前客户端阻塞读取数据
  brpop key1 [key2] timeout	#从右边操作
  ```

* 复制操作

  ```sh
  brpoplpush source destination timeout	#从source获取数据放入destination，假如在指定时间内没有任何元素被弹出，则返回一个nil和等待时长。反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长
  ```

```sh
127.0.0.1:6379> lpush mylist 1 2 aa bb 5
(integer) 5
127.0.0.1:6379> lrange mylist 0 -1
1) "5"
2) "bb"
3) "aa"
4) "2"
5) "1"
127.0.0.1:6379> llen mylist
(integer) 5
127.0.0.1:6379> lrange mylist 0 3
1) "5"
2) "bb"
3) "aa"
4) "2"
127.0.0.1:6379> lindex mylist -1
"1"
```



注意事项

1. list 中保存的数据都是 string 类型的，数据总容量是有限的，最多 2^32 - 1 个元素（4294967295）
2. list 具有索引的概念，但操作数据时通常以队列的形式进行入队出队，或以栈的形式进行入栈出栈
3. 获取全部数据操作结束索引设置为 -1
4. list 可以对数据进行分页操作，通常第一页的信息来自于 list，第 2 页及更多的信息通过数据库的形式加载



***



#### 应用

企业运营过程中，系统将产生出大量的运营数据，如何保障多台服务器操作日志的统一顺序输出？

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/image-20220222112409951.png)

* 依赖 list 的数据具有顺序的特征对信息进行管理，右进左查或者左近左查
*  使用队列模型解决多路信息汇总合并的问题
* 使用栈模型解决最新消息的问题

微信文章订阅公众号：

* 比如订阅了两个公众号，它们发布了两篇文章，文章 ID 分别为 666 和 888，可以通过执行 `LPUSH key 666 888` 命令推送给我



****



#### 实现

##### 底层结构

在 Redis3.2 版本以前列表类型的内部编码有两种：ziplist（压缩列表）和 linkedlist（链表）

列表中存储的数据量比较小的时候，列表就会使用一块连续的内存存储，采用压缩列表的方式实现：

* 列表中保存的单个数据（有可能是字符串类型的）小于 64 字节
* 列表中数据个数少于 512 个

在 Redis3.2 版本 以后对列表数据结构进行了改造，使用 **quicklist（快速列表）**代替了 linkedlist



****



##### 链表结构

Redis 链表为**双向无环链表**，使用 listNode 结构表示

```c
typedef struct listNode{ 	
    // 前置节点 	struct listNode *prev; 	
    // 后置节点 	struct listNode *next; 	
    // 节点的值 	void *value;
} listNode;
```

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-链表数据结构.png)

- 双向：链表节点带有前驱、后继指针，获取某个节点的前驱、后继节点的时间复杂度为 O(1)
- 无环：链表为非循环链表，表头节点的前驱指针和表尾节点的后继指针都指向 NULL，对链表的访问以 NULL 为终点



***



##### 快速列表

quicklist 实际上是 ziplist 和 linkedlist 的混合体，将 linkedlist 按段切分，每一段使用 ziplist 来紧凑存储，多个 ziplist 之间使用双向指针串接起来，既满足了快速的插入删除性能，又不会出现太大的空间冗余

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-快速列表数据结构.png )



***



### set

#### 简介

数据存储需求：存储大量的数据，在查询方面提供更高的效率

数据存储结构：能够保存大量的数据，高效的内部存储机制，便于查询

set 类型：与 hash 存储结构哈希表完全相同，只是仅存储键不存储值（nil），所以添加，删除，查找的复杂度都是 O(1)，并且**值是不允许重复且无序的**

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/set结构图.png )



***



#### 操作

指令操作：

* 数据操作

  ```sh
  sadd key member1 [member2]	#添加数据
  srem key member1 [member2]	#删除数据
  ```

* 查询操作

  ```sh
  smembers key				#获取全部数据
  scard key					#获取集合数据总量
  sismember key member		#判断集合中是否包含指定数据
  ```

* 随机操作

  ```sh
  spop key [count]			#随机获取集中的某个数据并将该数据移除集合
  srandmember key [count]		#随机获取集合中指定(数量)的数据
  ```

* 集合的交、并、差

  ```sh
  sinter key1 [key2...]  		#两个集合的交集，不存在为(empty list or set)
  sunion key1 [key2...]  		#两个集合的并集
  sdiff key1 [key2...]		#两个集合的差集
  sinterstore destination key1 [key2...]	#两个集合的交集并存储到指定集合中
  sunionstore destination key1 [key2...]	#两个集合的并集并存储到指定集合中
  sdiffstore destination key1 [key2...]	#两个集合的差集并存储到指定集合中
  ```

* 复制

  ```sh
  smove source destination member			#将指定数据从原始集合中移动到目标集合中
  ```

```sh
127.0.0.1:6379> sadd myset a b c
(integer) 3
127.0.0.1:6379> smembers myset
1) "a"
2) "c"
3) "b"
127.0.0.1:6379> sismember myset a
(integer) 1
127.0.0.1:6379> sismember myset b
(integer) 1
127.0.0.1:6379> srem myset d
(integer) 0
127.0.0.1:6379> srem myset c
(integer) 1
127.0.0.1:6379> smembers myset
1) "a"
2) "b"
127.0.0.1:6379> sadd myset d
(integer) 1
127.0.0.1:6379> smembers myset
1) "a"
2) "d"
3) "b"
```



注意事项

1. set 类型不允许数据重复，如果添加的数据在 set 中已经存在，将只保留一份
2. set 虽然与 hash 的存储结构相同，但是无法启用 hash 中存储值的空间



***



#### 应用

应用场景：

1. 黑名单：资讯类信息类网站追求高访问量，但是由于其信息的价值，往往容易被不法分子利用，通过爬虫技术，快速获取信息，个别特种行业网站信息通过爬虫获取分析后，可以转换成商业机密。

   注意：爬虫不一定做摧毁性的工作，有些小型网站需要爬虫为其带来一些流量。

2. 白名单：对于安全性更高的应用访问，仅仅靠黑名单是不能解决安全问题的，此时需要设定可访问的用户群体， 依赖白名单做更为苛刻的访问验证

3. 随机操作可以实现抽奖功能

4. 集合的交并补可以实现微博共同关注的查看，可以根据共同关注或者共同喜欢推荐相关内容

5. 公司对旗下新的网站做推广，统计网站的PV（访问量）,UV（独立访客）,IP（独立IP）。
   PV：网站被访问次数，可通过刷新页面提高访问量
   UV：网站被不同用户访问的次数，可通过cookie统计访问量，相同用户切换IP地址，UV不变
   IP：网站被不同IP地址访问的总次数，可通过IP地址统计访问量，相同IP不同用户访问，IP不变利用set集合的数据去重特征，记录各种访问数据

   - 建立string类型数据，利用incr统计日访问量（PV）
   - 建立set模型，记录不同cookie数量（UV）
   - 建立set模型，记录不同IP数量（IP）



***



#### 实现

集合类型的内部编码有两种：

* intset（整数集合）：当集合中的元素都是整数且元素个数小于 set-maxintset-entries配置（默认 512 个）时，Redis 会选用 intset 来作为集合的内部实现，从而减少内存的使用

* hashtable（哈希表，字典）：当无法满足 intset 条件时，Redis 会使用 hashtable 作为集合的内部实现

整数集合（intset）是 Redis 用于保存整数值的集合抽象数据结构，可以保存类型为 int16_t、int32_t 或者 int64_t 的整数值，并且保证集合中的元素是**有序不重复**的





***



### sorted_set

#### 简介

数据存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式

数据存储结构：新的存储模型，可以保存可排序的数据

sorted_set类型：在 set 的存储结构基础上添加可排序字段，类似于 TreeSet

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-sorted_set结构图.png )



****



#### 操作

指令操作：

* 数据操作

  ```sh
  zadd key score1 member1 [score2 member2]	#添加数据
  zrem key member [member ...]				#删除数据
  zremrangebyrank key start stop 				#删除指定索引范围的数据
  zremrangebyscore key min max				#删除指定分数区间内的数据
  zscore key member							#获取指定值的分数
  zincrby key increment member				#指定值的分数增加increment
  ```

* 查询操作

  ```sh
  zrange key start stop [WITHSCORES]		#获取指定范围的数据，升序，WITHSCORES 代表显示分数
  zrevrange key start stop [WITHSCORES]	#获取指定范围的数据，降序
  
  zrangebyscore key min max [WITHSCORES] [LIMIT offset count]	#按条件获取数据，从小到大
  zrevrangebyscore key max min [WITHSCORES] [...]				#从大到小
  
  zcard key										#获取集合数据的总量
  zcount key min max								#获取指定分数区间内的数据总量
  zrank key member								#获取数据对应的索引（排名）升序
  zrevrank key member								#获取数据对应的索引（排名）降序
  ```

  * min 与 max 用于限定搜索查询的条件
  * start 与 stop 用于限定查询范围，作用于索引，表示开始和结束索引
  * offset 与 count 用于限定查询范围，作用于查询结果，表示开始位置和数据总量

* 集合的交、并操作

  ```sh
  zinterstore destination numkeys key [key ...]	#两个集合的交集并存储到指定集合中
  zunionstore destination numkeys key [key ...]	#两个集合的并集并存储到指定集合中
  ```

```sh
127.0.0.1:6379> zadd zset-key 10 m1
(integer) 1
127.0.0.1:6379> zadd zset-key 5 m2
(integer) 1
127.0.0.1:6379> zadd zset-key 20 m3
(integer) 1
127.0.0.1:6379> zadd zset-key 20 m3
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1
1) "m2"
2) "m1"
3) "m3"
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "m2"
2) "5"
3) "m1"
4) "10"
5) "m3"
6) "20"
127.0.0.1:6379> zrangebyscore zset-key 0 10 withscores
1) "m2"
2) "5"
3) "m1"
4) "10"
```



注意事项：

1. score 保存的数据存储空间是 64 位，如果是整数范围是 -9007199254740992~9007199254740992
2. score 保存的数据也可以是一个双精度的 double 值，基于双精度浮点数的特征可能会丢失精度，慎重使用
3. sorted_set 底层存储还是基于 set 结构的，因此数据不能重复，如果重复添加相同的数据，score 值将被反复覆盖，保留最后一次修改的结果



***



#### 应用

* 排行榜

* 基础服务+增值服务类网站会设定各位会员的试用，让用户充分体验会员优势。例如观影试用VIP、游戏
  VIP体验、云盘下载体验VIP、数据查看体验VIP。当VIP体验到期后，如果有效管理此类信息。即便对于正式
  VIP用户也存在对应的管理方式。
  网站会定期开启投票、讨论，限时进行，逾期作废。如何有效管理此类过期信息。 对于基于时间线限定的任务处理，将处理时间记录为score值，利用排序功能区分处理的先后顺序

  - 记录下一个要处理的时间，当到期后处理对应任务，移除redis中的记录，并记录下一个要处理的时间
  - 当新任务加入时，判定并更新当前下一个要处理的任务时间
  - 为提升sorted_set的性能，通常将任务根据特征存储成若干个sorted_set。
  - 例如1小时内，1天内，周内，
    月内，季内，年度等，操作时逐级提升，将即将操作的若干个任务纳入到1小时内处理的队列中

  redis 应用于定时任务执行顺序管理或任务过期管理

* 当任务或者消息待处理，形成了任务队列或消息队列时，对于高优先级的任务要保障对其优先处理，采用 score 记录权重 

  对于带有权重的任务，优先处理权重高的任务，采用score记录权重即可
  多条件任务权重设定
  如果权重条件过多时，需要对排序score值进行处理，保障score值能够兼容2条件或者多条件，例如外贸
  订单优先于国内订单，总裁订单优先于员工订单，经理订单优先于员工订单

  - 因score长度受限，需要对数据进行截断处理，尤其是时间设置为小时或分钟级即可（折算后）
  - 先设定订单类别，后设定订单发起角色类别，整体score长度必须是统一的，不足位补0。第一排序规则首
    位不得是0
  - 例如`外贸101`，`国内102`，`经理004`，`员工008`。
    - 员工下的外贸单score值为`101008`（优先）
    - 经理下的国内单score值为`102004`



***



#### 实现

##### 底层结构

有序集合是由 ziplist（压缩列表）或 skiplist（跳跃表）组成的

当数据比较少时，有序集合使用的是 ziplist 存储的，使用 ziplist 格式存储需要满足以下两个条件：

- 有序集合保存的元素个数要小于 128 个；
- 有序集合保存的所有元素大小都小于 64 字节

当元素比较多时，此时 ziplist 的读写效率会下降，时间复杂度是 O(n)，跳表的时间复杂度是 O(logn)



***



##### 跳跃表

Redis 使用跳跃表作为有序集合键的底层实现之一，如果一个有序集合包含的**元素数量比较多**，又或者有序集合中元素的**成员是比较长的字符串**时，Redis 就会使用跳跃表来作为有序集合健的底层实现

跳跃表在链表的基础上增加了多级索引以提升查找的效率，索引是占内存的，所以是一个**空间换时间**的方案。原始链表中存储的有可能是很大的对象，而索引结点只需要存储关键值和几个指针，并不需要存储对象，因此当节点本身比较大或者元素数量比较多的时候，其优势可以被放大，而缺点则可以忽略

* 基于单向链表加索引的方式实现

- Redis 的跳跃表实现由 zskiplist 和 zskiplistnode 两个结构组成，其中 zskiplist 用于保存跳跃表信息（比如表头节点、表尾节点、长度），而 zskiplistnode 则用于表示跳跃表节点
- Redis 每个跳跃表节点的层高都是 1 至 32 之间的随机数（Redis5 之后最大层数为 64）
- 在同一个跳跃表中，多个节点可以包含相同的分值，但每个节点的成员对象必须是唯一的。跳跃表中的节点按照分值大小进行排序，当分值相同时节点按照成员对象的大小进行排序

![](https://gitee.com/seazean/images/raw/master/DB/Redis-跳跃表数据结构.png)



参考文章：https://www.cnblogs.com/hunternet/p/11248192.html



***



### Bitmaps

#### 操作

Bitmaps 本身不是一种数据类型， 实际上就是字符串（key-value） ， 但是它可以对字符串的位进行操作

数据结构的详解查看 Java → Algorithm → 位图

指令操作：

* 获取指定 key 对应**偏移量**上的 bit 值

  ```sh
  getbit key offset
  ```

* 设置指定 key 对应偏移量上的 bit 值，value 只能是 1 或 0

  ```sh
  setbit key offset value
  ```

* 对指定 key 按位进行交、并、非、异或操作，并将结果保存到 destKey 中

  ```sh
  bitop option destKey key1 [key2...]
  ```

  option：and 交、or 并、not 非、xor 异或

* 统计指定 key 中1的数量

  ```sh
  bitcount key [start end]
  ```



***



#### 应用

- 解决 Redis 缓存穿透，判断给定数据是否存在， 防止缓存穿透

  ![]( https://gitee.com/seazean/images/raw/master/DB/Redis-Bitmaps应用之缓存穿透.png )

- 垃圾邮件过滤，对每一个发送邮件的地址进行判断是否在布隆的黑名单中，如果在就判断为垃圾邮件

- 爬虫去重，爬给定网址的时候对已经爬取过的 URL 去重

- 信息状态统计



***



### Hyper

基数是数据集去重后元素个数，HyperLogLog 是用来做基数统计的，运用了 LogLog 的算法

```java
{1, 3, 5, 7, 5, 7, 8} 	基数集： {1, 3, 5 ,7, 8} 	基数：5{1, 1, 1, 1, 1, 7, 1} 	基数集： {1,7} 				基数：2
```

相关指令：

* 添加数据

  ```sh
  pfadd key element [element ...]
  ```

* 统计数据

  ```sh
  pfcount key [key ...]
  ```

* 合并数据

  ```sh
  pfmerge destkey sourcekey [sourcekey...]
  ```

应用场景：

* 用于进行基数统计，不是集合不保存数据，只记录数量而不是具体数据，比如网站的访问量
* 核心是基数估算算法，最终数值存在一定误差
* 误差范围：基数估计的结果是一个带有 0.81% 标准错误的近似值
* 耗空间极小，每个 hyperloglog key 占用了12K的内存用于标记基数
* pfadd 命令不是一次性分配12K内存使用，会随着基数的增加内存逐渐增大
* Pfmerge 命令合并后占用的存储空间为12K，无论合并之前数据量多少



***



### GEO

GeoHash 是一种地址编码方法，把二维的空间经纬度数据编码成一个字符串

* 添加坐标点

  ```sh
  geoadd key longitude latitude member [longitude latitude member ...]georadius key longitude latitude radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
  ```

* 获取坐标点

  ```sh
  geopos key member [member ...]georadiusbymember key member radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
  ```

* 计算距离

  ```sh
  geodist key member1 member2 [unit]	#计算坐标点距离geohash key member [member ...]		#计算经纬度
  ```

redis 应用于地理位置计算





***

## 应用总结

### 应用一

针对非会员用户提供每分钟10次调用服务接口的服务

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/image-20220222153926066.png)



### 应用二

> 使用微信的过程中，当微信接收消息后，会默认将最近接收的消息置顶，当多个好友及关注的订阅号同时发
> 送消息时，该排序会不停的进行交替。同时还可以将重要的会话设置为置顶。一旦用户离线后，再次打开微
> 信时，消息该按照什么样的顺序显示？

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/image-20220222155022208.png)

- 依赖list的数据具有顺序的特征对消息进行管理，将list结构作为栈使用
- 对置顶与普通会话分别创建独立的list分别管理
- 当某个list中接收到用户消息后，将消息发送方的id从list的一侧加入list（此处设定左侧）
- 多个相同id发出的消息反复入栈会出现问题，在**入栈之前无论是否具有当前id对应的消息，先删除对应id**
- 推送消息时**先推送置顶会话list**，再推送普通会话list，推送完成的list清除所有数据
- **消息的数量**，也就是微信用户对话数量采用计数器的思想另行记录，伴随list操作同步更新



### 总结

- redis用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性
- redis 控制数据的生命周期，通过数据是否失效控制业务行为，适用于所有具有时效性限定控制的操作
- redis应用于各种结构型和非结构型高热度数据访问加速
- redis 应用于购物车数据存储设计
- redis 应用于抢购，限购类、限量发放优惠卷、激活码等业务的数据存储设计
- redis 应用于具有操作先后顺序的数据控制
- redis 应用于最新消息展示
- redis 应用于随机推荐类信息检索，例如热点歌单推荐，热点新闻推荐，热卖旅游线路，应用APP推荐，大V推荐等
- redis 应用于同类信息的关联搜索，二度关联搜索，深度关联搜索
- redis 应用于同类型不重复数据的合并、取交集操作
- redis 应用于同类型数据的快速去重
- redis 应用于基于黑名单与白名单设定的服务控制
- redis 应用于计数器组合排序功能对应的排名
- redis 应用于定时任务执行顺序管理或任务过期管理
- redis 应用于及时任务/消息队列执行管理
- redis 应用于按次结算的服务控制
- redis 应用于基于时间顺序的数据操作，而不关注具体时间



## Jedis

### 基本使用

Jedis 用于 Java 语言连接 redis 服务，并提供对应的操作 API

1. jar 包导入

   * 下载地址：https://mvnrepository.com/artifact/redis.clients/jedis

   * 基于 maven：

     ```xml
     <dependency>	
         <groupId>redis.clients</groupId>	
         <artifactId>jedis</artifactId>	
         <version>2.9.0</version>
     </dependency>
     ```

2. 客户端连接 redis
   API 文档：http://xetorthio.github.io/jedis/

   连接 redis：`Jedis jedis = new Jedis("192.168.0.185", 6379);`
   操作 redis：`jedis.set("name", "seazean");  jedis.get("name");`
   关闭 redis：`jedis.close();`

代码实现：

```java
public class JedisTest {    
    public static void main(String[] args) {        
        //1.获取连接对象        
        Jedis jedis = new Jedis("192.168.2.185",6379);        
        //2.执行操作        
        jedis.set("age","39");        
        String hello = jedis.get("hello");        
        System.out.println(hello);        
        jedis.lpush("list1","a","b","c","d");        
        List<String> list1 = jedis.lrange("list1", 0, -1);        
        for (String s:list1 ) {            
            System.out.println(s);        
        }        
        jedis.sadd("set1","abc","abc","def","poi","cba");        
        Long len = jedis.scard("set1");        
        System.out.println(len);        
        //3.关闭连接        
        jedis.close();    
    }
}
```



***



### 工具类

连接池对象：
	JedisPool：Jedis 提供的连接池技术  
	poolConfig：连接池配置对象 
	host：redis 服务地址
	port：redis 服务端口号

JedisPool 的构造器如下：

```java
public JedisPool(GenericObjectPoolConfig poolConfig, String host, int port) {
    this(poolConfig, host, port, 2000, (String)null, 0, (String)null);
}
```



* 创建配置文件 redis.properties

  ```properties
  redis.maxTotal=50redis.maxIdel=10redis.host=192.168.2.185redis.port=6379
  ```

* 工具类：

  ```java
  public class JedisUtils {    
      private static int maxTotal;    
      private static int maxIdel;    
      private static String host;    
      private static int port;    
      private static JedisPoolConfig jpc;    
      private static JedisPool jp;    
      static {        
          ResourceBundle bundle = ResourceBundle.getBundle("redis");        
              //最大连接数        
              maxTotal = Integer.parseInt(bundle.getString("redis.maxTotal"));       
              //活动连接数        
              maxIdel = Integer.parseInt(bundle.getString("redis.maxIdel"));        
              host = bundle.getString("redis.host");        
              port = Integer.parseInt(bundle.getString("redis.port"));        
              //Jedis连接配置        
              jpc = new JedisPoolConfig();        
          jpc.setMaxTotal(maxTotal);        
          jpc.setMaxIdle(maxIdel);        
          //连接池对象        
          jp = new JedisPool(jpc, host, port);    
      }    
      //对外访问接口，提供jedis连接对象，连接从连接池获取    
      public static Jedis getJedis() {
          return jp.getResource();    
      }
  }
  ```
  
  

****

## SpringBoot整合

### 基本使用

1. 导入依赖

   ```java
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
           </dependency>
   ```

   

2. 添加配置文件

   ```java
   spring:
     redis:
       host: x.x.x.x
       port: 6379
   ```

3. 测试代码

   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class test {
       @Autowired
       private RedisTemplate redisTemplate;
   
       @Test
       public void testRedis() {
           redisTemplate.opsForValue().set("id", 11);
           System.out.println(redisTemplate.opsForValue().get("id"));
   
           // boundValueOps绑定了一个key后操作，本质是重新封装了opsForValue
           redisTemplate.boundValueOps("name").set("haha");
           System.out.println(redisTemplate.boundValueOps("name").get());
       }
   }
   ```



### 阅读源码

~~~shell
# Spring Boot 所有的配置类，都有一个自动配置类  RedisTemplate
# 自动配置类都会绑定一个 properties 配置文件。  RedisProperties
~~~

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/YycBTJ.png)

```java
@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    ) // 我们可以自己定义一个 RedisTemplate 来替换这个默认的。
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        // 默认的 RedisTemplate 没有过多的设置， Redis 对象都是需要序列化的。
        // 两个泛型都是 Object, Object 的类型，我们需要强制装换为 <String, Obejct>
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean // 由于 String 类型是 Redis 中最常用的，所以单独提出来一个 bean .
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```



看一下源码：RedisTemplate.class

```java
// 序列化配置
@Nullable
private RedisSerializer keySerializer = null;
@Nullable
private RedisSerializer valueSerializer = null;
@Nullable
private RedisSerializer hashKeySerializer = null;
@Nullable
private RedisSerializer hashValueSerializer = null;
private RedisSerializer<String> stringSerializer = RedisSerializer.string();
public void afterPropertiesSet() {
    super.afterPropertiesSet();
    boolean defaultUsed = false;
    if (this.defaultSerializer == null) {
        // 默认使用了 JDK 的序列化，会使得字符串转义
        this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
    }

    // ...
}
```

我们使用 Json 序列化，所以需要自定义配置类

### 序列化

编写一个实体类 User，测试序列化。

```java
package com.xiaopizhu.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.stereotype.Component;

@Component
@NoArgsConstructor
@AllArgsConstructor
@Data
public class User {
    private String name;
    private int age;
}
```

测试序列化：

```java
 @Test
public void test() throws JsonProcessingException {
    User user = new User("xiaoming", 3);
    redisTemplate.opsForValue().set("user", user);
    System.out.println(redisTemplate.opsForValue().get("user"));
}
```

抛出异常：

```bash
Caused by: java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable payload but received an object of type [com.xiaopizhu.pojo.User]
	at org.springframework.core.serializer.DefaultSerializer.serialize(DefaultSerializer.java:43)
	at org.springframework.core.serializer.support.SerializingConverter.convert(SerializingConverter.java:63)
	... 35 more
```

`DefaultSerializer requires a Serializable`默认的序列化需要实体类实现序列化接口。所以修改 User：

```java
public class User implements Serializable {
    private String name;
    private int age;
}
```

结果：

```java
User(name=xiaoming, age=3)
```

结果显示正常，但是控制台还是转义的。

```bash
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04user"
127.0.0.1:6379>
```

使用 jackson 的序列化：

```java
@Test
public void test() throws JsonProcessingException {
    // 一般开发中都会使用 json 来传递对象
    User user = new User("xiaoming", 3);
    String jsonUser = new ObjectMapper().writeValueAsString(user);
    redisTemplate.opsForValue().set("user", jsonUser);
    System.out.println(redisTemplate.opsForValue().get("user")); // {"name":"xiaoming","age":3}
}
```

无论 User 是否实现了 Serializable 接口，控制台结果显示正常，但是客户端中查看还是被转义了。

如果不想使用 JDK 的序列化，可以自己编写 RedisTemplate。



### 自定义RedisTemplate

```java
package com.xiaopizhu.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * 编写的自己的 RedisTemplate
 */
@Configuration
public class RedisConfig {

    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 为了开发方便，一般使用 <String, Object>
        RedisTemplate<String, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);

        // 序列化配置
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        // String 的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key 采用 String 的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash 的 key 也采用 String 的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value 序列化方式采用 Jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash 的 value 序列化方式采用 Jackson
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();

        return template;
    }
}
```

注入和测试：

```java
@Autowired
@Qualifier("redisTemplate")
private RedisTemplate redisTemplate;

@Test
public void test() throws JsonProcessingException {
    // 一般开发中都会使用 json 来传递对象
    User user = new User("xiaoming", 3);
    String jsonUser = new ObjectMapper().writeValueAsString(user);
    redisTemplate.opsForValue().set("user", jsonUser);
    System.out.println(redisTemplate.opsForValue().get("user")); // {"name":"xiaoming","age":3}
}
```

客户端中查看：

```bash
127.0.0.1:6379> keys *
1) "user"
127.0.0.1:6379>
```

这个时候的对象就没有被转义。

或者直接使用 RedisTemplate<String,String> 或者 StringRedisTemplate 即可。

---



## 可视化

Redis Desktop Manager

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-可视化工具.png )





****



## 持久化

### 概述

持久化：利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复的工作机制称为持久化 

作用：持久化用于防止数据的意外丢失，确保数据安全性，因为 Redis 是内存级，所以需要持久化到磁盘

计算机中的数据全部都是二进制，保存一组数据有两种方式
![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-持久化的两种方式.png )

第一种：将当前数据状态进行保存，快照形式，存储数据结果，存储格式简单

第二种：将数据的操作过程进行保存，日志形式，存储操作过程，存储格式复杂



***



### RDB

#### save

save 指令：手动执行一次保存操作

配置 redis.conf：

```sh
dir path				#设置存储.rdb文件的路径，通常设置成存储空间较大的目录中，目录名称
datadbfilename "x.rdb"		#设置本地数据库文件名，默认值为dump.rdb，通常设置为dump-端口号.
rdbrdbcompression yes|no	#设置存储至本地数据库时是否压缩数据，默认yes，设置为no节省CPU运行时间
rdbchecksum yes|no		#设置读写文件过程是否进行RDB格式校验，默认yes，设置为no，节约读写10%时间						#消耗，但存在数据损坏的风险
```

工作原理：redis 是个**单线程的工作模式**，会创建一个任务队列，所有的命令都会进到这个队列排队执行。当某个指令在执行的时候，队列后面的指令都要等待，所以这种执行方式会非常耗时。

save 指令的执行会阻塞当前 Redis 服务器，直到当前 RDB 过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用



***



#### bgsave

指令：bgsave（bg 是 background，后台执行的意思）

配置 redis.conf

```sh
stop-writes-on-bgsave-error yes|no	#后台存储过程中如果出现错误，是否停止保存操作，默认
yesdbfilename filename  
dir path  
rdbcompression yes|no  
rdbchecksum yes|no
```

bgsave 指令工作原理：

![](https://gitee.com/seazean/images/raw/master/DB/Redis-bgsave工作原理.png)

流程：当执行 bgsave 的时候，客户端发出 bgsave 指令给到 redis 服务器，服务器返回后台已经开始执行的信息给客户端，同时使用 fork 函数创建一个子进程，让子进程去执行 save 相关的操作。持久化过程是先将数据写入到一个临时文件中，持久化操作结束再用这个临时文件**替换**上次持久化的文件，在这个过程中主进程是不进行任何 IO 操作的，这确保了极高的性能 

bgsave 分成两个过程：第一个是服务端收到指令直接告诉客户端开始执行；另外一个过程是 fork 的子进程在完成后台的保存操作，操作完以后返回消息。**两个进程不相互影响**，所以在持久化期间 Redis 可以正常工作

注意：bgsave 命令是针对 save 阻塞问题做的优化，Redis 内部所有涉及到 RDB 操作都采用 bgsave 的方式，save 命令可以放弃使用



***



#### 自动

配置文件自动 RDB，无需显式调用相关指令，save 配置启动后底层执行的是 bgsave 操作

配置 redis.conf：

```sh
save second changes #设置自动持久化条件，满足限定时间范围内key的变化数量就进行持久化(bgsave)
```

参数：

* second：监控时间范围
* changes：监控 key 的变化量

说明： save 配置中对于 second 与 changes 设置通常具有互补对应关系，尽量不要设置成包含性关系

示例：

```sh
save 300 10	#300s内10个key发生变化就进行持久化
```

判定 key 变化的原理：

* 对数据产生了影响
* 不进行数据比对，比如 name 键存在，重新 set name seazean 也算一次变化

save 配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的

RDB三种启动方式对比：

| 方式           | save指令 | bgsave指令 |
| -------------- | -------- | ---------- |
| 读写           | 同步     | 异步       |
| 阻塞客户端指令 | 是       | 否         |
| 额外内存消耗   | 否       | 是         |
| 启动新进程     | 否       | 是         |



***



#### 总结

* RDB 特殊启动形式的指令（客户端输入）

  * 服务器运行过程中重启

    ```sh
    debug reload
    ```

  * 关闭服务器时指定保存数据

    ```sh
    shutdown save
    ```

    默认情况下执行 shutdown 命令时，自动执行 bgsave（如果没有开启 AOF 持久化功能）

  * 全量复制：主从复制部分详解

* RDB 优点：

  - RDB 是一个紧凑压缩的二进制文件，存储效率较高，但存储数据量较大时，存储效率较低
  - RDB 内部存储的是 redis 在某个时间点的数据快照，非常**适合用于数据备份，全量复制**等场景
  - RDB 恢复数据的速度要比 AOF 快很多，因为是快照，直接恢复

* RDB 缺点：

  - bgsave 指令每次运行要执行 fork 操作创建子进程，会牺牲一些性能
  - RDB 方式无论是执行指令还是利用配置，无法做到实时持久化，具有丢失数据的可能性，最后一次持久化后的数据可能丢失
  - Redis 的众多版本中未进行 RDB 文件格式的版本统一，可能出现各版本之间数据格式无法兼容

* 应用：服务器中每 X 小时执行 bgsave 备份，并将 RDB 文件拷贝到远程机器中，用于灾难恢复



***



### AOF

#### 概述

AOF（append only file）持久化：以独立日志的方式记录每次写命令（不记录读），**增量保存**，只许追加文件但不可以改写文件，重启时再重新执行 AOF 文件中命令达到恢复数据的目的，**与 RDB 相比可以简单理解为由记录数据改为记录数据的变化**

AOF 主要作用是解决了数据持久化的实时性，目前已经是 Redis 持久化的主流方式

AOF 写数据过程：
![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-AOF工作原理.png)



***



#### 策略

客户端的请求写命令会被 append 追加到 AOF 缓冲区内

启动 AOF 基本配置：

```sh
appendonly yes|no				#开启AOF持久化功能，默认no，即不开启状态
appendfilename filename			#AOF持久化文件名，默认appendonly.aof，建议设置appendonly-端口号.
aofdir							#AOF持久化文件保存路径，与RDB持久化文件路径保持一致即可
```

```sh
appendfsync always|everysec|no	#AOF写数据策略：默认为everysec
```

AOF 持久化数据的三种策略（appendfsync）：

- always（每次）：每次写入操作均同步到 AOF 文件中，**数据零误差，性能较低**，不建议使用。


- everysec（每秒）：每秒将缓冲区中的指令同步到 AOF 文件中，在系统突然宕机的情况下丢失 1 秒内的数据 数据**准确性较高，性能较高**，建议使用，也是默认配置


- no（系统控制）：由操作系统控制每次同步到 AOF 文件的周期，整体过程**不可控**

**AOF 缓冲区同步文件策略**，系统调用 write 和 fsync：

* write 操作会触发延迟写（delayed write）机制，Linux 在内核提供页缓冲区用来提高硬盘 IO 性能，write 操作在写入系统缓冲区后直接返回
* 同步硬盘操作依赖于系统调度机制，比如缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失
* fsync 针对单个文件操作（比如 AOF 文件）做强制硬盘同步，fsync 将阻塞到写入硬盘完成后返回，保证了数据持久化

异常恢复：AOF 文件损坏，通过 redis-check-aof--fix appendonly.aof 进行恢复，重启 Redis，然后重新加载



***



#### 重写

##### 介绍

随着命令不断写入 AOF，文件会越来越大，为了解决这个问题 Redis 引入了 AOF 重写机制压缩文件体积

AOF 重写：将 Redis 进程内的数据转化为写命令同步到**新** AOF 文件的过程，简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录

AOF 重写作用：

- 降低磁盘占用量，提高磁盘利用率
- 提高持久化效率，降低持久化写时间，提高 IO 性能
- 降低数据恢复的用时，提高数据恢复效率

AOF 重写规则：

- 进程内具有时效性的数据，并且数据已超时将不再写入文件 


- 非写入类的无效指令将被忽略，只保留最终数据的写入命令

  如 del key1、 hdel key2、srem key3、set key4 111、set key4 222等，select 指令虽然不更改数据，但是更改了数据的存储位置，此类命令同样需要记录

- 对同一数据的多条写命令合并为一条命令

  如 lpushlist1 a、lpush list1 b、lpush list1 c 可以转化为：lpush list1 a b c。

  为防止数据量过大造成客户端缓冲区溢出，对 list、set、hash、zset 等类型，每条指令最多写入 64 个元素





***



##### 方式

* 手动重写

  ```sh
  bgrewriteaof
  ```

  原理分析：

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-AOF手动重写原理.png)

* 自动重写

  触发时机：Redis 会记录上次重写时的 AOF 大小，默认配置是当 AOF 文件大小是上次重写后大小的一倍且文件大于 64M 时触发

  ```sh
  auto-aof-rewrite-min-size size		#设置重写的基准值，最小文件 64MB，达到这个值开始重写
  auto-aof-rewrite-percentage percent	#触发AOF文件执行重写的增长率，当前AOF文件大小超过上一次重写的AOF文件大小的百分之多少才会重写，比如文件达到 100% 时开始重写就是两倍时触发
  ```
  
  自动重写触发比对参数（ 运行指令 `info Persistence` 获取具体信息 ）：
  
  ```sh
  aof_current_size					#AOF文件当前尺寸大小（单位:字节）
  aof_base_size						#AOF文件上次启动和重写时的尺寸大小（单位:字节）
  ```

  自动重写触发条件公式：
  
  - aof_current_size > auto-aof-rewrite-min-size
  - (aof_current_size - aof_base_size) / aof_base_size >= auto-aof-rewrite-percentage



***





#### 流程

持久化流程：

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-AOF重写流程1.png)

重写流程：

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-AOF重写流程2.png)

使用**新的 AOF 文件覆盖旧的 AOF 文件**，完成 AOF 重写



****



### 对比

AOF 和 RDB 同时开启，系统默认取 AOF 的数据（数据不会存在丢失）

 RDB 与 AOF 对比：

| 持久化方式   | RDB                | AOF                |
| ------------ | ------------------ | ------------------ |
| 占用存储空间 | 小（数据级：压缩） | 大（指令级：重写） |
| **存储速度** | 慢                 | 快                 |
| **恢复速度** | 快                 | 慢                 |
| 数据安全性   | 会丢失数据         | 依据策略决定       |
| 资源消耗     | 高/重量级          | 低/轻量级          |
| 启动优先级   | 低                 | 高                 |

应用场景：

- 对数据**非常敏感**，建议使用默认的 AOF 持久化方案

  AOF 持久化策略使用 everysecond，每秒钟 fsync 一次，该策略 redis 仍可以保持很好的处理性能，当出现问题时，最多丢失 1 秒内的数据

  注意：AOF 文件存储体积较大，恢复速度较慢，因为要执行每条指令

- 数据呈现**阶段有效性**，建议使用 RDB 持久化方案

  数据可以良好的做到阶段内无丢失，且恢复速度较快，阶段内数据恢复通常采用 RDB 方案

  注意：利用 RDB 实现紧凑的数据持久化，存储数据量较大时，存储效率较低

综合对比：

- RDB 与 AOF 的选择实际上是在做一种权衡，每种都有利有弊
- 如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用 AOF
- 如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用 RDB
- 灾难恢复选用 RDB
- 双保险策略，同时开启 RDB 和 AOF，重启后 Redis 优先使用 AOF 来恢复数据，降低丢失数据的量
- 不建议单独用 AOF，因为可能会出现 Bug，如果只是做纯内存缓存，可以都不用



***



### fork

#### 介绍

fork() 函数创建一个子进程，子进程与父进程几乎是完全相同的进程，系统先给子进程分配资源，然后把原来的进程的所有数据都复制到子进程中，只有少数值与父进程的值不同，相当于克隆了一个进程

在完成对其调用之后，会产生 2 个进程，且每个进程都会**从 fork() 的返回处开始执行**，这两个进程将执行相同的程序段，但是拥有各自不同的堆段，栈段，数据段，每个子进程都可修改各自的数据段，堆段，和栈段

```c
#include<unistd.h>pid_t fork(void);// 父进程返回子进程的pid，子进程返回0，错误返回负值，根据返回值的不同进行对应的逻辑处理
```

fork 调用一次，却能够**返回两次**，可能有三种不同的返回值：

* 在父进程中，fork 返回新创建子进程的进程 ID
* 在子进程中，fork 返回 0
* 如果出现错误，fork 返回一个负值，错误原因：
  * 当前的进程数已经达到了系统规定的上限，这时 errno 的值被设置为 EAGAIN
  * 系统内存不足，这时 errno 的值被设置为 ENOMEM

fpid 的值在父子进程中不同：进程形成了链表，父进程的 fpid 指向子进程的进程 id，因为子进程没有子进程，所以其 fpid 为0

创建新进程成功后，系统中出现两个基本完全相同的进程，这两个进程执行没有固定的先后顺序，哪个进程先执行要看系统的调度策略

每个进程都有一个独特（互不相同）的进程标识符 process ID，可以通过 getpid() 函数获得；还有一个记录父进程 pid 的变量，可以通过 getppid() 函数获得变量的值



***



#### 使用

基本使用：

```c
#include <unistd.h>  
#include <stdio.h>   
int main ()   {       
    pid_t fpid; // fpid表示fork函数返回的值      
    int count = 0;      
    fpid = fork();       
    if (fpid < 0)           
        printf("error in fork!");       
    else if (fpid == 0) {          
        printf("i am the child process, my process id is %d/n", getpid());            
        count++;      
    }      
    else {          
        printf("i am the parent process, my process id is %d/n", getpid());           
        count++;      
    }      
    printf("count: %d/n",count);// 1      
    return 0;  
}  
/*输出内容：    
i am the child process, my process id is 5574    
count: 1    
i am the parent process, my process id is 5573    
count: 1
*/
```

进阶使用：

```c
#include <unistd.h>  #include <stdio.h>  int main(void)  {     int i = 0;     // ppid 指当前进程的父进程pid     // pid 指当前进程的pid,     // fpid 指fork返回给当前进程的值，在这可以表示子进程   for(i = 0; i < 2; i++){         pid_t fpid = fork();         if(fpid == 0)             printf("%d child  %4d %4d %4d/n",i, getppid(), getpid(), fpid);         else             printf("%d parent %4d %4d %4d/n",i, getppid(), getpid(),fpid);     }     return 0;  } /*输出内容：	i        父id  id  子id	0 parent 2043 3224 3225    0 child  3224 3225    0    1 parent 2043 3224 3226    1 parent 3224 3225 3227    1 child     1 3227    0    1 child     1 3226    0 */
```

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-fork函数使用演示.png )

在 p3224 和 p3225 执行完第二个循环后，main 函数退出，进程死亡。所以 p3226，p3227 就没有父进程了，成为孤儿进程，所以 p3226 和 p3227 的父进程就被置为 ID 为 1的 init 进程（笔记 Tool → Linux → 进程管理详解）

参考文章：https://blog.csdn.net/love_gaohz/article/details/41727415



***



#### 内存

fork() 调用之后父子进程的内存关系

早期 Linux 的 fork() 实现时，就是全部复制，这种方法效率太低，而且造成了很大的内存浪费，现在 Linux 实现采用了两种方法：

* 父子进程的代码段是相同的，所以代码段是没必要复制的，只需内核将代码段标记为只读，父子进程就共享此代码段。fork() 之后在进程创建代码段时，子进程的进程级页表项都指向和父进程相同的物理页帧

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-fork以后内存关系1.png )

* 对于父进程的数据段，堆段，栈段中的各页，由于父子进程要相互独立，采用**写时复制**的技术，来提高内存以及内核的利用率

  在 fork 之后两个进程用的是相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间，**两者的虚拟空间不同，但其对应的物理空间是同一个**。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间，如果两者的代码完全相同，代码段继续共享父进程的物理空间；而如果两者执行的代码不同，子进程的代码段也会分配单独的物理空间。   

  fork 之后内核会将子进程放在队列的前面，让子进程先执行，以免父进程执行导致写时复制，而后子进程再执行，因无意义的复制而造成效率的下降

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-fork以后内存关系2.png )

补充知识：

vfork（虚拟内存 fork virtual memory fork）：调用 vfork() 父进程被挂起，子进程使用父进程的地址空间。不采用写时复制，如果子进程修改父地址空间的任何页面，这些修改过的页面对于恢复的父进程是可见的



参考文章：https://blog.csdn.net/Shreck66/article/details/47039937





****



## 事务机制

### 基本操作

Redis 事务的主要作用就是串联多个命令防止别的命令插队

* 开启事务

  ```sh
  multi	#设定事务的开启位置，此指令执行后，后续的所有指令均加入到事务中
  ```

* 执行事务

  ```sh
  exec	#设定事务的结束位置，同时执行事务，与multi成对出现，成对使用
  ```

  加入事务的命令暂时进入到任务队列中，并没有立即执行，只有执行 exec 命令才开始执行

* 取消事务

  ```sh
  discard	#终止当前事务的定义，发生在multi之后，exec之前
  ```

  一般用于事务执行过程中输入了错误的指令，直接取消这次事务，类似于回滚

Redis 事务的三大特性：

* Redis 事务是一个单独的隔离操作，将一系列预定义命令包装成一个整体（一个队列），当执行时按照添加顺序依次执行，中间不会被打断或者干扰
* Redis 事务**没有隔离级别**的概念，队列中的命令在事务没有提交之前都不会实际被执行
* Redis 单条命令式保存原子性的，但是事务**不保证原子性**，事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚



***



### 工作流程

事务机制整体工作流程：

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-事务的工作流程.png)

几种常见错误：

* 定义事务的过程中，命令格式输入错误，出现语法错误造成，**整体事务中所有命令均不会执行**，包括那些语法正确的命令

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-事务中出现语法错误.png )

* 定义事务的过程中，命令执行出现错误，例如对字符串进行 incr 操作，能够正确运行的命令会执行，运行错误的命令不会被执行

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-事务中出现执行错误.png )

* 已经执行完毕的命令对应的数据不会自动回滚，需要程序员在代码中实现回滚，应该尽可能避免：

  事务操作之前记录数据的状态

  * 单数据：string
  * 多数据：hash、list、set、zset

  设置指令恢复所有的被修改的项

  * 单数据：直接 set（注意周边属性，例如时效）
  * 多数据：修改对应值或整体克隆复制



***



### 监控锁

对 key 添加监视锁，是一种乐观锁，在执行 exec 前如果其他客户端的操作导致 key 发生了变化，执行结果为 nil

* 添加监控锁

  ```sh
  watch key1 [key2……]	#可以监控一个或者多个key
  ```

* 取消对所有 key 的监视

  ```sh
  unwatch
  ```

应用：基于状态控制的批量任务执行，防止其他线程对变量的修改

悲观锁：很悲观，认为什么时候都会出问题，无论什么都会加锁。影响效率，实际情况一般会使用乐观锁。

乐观锁：很乐观，认为什么时候都不会出现问题，所以不上锁。更新数据的时候会判断一下，在此期间是否修改过监视的数据。

首先要了解redis事务中watch的作用，watch命令可以监控一个或多个key，一旦其中有一个key被修改（或删除），之后的事务就不会执行。监控一直持续到exec命令（事务中的命令是在exec之后才执行的，所以在multi命令后可以修改watch监控的键值）。假设我们通过watch命令在事务执行之前监控了多个Keys，倘若在watch之后有任何Key的值发生了变化，exec命令执行的事务都将被放弃，同时返回Null multi-bulk应答以通知调用者事务执行失败。

所以，需要注意的是watch监控键之后，再去操作这些键，否则watch可能会起不到效果。

> Redis 监视测试

正常测试：

```bash
127.0.0.1:6379> set money 100		
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money		# 监视 money 对象
OK
127.0.0.1:6379> multi		# 事务正常结束，执行期间，money 没有变动，这个时候就能执行成功了
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20
127.0.0.1:6379> 
```

测试多线程修改值，使用 watch 可以当做 Redis 的乐观锁操作。

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 10
OK
127.0.0.1:6379> watch money	# 监视 money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 10
QUEUED
127.0.0.1:6379> DECRBY out 10
QUEUED
127.0.0.1:6379> exec		# 执行之前，在另外一个线程 B 中修改 money 的值，下面就是执行失败。
(nil)
127.0.0.1:6379> 
```

B 线程：

```bash
[root@coder bin]# redis-cli -p 6379
127.0.0.1:6379> set money 30
OK
```

如果修改失败，获取最新的值就好。

```bash
127.0.0.1:6379> UNWATCH		# 事务执行失败，先解锁
OK
127.0.0.1:6379> WATCH money		# 获取最新的值，再次监视。相当于 MySQL 中的 select version
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 1
QUEUED
127.0.0.1:6379> INCRBY out 1
QUEUED
127.0.0.1:6379> exec		# 执行的时候会对比监视的值，如果发生变化会执行失败。
1) (integer) 29
2) (integer) 11
127.0.0.1:6379> 
```

## 



***



### 分布式锁

#### 基本操作

由于分布式系统多线程并发分布在不同机器上，这将使单机部署情况下的并发控制锁策略失效，需要分布式锁

Redis 分布式锁的基本使用，悲观锁

* 使用 setnx 设置一个公共锁

  ```sh
  setnx lock-key value	# value任意数，返回为1设置成功，返回为0设置失败
  ```

  * 对于返回设置成功的，拥有控制权，进行下一步的具体业务操作
  * 对于返回设置失败的，不具有控制权，排队或等待

  `NX`：只在键不存在时，才对键进行设置操作，`SET key value NX` 效果等同于 `SETNX key value`

  `XX` ：只在键已经存在时，才对键进行设置操作

  `EX`：设置键 key 的过期时间，单位时秒

  `PX`：设置键 key 的过期时间，单位时毫秒

  说明：由于 `SET` 命令加上选项已经可以完全取代 SETNX、SETEX、PSETEX 的功能，Redis 不推荐使用这几个命令

* 操作完毕通过 del 操作释放锁

  ```sh
  del lock-key 
  ```

* 使用 expire 为锁 key 添加存活（持有）时间，过期自动删除（放弃）锁

  ```sh
  expire lock-key second pexpire lock-key milliseconds
  ```

  通过 expire 设置过期时间缺乏原子性，如果在 setnx 和 expire 之间出现异常，锁也无法释放

* 在 set 时指定过期时间

  ```sh
  SET key value [EX seconds | PX milliseconds] NX
  ```

应用：解决抢购时出现超卖现象



****



#### 防误删

setnx 获取锁时，设置一个指定的唯一值（uuid），释放前获取这个值，判断是否自己的锁，防止出现线程之间误删了其他线程的锁

```java
// 加锁, unique_value作为客户端唯一性的标识SET 
lock_key unique_value NX PX 10000
```

unique_value 是客户端的唯一标识，可以用一个随机生成的字符串来表示，PX 10000 则表示 lock_key 会在 10s 后过期，以免客户端在这期间发生异常而无法释放锁



***



## 删除策略

### 过期数据

Redis 是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过 TTL 指令获取其状态

TTL 返回的值有三种情况：正数，-1，-2

- 正数：代表该数据在内存中还能存活的时间
- -1：永久有效的数据
- 2 ：已经过期的数据或被删除的数据或未定义的数据

删除策略：**删除策略就是针对已过期数据的处理策略**，已过期的数据不一定被立即删除，在不同的场景下使用不同的删除方式会有不同效果，这就是删除策略的问题

过期数据是一块独立的存储空间，Hash 结构，field 是内存地址，value 是过期时间，保存了所有 key 的过期描述，在最终进行过期处理的时候，对该空间的数据进行检测， 当时间到期之后通过 field 找到内存该地址处的数据，然后进行相关操作

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-时效性数据的存储结构.png )



****



### 数据删除

#### 删除策略

在内存占用与 CPU 占用之间寻找一种平衡，顾此失彼都会造成整体 Redis 性能的下降，甚至引发服务器宕机或内存泄露

针对过期数据有三种删除策略：

- 定时删除
- 惰性删除
- 定期删除



***



#### 定时删除

创建一个定时器，当 key 设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作

- 优点：节约内存，到时就删除，快速释放掉不必要的内存占用
- 缺点：无论 CPU 此时负载量多高，均占用 CPU，会影响 Redis 服务器响应时间和指令吞吐量
- 总结：用处理器性能换取存储空间（拿时间换空间）



***



#### 惰性删除

数据到达过期时间，不做处理，等下次访问该数据时，需要判断：

* 如果未过期，返回数据
* 如果已过期，删除，返回不存在

在任何 get 操作之前都要执行 **expireIfNeeded()**，相当于绑定在一起

特点：

* 优点：节约 CPU 性能，发现必须删除的时候才删除
* 缺点：内存压力很大，出现长期占用内存的数据
* 总结：用存储空间换取处理器性能（拿空间换时间）



***



#### 定期删除 

定时删除和惰性删除这两种方案都是走的极端，定期删除就是折中方案

定期删除是周期性轮询 Redis 库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度

定期删除方案：

- Redis 启动服务器初始化时，读取配置 server.hz 的值，默认为 10，执行指令 info server 可以查看

- 每秒钟执行 server.hz 次 serverCron() → databasesCron() → activeExpireCycle() 

- databasesCron() 操作是**轮询每个数据库**

- activeExpireCycle() 对某个数据库中的每个 expires 进行检测，每次执行耗时：250ms/server.hz

  对某个 expires[*] 检测时，随机挑选 W 个 key 检测

  - 如果 key 超时，删除 key
  - 如果一轮中删除的 key 的数量 > W*25%，循环该过程
  - 如果一轮中删除的 key 的数量 ≤ W*25%，检查下一个expires[]，0-15 循环
  - W 取值 = ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP 属性值，自定义值

* 参数 current_db 用于记录 activeExpireCycle() 进入哪个expires[*] 执行
* 如果 activeExpireCycle() 执行时间到期，下次从 current_db 继续向下执行

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-定期删除.png )

定期删除特点：

- CPU 性能占用设置有峰值，检测频度可自定义设置
- 内存压力不是很大，长期占用内存的冷数据会被持续清理
- 周期性抽查存储空间（随机抽查，重点抽查）



***



#### 策略对比

|          | 优点             | 缺点                          | 特点               |
| -------- | ---------------- | ----------------------------- | ------------------ |
| 定时删除 | 节约内存，无占用 | 不分时段占用CPU资源，频度高   | 拿时间换空间       |
| 惰性删除 | 内存占用严重     | 延时执行，CPU利用率高         | 拿空间换时间       |
| 定期删除 | 内存定期随机清理 | 每秒花费固定的CPU资源维护内存 | 随机抽查，重点抽查 |



***



### 数据淘汰

#### 逐出算法

数据淘汰策略：当新数据进入 Redis 时，在执行每一个命令前，会调用 **freeMemoryIfNeeded()** 检测内存是否充足。如果内存不满足新加入数据的最低存储要求，Redis 要临时删除一些数据为当前指令清理存储空间，清理数据的策略称为**逐出算法**

逐出数据的过程不是 100% 能够清理出足够的可使用的内存空间，如果不成功则反复执行，当对所有数据尝试完毕，如不能达到内存清理的要求，**出现 Redis 内存打满异常**：

```sh
(error) OOM command not allowed when used memory >'maxmemory'
```



****



#### 策略配置

Redis 如果不设置最大内存大小或者设置最大内存大小为 0，在 64 位操作系统下不限制内存大小，在 32 位操作系统默认为 3GB 内存，一般推荐设置 Redis 内存为最大物理内存的四分之三

内存配置方式：

* 通过修改文件配置（永久生效）：修改配置文件 maxmemory 字段，单位为字节

* 通过命令修改（重启失效）：

  * `config set maxmemory 104857600`：设置 Redis 最大占用内存为 100MB
  * `config get maxmemory`：获取 Redis 最大占用内存

  * `info` ：可以查看 Redis 内存使用情况，`used_memory_human` 字段表示实际已经占用的内存，`maxmemory` 表示最大占用内存

影响数据淘汰的相关配置如下，配置 conf 文件：

* 每次选取待删除数据的个数，采用随机获取数据的方式作为待检测删除数据，防止全库扫描，导致严重的性能消耗，降低读写性能

  ```sh
  maxmemory-samples count
  ```

* 达到最大内存后的，对被挑选出来的数据进行删除的策略

  ```sh
  maxmemory-policy policy
  ```

  数据删除的策略 policy：3 类 8 种

  第一类：检测易失数据（可能会过期的数据集 server.db[i].expires）：

  ```sh
  volatile-lru	# 对设置了过期时间的 key 选择最近最久未使用使用的数据淘汰
  volatile-lfu	# 对设置了过期时间的 key 选择最近使用次数最少的数据淘汰
  volatile-ttl	# 对设置了过期时间的 key 选择将要过期的数据淘汰
  volatile-random	# 对设置了过期时间的 key 选择任意数据淘汰
  ```
  
  第二类：检测全库数据（所有数据集 server.db[i].dict ）：
  
  ```sh
  allkeys-lru		# 对所有 key 选择最近最少使用的数据淘汰
  allkeLyRs-lfu	# 对所有 key 选择最近使用次数最少的数据淘汰
  allkeys-random	# 对所有 key 选择任意数据淘汰，相当于随机
  ```
  
  第三类：放弃数据驱逐
  
  ```sh
  no-enviction	#禁止驱逐数据(redis4.0中默认策略)，会引发OOM(Out Of Memory)
  ```

数据淘汰策略配置依据：使用 INFO 命令输出监控信息，查询缓存 hit 和 miss 的次数，根据需求调优 Redis 配置





***



## 主从复制

### 基本介绍

**三高**架构：

- 高并发：应用提供某一业务要能支持很多客户端同时访问的能力，我们称为并发

- 高性能：性能带给我们最直观的感受就是：速度快，时间短

- 高可用：
  - 可用性：应用服务在全年宕机的时间加在一起就是全年应用服务不可用的时间
  - 业界可用性目标 5 个 9，即 99.999%，即服务器年宕机时长低于 315 秒，约 5.25 分钟

主从复制：

* 概念：将 master 中的数据即时、有效的复制到 slave 中

* 特征：一个 master 可以拥有多个 slave，一个 slave 只对应一个 master

* 职责：master 和 slave 各自的职责不一样

  master：

  * **写数据**，执行写操作时，将出现变化的数据自动同步到 slave
  * 读数据（可忽略）

  slave

  * **读数据**
  * 写数据（禁止）

主从复制的机制：

* **薪火相传**：一个 slave 可以是下一个 slave 的 master，slave 同样可以接收其他 slave 的连接和同步请求，那么该 slave 作为了链条中下一个的 master, 可以有效减轻 master 的写压力，去中心化降低风险

  注意：主机挂了，从机还是从机，无法写数据了

* **反客为主**：当一个 master 宕机后，后面的 slave 可以立刻升为 master，其后面的 slave 不做任何修改

  将从机变为主机的命令：`slaveof no one`

主从复制的作用：

- **读写分离**：master 写、slave 读，提高服务器的读写负载能力
- **负载均衡**：基于主从结构，配合读写分离，由 slave 分担 master 负载，并根据需求的变化，改变 slave 的数量，通过多个从节点分担数据读取负载，大大提高 Redis 服务器并发量与数据吞吐量
- 故障恢复：当 master 出现问题时，由 slave 提供服务，实现快速的故障恢复
- 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式
- 高可用基石：基于主从复制，构建哨兵模式与集群，实现 Redis 的高可用方案

主从复制的应用场景：

* 机器故障：硬盘故障、系统崩溃，造成数据丢失，对业务形成灾难性打击，基本上会放弃使用redis

* 容量瓶颈：内存不足，放弃使用 redis

* 解决方案：为了避免单点 Redis 服务器故障，准备多台服务器，互相连通。将数据复制多个副本保存在不同的服务器上连接在一起，并保证数据是同步的。即使有其中一台服务器宕机，其他服务器依然可以继续提供服务，实现 Redis 高可用，同时实现数据冗余备份

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-主从复制多台服务器连接方案.png )



***



### 工作流程

主从复制过程大体可以分为3个阶段

* 建立连接阶段（即准备阶段）
* 数据同步阶段
* 命令传播阶段

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-主从复制工作流程.png)



***



### 建立连接

#### 建立流程

建立连接阶段：建立 slave 到 master 的连接，使 master 能够识别 slave，并保存 slave 端口号

流程如下：

1. 设置 master 的地址和端口，保存 master 信息
2. 建立 socket 连接
3. 发送 ping 命令（定时器任务）
4. 身份验证（可能没有）
5. 发送 slave 端口信息
6. 主从连接成功

连接成功的状态：

* slave：保存 master 的地址与端口

* master：保存 slave 的端口

* 主从之间创建了连接的 socket

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-主从复制建立连接.png )



***



#### 相关指令

* master 和 slave 互联

  方式一：客户端发送命令

  ```sh
  slaveof masterip masterport
  ```

  方式二：服务器带参启动

  ```sh
  redis-server --slaveof masterip masterport
  ```

  方式三：服务器配置（主流方式）

  ```sh
  slaveof masterip masterport
  ```

  * slave 系统信息：info 指令

    ```sh
    master_link_down_since_secondsmasterhost & masterport
    ```

  * master 系统信息：

    ```sh
    uslave_listening_port(多个)
    ```

  * 系统信息：

    ```sh
    info replication
    ```

* 主从断开连接：断开 slave 与 master 的连接，slave 断开连接后，不会删除已有数据，只是不再接受 master 发送的数据

  slave客户端执行命令：

  ```sh
  slaveof no one	
  ```

* 授权访问：master 有服务端和客户端，slave 也有服务端和客户端，不仅服务端之间可以发命令，客户端也可以

  master 客户端发送命令设置密码：

  ```sh
  requirepass password
  ```

  master 配置文件设置密码：

  ```sh
  config set requirepass passwordconfig get requirepass
  ```

  slave 客户端发送命令设置密码：

  ```sh
  auth password
  ```

  slave 配置文件设置密码：

  ```sh
  masterauth password
  ```

  slave 启动服务器设置密码：

  ```sh
  redis-server –a password
  ```

  

***



### 数据同步

#### 同步流程

数据同步需求：

- 在 slave 初次连接 master 后，复制 master 中的所有数据到 slave
- 将 slave 的数据库状态更新成 master 当前的数据库状态

同步过程如下：

1. 请求同步数据
2. 创建 RDB 同步数据
3. 恢复 RDB 同步数据（从服务器会**清空原有数据**）
4. 请求部分同步数据
5. 恢复部分同步数据
6. 数据同步工作完成

同步完成的状态：

* slave：具有 master 端全部数据，包含 RDB 过程接收的数据

* master：保存 slave 当前数据同步的位置

* 主从之间完成了数据克隆

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-主从复制数据同步.png )



***



#### 同步优化

* 数据同步阶段 master 说明

  1. master 数据量巨大，数据同步阶段应避开流量高峰期，避免造成 master 阻塞，影响业务正常执行

  2. 复制缓冲区大小设定不合理，会导致**数据溢出**。比如进行全量复制周期太长，进行部分复制时发现数据已经存在丢失的情况，必须进行第二次全量复制，致使 slave 陷入死循环状态

     ```sh
     repl-backlog-size ?mb
     ```

     建议设置如下：

     * 测算从 master 到 slave 的重连平均时长 second
     * 获取 master 平均每秒产生写命令数据总量 write_size_per_second
     * 最优复制缓冲区空间 = 2 * second * write_size_per_second

  3. master 单机内存占用主机内存的比例不应过大，建议使用 50%-70% 的内存，留下 30%-50% 的内存用于执行 bgsave 命令和创建复制缓冲区

* 数据同步阶段 slave 说明

  1. 为避免 slave 进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的对外服务

     ```sh
     slave-serve-stale-data yes|no
     ```

  2. 数据同步阶段，master 发给 slave 信息可以理解 master是 slave 的一个客户端，主动向 slave 发送命令

  3. 多个 slave 同时对 master 请求数据同步，master 发送的 RDB 文件增多，会对带宽造成巨大冲击，如果 master 带宽不足，因此数据同步需要根据业务需求，适量错峰

  4. slave 过多时，建议调整拓扑结构，由一主多从结构变为树状结构，中间的节点既是 master，也是 slave。注意使用树状结构时，由于层级深度，导致深度越高的 slave 与最顶层 master 间数据同步延迟较大，数据一致性变差，应谨慎选择



***



### 命令传播

#### 传播原理

命令传播：当 master 数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的状态，同步的动作称为命令传播

命令传播的过程：master 将接收到的数据变更命令发送给 slave，slave 接收命令后执行命令

命令传播阶段出现了断网现象：

* 网络闪断闪连：忽略
* 短时间网络中断：部分复制
* 长时间网络中断：全量复制

部分复制的三个核心要素：服务器的运行 id（run id）、主服务器的复制积压缓冲区、主从服务器的复制偏移量

* 服务器运行ID（runid）：服务器运行 ID 是每一台服务器每次运行的身份识别码，一台服务器多次运行可以生成多个运行 ID，由 40 位字符组成，是一个随机的十六进制字符

  作用：用于在服务器间进行传输识别身份，如果想两次操作均对同一台服务器进行，必须每次操作携带对应的运行 ID，用于对方识别

  实现：运行 ID 在每台服务器启动时自动生成，master 在首次连接 slave 时，将运行 ID 发送给 slave，slave 保存此 ID，通过 info Server 命令，可以查看节点的 runid

* 复制缓冲区：复制积压缓冲区，是一个先进先出（FIFO）的队列，用于存储服务器执行过的命令

  作用：用于保存 master 收到的所有指令（仅影响数据变更的指令，例如 set，select）

  实现方式：每次传播命令，master 都会将传播的命令记录下来，并存储在复制缓冲区，复制缓冲区默认数据存储空间大小是 1M，当入队元素的数量大于队列长度时，最先入队的元素被弹出，新元素会被放入队列

* 复制偏移量：一个数字，描述复制缓冲区中的指令字节位置

  - master 复制偏移量：记录发送给所有 slave 的指令字节对应的位置（多个）
  - slave 复制偏移量：记录 slave 接收 master 发送过来的指令字节对应的位置（一个）

  作用：同步信息，比对 master 与 slave 的差异，当 slave 断线后，恢复数据使用

  数据来源：

  - master 端：发送一次记录一次
  - slave 端：接收一次记录一次

**工作原理**：

- 通过 offset 区分不同的 slave 当前数据传播的差异
- master 记录已发送的信息对应的 offset
- slave 记录已接收的信息对应的 offset

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-主从复制复制缓冲区原理.png )



****



#### 复制流程

全量复制/部分复制

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-主从复制流程更新.png)





***



#### 心跳机制

心跳机制：进入命令传播阶段，master 与 slave 间需要信息交换，使用心跳机制维护，实现双方连接保持在线

master 心跳任务：

- 内部指令：PING
- 周期：由 `repl-ping-slave-period` 决定，默认10秒
- 作用：判断 slave 是否在线
- 查询：INFO replication  获取 slave 最后一次连接时间间隔，lag 项维持在 0 或 1 视为正常

slave 心跳任务

- 内部指令：REPLCONF ACK {offset}
- 周期：1秒
- 作用：汇报 slave 自己的复制偏移量，获取最新的数据变更指令；判断 master 是否在线

心跳阶段注意事项：

* 当 slave 多数掉线，或延迟过高时，master 为保障数据稳定性，将拒绝所有信息同步

  slave 数量少于 2 个，或者所有 slave 的延迟都大于等于 8 秒时，强制关闭 master 写功能，停止数据同步

  ```sh
  min-slaves-to-write 2min-slaves-max-lag 8
  ```

* slave 数量由 slave 发送 REPLCONF ACK 命令做确认


- slave 延迟由 slave 发送 REPLCONF ACK 命令做确认



****



### 常见问题

#### 重启恢复

系统不断运行，master 的数据量会越来越大，一旦 master 重启，runid 将发生变化，会导致全部 slave 的全量复制操作

解决方法：本机保存上次 runid，重启后恢复该值，使所有 slave 认为还是之前的 master

优化方案：

* master 内部创建 master_replid 变量，使用 runid 相同的策略生成，长度41位，并发送给所有 slave

* 在master关闭时执行命令 `shutdown save`，进行RDB持久化，将 runid 与 offset 保存到RDB文件中

  `redis-check-rdb dump.rdb` 命令可以查看该信息，保存为 repl-id 和 repl-offset

* master 重启后加载 RDB 文件，恢复数据

  重启后，将RDB文件中保存的 repl-id 与 repl-offset 加载到内存中

  * master_repl_id = repl-id，master_repl_offset = repl-offset
  * 通过 info 命令可以查看该信息

 

***



#### 网络中断

master 的 CPU 占用过高或 slave 频繁断开连接

* 出现的原因：

  * slave 每1秒发送 REPLCONF ACK 命令到 master
  * 当 slave 接到了慢查询时（keys * ，hgetall等），会大量占用 CPU 性能
  * master 每1秒调用复制定时函数 replicationCron()，比对 slave 发现长时间没有进行响应

  最终导致 master 各种资源（输出缓冲区、带宽、连接等）被严重占用

* 解决方法：通过设置合理的超时时间，确认是否释放 slave

  ```sh
  repl-timeout	# 该参数定义了超时时间的阈值（默认60秒），超过该值，释放slave
  ```

slave 与 master 连接断开

* 出现的原因：

  * master 发送 ping 指令频度较低
  * master 设定超时时间较短
  * ping 指令在网络中存在丢包

* 解决方法：提高 ping 指令发送的频度

  ```sh
  repl-ping-slave-period	
  ```

  超时时间 repl-time 的时间至少是 ping 指令频度的5到10倍，否则 slave 很容易判定超时



****



#### 一致性

网络信息不同步，数据发送有延迟，导致多个 slave 获取相同数据不同步

解决方案：

* **优化主从间的网络环境**，通常放置在同一个机房部署，如使用阿里云等云服务器时要注意此现象

* 监控主从节点延迟（通过offset）判断，如果 slave 延迟过大，**暂时屏蔽程序对该 slave 的数据访问**

  ```sh
  slave-serve-stale-data yes|no
  ```

  开启后仅响应 info、slaveof 等少数命令（慎用，除非对数据一致性要求很高）





***



## 哨兵模式

### 哨兵概述

如果 Redis 的 master 宕机了，需要从 slave 中重新选出一个 master，要实现这些功能就需要 Redis 的哨兵

哨兵（sentinel）是一个分布式系统，用于对主从结构中的每台服务器进行**监控**，当出现故障时通过**投票机制选择**新的 master 并将所有 slave 连接到新的 master

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-哨兵模式.png )

哨兵的作用：

- 监控：监控 master 和 slave，不断的检查 master 和 slave 是否正常运行，master 存活检测、master 与 slave 运行情况检测

- 通知：当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知


- 自动故障转移：断开 master 与 slave 连接，选取一个 slave 作为 master，将其他 slave 连接新的 master，并告知客户端新的服务器地址

注意：哨兵也是一台 Redis 服务器，只是不提供数据相关服务，通常哨兵的数量配置为单数（投票）



***



### 启用哨兵

配置哨兵：

* 配置一拖二的主从结构

* 配置三个哨兵（配置相同，端口不同），sentinel.conf

  ```sh
  port 26401dir "/redis/data"sentinel monitor mymaster 127.0.0.1 6401 2sentinel down-after-milliseconds mymaster 5000sentinel failover-timeout mymaster 20000sentinel parallel-sync mymaster 1sentinel deny-scripts-reconfig yes
  ```

  配置说明：

  * 设置哨兵监听的主服务器信息， sentinel_number 表示参与投票的哨兵数量

    ```sh
    sentinel monitor master_name master_host master_port sentinel_number
    ```

  * 指定哨兵在监控 Redis 服务时，设置判定服务器宕机的时长，该设置控制是否进行主从切换

    ```sh
    sentinel down-after-milliseconds master_name million_seconds
    ```

  * 出现故障后，故障切换的最大超时时间，超过该值，认定切换失败，默认3分钟

    ```sh
    sentinel failover-timeout master_name	million_seconds
    ```

  * 指定同时进行主从的 slave 数量，数值越大，要求网络资源越高，要求约小，同步时间约长

    ```sh
    sentinel parallel-syncs master_name sync_slave_number
    ```

启动哨兵：

* 服务端命令（Linux 命令）：

  ```sh
  redis-sentinel filename
  ```



***



### 工作原理

#### 监控阶段

哨兵在进行主从切换过程中经历三个阶段

- 监控
- 通知
- 故障转移

监控阶段作用：同步各个节点的状态信息

* 获取各个 sentinel 的状态（是否在线）


- 获取 master 的状态

  ```markdown
  master属性	prunid	prole：master	各个slave的详细信息	
  ```

- 获取所有 slave 的状态（根据 master 中的 slave 信息）

  ```markdown
  slave属性	prunid	prole：slave	pmaster_host、master_port	poffset
  ```

内部的工作原理：

sentinel 1 首先连接 master，建立 cmd 通道，根据主节点访问从节点，连接完成

sentinel 2 首先连接 master，然后通过 master 中的 sentinels 发现其他哨兵，然后寻找哨兵建立连接，哨兵之间同步数据

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-哨兵模式监控工作原理.png )



***



#### 通知阶段

sentinel 在通知阶段不断的去获取 master/slave 的信息，然后在各个 sentinel 之间进行共享，流程如下：

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-哨兵模式通知工作流程.png)



***



#### 故障转移

当 master 宕机后，sentinel 会判断出 master 是否真的宕机，具体的操作流程：

* 检测 master

  sentinel1 检测到 master 下线后会做 flag:SRI_S_DOWN 标志，此时 master 的状态是主观下线，并通知其他哨兵，其他哨兵也会尝试与 master 连接，如果大于 (n/2) + 1 个sentinel 检测到 master 下线，就达成共识更改 flag，此时 master 的状态是客观下线

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-哨兵模式故障转移工作流程1.png)

* 当 sentinel 认定 master 下线之后，此时需要决定更换 master，选举某个 sentinel 处理事故

  在选举的时候每一个 sentinel 都有一票，于是每个 sentinel 都会发出一个指令，在内网广播要做主持人；比如 sentinel1 和 sentinel4 发出这个选举指令了，那么 sentinel2 既能接到 sentinel1 的也能接到 sentinel4 的，sentinel2 会把一票投给其中一方，投给指令最先到达的 sentinel。选举最终得票多的，就成为了处理事故的哨兵，需要注意在这个过程中有可能会存在失败的现象，就是一轮选举完没有选取，那就会接着进行第二轮第三轮直到完成选举。

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-哨兵模式故障转移工作流程2.png)

选择新的 master，在服务器列表中挑选备选 master 的原则：

- 不在线的 OUT

- 响应慢的 OUT
- 与原 master 断开时间久的 OUT

- 优先原则：先根据优先级 → offset → runid

选出新的 master之后，发送指令（sentinel ）给其他的 slave

* 向新的 master 发送 slaveof no one
* 向其他 slave 发送 slaveof 新 masterIP 端口





****



## 集群模式

### 集群概述

集群就是使用网络将若干台计算机联通起来，并提供统一的管理方式，使其对外呈现单机的服务效果

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-集群图示.png)

**集群作用：**

- 分散单台服务器的访问压力，实现负载均衡
- 分散单台服务器的存储压力，实现可扩展性
- 降低单台服务器宕机带来的业务灾难



***



### 结构设计

**数据存储设计：**

1. 通过算法设计，计算出 key 应该保存的位置（类似哈希寻址）

   ```markdown
   key -> CRC16(key) -> 值 -> %16384 -> 存储位置
   ```

2. 将所有的存储空间计划切割成 16384 份，每台主机保存一部分

   注意：每份代表的是一个存储空间，不是一个 key 的保存空间，可以存储多个 key

3. 将 key 按照计算出的结果放到对应的存储空间

   ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-集群存储空间.png )

查找数据：

- 各个数据库相互通信，保存各个库中槽的编号数据
- 一次命中，直接返回
- 一次未命中，告知具体位置，最多两次命中

设置数据：系统默认存储到某一个

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-集群查找数据.png )



***



### 结构搭建

整体框架：

- 配置服务器（3 主 3 从）
- 建立通信（Meet）
- 分槽（Slot）
- 搭建主从（master-slave）

创建集群 conf 配置文件：

* redis-6501.conf

  ```sh
  port 6501dir "/redis/data"dbfilename "dump-6501.rdb"cluster-enabled yescluster-config-file "cluster-6501.conf"cluster-node-timeout 5000#其他配置文件参照上面的修改端口即可，内容完全一样
  ```

* 服务端启动：

  ```sh
  redis-server config_file_name
  ```

* 客户端启动：

  ```sh
  redis-cli -p 6504 -c
  ```

**cluster 配置：**

- 是否启用 cluster，加入 cluster 节点

  ```properties
  cluster-enabled yes|no
  ```

- cluster 配置文件名，该文件属于自动生成，仅用于快速查找文件并查询文件内容

  ```properties
  cluster-config-file filename
  ```

- 节点服务响应超时时间，用于判定该节点是否下线或切换为从节点

  ```properties
  cluster-node-timeout milliseconds
  ```

- master 连接的 slave 最小数量

  ```properties
  cluster-migration-barrier min_slave_number
  ```

客户端启动命令：

**cluster 节点操作命令（客户端命令）：**

- 查看集群节点信息

  ```properties
  cluster nodes
  ```

- 更改 slave 指向新的 master

  ```properties
  cluster replicate master-id
  ```

- 发现一个新节点，新增 master

  ```properties
  cluster meet ip:port
  ```

- 忽略一个没有 solt 的节点

  ```properties
  cluster forget server_id
  ```

- 手动故障转移

  ```properties
  cluster failover
  ```

**集群操作命令（Linux）：**

* 创建集群

  ```properties
  redis-cli –-cluster create masterhost1:masterport1 masterhost2:masterport2  masterhost3:masterport3 [masterhostn:masterportn …] slavehost1:slaveport1  slavehost2:slaveport2 slavehost3:slaveport3 -–cluster-replicas n
  ```

  注意：master 与 slave 的数量要匹配，一个 master 对应 n 个 slave，由最后的参数 n 决定。master 与 slave 的匹配顺序为第一个 master 与前 n 个 slave 分为一组，形成主从结构


* 添加 master 到当前集群中，连接时可以指定任意现有节点地址与端口

  ```properties
  redis-cli --cluster add-node new-master-host:new-master-port now-host:now-port
  ```

* 添加 slave

  ```properties
  redis-cli --cluster add-node new-slave-host:new-slave-port master-host:master-port --cluster-slave --cluster-master-id masterid
  ```

* 删除节点，如果删除的节点是 master，必须保障其中没有槽 slot

  ```properties
  redis-cli --cluster del-node del-slave-host:del-slave-port del-slave-id
  ```

* 重新分槽，分槽是从具有槽的 master 中划分一部分给其他 master，过程中不创建新的槽

  ```properties
  redis-cli --cluster reshard new-master-host:new-master:port --cluster-from src-  master-id1, src-master-id2, src-master-idn --cluster-to target-master-id --  cluster-slots slots
  ```

  注意：将需要参与分槽的所有 masterid 不分先后顺序添加到参数中，使用 `,` 分隔，指定目标得到的槽的数量，所有的槽将平均从每个来源的 master 处获取


* 重新分配槽，从具有槽的 master 中分配指定数量的槽到另一个 master 中，常用于清空指定 master 中的槽

  ```properties
  redis-cli --cluster reshard src-master-host:src-master-port --cluster-from src-  master-id --cluster-to target-master-id --cluster-slots slots --cluster-yes
  ```

  



***



## 缓存方案

### 缓存模式

#### 旁路缓存

缓存本质：弥补 CPU 的高算力和 IO 的慢读写之间巨大的鸿沟

旁路缓存模式 Cache Aside Pattern 是平时使用比较多的一个缓存读写模式，比较适合读请求比较多的场景

Cache Aside Pattern 中服务端需要同时维系 DB 和 cache，并且是以 DB 的结果为准

* 写操作：<font color="red">先更新 DB，然后直接删除 cache</font>
* 读操作：从 cache 中读取数据，读取到就直接返回；读取不到就从 DB 中读取数据返回，并放到 cache 

时序导致的不一致问题：

* 在写数据的过程中，不能先删除 cache 再更新 DB，因为会造成缓存的不一致。比如请求 1 先写数据 A，请求 2 随后读数据 A，当请求 1 删除 cache 后，请求 2 直接读取了 DB，此时请求 1 还没写入 DB（延迟双删）

* 在写数据的过程中，先更新 DB 再删除 cache 也会出现问题，但是概率很小，因为缓存的写入速度非常快

旁路缓存的缺点：

* 首次请求数据一定不在 cache 的问题，一般采用缓存预热的方法，将热点数据可以提前放入 cache 中
* 写操作比较频繁的话导致 cache 中的数据会被频繁被删除，影响缓存命中率



****



#### 读写穿透

读写穿透模式 Read/Write Through Pattern：服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中，cache 负责将此数据同步写入 DB，从而减轻了应用程序的职责

* 写操作：<font color="red">先查 cache，cache 中不存在，直接更新 DB；cache 中存在则先更新 cache，然后 cache 服务更新 DB（同步更新 cache 和 DB）</font>

* 读操作：从 cache 中读取数据，读取到就直接返回 ；读取不到先从 DB 加载，写入到 cache 后返回响应

  Read-Through Pattern 实际只是在 Cache-Aside Pattern 之上进行了封装。在 Cache-Aside Pattern 下，发生读请求的时候，如果 cache 中不存在对应的数据，是由客户端负责把数据写入 cache，而 Read Through Pattern 则是 cache 服务自己来写入缓存的，对客户端是透明的

Read-Through Pattern 也存在首次不命中的问题，采用缓存预热解决



***



#### 异步缓存

异步缓存写入 Write Behind Pattern 由 cache 服务来负责 cache 和 DB 的读写，对比读写穿透不同的是 Write Behind Caching 是<font color="red">只更新缓存，不直接更新 DB，改为**异步批量**的方式来更新 DB，可以减小写的成本</font>

缺点：这种模式对数据一致性没有高要求，可能出现 cache 还没异步更新 DB，服务就挂掉了

应用：

* DB 的写性能非常高，适合一些数据经常变化又对数据一致性要求不高的场景，比如浏览量、点赞量

* MySQL 的 InnoDB Buffer Pool 机制用到了这种策略



****



### 缓存一致

使用缓存代表不需要强一致性，只需要最终一致性

缓存不一致的方法：

* 数据库和缓存数据强一致场景：
  * 更新 DB 时同样更新 cache，加一个锁来保证更新 cache 时不存在线程安全问题，这样可以增加命中率
  * 延迟双删：先淘汰缓存再写数据库，休眠 1 秒再次淘汰缓存，可以将 1 秒内造成的缓存脏数据再次删除
  * CDC 同步：通过 canal 订阅 MySQL binlog 的变更上报给 Kafka，系统监听 Kafka 消息触发缓存失效
* 可以短暂允许数据库和缓存数据不一致场景：更新 DB 的时候同样更新 cache，但是给缓存加一个比较短的过期时间，这样就可以保证即使数据不一致影响也比较小



参考文章：http://cccboke.com/archives/2020-09-30-21-29-56



***



### 企业方案

#### 缓存预热

场景：宕机，服务器启动后迅速宕机

问题排查：

1. 请求数量较高，大量的请求过来之后都需要去从缓存中获取数据，但是缓存中又没有，此时从数据库中查找数据然后将数据再存入缓存，造成了短期内对 redis 的高强度操作从而导致问题

2. 主从之间数据吞吐量较大，数据同步操作频度较高

解决方案：

- 前置准备工作：

  1. 日常例行统计数据访问记录，统计访问频度较高的热点数据

  2. 利用 LRU 数据删除策略，构建数据留存队列例如：storm 与 kafka 配合

- 准备工作：

  1. 将统计结果中的数据分类，根据级别，redis 优先加载级别较高的热点数据

  2. 利用分布式多服务器同时进行数据读取，提速数据加载过程

  3. 热点数据主从同时预热

- 实施：

  4. 使用脚本程序固定触发数据预热过程

  5. 如果条件允许，使用了 CDN（内容分发网络），效果会更好

总的来说：缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题，用户直接查询事先被预热的缓存数据！



***



#### 缓存雪崩

场景：数据库服务器崩溃，一连串的问题会随之而来

问题排查：在一个较短的时间内，**缓存中较多的 key 集中过期**，此周期内请求访问过期的数据 Redis 未命中，Redis 向数据库获取数据，数据库同时收到大量的请求无法及时处理。

解决方案：

1. 加锁，慎用
2. 设置热点数据永远不过期，如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中
3. 缓存数据的<font color="red">过期时间设置随机</font>，防止同一时间大量数据过期现象发生
4. 构建**多级缓存**架构，Nginx 缓存 + Redis 缓存 + ehcache 缓存
5. 灾难预警机制，监控 Redis 服务器性能指标，CPU 使用率、内存容量、平均响应时间、线程数
6. 限流、降级：短时间范围内牺牲一些客户体验，限制一部分请求访问，降低应用服务器压力，待业务低速运转后再逐步放开访问


总的来说：缓存雪崩就是瞬间过期数据量太大，导致对数据库服务器造成压力。如能够有效避免过期时间集中，可以有效解决雪崩现象的出现（约 40%），配合其他策略一起使用，并监控服务器的运行数据，根据运行记录做快速调整。



***



#### 缓存击穿

场景：系统平稳运行过程中，数据库连接量瞬间激增，Redis 服务器无大量 key 过期，Redis 内存平稳无波动，Redis 服务器 CPU 正常，但是数据库崩溃

问题排查：

1. **Redis 中某个 key 过期，该 key 访问量巨大**

2. 多个数据请求从服务器直接压到 Redis 后，均未命中

3. Redis 在<font color="red">短时间内发起了大量对数据库中同一数据的访问</font>

简而言之两点：单个 key 高热数据，key 过期

解决方案：

1. 预先设定：以电商为例，每个商家根据店铺等级，指定若干款主打商品，在购物节期间，加大此类信息 key 的过期时长 注意：购物节不仅仅指当天，以及后续若干天，访问峰值呈现逐渐降低的趋势

2. 现场调整：监控访问量，对自然流量激增的数据**延长过期时间或设置为永久性 key**

3. 后台刷新数据：启动定时任务，高峰期来临之前，刷新数据有效期，确保不丢失

4. **二级缓存**：设置不同的失效时间，保障不会被同时淘汰就行

5. 加锁：分布式锁，防止被击穿，但是要注意也是性能瓶颈，慎重

总的来说：缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中 Redis 后，发起了大量对同一数据的数据库访问，导致对数据库服务器造成压力。应对策略应该在业务数据分析与预防方面进行，配合运行监控测试与即时调整策略，毕竟单个 key 的过期监控难度较高，配合雪崩处理策略即可



***



#### 缓存穿透

场景：系统平稳运行过程中，应用服务器流量随时间增量较大，Redis 服务器命中率随时间逐步降低，Redis 内存平稳，内存无压力，Redis 服务器 CPU 占用激增，数据库服务器压力激增，数据库崩溃

问题排查：

1. Redis 中大面积出现未命中

2. 出现非正常 URL 访问

问题分析：

- 访问了不存在的数据，跳过了 Redis 缓存，数据库页查询不到对应数据
- Redis 获取到 null 数据未进行持久化，直接返回
- 出现黑客攻击服务器

解决方案：

1. 缓存 null：对查询结果为 null 的数据进行缓存，设定短时限，例如 30-60 秒，最高 5 分钟

2. 白名单策略：提前预热各种分类**数据 id 对应的 bitmaps**，id 作为 bitmaps 的 offset，相当于设置了数据白名单。当加载正常数据时放行，加载异常数据时直接拦截（效率偏低），也可以使用布隆过滤器（有关布隆过滤器的命中问题对当前状况可以忽略）

3. 实时监控：实时监控 Redis 命中率（业务正常范围时，通常会有一个波动值）与 null 数据的占比

   * 非活动时段波动：通常检测 3-5 倍，超过 5 倍纳入重点排查对象
   * 活动时段波动：通常检测10-50 倍，超过 50 倍纳入重点排查对象

   根据倍数不同，启动不同的排查流程。然后使用黑名单进行防控

4. key 加密：临时启动防灾业务 key，对 key 进行业务层传输加密服务，设定校验程序，过来的 key 校验；例如每天随机分配 60 个加密串，挑选 2 到 3 个，混淆到页面数据 id 中，发现访问 key 不满足规则，驳回数据访问

总的来说：缓存击穿是指访问了不存在的数据，跳过了合法数据的 Redis 数据缓存阶段，每次访问数据库，导致对数据库服务器造成压力。通常此类数据的出现量是一个较低的值，当出现此类情况以毒攻毒，并及时报警。无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除



参考视频：https://www.bilibili.com/video/BV15y4y1r7X3





***



### 性能指标

Redis 中的监控指标如下：

* 性能指标：Performance

  响应请求的平均时间：

  ```sh
  latency
  ```

  平均每秒处理请求总数：

  ```sh
  instantaneous_ops_per_sec
  ```

  缓存查询命中率（通过查询总次数与查询得到非nil数据总次数计算而来）：

  ```sh
  hit_rate(calculated)
  ```

* 内存指标：Memory

  当前内存使用量：

  ```sh
  used_memory
  ```

  内存碎片率（关系到是否进行碎片整理）：

  ```sh
  mem_fragmentation_ratio
  ```

  为避免内存溢出删除的key的总数量：

  ```sh
  evicted_keys
  ```

  基于阻塞操作（BLPOP等）影响的客户端数量：

  ```sh
  blocked_clients
  ```

* 基本活动指标：Basic_activity

  当前客户端连接总数：

  ```sh
  connected_clients
  ```

  当前连接 slave 总数：

  ```sh
  connected_slaves
  ```

  最后一次主从信息交换距现在的秒：

  ```sh
  master_last_io_seconds_ago
  ```

  key 的总数：

  ```sh
  keyspace
  ```

* 持久性指标：Persistence

  当前服务器其最后一次 RDB 持久化的时间：

  ```sh
  rdb_last_save_time
  ```

  当前服务器最后一次 RDB 持久化后数据变化总量：

  ```sh
  rdb_changes_since_last_save
  ```

* 错误指标：Error

  被拒绝连接的客户端总数（基于达到最大连接值的因素）：

  ```sh
  rejected_connections
  ```

  key未命中的总次数：

  ```sh
  keyspace_misses
  ```

  主从断开的秒数：

  ```sh
  master_link_down_since_seconds
  ```

要对redis的相关指标进行监控，我们可以采用一些用具：

- CloudInsight Redis
- Prometheus
- Redis-stat
- Redis-faina
- RedisLive
- zabbix

命令工具：

* benchmark

  测试当前服务器的并发性能：

  ```sh
  redis-benchmark [-h ] [-p ] [-c ] [-n <requests]> [-k ]
  ```

  范例：100 个连接，5000 次请求对应的性能

  ```sh
  redis-benchmark -c 100 -n 5000
  ```

  ![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/redis/Redis-redis-benchmark指令.png)

* redis-cli

  monitor：启动服务器调试信息

  ```sh
  monitor
  ```

  slowlog：慢日志

  ```sh
  slowlog [operator]    #获取慢查询日志
  ```

  * get ：获取慢查询日志信息
  * len ：获取慢查询日志条目数
  * reset ：重置慢查询日志

  相关配置：

  ```sh
  slowlog-log-slower-than 1000 #设置慢查询的时间下线，单位：微妙
  slowlog-max-len 100	#设置慢查询命令对应的日志显示长度，单位：命令数
  ```

  
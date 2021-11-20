

# 1. 安装配置

安装

```shell
$ wget https://download.redis.io/releases/redis-6.2.6.tar.gz
$ tar xzf redis-6.2.6.tar.gz
$ cd redis-6.2.6
$ make
```



启动服务端` ./src/redis-server ./redis.conf`

启动客户端`src/redis-cli`

测试是否成功`ping` 回应`PONG`

关闭连接`quit`

切换数据库`select index`



# 2. 数据类型

[查询手册](https://redis.io/commands)



## 2.1 String字符串

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



## 2.2 Set集合

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



## 2.3 list列表

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



## 2.4 Hashset哈希集合

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



## 2.5 Sorted Sets排序集合

```shell
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




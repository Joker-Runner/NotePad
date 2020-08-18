# Redis学习





## 一、安装配置

### 1.1 安装

```shell
# 将程序拷贝至程序目录下
sudo mv redis-6.0.5.tar.gz /usr/local/redis-6.0.5.tar.gz
# 解压程序包
sudo tar xzf redis-6.0.5.tar.gz
# 编译测试
sudo make test 
# 编译安装
sudo make install
# 拷贝启动文件，可能要先创建bin文件夹
sudo cp src/mkreleasehdr.sh src/redis-benchmark src/redis-check-rdb src/redis-cli src/redis-server bin/

# 在编译过程中出现错误，如下命令清理了再编译： 
sudo make distclean && sudo make && sudo make test
```

### 1.2 配置

```shell
#修改为守护模式
daemonize yes
#设置进程锁文件
pidfile /usr/local/redis-4.0.9/redis.pid
#端口
port 6379
#客户端超时时间
timeout 300
#日志级别
loglevel debug
#日志文件位置
logfile /usr/local/redis-4.0.9/log-redis.log
#设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
databases 16
##指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
#save <seconds> <changes>
#Redis默认配置文件中提供了三个条件：
save 900 1
save 300 10
save 60 10000
#指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，
#可以关闭该#选项，但会导致数据库文件变的巨大
rdbcompression yes
#指定本地数据库文件名
dbfilename dump.rdb
#指定本地数据库路径
dir /usr/local/redis-4.0.9/db/
#指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能
#会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有
#的数据会在一段时间内只存在于内存中
appendonly no
#指定更新日志条件，共有3个可选值：
#no：表示等操作系统进行数据缓存同步到磁盘（快）
#always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
#everysec：表示每秒同步一次（折衷，默认值）
appendfsync everysec
```





## 二、启动

### 2.1 启动

```shell
# 此命令启动Redis服务
redis-server

# 关闭服务需要启动Redis客户端，输入shutdown命令，带参数
redis-cli
>shutdown [nosave|save]
```

### 2.2 基础操作

Redis中默认有16个数据，默认使用第0个

```shell
# 切换数据库
127.0.0.1:6379> select 3
OK
# set值
127.0.0.1:6379[3]> set name Joker
OK
# 查看数据库大小
127.0.0.1:6379[3]> dbsize
(integer) 1
# 获取插入的值
127.0.0.1:6379[7]> get name
"Joker"
# 查看数据库所有的key
127.0.0.1:6379[3]> keys *
1) "name"
# 清空当前库
127.0.0.1:6379[3]> flushdb
OK
# 清空所有库
127.0.0.1:6379[3]> flushall
OK

```

其他操作

```shell
# 移动key到指定数据库
127.0.0.1:6379[3]> move last 0
(integer) 1
# 判断key是否存在
127.0.0.1:6379[3]> exists last
(integer) 0
# 删除key
127.0.0.1:6379[3]> del name
(integer) 1

# 设置过期时间，秒
127.0.0.1:6379> expire last 10
(integer) 1
# 查看过期剩余时间
127.0.0.1:6379> ttl last
(integer) 3

127.0.0.1:6379> set name Joker
OK
127.0.0.1:6379> set age 1
OK
# 获取key值类型
127.0.0.1:6379> type name
string
127.0.0.1:6379> type age
string
```

## 三、各种数据类型

### 3.1 String

```shell
# 往某个key中追加字符串
127.0.0.1:6379> append name  Runner
(integer) 11
# 获取字符串的长度
127.0.0.1:6379> strlen name
(integer) 11
# set、append的字符串前后的空格无意义
127.0.0.1:6379> get name
"JokerRunner"

127.0.0.1:6379> set count 0
OK
# 自增
127.0.0.1:6379> incr count
(integer) 1
127.0.0.1:6379> incr count
(integer) 2
# 自减
127.0.0.1:6379> decr count
(integer) 1
# 自增设置步长
127.0.0.1:6379> incrby count 10
(integer) 11
127.0.0.1:6379> decrby count 5
(integer) 6


127.0.0.1:6379> set name Joker,Redis
OK
# 截取子字符串
127.0.0.1:6379> getrange name 0 3
"Joke"

127.0.0.1:6379> get name
"Joker,Redis"
# Replace，偏移量 目标字符串
127.0.0.1:6379> setrange name 2 xx
(integer) 11
127.0.0.1:6379> get name
"Joxxr,Redis"

########################################
>setex(set with expire) # 设置过期时间
>setnx(set if not exists) # 不存在再设置（常用于分布式锁中）

127.0.0.1:6379> setex key3 30 Hello
OK
127.0.0.1:6379> ttl key3
(integer) 24
127.0.0.1:6379> setnx key3 ll
(integer) 0
127.0.0.1:6379> ttl key3
(integer) 9
127.0.0.1:6379> setnx key3 Redis
(integer) 0
127.0.0.1:6379> setnx key3 Redis
(integer) 1


########################################
# 批量操作
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
# 不存在再设置，原子性操作，比较适合用于分布式锁
127.0.0.1:6379> msetnx k1 v1 k4 v4
(integer) 0 # 失败

# 组合命令，先获取，再设置
127.0.0.1:6379> getset db redis
(nil)
127.0.0.1:6379> getset db mongodb
"redis"
```

String的使用场景

* 除了是字符串还可以是数字，比如用作计数器。
* 

### 3.2 List

```shell
# 给一个List插入到头部
127.0.0.1:6379> lpush list one
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
# 获取List的值，参数为起止下标
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
# rpush 将值插入到后面
127.0.0.1:6379> rpush list four
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "four"

# 移除值（左pop，右pop）
127.0.0.1:6379> lpop list
"three"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
3) "four"
127.0.0.1:6379> rpop list
"four"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"

# 查看list的长度
127.0.0.1:6379> llen list
(integer) 2
# 根据索引获取值
127.0.0.1:6379> lindex list 1
"one"
#llen，lindex，都没有对应的rlen，rindex
127.0.0.1:6379> rindex list 1
(error) ERR unknown command `rindex`, with args beginning with: `list`, `1`, 
127.0.0.1:6379> 


############################################
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "three"
3) "three"
4) "two"
5) "one"
# 移除1个three
127.0.0.1:6379> lrem list 1 three
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
# 移除2个three
127.0.0.1:6379> lrem list 2 three
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"

# 通过下标截断list
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
3) "three"
127.0.0.1:6379> ltrim list 1 2
OK
127.0.0.1:6379> lrange list 0 -1
1) "one"
2) "three"
```



### 3.3 Set

```shell
# 往set添加值
127.0.0.1:6379> sadd set one
(integer) 1
127.0.0.1:6379> sadd set two
(integer) 1
127.0.0.1:6379> sadd set three
(integer) 1
# 查看set中的值
127.0.0.1:6379> smembers set
1) "two"
2) "one"
3) "three"
# 查看set中是否存在值
127.0.0.1:6379> sismember set one
(integer) 1
# 查看set中的元素数量
127.0.0.1:6379> scard set
(integer) 3
# 移除set中的指定元素
127.0.0.1:6379> srem set one
(integer) 1
# 获取随机的元素
127.0.0.1:6379> srandmember set
"two"
127.0.0.1:6379> srandmember set
"three"
# 随即移除集合中一个元素
127.0.0.1:6379> spop set
"three"

####################################### 
127.0.0.1:6379> smembers s1
1) "d"
2) "a"
3) "b"
4) "c"
127.0.0.1:6379> smembers s2
1) "e"
2) "c"
# 取s1，s2的差集
127.0.0.1:6379> sdiff s1 s2
1) "d"
2) "a"
3) "b"
# 取s1，s2的交集
127.0.0.1:6379> sinter s1 s2
1) "c"
# 取s1，s2的并集
127.0.0.1:6379> sunion s1 s2
1) "a"
2) "b"
3) "c"
4) "e"
5) "d"
```














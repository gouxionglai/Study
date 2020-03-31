# Redis

> https://www.cnblogs.com/zuidongfeng/p/8032505.html>

概述：

简单，高效，分布式，基于内存的key-value式的缓存工具。

C语言编写，**单线程操作**，不会有线程安全问题。

分布式数据库中原理：CAP+BASE

# 基础

## 概念

### 传统数据库

重要的是**ACID**：

- A：automicity	原子性
- C：consistency	一致性
- I：isolation	独立性
- D：Durability	持久性

### NOSQL数据库

重要的是**CAP+BASE**

- C：consistency	一致性
- A：availability	可用性
- P：partition tolerance	分区容错性



但是一个分布式系统不可能3个都满足，即要么CA,AP,或者CP三大类

- CA：单点集群，可扩展性不强，即传统数据库Oracle
- **AP**：不满足一致性，对数据准确度要求不高。大多数网站架构选择。**重点**
- CP：不满足可用性，性能不高。 比如Redis, Mongodb



既然只能满足两个，**即牺牲暂时的一致性，换来系统的性能**。但是最后在资源不紧张的时候也来满足一致性。即需要BASE条件。

- BA：basically available基本可用
- S：soft state 软状态
- E：Eventually consistency 最终一致

### 分布式和集群

分布式：不同服务器上部署不同的服务模块。通过rpc/rmi进行通信和调用。对外提供服务

集群：不同服务器上部署相同的服务模块。通过分布式调度软件进行统一调度（负载均衡），对外提供服务

## 进阶场景

单服务单数据库，直接交互

单服务多数据库，**负载均衡**，直接交互（**垂直拆分**）

单服务，负载均衡，**缓存**，多数据库，间接交互

单服务，负载均衡，缓存，多数据库（**主从模式，读写分离**），间接交互

单服务，负载均衡，缓存，多数据库（**主从模式，读写分离，分库分表，水平拆分mysql集群**），间接交互

no-sql用于处理大量的数据，大数据时代，海量，多样，实时数据交互

## 使用场景

expire key seconds

1. 限时优惠活动信息  
2. 网站数据缓存（对于一些需要定时更新的数据，比如排行榜）ZSET
3. 手机验证码，经常是1分钟内有效
4. 限制网站访客访问频率（1分钟最多访问10次，防止恶意攻击等）

INCR 自增指令 原子性操作

1. 网站统计访问量



## 安装

参考：

<https://www.cnblogs.com/zuidongfeng/p/8032505.html>

<https://blog.csdn.net/u010309394/article/details/81807597>    //设置了密码

## 操作

参考：

<https://www.cnblogs.com/sunzhiqi/p/10869638.html>

<https://www.cnblogs.com/kim-yang/p/10165394.html>	//更深入一些

### 基本类型

**注意key区分大小写**

#### string类型

字符串类型是 Redis 中最为基础的数据存储类型，它在 Redis 中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。

##### 添加(修改)

//如果已存在就是覆盖操作

--添加单个 set

```txt
set [key] [value]		--返回ok
```

--添加多个 mset

```txt
mset [key1] [value1] [key2] [value2] ...
```

--添加时指定超时时间

```txt
setex [key] [seconds] [value]
```

**--不存在是才添加，避免覆盖**

```txt
setnx [key] [value]	//set if not exists
```



##### 追加

//在原value后拼接

```txt
append [key] [value]	--返回添加之后的值的长度
```

例子：

```txt
>set key1 value1
OK

>append key1 23
(integer) 8

>get key1
"value123"
```



##### 获取

--单个get

```txt
get [key]		--返回value
```

--多个mget

```txt
mget [key1] [key2]
```

例子：

```txt
127.0.0.1:6379> mget key1 key2 key3
1) "value123"
2) "value2"
3) (nil)	//没有这个key就返回(nil)

```

#### hash类型

功能其实string也能实现，但是hash修改值更灵活。所以如果只是取值修改少，宁愿存string

可以理解成map

- hash⽤于存储对象，对象的结构为属性、值
- 值的类型为string
- 设置单个属性


##### 添加(修改)

```txt
hset [key] [field] [value]
//不存在才添加
hsetnx [key] [field] [value]
```

java中：


```java
Map<String,Object> map = new HashMap<>();
map.put("field1","value1");
map.put("field2","value2");
```

redis中:

```txt
hset map filed1 value1 filed2 value2
```

##### 获取

--获取单个 hget

```txt
hget [key] [field]
```

--获取多个hmget

```txt
hmget [key] [field1] [field2]..
```

--获取所有的key

```txt
hkeys [key]
```

--获取所有的value

```txt
hvals [key]
```

--获取所有的key+value

```txt
hgetall [key]
```

##### 判断

hexists [key]  用于判断某个field是否存在。

例子：

```txt
//添加
127.0.0.1:6379> hset map key1 value1 key2 value2
(integer) 2   //返回成功的数量而不是当前key的所有field数量

//查询单个
127.0.0.1:6379> hget map key1
"value1"

//查询多个
127.0.0.1:6379> hmget map key1 key2 key3
1) "value1"
2) "value2"
3) (nil)

//获取所有
127.0.0.1:6379> hvals map
1) "value1"
2) "value2"

```

##### 删除

--删除整个key使用通用操作中的del

--删除key下的某个field,  成功会返回删除的数量。如果没有返回0

```txt
hdel [key] [field]
```

##### 自增

```txt
//整型量
hincrby [key] [field] increment
//float型增长
hincrbyfloat [key] [field] increment

```



#### list类型

参考java类型的list

- 列表的元素类型为string
- 按照插⼊顺序排序
- 允许从左侧操作Lxx，也可以从右侧操作RXX。
- **左侧是头部，右侧是尾部**
- **允许value重复**

##### 添加(追加)

//如果有相同的key，是追加操作

```txt
//左侧添加
Lpush [key] [value1] [value2] ...	//先入后出
//右侧添加
Rpush [key] [value1] [value2] ...  //所以Rpush才是我们正常理解的先入先出
```



##### 插入

//在某key的指定元素oldValue的前面或者后面添加新元素newValue

```txt
linsert [key] before/after [oldValue] [newValue]
```

例子：

```txt
127.0.0.1:6379> lpush list1 a b c d
(integer) 4
127.0.0.1:6379> linsert list1 e f g
(integer) 7
//查看list1中从0到7的值
127.0.0.1:6379> lrange list1 0 7
//读取非常有意思 ，像压栈，先进后出。
//最后压栈的在最上面。编号是最小的。
1) "g"
2) "f"
3) "e"
4) "d"
5) "c"
6) "b"
7) "a"
//在前面添加
127.0.0.1:6379> linsert list1 before e 11
(integer) 8
//在后面添加
127.0.0.1:6379> linsert list1 after e 22
(integer) 9
127.0.0.1:6379> lrange list1 0 9
1) "g"
2) "f"
3) "11"
4) "e"
5) "22"
6) "d"
7) "c"
8) "b"
9) "a"

```



##### 查看

**因为有了顺序，所以从右往左算是从0开始；从左往右算是从-1开始,倒数第二个是-2。**使用于其他有顺序的类型，比如Zset

- start从0开始数
- stop实际最大值是数组长度。如果超过最大，也只读取整个数组。(**最后一位可以用实际长度值或者-1表示**)
- **读取和写入是反着的。即压栈的过程，先进后出。所以编号0的值是最后一个插入的。**（可以参考上面插入的例子）

```txt
lrange [key] [start] [stop]
```

例子：

```txt
获取全部：
lrange list1 0 -1
获取第三位以及之后的:
lrange list1 2 -1
```

##### 获取

出栈操作，获取并删除元素

```txt
LPOP [key]	//弹出左边的元素，返回元素的值  
RPOP [key]	//弹出右边的元素，返回元素的值 
//阻塞式访问 BRPOP 和 BLPOP
//但是POP可能不安全，因为取出来过程中，可能中间环节有问题，比如网络断了，导致并没有到达目的地。所以提供了另一个BRpopLpush
//将source的最后一个元素放到destination的最前面
BRpopLpush [source] [destination]
//增加了时间的概念 在规定时间内取，有就取值，无就返回nil
bLpop [key...] [timeout]
bRpop [key...] [timeout]
```

例子：

```txt
RpopLpush a1 b1
RpopLpush a1 a1   	//循环列表，想象成一个圆环，切开一个口就有头有尾，RpopLpush就好像是挪了一个位置切。

127.0.0.1:6379> lpush a1 1 2 3 4 5
(integer) 5
127.0.0.1:6379> lrange a1 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
127.0.0.1:6379> lpush a2 a b c d e
(integer) 5
127.0.0.1:6379> lrange a2 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"
//把a1 最后元素放到a2前面
127.0.0.1:6379> rpoplpush a1 a2
"1"
127.0.0.1:6379> lrange a2 0 -1
1) "1"
2) "e"
3) "d"
4) "c"
5) "b"
6) "a"

```



##### 修改

修改指定位置的值

```txt
lset [key] [index] [value]
```

//需要注意的是index的值，记住后添加的序列号最小。index从0开始

##### 截取

可以使用[LTRIM](http://www.redis.cn/commands/ltrim.html)把list从左边截取指定长度。

//注意 没有Rtrim

```
LTRIM [key] [start] [stop]
```



##### 删除

删除指定元素，将列表中前count次出现的值为value的元素移除。

```txt
lrem [key] [count] [value]
```

- count > 0: 从头往尾移除
- count < 0: 从尾往头移除
- **count = 0: 移除所有**

//通俗理解就是：删除第count次出现的value。只不过count具有方向性。如果只出现了1次， count写为3，也当作删除这一个。

```txt
127.0.0.1:6379> lrem list1 1 pp
(integer) 0	//数组中没有pp的值，所以返回0删除失败
127.0.0.1:6379> lrem list1 3 a
(integer) 1	//数组中只有一个a  并没有出现3次，直接删除第一次出现的a. 删除成功。
127.0.0.1:6379> lrem list1 0 b
(integer) 3  //数组中出现了3次b，  则直接全部删除掉。返回删除的个数

```

#### set类型

参考java类型的set。 和list类似，只是没有顺序的概念。

- ⽆序集合
- 元素为string类型
- 元素具有唯⼀性，**不重复**
- 说明：对于集合没有修改操作

##### 添加(追加)

//如果value重复了就不会追加了

```txt
sadd [key] [value1] [value2] [...]
```

##### 查看

```txt
//获取所有的值
smembers [key]
//获取值的数量
scard [key]
//判断是否存在
sismember [key] [member]
```

##### 获取

随机获取数据 可以指定个数

srandmember [key] [count]

随机获取数据 （类似于pop 获取之后集合数据就少了，不会重复。）

spop [key]



##### 删除

```txt
srem [key] [value1] [value2]
```

例子：

```txt
//添加
127.0.0.1:6379> sadd set1 val1 val2 val3 val1
(integer) 3
//查看
127.0.0.1:6379> smembers set1
1) "val3"
2) "val1"
3) "val2"
//添加(追加)
127.0.0.1:6379> sadd set1 val4
(integer) 1
//重复的元素不会添加
127.0.0.1:6379> sadd set1 val4
(integer) 0
127.0.0.1:6379> smembers set1
1) "val4"
2) "val3"
3) "val1"
4) "val2"
//删除指定元素
127.0.0.1:6379> srem set1 val3 val2
(integer) 2
127.0.0.1:6379> smembers set1
1) "val4"
2) "val1"

```

#### 扩展

交集，并集，差集

实际场景：共同好友，推荐好友等

```txxt
//交集
sinter [key1] [key2]
//并集
sunion [key1] [key2]
//差集
sdiff [key1] [key2]

//求集合并保存到指定集合中
sinterstore [destination] [key1] [key2]
sunionstore [destination] [key1] [key2]
sdiffstore [destination] [key1] [key2]

//将指定数据从原始集合移动到目标集合
smove [source] [destination] [member]
```



#### zset类型

相比set多了顺序的概念。

- sorted set，有序集合
- 元素为string类型
- 元素具有唯⼀性，不重复
- 每个元素都会关联⼀个double类型的**score，表示权重**，通过权重将元素从⼩到⼤排序。**小的先取出来。**
- 说明：没有修改操作

##### 添加

```txt
zadd [key] [score1] [member1] [score2] [member2] ...
```

##### 查询

根据下标查询zrange

//因为有顺序，所以查询类似list，而不是set

```txt
zrange [key] [start] [stop]
```



根据权重查询zrange(正序)，zrangebyscore(逆序)

//权重是double类型， **比如3，5中间可能不止有一个值，或者根本没有值。**

```txt
zrangescore [key] [min] [max]
```

根据value查询对应的权重

```txt
zscore [key] [member]
```

##### 删除

删除指定元素

```txt
zrem key member1 member2 ...
```

删除指定权重范围的元素

```txt
zremrangebyscore key min max
```



##### 举例

积分，成绩等等排行榜。

```java
//1.插入学生成绩信息
zadd student 91 a 98 b 68 c 55 d 88 e 73 f
//2.按成绩由高到低，查询前三名成绩	即逆序排序
zrevrange student 0 2
//3.查询成绩在60-80直接的学生 //因为权重刚好就是分数
zrangebyscore student 60 80
```







### 通用操作

#### 获取所有的key

//*代表匹配后面所有

//? 代表匹配后面一个字符

```txt
keys [匹配规则]
//比如：
keys *
keys h*
keys h? 
```

#### 查看某个key是否存在

```txt
exists [key]	//返回1代表存在，0代表不存在
```



#### 设置key失效时间

//不设置默认是永久

```txt
expire [key] [seconds]
```

#### 查看key的剩余失效时间

--以秒为单位

```txt
ttl [key]
```

```txt
>0	代表具体时间
-1	代表永久有效(不设置是默认)
-2	代表已经失效(节约内存)
```

#### 移除key的失效时间

```txt
persist [key]
//或者重复设置一下值就行。
//比如set [key] [value] 相当于覆盖
```



#### 查看键值对类型type

//返回对应value的类型

```txt
type [key]
```

#### 删除

//不管什么类型都能删除

//支持同时删除多个， 空格隔开

```txt
del [key...]
```

#### 清空

清空当前库的所有key：flushdb

清空整个redis的所有key：flushall

#### 自增自减

线程安全的

```txt
//自增
INCR [key]
//指定步长增加
INCRBY [key] [increment]
//自减
DECR [key]
//指定步长减少
DECRBY [key] [increment]
```



#### 改名

### 编码规范

#### 命名规范

- key不要太长，太长消耗内存，并且降低搜索效率
- 增加可读性，**使用冒号隔开**，key最好通俗易懂，比如user:123:password    代表user是123的密码。value就是具体的密码
- 一般是用=表名:主键名:主键值:属性名 



### 

# 进阶

## 事务

- 开启事务： multi
- 执行语句：只是暂时加入队列，等待一起提交。
- 提交事务：exec
- 回滚事务：discard
- 监视：watch key     //可以防止其他进程去改变key的值，如果已经被改变则本次事务会回滚。

注意：

​	事务保证一起提交不被插队。语法错误不会导致回滚，但是命令错误会导致回滚。

## 持久化

### RDB

### AOF

### 混合持久化

## 键值过期策略

### 管道技术Pipeline

### 附件的人

统计算法

过滤器

内存淘汰机制

### 消息队列

# 高级

### 分布式锁

### 延迟队列

### 定时任务

### RedisSearch

### 性能优化

### 主从设置

### 哨兵模式

### 集群模式

无中心化，至少需要3主3从才能构成集群。

#### master不可用

投票机制来决定。所有的master参与投票，半数以上的master与其他master节点通信超时，则认为master不可用。

#### 集群不可用

集群任意master挂掉，且当前master没有slave，集群进入fail。

集群超过半数以上master挂掉，无论有无slave，集群进入fail状态

#### 配置集群

--cluster create

```txt
//redis-cli中输入命令
--cluster create ip1:端口1 ip2:端口2 .....ipN:端口N --cluster-replicas 1
//会自动帮你配置主从关系，回复yes就生效，no就自己配置
//--cluster-replicas 1 代表每个主机都希望至少有一个从机

//连接某个节点
//-c 是重点，表示要求连接上集群 
//-h host主机  -p port端口 -a auth 密码
redis-cli -h 127.0.0.1 -c -p 7001 -a password
//查看当前节点信息
info replication

//查看集群节点信息,会罗列所有节点。
//每个节点都会生成一个唯一的id.作为唯一标识。
cluster nodes
```

脚本控制一键集群开启or关闭：shutdown.sh

```shell

/usr/local/redis_cluster/src/redis-cli -c h 127.0.0.1 -p 7000 -a pass shutdown

/usr/local/redis_cluster/src/redis-cli -c h 127.0.0.1 -p 7001  -a pass shutdown

/usr/local/redis_cluster/src/redis-cli -c h 127.0.0.1 -p 7002  -a pass shutdown

/usr/local/redis_cluster/src/redis-cli -c h 127.0.0.1 -p 7003  -a pass shutdown
```

使脚本变成可执行的文件。

```shell
chmod u+x shutdown.sh	//修改文件属性，变成可执行文件
```





### 总结


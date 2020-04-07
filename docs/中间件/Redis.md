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





### 高级数据类型

#### bitmap

利用bit里面的位来存数据。比如是否，是即1，否即0.节约空间。

#### hyperlog

...

#### GEO

坐标轴距离计算。像附近的人，外卖显示距离等。

- 添加坐标点:    key + 经度 + 维度 + person1 

  geoadd key longitude latitude member

- 获取坐标点

  geopos key [member...]

- **计算坐标点距离**

  geodist key member1 member2 [unit]

- 根据**坐标**求范围内数据

  georedius key longitude latitude [unit]

- 根据**点**求**范围**内数据

  georadiusbymember key member radius [unit] //redius是半径

- 获取指定点对应的坐标hash值

  geohash key [member...]

  

  



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

```txt
//注意：如果新keyname已经存在，是会覆盖的。并且如果类型不一样，会导致值变为空
rename [oldkeyname] [newkeyname]
//优化如下：
//如果不存在才改名
renamenx [oldkeyname] [newkeyname]
```

#### 排序

//只对list，set有用

sort [key] 

#### 查看帮助

help  code

#### 其他

//查看redis情况

**info**

//查看服务是否通

ping	//会返回pong

//选择数据库

select  db（一共16个 即最大15最小0）

//移动数据

move key db

//数据清除

清空当前库的所有key：flushdb

清空整个redis的所有key：flushall

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

​	**watch需要在事务之前。**

​	watch的值改变之后，其后面的事务，不管有么有修改到监视的key，都会回滚。

​	事务保证一起提交不被插队。语法错误不会导致回滚，但是命令错误会导致回滚。

```java
//在redisTemplate里面，加事务也是一样。
redisTemplate.multi();
redisTemplate.xxx1
redisTemplate.xxx2
redisTemplate.exec();
```





## 持久化

注意修改配置需要重启服务。

关注结果：快照形式。存储格式简单，存储最终结果。关注点在数据。即RDB

关注过程：日志形式。存储格式复杂，存储操作步骤，关注点在操作。即AOF

### RDB



**手动持久化**

#### 命令

#### (redis-cli中输入)

- save：量大的化很消耗性能，可能会长时间阻塞线程。**弃用**
- bgsave：（back ground save）后台择机，利用fork函数新建子线程执行，所以不会阻塞主线程。
- save second changes：配置的自动持久化（其本质还是gbsave）。**在配置文件中**设置在second秒内达到了changes次变化就**触发**进行持久化。注意：是变化，而不是新增。即删除或者重新set值都算变化（即使set同一个值也算）。

#### 配置

（配置文件中修改）

- dbfilename：文件名，通常设置成dump-端口号.rdb
- dir： 文件保存路径，通常放在存储空间大的目录中的data下
- rdbcompression：yes 持久化使是否压缩。采用LZF压缩。
- rdbchecksub：yes 是否进行文件格式校验。读写均要校验。
- **stop-writes-on-bgsave-error**：yes.  后台序列化出错的时候是否停止 (bgsave才有)

优点：

- 存储压缩的二进制数据，存储效率高
- 存储是在某个时间点执行的，很适合数据备份，全量复制等场景。
- RDB速度比AOF快很多
- 应用：每隔几小时备份一次，用于灾难恢复。

缺点：

- 无法实时持久化，肯定会丢失一部分数据。
- bgsave要执行fork命令新建线程执行持久化，需要多消耗内存。
- 版本未统一，可能不同的redis版本存在兼容性。

### AOF

Append only file

#### 三种策略

- always：**每次**写入都会同步到AOF文件中，零误差，但是性能差
- everysec：**每秒**同步到AOF文件中，准确性相对较高，性能也较高（推荐）
- no：**系统控制**每次同步AOF文件，整体不可控。

#### 配置

- appendonly： 默认no，需要修改为yes，即使用AOF持久化

- appendfsync：使用何种策略，三选一，即always, everysec, no
- appendfilename：修改aof名称。建议appendonly-端口号.aof

#### 重写

重写规则：

- 超时的数据不写入AOF文件
- 忽略无效指令，只写入最终值。比如del key,  set key1 value1 set key2 value2
- 对同一数据多条写命令合并为一条。比如lpush list1 a, lpush list1 b, lpush list1 c 可以合并为lpush list1 a b c .

重写方式：

- 手动重写：bgrewriteaof。  即bg- rewirte- aof
- 自动重写：
  - auto-aof-rewirte-min-size  size.     即按增长大小来，达到即重写
  - auto-aof-rewrite-percentage percentage     即按增长比例来，达到即重写。，比如10%,原本30%，现在40%了就会触发重写。

### 混合持久化



## 键值过期策略

### 概念

redis里面内存模型有点像java那种堆栈型，每个数据都有其值和对应的内存地址，expire过期的时候直接修改地址值就行了。

但是遇到cpu比较繁忙的时候，并不要求立刻实现，而是闲下来的时候去执行，所以就需要策略去控制这种情况。

### 策略

- 定时删除：
  - 创建定时器，到时间就执行清理。
  - 用处理器性能换存储空间。会增加cpu压力，但是内存可以很好的控制。

- 惰性删除：
  - 过期不处理，但是有访问的时候再处理。
  - 与上面相反，拿内存换cpu性能。

- 定期删除
  - 简单理解就是每秒去遍历（双重遍历数据库的key，随机抽取，有过期就删除，并且看过期比例，如果超过设置的比例，则代表可能过期的很多了，则再次抽取，直到小于这个比例；再去下一个数据库执行相同的事情。redis里面有16个库）
  - 这种方案

## 逐出算法

### 含义

可能会内存不够，所以每次写入都会调用freeMemoryIfNeeded()去检测是否足够。然后根据**逐出算法**删除一些key。如果确实不够，就会抛错。

### 配置

- 最大可用内存：maxmemory
- 待删除数据个数：maxmemory-samples
- 删除策略：maxmemory-policy

### 策略

- 检测易失数据：可能会过期的数据
  - volatile-LRU：least recently used    同一时间段内，时间最长没被用的淘汰
  - volatile-LFU：least Frequently Used  同一时间段内，使用次数最少的淘汰
  - volatile-ttl：挑选将要过期的淘汰
  - volatile-random：随机淘汰
- 检测全库数据：所有数据
  - allkeys-LRU
  - allkeys-LFU
  - allkeys-random
- 放弃数据驱逐：即不删除，直接对外抛错，内存不够了

## 配置

redis.conf文件的设置

### 服务器设置

- 以守护进程的方式运行（后台）

  daemonize yes|no 

- 绑定主机地址

  bind ip

- 端口号

  port 6379

- 数据库数量

  database 16

### 日志配置

- 日志级别  默认时verbose，  生产环境一般用notice

  loglevel debug|**verbose**|notice|warning

- 日志名

  logfile  端口号.log

### 客户端配置

- 同一时间最大连接数。默认无限制

  maxclients 0

- 最大等待时长，达到后即断开，如果需要关闭，设置为0

  timeout 300

### 多服务器配置

可能很多redis服务器，并且大部分配置都相似。一改要改很多。所以提供了提起公共的功能，便于维护。

​	include /path/server-端口号.conf

# 高级

## 分布式锁

redission.org //实现了分布式锁

在分布式架构下，jdk自带的synchronized关键字并不管用。

所以可以利用redis里面的某些排他操作实现分布式锁。比如setnx key value。

原理：利用排他性

```java
public String sellTicket() throws Exception{
    String lockKey = "lockKey";
    String uuid = UUID.randomUUID().toString();
try{
    /***
    //成功的会返回true,而其他线程则会返回false.直到运行完为止。
    Boolean flag =  redisTemplate.opsForValue().setIfAbsent(lockKey,"1");
    //改进2：如果宕机等清空，不执行了，也让锁自动消失。
    redisTemplate.expire(lockKey,30,TimeUnit.SECENDS);
    */
    
    //改进3:如果第一行boolean就宕机了，那后面也不会执行。
    //所以希望两行是一个原子操作。即下面
    //Boolean flag =  redisTemplate.opsForValue().setIfAbsent(lockKey,"1",30,TimeUnit.SECENDS);    
    Boolean flag =  redisTemplate.opsForValue().setIfAbsent(lockKey,uuid,30,TimeUnit.SECENDS);
    if(!flag){
        retrun "error";
    }
    //执行正常的业务逻辑，比如扣减库存等
    .....
}
    //改进1：上面的抛异常之后也需要执行删除操作，避免死锁。
finally{
    //改进4：自己设置的锁自己释放。才能控制并发。释放锁的时候比较值是否一样。uuid
    if(uuid.equals(redisTemplate.opsForValue().get(lockKey))){
        //删除掉锁，避免之后的都进不去。
       redisTemplate.delete(lockKey); 
    }
    
}
return "end";
}
```



## 延迟队列

## 定时任务

## 主从复制

互联网三高架构：高并发，高性能，高可用。 

可用性：高可用业界目标5个9，即99.999%。平均一年宕机时长在全年中占比不到0.001%.即315s以下。

单机redis的风险：机器故障+容量瓶颈

### 思想

- 一主多从
- 主可写可读（一般只写），从只读
- 主机master写数据的时候，自动同步到所有的从机slave中
- 从可以设置均衡负载，分摊压力
- 故障恢复，数据备份。

### 工作流程

1. 建立连接阶段
2. 数据同步阶段
3. 命令传播阶段

#### 建立连接

slave连接master（从机发出）

- slaveof <masterIP> <masterPORT>

- redis-server --slaveof <masterIP> <masterPORT>

断开主从（从机发出）

​	slaveof no one  

![img](D:\gitubDATA\Study\docs\中间件\images\主从工作流程.png)



#### 同步数据

问：如果是有多台slave先后加入 ，后续的slave同步也需要先创建RDB吗  还是使用之前的RDB？？？

答：每新加入一个，发送现有的rdb文件给slave.（之前的rdb文件，至于rdb文件什么更新，根据策略来。）

全量复制情况：runid不一致，offset在缓冲区已经找不到了，全新的slave..

1. slave请求同步
2. master创建RDB同步数据（全量复制）
3. slave恢复RDB同步数据( 通过socket接收)
4. slave请求同步增量数据（部分复制）
5. slave恢复增量数据完成。

![img](D:\gitubDATA\Study\docs\中间件\images\数据同步+命令传播.png)

#### 命令传播阶段

//每台slave都有自己唯一的标识runid，请求数据则是根据offset来看还需要取哪些数据。

slave发送replconf ack <offset>	//根据offset请求同步数据

### 复制缓冲区

master对应了很多slave，不可能每一个slave都重新去拿，而是维护了一个**复制缓冲区**，记录了每一次的操作记录（写的 会改变数据的）。

同步的时候，slave会从缓冲区里面拿数据，每个slave可能因为网络原因进度不一样。所以每个slave会单独记录自己的**offset偏移量**。 master也会记录每个slave的offset。

如果offset一样，则继续拿；如果不一样则说明中间断了没收到，则以slave的为准重新拿。



### 心跳机制

即探测双方是否都在线。

master：

- 指令：ping
- 周期：由repl-ping-slave-period决定。默认10s/次
- 作用：判断slave是否在线。
- 查询：INFO replication   //获取最后一次连接时间间隔，lag项维持在0或1则正常

slave：

- 指令：REPLCONF ACK {offset}
- 1s
- 作用1：汇报当前offset，获取最新的数据变更指令
- 作用2：看master是否在线。

### 常见问题

1. master重启，runid重新生成，导致全量复制
   1. 原因：master数据量越来越大，重启之后，runid会重新生成，导致slave需要重新重头获取数据（因为runid改变了）
   2. 优化方案：控制重启后runid不变
2. 复制缓冲区过小，导致全量复制
   1. 优化方案：修改缓冲区大小，repl-backlog-size
   2. 缓冲区大小设定：
      1. 测算master-slave重连的平均时长second
      2. 获取master每秒产生写指令数据总量：write_size_per_second
      3. 则缓冲区大小为：2* second* write_size_per_second
3. 网络不稳定/中断，导致slave和master断开连接
   1. 优化方案：
      1. 调低slave发送replconf ack <offset>的频率
      2. 调高master ping slave的频率
4. 多个slave拿到数据不一致
   1. 原因：网络延迟
   2. 优化方案：
      1. 通常将所有的slave放在一起，最大限度降低网络延迟干扰
      2. 监控主从节点延迟（通过offset判断），如果某些slave延迟过大，则暂时屏蔽访问。（slave-server-stale-data   yes/no）。慎用！！除非对数据一致性要求很高。

## 哨兵

顾名思义，哨兵(sentinel)是一个分布式系统，一个**监控**措施，投票**选择**新的master.

1. 监控：同步信息
2. 通知：保持联通
3. 故障转移：
   1. 发现问题
   2. 竞选负责人
   3. 选择新master
   4. 让其他slave切换到新master。 

s_down：哨兵发现master宕机了，标记状态。

o_down：超过半数发现master宕机了，标记真正的认定宕机。	//这个半数票是可以通过配置文件设置。

## 集群

无中心化，至少需要3主3从才能构成集群。

### 原理

多个master共同分摊请求，负载均衡。处理写请求逻辑是：所有的master都分配自己的区域slots(总共16384 = 16*1024，根据master数量平均分配)。

key来的时候会根据某个算法计算出一个hash值，然后去%16384得到一个值即slots的值（和master里面的slots一样），就知道写哪个了。（**每个master都会维护其他master的分区信息。**）

### master不可用

投票机制来决定。所有的master参与投票，半数以上的master与其他master节点通信超时，则认为master不可用。

### 集群不可用

集群任意master挂掉，且当前master没有slave，集群进入fail。

集群超过半数以上master挂掉，无论有无slave，集群进入fail状态

### 配置集群

--cluster create

--操作数据记得加 **-c**

```txt
======================配置================
//设置加入cluster
cluster-enabled yes|no

//cluster配置文件名
cluster-config-file <filename>

//节点服务响应时间，判断是否该节点已下线
cluster-node-timeout <milliseconds>

//master的最小slave数量
cluster-migration-barrier <count>  


=====================启动==========================
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





# 企业级解决方案

## 缓存预热

### 概念

即才启动服务器，缓存是空的，假设请求瞬间很高，就相当于redis没起作用，压力会很大。所以需要提前准备好数据。

### 方案

根据日常使用情况，将缓存分门别类，根据访问频次优先级考虑，进行准备。

使用固定脚本进行触发。 

## 缓存雪崩

### 概念

在较短的时间内，大量的key过期导致访问不到。发生连续效应

### 方案

**中心思想：多级过滤，稀释集中过期的key**

1. 页面静态化处理，能不访问后台就不访问
2. 构建多级缓存：ngix缓存--》redis缓存--》ehcache缓存
3. mysql查询优化
4. 灾难预警机制：监控redis服务器cpu，内存，响应时间等
5. 限流：牺牲部分用户体验，从源头减少访问。

## 缓存击穿

### 概念

相对于雪崩，要好一些，只是**少数高频**的key过期，导致访问不到而去请求数据库。

可能更常见一些。比如秒杀商品的访问。

### 方案

1. 预先设定其过期时间延长。
2. 其余方案和雪崩类似。
3. 加锁：分布式锁。慎重！？

## 缓存穿透

### 概念

- redis大面积的命中失效
- 非法url访问 //黑客攻击

### 方案

- 白名单，过滤
- 实时监控，比如高峰时段超过正常的50倍，非高峰时段超过5倍即报警
- key加密，过滤

## 性能指标

### 指标

| name                      | description                |      |
| ------------------------- | -------------------------- | ---- |
| latency                   | 响应时长                   |      |
| instantaneous_ops_per_sec | QPS （平均每秒处理请求数） |      |
| hit rate                  | 命中率                     |      |



### 监控方式

- redis-benchmark

  性能测试

```shell
//redis自带的压测，比如100个连接，5000次请求

//注意：和redis-cli类似的命令，**而不是在redis-cli里面执行**。

redis-benchmark  -c 100 -n 5000
```

- monitor

  监控日志

- slowlog  

  慢查询的日志

```shell
slowlog get
  
slowlog len
  
slowlog reset  //重置
```

  
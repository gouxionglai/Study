# Redis

> https://www.cnblogs.com/zuidongfeng/p/8032505.html>

概述：

简单，高效，分布式，基于内存的key-value式的缓存工具。

C语言编写，单线程操作，不会有线程安全问题。

# 使用场景

expire key seconds

1. 限时优惠活动信息  
2. 网站数据缓存（对于一些需要定时更新的数据，比如排行榜）
3. 手机验证码，经常是1分钟内有效
4. 限制网站访客访问频率（1分钟最多访问10次，防止恶意攻击等）

INCR 自增指令 原子性操作

1. 网站统计访问量



# 安装

参考：

<https://www.cnblogs.com/zuidongfeng/p/8032505.html>

<https://blog.csdn.net/u010309394/article/details/81807597>    //设置了密码

# 操作

参考：

<https://www.cnblogs.com/sunzhiqi/p/10869638.html>

<https://www.cnblogs.com/kim-yang/p/10165394.html>	//更深入一些

## 基本类型

**注意key区分大小写**

### string类型

字符串类型是 Redis 中最为基础的数据存储类型，它在 Redis 中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或Json对象描述信息等。

#### 添加(修改)

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



#### 追加

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



#### 获取

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

### hash类型

可以理解成map

- hash⽤于存储对象，对象的结构为属性、值
- 值的类型为string
- 设置单个属性


#### 添加(修改)

```txt
hset [key] [filed] [value]
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

#### 获取

--获取单个 hget

```txt
hget [key] [field]
```

--获取多个hmget

```txt
hmget [key] [field1] [field2]..
```

--获取所有的值

```txt
hvals [key]
```

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

#### 删除

--删除整个key使用通用操作中的del

--删除key下的某个field,  成功会返回删除的数量。如果没有返回0

```txt
hdel [key] [field]
```



### list类型

参考java类型的list

- 列表的元素类型为string
- 按照插⼊顺序排序
- 允许从左侧操作Lxx，也可以从右侧操作RXX。
- **左侧是头部，右侧是尾部**
- **允许value重复**

#### 添加(追加)

//如果有相同的key，是追加操作

```txt
//左侧添加
Lpush [key] [value1] [value2] ...	//先入后出
//右侧添加
Rpush [key] [value1] [value2] ...  //所以Rpush才是我们正常理解的先入先出
```



#### 插入

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



#### 查看

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

#### 获取

出栈操作，获取并删除元素

```txt
LPOP [key]	//弹出左边的元素，返回元素的值  
RPOP [key]	//弹出右边的元素，返回元素的值 
//阻塞式访问 BRPOP 和 BLPOP
//但是POP可能不安全，因为取出来过程中，可能中间环节有问题，比如网络断了，导致并没有到达目的地。所以提供了另一个BRpopLpush
//将source的最后一个元素放到destination的最前面
BRpopLpush [source] [destination]
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



#### 修改

修改指定位置的值

```txt
lset [key] [index] [value]
```

//需要注意的是index的值，记住后添加的序列号最小。index从0开始

#### 截取

可以使用[LTRIM](http://www.redis.cn/commands/ltrim.html)把list从左边截取指定长度。

//注意 没有Rtrim

```
LTRIM [key] [start] [stop]
```



#### 删除

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

### set类型

参考java类型的set。 和list类似，只是没有顺序的概念。

- ⽆序集合
- 元素为string类型
- 元素具有唯⼀性，**不重复**
- 说明：对于集合没有修改操作

#### 添加(追加)

//如果value重复了就不会追加了

```txt
sadd [key] [value1] [value2] [...]
```

#### 查看

```txt
smembers [key]
```

#### 删除

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



### zset类型

相比set多了顺序的概念。

- sorted set，有序集合
- 元素为string类型
- 元素具有唯⼀性，不重复
- 每个元素都会关联⼀个double类型的**score，表示权重**，通过权重将元素从⼩到⼤排序。**小的先取出来。**
- 说明：没有修改操作

#### 添加

```txt
zadd [key] [score1] [member1] [score2] [member2] ...
```

#### 查询

根据下标查询zrange

//因为有顺序，所以查询类似list，而不是set

```txt
zrange [key] [start] [stop]
```



根据权重查询zrangebyscore

//权重是double类型， **比如3，5中间可能不止有一个值，或者根本没有值。**

```txt
zrangescore [key] [min] [max]
```

根据value查询对应的权重

```txt
zscore [key] [member]
```

#### 删除

删除指定元素

```txt
zrem key member1 member2 ...
```

删除指定权重范围的元素

```txt
zremrangebyscore key min max
```







## 通用操作

### 获取所有的key

//*代表匹配后面所有

//? 代表匹配后面一个字符

```txt
keys [匹配规则]
//比如：
keys *
keys h*
keys h? 
```

### 查看某个key是否存在

```txt
exists [key]	//返回1代表存在，0代表不存在
```



### 设置key失效时间

//不设置默认是永久

```txt
expire [key] [seconds]
```

### 查看key的剩余失效时间

--以秒为单位

```txt
ttl [key]
```

```txt
>0	代表具体时间
-1	代表永久有效(不设置是默认)
-2	代表已经失效(节约内存)
```

### 移除key的失效时间

```txt
persist [key]
```



### 查看键值对类型type

//返回对应value的类型

```txt
type [key]
```

### 删除

//不管什么类型都能删除

//支持同时删除多个， 空格隔开

```txt
del [key...]
```

### 自增自减

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



# 编码规范

## 命名规范

- key不要太长，太长消耗内存，并且降低搜索效率
- 增加可读性，key最好通俗易懂，比如user:123:password    代表user是123的密码。value就是具体的密码
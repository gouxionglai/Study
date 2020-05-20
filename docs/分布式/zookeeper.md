# Zookeeper

## 入门

### 概述

文件系统+通知系统

### 特点

- 一领导Leader，多跟随者Follower组成的集群。
- 半数以上节点存活就能工作。
- 全局数据一致。
- 同一节点的更新请求，依次执行。
- 数据原子性，要么成功要么失败。
- 实时性：一定时间范围内，能读取到最新的数据。

### 数据结构

树形结构，每个节点叫ZNode，默认存储1MB数据。每个ZNode都有其唯一标识。

### 应用场景

- 统一命名服务：分布式应用统一命名。比如baidu.com。 肯定是由很多台服务器组建而成，每个有不同的ip地址。但是我们只需要baidu.com即可，而不需要知道具体某一台的ip地址。
- 统一配置管理：配置文件同步。修改一个ZNode，服务器都监听这个ZNode，发现修改后同步进行修改。
- 统一集群管理：将所有服务器节点信息都写入一个ZNode，监听这个ZNode就可以知道服务器节点的变化。
- 服务器节点动态上下线：基本同上，知道其中一个服务器挂掉/启用后，通知客户端可以不访问/访问。
- 软均衡负载

## 安装

### 步骤

1. 下载安装包：前提，需要jdk

2. 解压

   ```shell
   //-zxvf 是解压.gz结尾的包
   //-xvf 是解压.tar结尾的包
   tar -zxvf zookeeper-xxx.tar.gz -c /opt/module/
   ```

3. 修改配置

   ```shell
   //1. 将/opt/module/zookeeper-xxx/config 路径下的zoo_sample.cfg修改为zoo.cfg
   mv zoo_sample.cfg zoo.cfg
   //2. 打开zoo.cfg，修改dataDIR路径
   vim zoo.cfg
   dataDir=/opt/module/zookeeper-xxx/zkData
   //3. 创建zkData文件夹
   mkdir zkData
   
   ```

4. zookeeper 操作

   ```shell
   //启动
   bin/zkServer.sh start
   //停止
   bin/zkServer.sh stop
   //查看状态
   bin/zkServer.sh status
   //显示当前所有java进程pid的命令
   jps
   ```

### 配置文件说明

```shell
#心跳时间 单位毫秒ms
tickTime=2000
#启动超时次数。  所以时间=2s*10
initLimit=10
#每次请求超时次数。  所以时间=2s*5
syncLimit=5
#存放数据目录
dataDir=/opt/module/zookeeper-xxx/zkData
#客户端连接端口
clientPort=2181 
```

## 原理

### 选举机制

半数机制。一半以上节点存活则集群存活。所以zookeeper节点都是单数。

### 节点类型

- 持久：客户端和服务器断开后，创建的节点不删除（由zookeeper给该节点进行编号，顺序递增。）
- 短暂：客户端和服务器断开后，创建的节点自动删除

### stat结构体

### 监听器原理

1. 客户端建立连接是有两个线程，一个connect负责通信，一个listener负责监听。
2. 客户端通过connect发送监听事件给zookeeper。
3. zookeeper接收到监听后添加到监听列表中
4. 监听节点发生变化后，就会发送给Listener线程。内部调用了自己需要编写的process()方法。

常用的监听有：

- 监听节点数据变化：get path [watch]
- 监听子节点增减变化： ls path [watch]

### 写数据流程

节点收到写请求后转发给Leader，然后由Leader分发给所有节点。超过半数成功即成功。

## 实战

### 配置zookeeper集群

比如有三台服务器102,103,104

1. 先在102上安装好zookeeper（同入门安装步骤。），copy file到其他两台服务器（scp 命令）

2. 配置服务器编号：三台服务器上分别在zkData目录下新建myid文件，vim设置各自的编号。（也可以scp后再改）

   ```shell
   #在zookeeper安装目录下新建
   mkdir -p zkData
   #新建myid文件
   touch myid
   #设置编号
   vim myid
   
   ```

3. 配置zoo.cfg文件：同样需要同步到其他节点上。

   ```shell
   #增加如下配置
   #################cluster#####################
   #server.A=B:C:D
   #A即myid里面的值。
   #B即具体哪一台服务器（ip地址）
   #C即该服务器与Leader服务器通信的端口
   #D即leader挂掉后进行选举的端口（替补端口 ，因为leader挂了C肯定断了。）
   server.2=XXXX102:2888:3888
   server.3=XXXX103:2888:3888
   server.4=XXXX104:2888:3888
   ```

   

### 操作命令

| 命令             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| help             | 帮助                                                         |
| ls path [watch]  | 查看当前节点信息                                             |
| ls2 path [watch] | 查看当前节点信息，以及明细信息                               |
| create           | create /nodeName nodeValue 普通创建<br /><br />create -s /nodeName nodeValue  名称带有序列号，自增<br /><br />create -e /nodeName nodeValue 临时节点，客户端断开后消失 |
| get path [watch] | 获取节点的值                                                 |
| set              | 设置节点的值                                                 |
| stat             | 查看节点状态                                                 |
| delete           | 删除节点                                                     |
| rmr              | 递归删除节点                                                 |

### 编程

#### 创建客户端

#### 创建节点

#### 修改节点

#### 删除节点

#### 监听节点


## 面试重点

- zookeeper选举机制

- zookeeper监听原理

- zookeeper部署方式有几种？集群中的角色有哪些？集群最少需要几台服务器
  - 单机+集群
  - Leader + Follower
  - 3
- zookeeper常用命令


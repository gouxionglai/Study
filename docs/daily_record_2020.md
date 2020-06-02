# 每天的进度记录

2020年了 希望效率更好 学习更上一步

# 2020-01-06

从之前到2020-01-06都是在做MFC

记录了bat，mfc的一些知识点。放到了gitub上

<https://github.com/gouxionglai/mfc>

# 2020-01-08

昨天休假了，今天正式拾起前面耽搁的学习任务。从spring mvc开始

# 2020-01-09

实现MFC窗口中运行第三方exe程序。

难点在于createprocess时获取该程序的pid，通过pid去找目标窗口，把窗口挪到mfc中去。

# 2020-01-13

1.spring mvc 上传

2.spring mvc全局异常处理

3.spring mvc 拦截器

4.spring mvc 执行步骤源码分析

# 2020-01-14

1.spring mvc 源码分析 总结

2.mybatis 学习

# 2020-01-15

1.spring mvc 另一种视频查缺补漏

# 2020-01-16

1.spring mvc 另一种视频查缺补漏

# 2020-01-17

1.mybatis源码分析

1. 增删改：一切均调用的是update, 传递的参数不一样而已。

2. 增删改查：最后的底层都是掉原始jdbc的方法， preparestatement.execute()

   1. 传递的参数，经过一系列的封装转换到了preparestatement里面了

3. 连接池：LinkedList

   1. 如果有空闲，每次取第一个连接。
   2. 没有空闲，则看目前的连接数有没有超过最大活动连接数。
      1. 未超过：则直接创建一个新的连接， 加到连接池返回。
      2. 超过了（实际上只可能相等，不可能超过）：则在活动连接中取最早的连接，判断是否超过了timeout时间
   3. 用完后还回去到最后。

4. 事务：

   1. connection.begin();   //开启事务
   2. preparestatement.execute();   //执行目标sql
   3. connection.commit();    //正常执行未报错会提交事务
   4. connection.rollback();  //catch异常了会执行事务回滚

# 2020-01-21

放假回家过年

# 2020-02-03

在家学习spring boot

# 2020-02-12

spring boot学习完毕

# 2020-02-13

中间件：redis

1.学习缓存中间件Redis
	1.1 cenos 安装及操作redis 
	1.2 redis数据类型及操作，重点理解了list类型

# 2020-02-14

​	1.3 java运用redis进行操作
​	1.4 redis使用场景训练

# 2020-02-17

# 2020-03-16



# 2020-03-17

1.增加上帝视角

# 2020-03-18

1.性能优化：遮罩，渲染剔除

# 2020-03-30

1.redis 进阶

# 2020-04-08

第一次感受凌晨3:30的学习   希望up up up

1.redis 完结 （好多知识还是太书面了，感觉实际运用还是摸不到门）

2.rabbitMQ

# 2020-04-14

中间玩了几天

从周一开始看rabbitMQ，预计15号完成

# 2020-04-26

rabbitMQ完成

接下来：多线程

day01 多线程

# 2020-04-27

day02 多线程

多线程视频 ，表示水分很重。没学到什么精华。

明天再找个质量高点的看看吧。

多线程 ！！

# 2020-04-28

day03 多线程   more and more

实际：今天修复线上bug去了 dss双选系统。用的strut2做webmvc  。

问题：有nginx情况下，用户发送给tomcat的请求，并且是代码上有重定向的（struts2配置），正常请求没问题，但是重定向之后端口被改成了80（原端口可能是10080）。导致找不到资源.

```java
<action name="PaperTitleRejectMgrEdit" class="xxxx.dss.PaperTitleAction" method="reject">
    <!--<result name="success" type="redirectAction">PaperTitleAuditMgrList</result>-->   
    <result name="success" type="chain">PaperTitleAuditMgrList</result>
        </action>
```



# 2020-04-29

day04 多线程

# 2020-04-30

day05 多线程 ！必须完成了！！！

//呵呵哒。。。

# 2020-05-01

休假到05-06



# 2020-05-10

多线程其实已经完成了。只是看看其实视频，有么有遗漏的地方。

# 2020-05-18

正式完成多线程。 11-17号什么都没干。。

预计后面一周完成：zookeeper，dubbo, nginx



# 2020-05-19

zookeeper完毕

dubbo ing

# 2020-06-02

面试题

# 2020-06-03

git命令操作


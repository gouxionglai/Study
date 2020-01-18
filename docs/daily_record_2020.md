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

   

   


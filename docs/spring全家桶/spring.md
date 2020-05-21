# Spring

![img](images/spring_framework.png)

## spring hello-world

spring tool suite   //eclipse的sring插件

- 引用maven包

```xml
<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
```

- 配置config.xml

```xml
<bean id="helloworld" class="com.cn.springdemo.bean.Helloworld">
  <property name="name2" value="aha spring test"/>
</bean>
```

  

- 测试

```java
//1 获取容器: 自动将配置文件中定义的bean全部初始化
//已经打印了constructor init...  set name...
ApplicationContext context = new ClassPathXmlApplicationContext("config/applicationContext.xml");

//2 从容器中取出bean
Helloworld helloworld = (Helloworld) context.getBean("helloworld");
```

  

## spring IOC

### 概述

注意：如果注解和xml都配置了bean ，则优先使用xml中配置的信息

IOC:  **即inversion of control .控制反转**。 容器主动将资源推送给所需的组件，组件只需要以合适的形式接收即可。相当于配置<bean>

DI: 即**Dependency Injection. 依赖注入**。比如Controller里面的service, 加上@Autowired注解，则自动注入对应的service，而不用去new。  相当于配置bean中的<property>

```java
@Controller
public class TicketSellController {
    @Resource	//前提是有一个service，如下@service注解
    private TicketRecordService recordService;
```

```java
@Service
public class TicketRecordServiceImpl implements TicketRecordService {
....
}
```

- IOC容器： BeanFactory & ApplicationContext
  - BeanFactory是IOC的基本实现。用于生成任何bean，采用延迟加载，**第一次getBean时才会初始化。**
  - ApplicationContext提供了更多高级特性，**是Beanfacotry的子接口**。ApplicationContext面向使用spring的开发者，几乎所有场合都适用
  - 无论何种方式，配置文件是相同的
- ApplicationContext
  - 实现关系：ApplicationContext-- ConfigurableApplicationContext -- ClassPath/FileSystemXmlApplicationContext
  - ConfigurableApplicationContext 新增两个功能：refresh(), close()启动刷新关闭上下文功能。
  - ClassPathXmlApplicationContext从类路径下获取配置
  - FileSystemXmlApplicationContext从文件路径下获取配置
  - WebApplicationContext专门为web应用准备的，允许从相对于web根目录路径完成初始化。

### bean的种类

- 普通bean：之前操作的都是普通bean。<bean id="" class="A"> ，spring直接创建A实例，并返回
- FactoryBean：是一个特殊的bean，具有工厂生成对象能力，只能生成特定的对象。
  - bean必须使用 FactoryBean接口，此接口提供方法 getObject() 用于获得特定bean。

<bean id="" class="FB"> 先创建FB实例，使用调用getObject()方法，并返回方法的返回值

FB fb = new FB();

return fb.getObject();

- BeanFactory 和 FactoryBean 对比？
  - BeanFactory：工厂，用于生成任意bean。
  - FactoryBean：特殊bean，用于生成另一个特定的bean。例如：ProxyFactoryBean ，此工厂bean用于生产代理。<bean id="" class="....ProxyFactoryBean"> 获得代理对象实例。AOP使用





### 配置bean基于xml
#### 实例化方式

 实例化方式有3种，默认构造，静态工厂构造，实例工厂构造

- 默认构造

  - 要求bean必须有无参构造，不然会报错

  - id: 标记是哪个bean ，所以唯一

  - class: 类的全路径

    ```xml
    <bean id="" class="">
    ```

    

- 静态工厂构造

  - 意义：**常用与spring整合其他框架（工具）**

  - 静态工厂：用于生成实例对象，所有的方法必须是static

    ```xml
    <bean id="实例对象名" class="factory的class" factory-method="静态方法名" >
    ```

- 实例工厂构造

  - 必须先有工厂实例对象，通过实例对象创建对象。提供所有的方法都是“非静态”的。
  - 即先有一个工厂的bean， 再有一个实例对象的bean.（静态工厂只有一个bean）

```xml
<!-- 创建工厂实例 --> 
<bean id="myBeanFactoryId" class="com.itheima.c_inject.c_factory.MyBeanFactory"></bean> 
<!-- 获得userservice * factory-bean 确定工厂实例 * factory-method 确定普通方法 --> 
<bean id="userServiceId" factory-bean="myBeanFactoryId" factory-method="createService"></bean>
```

​      


#### 属性依赖注入

- 属性注入

  - 注入基本数据类型（即<property name="name2" value="aha spring test"/>）

  - 注入引用对象：<property name="car" ref="car2"/>   //前提是bean中已经有一个id=car2

  - 注入集合属性 list  set map等

    ```xml
    <property name="cars">
        <list>
        	<ref bean="car1"/>
            <ref bean="car2"/>
            <ref bean="car3"/>
        </list>
    </property>
    
    <property name="dataMap">
        <map>
            <entry key="A" value="AAAA"/>
            <entry key="B" value="BBB"/>
            <entry key="C" value="CCC"/>
        </map>
    </property>
    
    <property name="dataMap">
        <map>
            <entry key="A" value="AAAA"/>
            <entry key="B" value="BBB"/>
            <entry key="C" value="CCC"/>
        </map>
    </property>
    ```

    

- 构造器注入

  - （即<constructor-arg value="value test" index ="0" type ="java.lang.String"></construtor-arg>）  
  - //index 和 type可不标明。 要求必须有相应的构造方法
  - 特殊字符比如<  > =这些 ，可以使用<![CDATA[xxxxx]]  把特殊的包裹起来。类似转义

- 工厂注入 （不推荐）

#### bean的作用域

即创建的bean的实例个数。

```xml
<bean id="xx" class=""  scope=""></bean>
```

用scope来标记。取值有 singleton， prototype，web环境作用域（request, session, globalSession）

- singleton：单例，默认值，创建容器时自动注入
- prototype：多例，创建容器时不会自动注入，使用时才注入。Struts2就是这样

#### 引用外部属性文件

- 比如数据库配置文件jdbc.properties

```properties
url: jdbc:mysql://localhost:3306/airshow_test?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&useSSL=FALSE
username: airshow_dba
password: e7Qaslntoilsedet1rQnsitMc8Aaogcdna9
driverClassName: com.mysql.cj.jdbc.Driver
```

```xml
    <!-- 加载配置文件-->
    <context:property-placeholder location="classpath:config/jdbc.properties"/>

    <!--如果是多个配置文件 则采用逗号分隔
    <context:property-placeholder location="classpath:config/jdbc.properties,classpath:config/redis-config.properties"/>
    -->
 <!--   多个文件 还可以修改成如下形式
<bean id="propertyConfigurer"	  class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:jdbc.properties</value>
				<value>classpath:quartzJobConfig.properties</value>
			</list>
		</property>
	</bean>
-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
        <property name="driverClassName" value="${driverClassName}"/>
    </bean>
```



#### spEL


  - 类似EL表达式   使用<property name="price" value="#{T(java.lang.Math).PI*80}">


#### bean的生命周期

```xml
<bean id="helloworld" 
      class="com.cn.springdemo.bean.Helloworld" 
      init-method="init" 
      destroy-method="destroy">
```


  - 含义：主动管理bean的生命周期
  - 过程：

    - 通过构造器或工厂方法创建bean实例	<bean>
    - 为bean的属性设置值和对其他bean的引用  <property>
    - 调用bean的初始化方法
    - 创建完成
    - 关闭容器时，调用bean的销毁方法


#### 后处理bean：BeanPostProcessor

主要有两个方法：

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```



- spring 提供一种机制，只要实现此接口BeanPostProcessor，并将实现类提供给spring容器，spring容器将自动执行，在初始化方法前执行before()，在初始化方法后执行after() 。 配置<bean class="">

- Factory hook(勾子) that allows for custom modification of new bean instances, e.g. checking for marker interfaces or wrapping them with proxies.

- spring提供工厂勾子，用于修改实例对象，可以生成代理对象，是AOP底层。

- 模拟代码情况

  ```txt
  A a =new A();
  a = B.before(a) --> 将a的实例对象传递给后处理bean，可以生成代理对象并返回。
  a.init();
  a = B.after(a);
  a.addUser(); //生成代理对象，目的在目标方法前后执行（例如：开启事务、提交事务）
  a.destroy()
  ```

  

### 配置bean基于注解

- xml中需要配置待扫描的包 <context:component-scan>

- xml中需要配置不扫描目标类 <context:exclude-filter>     //放在component-scan里面

- xml中需要配置子节点要包含的目标类 <context:include-filter>     //只匹配这个。 放在component-scan里面  需要配合use-default-filters=“false”使用

  ```xml
  <!-- 开启注解
  扫描包以及子包：base-package ="com.cn.springdemo.annotation"
  匹配包名：resource-pattern="annotation/*.class"
  -->
      <context:component-scan base-package="com.cn.springdemo.annotation">
  	<!--不扫描Service注解-->
      <!--<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>-->
      <!--只扫描Controller注解  和不扫描互斥，并且需要再component-scan 配置use-default-filters="false" -->
          <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
      </context:component-scan>
  ```




## spring AOP

切面编程。最常见的例子就是日志模块   前置or后置通知  ；还有参数校验之类的

### **AOP原理**

其本质是动态代理

```java
InvocationHandler handler = new InvocationHandler() {
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            //前置通知在这里 @Before
            Object result = null;
            try {
                result = method.invoke(args);
                //返回通知，能拿到返回值 @AfterReturning
            } catch (Exception e) {
                e.printStackTrace();
                //异常通知 @AfterThrowing
            }
            //后置通知，可能没有返回值 @After
            return result;
        }
    };
```



### AOP术语

什么时候，执行什么， 在哪里执行，

1. target目标类：需要被代理的类。例如：UserService
2. Joinpoint连接点：所谓连接点是指那些可能被拦截到的方法。例如：所有的方法
3. PointCut切入点：已经被增强的连接点。例如：addUser()
4. advice通知/增强，增强代码。例如：after、before
5. Weaving织入：是指把增强advice应用到目标对象target来创建新的代理对象proxy的过程.
6. proxy代理类
7. Aspect切面：是切入点pointcut和通知advice的结合

### 使用步骤

#### 注解aop

```txt
@Aspect  声明切面，修饰切面类，从而获得 通知。
通知
	@Before 前置
	@AfterReturning 后置
	@Around 环绕
	@AfterThrowing 抛出异常
	@After 最终
切入点
	@PointCut ，修饰方法 private void xxx(){}  之后通过“方法名”获得切入点引用

```



1. maven中加入aspectJ的相关包

2. xml中开启aop自动代理

   ```xml
       <!--开启自动aop注解-->
       <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
       <context:component-scan base-package="com.cn.springdemo"/>
   ```

3. 切面编程

   1. 声明容器
   2. 声明AOP切面
   3. 定义切入点即Poitcut(execution(表达式))
   4. 具体的通知方法：前置,后置,xxx

   ```java
   //声明需要加入ioc容器
   @Component
   //声明是一个切面
   @Aspect
   public class MathicAspect {
       //定义切入点
       @Pointcut("execution(* com.cn.springdemo.aop.*.*(..))")
       private void loggingPoint(){
   
       }
   	//前置通知
       //通知的地方为execution中匹配的方法
   //    @Before("execution(* com.cn.springdemo.aop.*.*(..))")
       @Before("loggingPoint()")
       public void beforeMethod(JoinPoint joinPoint) {
           String methodName = joinPoint.getSignature().getName();
           List<Object> args = Arrays.asList(joinPoint.getArgs());
           System.out.println(methodName + "method start with args " + args);
           System.out.println("aop Before");
       }
   
       //后置通知
       @After("loggingPoint()")
       public void afterMethod(JoinPoint joinPoint) {
           System.out.println("aop After");
       }
   
       //返回值通知
       @AfterReturning(pointcut = "loggingPoint()")
       public void afterReturningMethod(JoinPoint joinPoint) {
           System.out.println("aop AfterReturning");
       }
   
       //异常通知
       @AfterThrowing(pointcut = "loggingPoint()", throwing = "e")
       public void afterThrowingMethod(JoinPoint joinPoint, Exception e) {
           System.out.println("aop AfterThrowing:" + e);
       }
       //全能型选手，环绕通知
       @Around("loggingPoint()")
       public Object aroundMethod(ProceedingJoinPoint joinPoint) {
           Object obj = null;
           try {
               //前置通知
               System.out.println("around before");
               //目标方法执行，必须手动执行
               obj = joinPoint.proceed();
               //返回通知
               System.out.println("around afterReturning"+ obj);
           } catch (Throwable throwable) {
               //异常通知
               System.out.println("around throwing"+ throwable);
               throwable.printStackTrace();
           }
           //后置通知
           System.out.println("around after");
           return obj;
       }
   }
   ```



#### xml aop

1. aspect：同注解aspect，只是不需要那些注解。将注解信息挪到了xml中配置

   

2. xml

```xml
<!--1 容器中加入目标类-->
    <bean id="mathic" class="com.cn.springdemo.aop.Mathic"/>
    <!--2 容器中加入切面类-->
    <bean id="mathicAspect" class="com.cn.springdemo.aopConfig.MathicAspect"/>

    <!--3 关联关系-->
    <aop:config >
        <!--定义切入点-->
        <aop:pointcut id="pointcut" expression="execution(* com.cn.springdemo.aop.Mathic.*(..))"/>
        <!--4 定义切面-->
        <aop:aspect ref="mathicAspect">
            <!--某一个面：定义什么通知使用在什么切入点-->
            <aop:before method="beforeMethod" pointcut-ref="pointcut"/>
            <aop:after method="afterMethod" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>
```



### 切面优先级

存在多个切面的时候，会按照一定的顺序执行。 @Order(int x) //值越小优先级越高

-- xml配置形式进行切面编程。 忽略。。

## spring jdbcTemplate

比较原生 不推荐使用，建议使用Mybatis   

## spring 事务

### 定义:acid

- 原子性：整体
- 一致性：完成
- 隔离性：并发
- 持久性：结果

### 事务传播行为

| 传播行为                  | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |

### 隔离级别

#### 几种常见现象解释：

- 脏读（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。
- 不可重复读（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
- 幻读（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

#### 隔离级别定义

| 隔离级别                   | 含义                                                         | 脏读 | 不可重复读 | 幻读 |
| -------------------------- | ------------------------------------------------------------ | ---- | ---------- | ---- |
| ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别（mysql默认是read_repeatable，同一事务里面可以读取到未提交的数据）（oracle默认是read_committed,也就是读已提交） |      |            |      |
| ISOLATION_READ_UNCOMMITTED | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读 | yes  | yes        | yes  |
| ISOLATION_READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生 | no   | yes        | yes  |
| ISOLATION_REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 | no   | no         | yes  |
| ISOLATION_SERIALIZABLE     | 最高的隔离级别，完全服从ACID的隔离级别。强制的进行排序，在每个读读数据行上添加共享锁。**会导致大量超时现象和锁竞争。** | no   | no         | no   |

### tx形式声明事务

主要在xml中配置。

1. xml中定义事务管理器  <bean id="txManager"  ...>
2. tx标签配置拦截器 <tx:advice>  + <aop:config>

### 注解形式声明事务 

xml中定义事务管理器，然后在java方法上添加@Transactional即可  略。


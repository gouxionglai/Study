# SpringCloud

学习视频：<https://www.bilibili.com/video/BV18E411x7eT?p=1>

0-4：0基础

5-9：初级

10-16：中级

17-20：高级



*CAP*原则又称*CAP*定理，指的是在一个分布式系统中，一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）。*CAP* 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。



![image](images\spring_cloud构成.png)

**springcloud和springboot版本如何选择？**

官网<https://spring.io/projects/spring-cloud>

springboot最新版本是2.3.1  （截至2020.06.30）但是并不和springcloud匹配。

![image](images\springboot_springcloud.png)

taggle test 按钮

# 微服务步骤

1. 建module：父module package= pom, N个子module  package = jar
2. 改pom： 依赖 
3. 写YML：  server.port  每个服务有不同的port
4. 主启动：@SpringbootApplication
5. 业务类：controller, service....

例子：消费者，生产者，公共实体类，拆分这三个module， 两个微服务，通过RestTemplate完成服务调用。基本架构完成。

RestConfig.java

```java
@Configuration
public class RestConfig {

    @LoadBalanced	//负载均衡用的，当微服务提供者是集群的时候使用
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

XXController.java

```java
@RequestMapping("/order")
@RestController
public class OrderController {
    private Logger logger = LoggerFactory.getLogger(OrderController.class);

    @Autowired
    private RestTemplate restTemplate;
    //未注册的时候
	private static final String PAYMENT_URL = "http://127.0.0.1：8001/api";
    //服务注册之后
    //private static final String PAYMENT_URL = "http://cloud-payment-service/api";

    @GetMapping("/payment/get/{id}")
    public CommonResult<Payment> getByID(@PathVariable("id") Long id){
        //restTemplate相当于模拟了一个postMan的请求，包装了一下
        return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id, CommonResult.class);
    }

}
```







# 服务注册中心

1. 解耦服务之间的依赖关系（感觉就像DI 依赖注入，并不需要自己去关联服务，由服务注册中心完成。而且重点是微服务一般是多台，都有不同的ip和端口。如果注册了就有一个统一的名字，而用户并不关心具体是哪个服务。方便维护。）
2. 对服务的管理（将挂掉的服务去除，保留好的服务）

## Eureka

**保证AP**

1. 服务注册

   新上线的服务通过REST请求方式向Eureka Server注册服务，提供ip,port,index等信息，Eureka收到后会存入双层map中

2. 服务续约

   维护服务状态。通过心跳机制，默认每隔30s请求一次map中存放的服务信息，看是否存活。

3. 服务同步

   Eureka server不仅仅一台，可能是一个集群，所以集群之间会存在信息同步，保持一致性。

4. 自我保护

   即有一种情况，Eureka发现怎么存储map里面的server都不通了（15mins内 服务失效比例>=85%）。按逻辑是不是应该都移除。但是自我保护就认为并不是他们不同了 ，而是自己的原因比如网断了，会暂时保留一定时间在重新试试。

   

### 服务器配置

注册中心集群的话，就是配置多台相互注册即可。

启动类增加@EnableEurekaServer注解

```yaml
server:
  port: 7001
eureka:
  instance:
    #集群 要用name而不能用ip
    hostname: eureka7001.com
  #单机hostname: localhost #eureka服务端的实例名称
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要检索服务
    fetch-registry: false
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      # 单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 相互注册 注册其他机器
      defaultZone: http://eureka7002.com:7002/eureka/
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server:
    #关闭自我保护模式，保证不可用服务被及时删除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```



### 客户端配置

注意：在注册中心看来，所有来注册的都是客户端（即使是微服务提供者也是客户端）

启动类增加@EnableEurekaClient注解

注意：如果要支付服务集群，spring.application.name必须一样，ip或者port不一样即可。其余一样。

```yaml
server:
  port: 8003
  servlet:
    session:
      timeout: 3000m
      persistent: true
    context-path: /api

spring:
  application:
  	#这个是注册到服务中心的名字，对外提供服务的请求地址
    name: cloud-payment-service
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka/ #单机
      #集群
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: payment8003
    prefer-ip-address: true
```



## Zookeeper

**保证CP**

## Consul

**保证CP**

### 安装

超级简单，官方下载后，console里面执行: consul agent -dev  

### 配置

#### 客户端配置

```yaml
#前提需要导包pom: spring-cloud-starter-consul-discovery
server:
  port: 80
spring:
  application:
    name: consul-consumer-order
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```



注册成功后会显示，断开服务后立马提示节点断开。保证CP（数据一致性）

![img](images\consul.png)





# 负载均衡

集中式LB：F5或者nginx

进程内LB：ribbon



## Ribbon

客户端的负载均衡组件

### IRule

根据特定算法中从注册的服务列表选取一个进行访问。

默认是7种规则

![image](images\loadbalance策略.png)

- 轮询 RoundRobinRule：依次遍历服务列表进行访问。公式： rest请求次数 % 总服务数 =  访问的index。  即 list.get(index)。 rest请求数从1开始（里面用了CAS来保证原子性）
- 随机 RandomRule：核心是ThreadLocalRandom.current().nextInt(serverCount);  //ThreadLocalRandom继承了Random类
- 最小并发 BestAvailableRule：
- 最可靠 AvailabilityFilteringRule：
- 请求时间加权 WeightedResponseTimeRule：
- 重试机制 RetryRule：
- 复合判断 ZoneAvoidanceRule：性能好 + 连接数少 





### 负载均衡算法

## LoadBalancer



# 服务调用

## OpenFeign

更加面向接口开发

```java
//启动类： OpenFeignOrderApplication
@EnableFeignClients //启动OpenFeign
@EnableDiscoveryClient  //服务注册
@SpringBootApplication
public class OpenFeignOrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OpenFeignOrderApplication.class,args);
    }
}

//OrderController
@RequestMapping("/order")
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService;
    @GetMapping("/get/{id}")
    public CommonResult<Payment> getByID(@PathVariable("id") Long id){
        return orderService.getByID(id);
    }
}

//OrderService
@FeignClient("cloud-payment-service")	//指明调用的是哪个微服务
@Service
public interface OrderService {
    //这样的话 就直接调用对应服务的该方法
    //但是这里的mapping地址也要像之前controller里面restTemplate调用一样，完整的url
    //比如有context-path: /api 的都要补全
    @GetMapping("/api/payment/get/{id}")
    CommonResult<Payment> getByID(@PathVariable("id") Long id);
}


//详细日志  需要结合配置开启日志
@Configuration
public class FeinConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return  Logger.Level.FULL;
    }
}

```

配置

```yml
#设置openFeign客户端超时时间（底层用的ribbon）
ribbon:
  #建立连接超时时间  单位毫秒
  ReadTimeout: 5000
  #指建立连接后获取资源超时时间  默认1s
  ConnectTimeout: 5000
logging:
  level:
    com.gxl.study.springcloud.service: debug
```

日志效果： 非常详细

```log
 [OrderService#getByID] <--- HTTP/1.1 200 (51ms)
 [OrderService#getByID] connection: keep-alive
 [OrderService#getByID] content-type: application/json;charset=UTF-8
 [OrderService#getByID] date: Wed, 29 Jul 2020 07:31:32 GMT
 [OrderService#getByID] keep-alive: timeout=60
 [OrderService#getByID] transfer-encoding: chunked
 [OrderService#getByID] 
 [OrderService#getByID] {"code":200,"message":"query success!!port:8001","data":{"id":4,"serial":"qwer12356"}}
 [OrderService#getByID] <--- END HTTP (86-byte body)
 Flipping property: cloud-payment-service.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
 Resolving eureka endpoints via configuration
```



# 断路器

许多依赖不可避免会调用失败，比如超时、异常等。此时就应该有一种机制，一个故障了不会导致整个故障。

## Hystrix

即能提供断路器一样的开关装置，故障后仍能返回一个符合预期的响应而不是一直等待到超时或者直接抛出异常。

### 降级

fallBack：超时不等待，以及兜底的措施。就像if else if  else{}的最后的else。  不管怎么判断都会有一个兜底的。或者说是trycatch,  始终有个兜底的finally

```java
//和所有功能一样，都需要在启动类上标注
@EnableCircuitBreaker


//针对某一个服务方法降级
@HystrixCommand(fallbackMethod = "timeOut_TimeoutFallbackHandler", commandProperties = {
    //触发条件：超时>=value
    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "5000")
})

//针对所有默认降级策略，但是必须要配合@HystrixCommand一起才有用。  标注在controller上
@DefaultProperties(defaultFallback="global_fallBackHandler")
//解耦：在调用方法里面指定降级的方法。就不用写在controller
@FeignClient(value = "cloud-provider-hystrix-payment",fallback = OrderServiceHystrixImpl.class)
```



### 熔断

break：顾名思义，达到某些条件后，触发的功能即熔断。就像保险丝的功能。

注意：和降级差别在于，第一。降级是固定的结果，而熔断是一个判断，在熔断打开之后才直接降级。如果没有触发熔断，是不会直接走到降级的。（虽然一些其他异常或者宕机会导致走到降级，但已经属于降级范畴了）。

第二。熔断是有一个**自我恢复**的机制在里面。



```java
@HystrixCommand(fallbackMethod = "circuitBreak_FallbackHandler", commandProperties = {
    //触发条件：
    // 3个参数放在一起，所表达的意思就是：
    //每当10个请求中，有60%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。直到20s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开。
    //开启断路器
    @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),
    //请求数达到value之后开始执行判断是否需要熔断 10次
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
    //休眠时间窗10000ms ，即多少时间之后服务重新可用
    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "20000"),
    //错误率达到多少跳闸  60%
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60")
})
```



### 限流

followLimit：高并发的时候，控制QPSs不能超过多少，自动排队。



## Sentinel



# 服务网关

下图相当形象的说明了分布式技术的组合。其中Gateway充当的是网关这个角色。

![image](images\gateway.png)

## Zuul2

//不维护了 



## gateway

路由转发 + 执行过滤连（一系列的filter）（在请求执行前，执行后都可以做处理）
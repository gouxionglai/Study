# SpringCloud

学习视频：<https://www.bilibili.com/video/BV18E411x7eT?p=1>

0-4：0基础

5-9：初级

10-16：中级

17-20：高级

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


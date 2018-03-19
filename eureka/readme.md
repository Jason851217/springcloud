##### 在默认情况下，Eureka会将自己也作为客户端尝试注册，所以在单机模式下，我们需要禁止该行为
```applicaiton.yml配置
  server:
      port: 8761  # 指定该Eureka实例的端口

  eureka:
      instance:
        hostname: localhost   # 指定该Eureka实例的主机名
      client:
        service-url:
           defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
        register-with-eureka: false # 禁止EurekaServer用Erueka注册
        fetch-registry: false #禁止抓取注册信息
  spring:
      application:
         name: discovery  #serviceId
```

##### eureka.instance.appname 与 spring.application.name 的区别
   
   - eureka.instance.appname
```
   找到类 org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean.java，其中有段：

    @Data
    @ConfigurationProperties("eureka.instance")
    public class EurekaInstanceConfigBean implements CloudEurekaInstanceConfig, EnvironmentAware {

    private static final String UNKNOWN = "unknown";

    /**
     * Get the name of the application to be registered with eureka.
     */
    private String appname = UNKNOWN;

可以看到，应用名称 就是在这里配置的；
```
  
   - spring.application.name
   
```
  找到类 org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean，其中片段：

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
        // set some defaults from the environment, but allow the defaults to use relaxed binding
        RelaxedPropertyResolver springPropertyResolver = new RelaxedPropertyResolver(this.environment, "spring.application.");
        String springAppName = springPropertyResolver.getProperty("name");
        if(StringUtils.hasText(springAppName)) {
            setAppname(springAppName);
            setVirtualHostName(springAppName);
            setSecureVirtualHostName(springAppName);
        }
    }

可以看到，这里是用 spring.application.name 配置的应用名称；
```
   - 结论
   
```
从以上可以看到，spring.application.name 的优先级比 eureka.instance.appname 高，例如：

    spring:
      application:
        name: jack
    eureka:
      client:
        serviceUrl:
          defaultZone: http://localhost:8761/eureka/
      instance:
        appname: client

两者都配置的时候，注册到Eureka Server上的 appname 是 jack
```

- Eureka的高可用：
  1. 多个Eureka之间相互注册(>3个Eureka时需要两两注册，把自己注册到非本身的Eureka Server上)
  2. 指定服务提供者的eureka.client.serviceUrl.defaultZone为多个url地址，之间用","隔开
  
  
- Eureka总结
   - @EnableEurekaServer  @EnableDiscoveryClient  @EnableEurekaClient  
     - spring cloud中discovery service有许多种实现（eureka、consul、zookeeper等等），@EnableDiscoveryClient基于spring-cloud-commons, @EnableEurekaClient基于spring-cloud-netflix。其实用更简单的话来说，就是如果选用的注册中心是eureka，那么就推荐@EnableEurekaClient，如果是其他的注册中心，那么推荐使用@EnableDiscoveryClient
   - EurekaServer会提供服务注册的功能，各个微服务启动后，会在EurekaServer中注册，这样EurekaServer就有了所有微服务节点的信息
   - 心跳检测(如果在系统的运行期间，某个服务实例挂了，也就是在规定的时间内没有发送心跳信号，这个服务实例就会被Eureka剔除掉)、健康检查、负载均衡等功能
   - Eureka的高可用，生产上建议至少两台以上
   - 在分布式系统中，服务注册中心是最重要的基础部分
   
- 服务注册与发现——Eureka
Eureka Server：服务的注册中心，负责维护注册的服务列表。作为一个独立的部署单元，以REST API的形式为服务实例提供了注册、管理和查询等操作.
Service Provider：服务提供方，作为一个Eureka Client，向Eureka Server做服务注册、续约和下线等操作，注册的主要数据包括服务名、机器ip、端口号、域名等等。
Service Consumer：服务消费方，作为一个Eureka Client，向Eureka Server获取Service Provider的注册信息，并通过远程调用与Service Provider进行通信。


属性配置相关类
服务注册相关配置:eureka.instance 为前缀

org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean
服务实例相关配置:eureka.client 为前缀

org.springframework.cloud.netflix.eureka.EurekaClientConfigBean
服务配置属性以eureka.server开头

org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean


 

Eureka Server 常用属性

//将IP注册到Eureka Server上，如果不配置就是机器的主机名
eureka.instance.prefer-ip-address=true

//设为false，关闭自我保护
eureka.server.enable-self-preservation=false

//表示是否将自己注册到Eureka Server，默认为true
eureka.client.register-with-eureka=false

//表示是否从Eureka Server获取注册信息，默认为true
eureka.client.fetch-registry=false

// 扫描失效服务的间隔时间（单位毫秒，默认是60*1000）即60秒
eureka.server.eviction-interval-timer-in-ms=5000

//设置 eureka server同步失败的等待时间 默认 5分
//在这期间，它不向客户端提供服务注册信息
eureka.server.wait-time-in-ms-when-sync-empty=5

//设置 eureka server同步失败的重试次数 默认为 5 次
eureka.server.number-of-replication-retries=5

//自我保护系数（默认0.85）
eureka.server.renewal-percent-threshold=0.49
 
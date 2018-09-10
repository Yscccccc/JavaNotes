# Spring Cloud学习笔记

spring cloud利用spring boot 简化构建分布式应用

- 微服务是一种架构风格
- 微服务由一系列微小的服务共同组成
- 跑在自己的进程里
- 每个服务为独立的业务开发
- 独立部署
- 分布式的管理

## 单体架构

（并不是说单体架构就是不能是分布式）

单体架构的优点

- 容易测试
- 容易部署

单体架构的缺点

- 开发效率低
- 代码维护难
- 部署不灵活（构建不灵活）
- 稳定性不高
- 扩展性不够

## 分布式架构

（微服务架构必定是分布式的）

旨在支持应用程序和服务的开发，可以利用物理架构由多个自治的处理元素，不共享主内存，但通过网络发送消息合作。

														--Leslie Lamport

### 微服务架构的基础框架/组件

- 服务注册发现
- 服务网关
- 后端通用服务（也成中间层服务Middle Tier Service）
- 前端服务（也称边缘服务Edge Service）



## Spring Cloud是什么

spring cloud是一个开发工具集，含多个子项目

- 利用spring boot 的开发便利
- 主要是基于对Netflix开源组件的进一步封装

spring Cloud简化了分布式开发



## Spring Cloud Eureka

- 基于Netfix Euraka做了二次封装
- 两个组件组成：

1. Eureka Server 注册中心
2. Eureka Client  服务注册



### Eureka Server

- 心跳检测、健康检查、负载均衡等功能
- Eureka高可用，生产上建议至少两台以上
- 分布式系统中，服务注册中心是最重要的基础部分

引用依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

在启动类加上@EnableEurekaServer注解

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

application.yml配置

register-with-eureka: false(关闭注册自己)

server.enable-self-preservation 配置关闭自我保护,并按需配置Eureka *Server*清理无效节点的时间间隔

```y&#39;m
server:
  port: 8761
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    register-with-eureka: false
  server:
    enable-self-preservation: false
spring:
  application:
    name: eureka
```

### Euraka Client

引用依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

在启动类加上@EnableDiscoveryClient注解

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

application.yml配置

instance.hostname 自定义链接

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  #instance:
    #hostname: clientName
spring:
  application:
    name: client
```

### 高可用

启用多台Euraka Server互相注册，Eureka Client注册多台Euraka Server

eureka server配置

- euraka1 配置端口8761,注册到2，3
- 同理euraka2配置端口8762,注册到1，3
- euraka3配置端口8763,注册到1，2

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8762/eureka/,http://localhost:8763/eureka/
    register-with-eureka: false
  server:
    enable-self-preservation: false
spring:
  application:
    name: eureka
```

euraka client配置

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/
spring:
  application:
    name: client
```

### 分布式系统中为什么需要服务发现

服务发现的两种方式

- 客户端发现（eureka采用）
- 服务端发现: 使用代理（nginx,zookeeper,kubernetes）

微服务的特点：异构

- 不同语言
- 不同类型的数据库

spring cloud的服务调用方式

- rest (spring cloud采用)
- rpc

## 业务形态不适合微服务

- 系统中包含很多强事务场景的
- 业务相对稳定，迭代周期长
- 访问压力不大，可用性要求不高

## 康威定律

任何组织在设计一套系统（广义概念上的系统）时，所交付的设计方案在结构上都与该组织的沟通结构保持一致。--沟通的问题会影响系统的设计

## 微服务的特点

- 一系列微小的服务共同组成
- 单独部署，跑在自己的进程里
- 每个服务为独立的业务开发
- 分布式的管理

## 服务拆分的方法论

拓展立方模型（Scale Cube）

- X 轴   水平复制
- Z 轴   数据分区
- Y 轴   功能解耦

如何拆“功能”

- 单一职责

- 松耦合（服务之间耦合度低，修改一个服务不用导致另外一个服务的更改）

- 高内聚（服务内部相关的行为都聚集在一个内部，而不是分散在别的服务中）

- 关注点分离

  （

      1.按职责：给服务进行分类

      2.按通用性：一些基础组件与业务无关的也可以划分出来作为一个服务

      3.按粒度级别

  ）

服务和数据的关系

- 先考虑业务功能，在考虑数据
- 无状态服务（状态：如果一个数据要被多个服务共享才能完成一个请求）

如何拆“数据”

- 每个微服务都有单独的数据存储
- 依据服务的特点选择不同结构的数据库类型

## 应用间通信

RPC(Dubbo)

HTTP(Spring Cloud)



spring cloud 中服务间两种调用方式

- RestTemplate
- Feign

客户端负载均衡器：Ribbon(基于netflix的ribbon实现)

- RestTemplate（使用LoadBalanceClient或者@LoadBalanced注解）
- Feign
- Zuul

## Ribbon（客户端负载均衡）

- 服务发现（发现依赖服务的列表）
- 服务选择规则（依据规则，在多种服务中选择有效的服务）
- 服务监听（检测失效的服务，做到高效剔除）

主要组件：

- ServerList
- IRule
- ServerListFilter

(首先通过ServerList获取服务列表，然后通过ServerListFilter过滤掉一些地址，最后通过IRule选择一个实例作为最终目标结果）

## Feign（声明式服务调用）

- 声明式REST客户端（伪RPC）
- 采用了基于接口的注解

使用：

1.在项目引入依赖 spring-cloud-starter-openfeign

2.在启动类加@EnableFeignClients注解

3.@FeignClient注解在一个接口上，声明式接口

## 为什么需要统一配置中心

- 不方便维护
- 配置内容安全与权限
- 更新配置项目需要重启

（在远端git拉取配置文件到本地的git，为各个服务进行调用）

server使用：

1.添加spring-cloud-config-server依赖，在启动类上加@EnableConfigServer注解

2.配置

```
spring:
  application:
    name: config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Yscccccc/config-repo
          username: xxx@xx.com
          password: xxx
          basedir: E:\IDEA\project\config\basedir
  rabbitmq:
    host: 192.168.99.100
    port: 5672
    username: guest
    password: guest
eureka:
  client:
    service-url:
     defaultZone: http://localhost:8761/eureka/
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

3. 访问

    /{name}-{profiles}.yml

   /{label}/{name}-{profiles}.yml

   name:服务名

   profiles:环境

   label:分支{branch}

client使用：

1.引入依赖：spring-cloud-config-client，在启动类加@EnableDiscoveryClient注解

2.将application.yml改成bootstrap.yml,表示先配置这个文件

```
spring:
  application:
    name: order
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG
      profile: test
```

3.使用ribbon进行负载均衡

4.使用spring cloud Bus 进行消息通知

在哪使用拉取的配置要加@RefreshScope

暴露接口地址/bus-fresh

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

5.使用webhook调用/monitor接口进行动态更新

## 消息总线：Spring Cloud Bus

使用：

1.引入依赖spring-cloud-starter-bus-amqp

## 消息驱动的微服务：Spring Cloud Stream

- rabbitMQ

  依赖：spring-cloud-starter-stream-rabbit

- kafka

## 异步

客户端请求不会阻塞进程，服务端的响应可以是非即时的



异步的常见形态：

- 通知（单项请求）
- 请求/异步响应（客户端发送请求到服务端，服务端异步响应请求，客户端不会阻塞，不会立刻收到响应）
- 消息

## MQ

MQ应用场景

- 异步处理
- 流量削峰
- 日志处理（kafka）
- 应用解耦

## 服务网关：Zuul

服务网关的要素：

- 稳定性，高可用
- 性能，并发性
- 安全性
- 拓展性

常用的网关方案：

- Nginx+Lua
- kong
- Tyk
- Spring Cloud Zuul(netflix出品基于jvm路由和服务端的负载均衡器)

Zuul的特点

- 路由+过滤器=Zuul
- 核心是一系列的过滤器

Zuul的四种过滤器API

- 前置（Pre）
- 路由（Route）
- 后置（Post）
- 错误（Error）

应用场景

1.前置（Pre）

- 限流
- 鉴权
- 参数校验调整

2.后置（post）

- 统计
- 日志

Zuul的高可用

- 多个Zuul节点注册到Eureka Server
- Nginx和Zuul "混搭"

Zuul：权限校验

- 在前置过滤器中实现相关的逻辑
- 分布式session 和 OAuth2

Zuul:跨域

- 在被调用的类或者方法上增加@CrossOrigin注解
- 在Zuul里增加CorsFilter过滤器

## 服务容错保护：Hystrix

防止雪崩利器

基于Netflix对应的Hystrix

使用：

在服务调用方引入依赖spring-cloud-starter-netflix-hystrix

在启动类上加@EnableCircuitBreaker

功能：

- 服务降级

1. 优先核心服务，非核心服务不可用或弱可用
2. 通过@HystrixCommand注解指定
3. fallbackMethod(回退函数)中具体实现降级逻辑

- 服务熔断
- 依赖隔离
- 监控



依赖隔离

- 线程池隔离（Hystrix为每个HystrixCommand创建一个独立的线程池，在某个被HystrixCommand包装下的服务出现了延迟过高的情况，也只是对该依赖的服务产生影响，不会拖慢其他服务）
- Hystrix自动实现了依赖隔离



服务熔断

circuitBreaker.enabled 设置熔断

circuitBreaker.requestVolumeThreshold 设置最小请求数

circuitBreaker.sleepWindowInMilliseconds 设置休眠时间窗

circuitBreaker.errorThresholdPercentage 设置错误百分比

circuitBreaker：断路器



结合feign使用Hystrix

配置

```
feign:
  hystrix:
    enabled: true
```

在@FeginClient 设置 fallback参数



使用HystrixDashboard：在启动类上加@EnableHystrixDashboard注解

## 分布式服务追踪: spring cloud Sleuth

链路监控：spring cloud sleuth

使用：

引入依赖spring-cloud-starter-sleuth

可视化依赖：spring-cloud-sleuth-zipkin

配置

```
spring:
  zipkin:
    base-url: http://192.168.99.100:9411/
  sleuth:
    sampler:
      probability: 1
```



## 分布式追踪系统

核心步骤：

- 数据采集(由于各种系统的api各不兼容，如果切换系统往往带来较大的改动，解决-->OpenTracing)

1. OpenTraing:来自CNCF（优势：提供平台无关，厂商无关的api,使得开发人员可以快速的添加或者更换厂商的实现）

- 数据存储
- 查询

Annotation(即时记录事件)

事件类型：

	cs(Client Send): 客户端发起请求的时间

	cr(Client Received): 客户端收到处理完请求的时间

	ss(Server Send): 服务端处理完逻辑的时间

	sr(Server Received): 服务端收到调用端请求的时间

（客户端调用时间=cr-cs，服务端处理时间=sr-ss）



## zipkin

traceId(全局的跟踪Id,是跟踪的入口点)

spanId(下一层的请求Id,一个traceId可以包括一个以上的spanId)

parentId(上一次的请求Id，用来将前后的请求串联起来)


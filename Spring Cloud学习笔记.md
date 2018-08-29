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
- 服务端发现（nginx,zookeeper,kubernetes）

微服务的特点：异构

- 不同语言
- 不同类型的数据库

spring cloud的服务调用方式

- rest (spring cloud采用)
- rpc


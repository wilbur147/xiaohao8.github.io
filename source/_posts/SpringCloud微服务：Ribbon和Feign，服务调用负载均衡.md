---
index_img: https://img-blog.csdnimg.cn/img_convert/be504de647aba578578a3fcb3b7ea37f.png
title: SpringCloud微服务：Ribbon和Feign，服务调用负载均衡
date: 2022-09-27 21:16:22
updated: 2022-09-27 21:16:22
tags:
  - Java
  - SpringCloud
categories:
  - 技术
comments: true
---

# 一、Ribbon简介

## 1、基本概念

> Ribbon是一个客户端的负载均衡（Load Balancer，简称LB）器，它提供对大量的HTTP和TCP客户端的访问控制。

## 2、负载均衡简介

目前主流的负载均衡方案可分成两类：

1）集中式

> 即在服务的消费方和提供方之间使用独立的LB设施，可以是硬件，如F5,也可以是软件，如nginx,由该设施负责把访问请求通过某种策略转发至服务的提供方；

2）进程内

> 将LB逻辑集成到消费方，消费方从服务注册中心获取可用服务列表，然后根据指定负载均衡策略选择合适的服务器。Ribbon就属于该方式。

## 3、Ribbon负载策略

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129171508.png)



```dart
1) RoundRobinRule 轮询
轮询服务列表List<Server>的index，选择index对应位置的服务。
2) RandomRule 随机
随机服务列表List<Server>的index，选择index对应位置的服务。
3) RetryRule 重试
指定时间内，重试(请求)某个服务不成功达到指定次数，则不再请求该服务。
```

# 二、Feign简介

1、基本概念

> Feign 是一个声明式的 Web Service 客户端。它的出现使开发 Web Service 客户端变得很简单。使用 Feign 只需要创建一个接口加上对应的注解，比如：@FeignClient 接口类注解。

2、执行流程

> 1. 主程序入口添加 @EnableFeignClients 注解开启对 FeignClient 接口扫描加载。接口使用@FeignClient注解。
> 2. 调用Feign 接口中的方法被时，通过JDK的代理的方式，生成具体的 RequestTemplate。
> 3. RequestTemplate 生成 Request请求，结合Ribbon实现服务调用负载均衡策略。

# 三、综合使用案例

## 1、项目结构图

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129171522.png)

1)、模块描述



```undefined
Eureka注册中心
node02-eureka-7001
两个服务提供方
node02-provider-6001
node02-provider-6002
Ribbon服务调用
node02-consume-8001
Feign服务调用
node02-consume-8002
```

2)、依赖Eureka知识

上篇文章Eureka使用：

## 2、Ribbon服务调用

代码所属模块：node02-consume-8001

1)、核心依赖



```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

2)、配置文件



```java
@Configuration
public class LoadConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate (){
        return new RestTemplate() ;
    }
    @Bean
    public IRule getIRule (){
        // 默认轮询算法
        // return new RoundRobinRule() ;
        // 重试算法:默认情况，访问某个服务连续三次失败，就不会再访问
        // return new RetryRule() ;
        // 随机算法
        return new RandomRule() ;
    }
}
```

3)、调用方式



```kotlin
@RestController
public class ConsumeController {

    @Autowired
    private RestTemplate restTemplate ;

    String server_name = "http://NODE02-PROVIDER" ;
    // http://localhost:8001/showInfo
    @RequestMapping("/showInfo")
    public String showInfo (){
        return restTemplate.getForObject(server_name+"/getInfo",String.class) ;
    }

}
```

这里的NODE02-PROVIDER就是服务提供方的配置文件。两个服务提供方的这块配置相同，Ribbon正基于此，实现多个服务调用的负载均衡。



```undefined
spring:
  application:
    name: node02-provider
```

4)、提供方接口



```kotlin
@RequestMapping("/getInfo")
public String getInfo (){
    LOG.info("provider-6002");
    return "success" ;
}
```

## 3、Feign服务调用

代码所属模块：node02-consume-8002

1)、核心依赖



```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

2)、配置文件



```kotlin
@FeignClient(value = "NODE02-PROVIDER")
public interface GetAuthorService {
    @RequestMapping(value = "/getAuthorInfo/{authorId}",method = RequestMethod.GET)
    String getAuthorInfo (@PathVariable("authorId") String authorId) ;
}
```

3)、调用方式



```kotlin
@RestController
public class ConsumeController {
    @Resource
    private GetAuthorService getAuthorService ;
    @RequestMapping(value = "/getAuthorInfo")
    public String getAuthorInfo () {
        return getAuthorService.getAuthorInfo("1") ;
    }
}
```

4)、启动类注解



```kotlin
// 因为包名路径不同，需要加basePackages属性
@EnableFeignClients(basePackages={"cloud.block.code.service"})
```

5)、提供方接口



```kotlin
@RequestMapping(value = "/getAuthorInfo/{authorId}",method = RequestMethod.GET)
public String getAuthorInfo (@PathVariable("authorId") String authorId) {
    LOG.info("provider-6002");
    return "翔基"+authorId ;
}
```


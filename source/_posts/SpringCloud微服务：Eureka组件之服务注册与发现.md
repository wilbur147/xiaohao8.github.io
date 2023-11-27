---
index_img: https://img-blog.csdnimg.cn/img_convert/be504de647aba578578a3fcb3b7ea37f.png
title: SpringCloud微服务：Eureka组件之服务注册与发现
date: 2022-09-24 22:18:13
updated: 2022-09-24 22:18:13
tags:
  - Java
  - SpringCloud
categories:
  - 技术
comments: true
---

# 一、Eureka基本架构

## 1、Eureka角色结构图

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129165112.png)

角色职责如下：
 1）、Register：服务注册中心，它是一个Eureka Server ，提供服务注册和发现功能。
 2）、Provider：服务提供者，它是一个Eureka Client ，提供服务。
 3）、Consumer：服务消费者，它是一个Eureka Cient ，消费服务。

## 2、Eureka中几个核心概念

1）、Registe服务注册
 当Client向Server 注册时，Client 提供自身的元数据，比如IP 地址、端口、运行状况指标的Uri 、主页地址等信息。
 2）、Renew服务续约
 Client 在默认的情况下会每隔30 秒发送一次心跳来进行服务续约。通过服务续约来告知Server该Client仍然可用。正常情况下，如果Server在90 秒内没有收到Client 的心跳，Server会将Client 实例从注册列表中删除。官网建议不要更改服务续约的间隔时间。
 3）、Fetch Registries获取服务注册列表信息
 Client 从Server 获取服务注册表信息，井将其缓存在本地。Client 会使用服务注册列表信息查找其他服务的信息，从而进行远程调用。该注册列表信息定时（每30 秒） 更新一次。
 4）Cancel服务下线
 Client 在程序关闭时可以向Eureka Server 发送下线请求。发送请求后，该客户端的实例信息将从Server 的服务注册列表中删除。该下线请求不会自动完成，需要在程序关闭时调用以下代码：



```css
DiscoveryManager.getinstance().shutdownComponent();
```

5） Eviction服务下线
 在默认情况下，当Client 连续90 秒没有向Server 发送服务续约（即心跳〉时，Server 会将该服务实例从服务注册列表删除，即服务下线。

# 二、Eureka案例代码

## 1、项目基本结构图



```bash
主要包括两个注册中心（集群）
node01-eureka-7001
node01-eureka-7002
一个服务提供方
node01-provider-8001
一个服务消费方
node01-consume-8002
```

## 2、配置本机的Host文件



```css
# cloud host
127.0.0.1 registry01.com
127.0.0.1 registry02.com
127.0.0.1 provider-8001.com
```

## 3、注册中心代码分解

1）Eureka注册中心依赖



```xml
<dependencies>
    <!--eureka-server服务端 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
```

2）核心配置文件



```objectivec
server:
  port: 7001
spring:
  application:
    name: node01-eureka-7001
eureka:
  instance:
    hostname: registry01.com
    prefer-ip-address: true
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示该端就是注册中心，维护服务实例，不去检索服务
    fetch-registry: false
    service-url:
      # 单点注册中心
      # defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      # 集群注册中心
      defaultZone: http://registry02.com:7002/eureka/
```

这里采用集群的配置，如果有多个集群，逗号分隔，如下写法就好：



```cpp
defaultZone: 
http://registry02.com:7002/eureka/,
http://registry02.com:7002/eureka/
```

3）启动类注解



```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
@SpringBootApplication
@EnableEurekaServer // 注册中心注解
public class Application_7001 {
    public static void main(String[] args) {
        SpringApplication.run(Application_7001.class,args) ;
    }
}
```

这样注册中心代码完成。
 4）启动项目，如图



![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129165235.jpg)

暂时没有服务注册进来。

## 4、服务提供方代码分解

1）核心依赖



```xml
<dependencies>
    <!-- 将微服务provider侧注册进eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```

2)配置文件如下



```csharp
server:
  port: 8003
spring:
  application:
    name: node01-provider-8001
eureka:
  instance:
    hostname: provider-8001
    prefer-ip-address: true
  client:
    service-url:
      # 集群注册中心
      defaultZone: http://registry01.com:7001/eureka/,http://registry02.com:7002/eureka/
```

3)提供一个接口服务



```dart
@RestController
public class ProviderController {
    @RequestMapping("/getInfo")
    public Map<String,String> getInfo (){
        Map<String,String> infoMap = new HashMap<>() ;
        infoMap.put("作者：","翔基") ;
        infoMap.put("主题：","SpringCloud微服务框架") ;
        return infoMap ;
    }
}
```

4)启动类上需要注解



```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient    // 本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient
public class Application_8001 {
    public static void main(String[] args) {
        SpringApplication.run(Application_8001.class,args) ;
    }
}
```

## 5、服务消费方代码分解

服务消费方和提供方的代码逻辑基本一致。
 1）配置文件
 这里不需要注册自己，只是单纯从注册中心获取服务提供方的消息。



```objectivec
server:
  port: 8002
spring:
  application:
    name: node01-consume-8002
eureka:
  client:
    register-with-eureka: false
    service-url:
      # 集群注册中心
      defaultZone: http://registry01.com:7001/eureka/,http://registry02.com:7002/eureka/
```

2）消费服务代码
 注意这里的【server_name】就服务提供方的



```yaml
spring:
  application:
    name: node01-provider-8001
```

使用RestTemplate调用服务。



```kotlin
@RestController
public class ConsumeController {
    @Autowired
    private RestTemplate restTemplate ;
    String server_name = "node01-provider-8001" ;
    @RequestMapping("/showInfo")
    public Map<String,String> showInfo (){
        return restTemplate.getForObject("http://"+server_name+":8001/getInfo",Map.class) ;
    }
}
```

到这里，一个基于Eureka的服务注册与发现就完成了。
 3）四个服务全部启动



![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129165249.png)

4)访问如下地址



```cpp
http://localhost:8002/showInfo
```

获取服务接口结果



```json
{
    "主题：": "SpringCloud微服务框架",
    "作者：": "翔基"
}
```

这样案例就结束了。




---
index_img: https://img-blog.csdnimg.cn/img_convert/be504de647aba578578a3fcb3b7ea37f.png
title: SpringCloud微服务：Zuul网关之路由配置
date: 2022-09-28 23:08:13
updated: 2022-09-28 23:08:13
tags:
  - Java
  - SpringCloud
categories:
  - 技术
comments: true
---

# 一、Zuul是什么？

Zuul 是 Netflix OSS 中的一员，是一个基于 JVM **路由器**和**服务端的负载均衡器**。提供路由、监控、弹性、安全等方面的服务框架。Zuul 能够与 Eureka、Ribbon、Hystrix 等组件配合使用。

zuul核心功能是**过滤器**、路由、异常处理，通过过滤器还能扩展出其他功能：

> 1）动态路由、2）请求监控、3）认证鉴权、4）压力测试、5）灰度发布

# 二、Zuul路由配置

## 1.创建项目

引入依赖：



```xml
<!--eureka-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>3.0.2</version>
</dependency>

<!--Zuul-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.2.7.RELEASE</version>
</dependency>
```

这里有个问题：

> **spring-cloud-starter-netflix-zuul:2.2.x 和Spring Boot 2.5.x之间的Spring 版本冲突**
>  **Spring Boot 2.4之后不支持zuul(改用Gateway)**

导致项目启动失败

## 2.路由规则配置

当 Zuul 集成 Eureka 之后，其实就可以为 Eureka 中所有的服务进行路由操作了，默认的转发规则就是“API 网关地址+访问的服务名称+接口 URI”。

配置文件如下：



```bash
server.port=8088
#server.servlet.context-path=/zuul-demo
spring.application.name=zuul-server

eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/

#zuul代理配置  zuul.routes.服务名.path,服务名要与注册的一致
zuul.routes.client-provider-server.path=/client-provider/**
#URL映射
#zuul.routes.client-provider-server.url=http://localhost:8081/
#应用名映射
zuul.routes.client-provider-server.service-id=client-provider-server
```

启动类设置@EnableZuulProxy注解，访问代理 http://localhost:8088/client-provider/api/provider

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129173241.png)



## 3.Zuul过滤器介绍及使用

Zuul 中的过滤器跟我们之前使用的 javax.servlet.Filter 不一样，javax.servlet.Filter 只有一种类型，可以通过配置 urlPatterns 来拦截对应的请求。

### 过滤器类型

| 类型  | 使用场景                                                     |
| ----- | ------------------------------------------------------------ |
| pre   | 可以在请求被路由之前调用。适用于身份认证的场景，认证通过后再继续执行下面的流程。 |
| route | 在路由请求时被调用。适用于灰度发布场景，在将要路由的时候可以做一些自定义的逻辑。 |
| error | 在 route 和 error 过滤器之后被调用。这种过滤器将请求路由到达具体的服务之后执行。适用于需要添加响应头，记录响应日志等应用场景。 |
| post  | 处理请求时发生错误时被调用。在执行过程中发送错误时会进入 error 过滤器，可以用来统一记录错误信息。 |

### 过滤器执行生命周期

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129172910.png)

 过滤器生命周期



通过上面的图可以清楚地知道整个执行的顺序，请求发过来首先到 pre 过滤器，再到 routing 过滤器，最后到 post 过滤器，任何一个过滤器有异常都会进入 error 过滤器。

### 自定义Zuul过滤器

如下，代码说明写在注释中



```java
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.apache.commons.lang.StringUtils;

import javax.servlet.http.HttpServletRequest;

/**
 * 自定义过滤器
 */
public class MyAccessFilter extends ZuulFilter {

    /**
     * 过滤器类型，可选值有 pre、route、post、error。
     *
     * @return
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 通过int值来定义过滤器的执行顺序
     * 过滤器的执行顺序，数值越小，优先级越高。
     *
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 是否执行该过滤器，true 为执行，false 为不执行
     * 这个也可以利用配置中心来实现，达到动态的开启和关闭过滤器。
     * 配置文件中禁用过滤器：
     * 【zuul.过滤器的类名.过滤器类型.disable=true，如：zuul.MyAccessFilter.pre.disable=true】
     *
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 过滤器具体逻辑
     *
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        System.out.println(String.format("%s AccessFilter request to %s", request.getMethod(), request.getRequestURL().toString()));
        String accessToken = request.getParameter("accessToken");
        // 有权限令牌
        if (!StringUtils.isEmpty(accessToken)) {
            // 设置是否路由到服务
            ctx.setSendZuulResponse(true);
            ctx.setResponseStatusCode(200);
            // 设置下一个过滤器是否执行，当为 true 的时候，后续的过滤器才执行，若为 false 则不执行。
            ctx.set("isSuccess", true);
        } else {
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.setResponseBody("{\"result\":\"accessToken is not correct!\"}");
            //可以设置一些值
            ctx.set("isSuccess", false);
        }
        return null;
    }
}
```

配置过滤器



```java
import com.local.springboot.zuul.zuulserver.filter.MyAccessFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfiguration {

    @Bean
    public MyAccessFilter myAccessFilter(){
        return new MyAccessFilter();
    }
}
```

访问：http://localhost:8088/client-provider/api/provider

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129172930.png)


 重新访问：http://localhost:8088/client-provider/api/provider?accessToken=12358

![img](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129172942.png)



### 异常处理

与Spring Boot中用 @ControllerAdvice 注解统一异常处理不同，@ControllerAdvice 注解主要用来针对 Controller 中的方法做处理，作用于 @RequestMapping 标注的方法上，只对我们定义的接口异常有效，在 Zuul 中是无效的。

我们可以定义一个 error 过滤器来记录异常信息或者通过实现ErrorController 的方式处理异常。



```java
public class ErrorFilter extends ZuulFilter {
    private Logger log = LoggerFactory.getLogger(ErrorFilter.class);
    @Override
    public String filterType() {
        return "error";
    }
    @Override
    public int filterOrder() {
        return 10;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        Throwable throwable = ctx.getThrowable();
        log.error("Filter Erroe : {}", throwable.getCause().getMessage());
        return null;
    }
}
```

### 动态路由

Zuul 实现动态路由，是因为Zuul 集成的有负载均衡、有负载均衡的效果。


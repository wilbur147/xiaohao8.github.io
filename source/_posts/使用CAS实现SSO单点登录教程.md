---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210907162847.png
title: 使用CAS实现SSO单点登录教程
date: 2021-09-07 19:36:23
updated: 2021-09-07 19:36:23
categories:
  - 技术
comments: true
---
## 1. 概述 

### 1.1. 什么是SSO?

单点登录（ Single Sign-On , 简称 SSO ）是目前比较流行的服务于企业业务整合的解决方案之一， SSO 使得在多个应用系统中，用户只需要 登录一次 就可以访问所有相互信任的应用系统。

### 1.2. 什么是CAS?

随着SSO技术的流行，相关产品也比较多，其中CAS就是一套解决方案，CAS（Central Authentication Service）中文翻译为统一身份认证服务或中央身份服务，它由服务端和客户端组成，实现SSO,并且容易进行企业应用的集成。

CAS是Yale大学（耶鲁）发起的一个开源项目，旨在为web应用系统提供一种可靠的单点登录方法，CAS在2004年12月正式成为JA-SIG的一个项目。

> 官网：https://www.apereo.org/projects/cas

![1](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210907162847.png)

CAS具有以下的特点：

- 开源的企业级单点登录解决方案
- CAS Server为需要独立部署的web应用
- CAS Client支持非常多的客户端（这里指单点登录系统中的各个web应用），包括 Java、.Net 、ISAPI、Php、Perl、uPortal、Acegi、Ruby、VBScript等客户端

有了CAS,我们的系统架构就演变成下面这样的：

![2](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210907162914.png)

从架构上可以看出，CAS包含两个部分：CAS Server和CAS Client.

- CAS Server需要独立部署，主要负责对用户的认证工作，CAS Client负责处理
- 对客户端受保护资源的访问请求，需要登录，重定向到CAS Server。

> 下面，我们一步步搭建CAS实现SSO.

### 1.3. 开发环境要求

Jdk1.8+ maven3.6 idea tomcat9.0+ windows10

## 2. CAS Server服务器端

#### 2.1. CAS服务器端软件包下载

- 下载版本为5.3

> 下载服务器的overlay地址: https://github.com/apereo/cas-overlay-template/tree/5.3

压缩包：`cas-overlay-template-5.3.zip`

解压好后用命令：`build.cmd package`

然后用编译的目录查看war包:

![3](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210907162925.png)

### 2.2. 服务器端的基本部署和测试

将war包放到tomcat的webapp中，然后启动tomcat

访问地址：`http://localhost:8080/cas` 或者 `http://localhost:8080/cas/login`

![kjsabfjkasbfjiasgbgfuio](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210817104223.png)

默认用户名和密码在`\webapps\cas\WEB-INF\classes\application.properties`里面 用户名：casuser 密码：Mellon

![640](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210817113509.webp)

CAS服务端启动成功

### 2.3. CAS Server服务器配置

#### 2.3.1 去除https认证

CAS默认使用的是HTTPS协议，如果使用HTTPS协议需要SSL安全证书（需向特定的机构申请和购买）。如果对安全要求不高或是在开发测试阶段，可使用HTTP协议。我们这里讲解通过修改配置，让CAS使用HTTP协议。

修改CAS服务端配置文件：

`\cas\WEB-INF\classes\application.properties`里添加如下内容:

```basic
cas.tgc.secure=false
cas.serviceRegistry.initFromJson=true
```

`\cas\WEB-INF\classes\services`目录下的HTTPSandIMAPS-10000001.json修改内容如下:

```json
"serviceId" : "^(https|http|imaps)://.*"
```

## 3. CAS Client客户端配置（自己项目）

Pom文件的依赖即pom.xml

```xml
<dependency>
    <groupId>net.unicon.cas</groupId>
    <artifactId>cas-client-autoconfig-support</artifactId>
    <version>2.1.0-GA</version>
</dependency>
```

application.yml配置文件

客户端1

```yaml
server:
  port: 9010
cas:
  server-url-prefix: http://localhost:8080/cas
  server-login-url: http://localhost:8080/cas/login
  client-host-url: http://localhost:9010
  validation-type: cas3
```

> 注：启动类追加开启CAS的注解@EnableCasClient

项目中新建一个测试类

```java
iimport io.swagger.annotations.Api;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Api(description = "SSO-CAS的测试")
public class TestController {

    @GetMapping("/test1")
    public String test1(){
        return "test1....";
    }
}
```

客户端2

```yaml
server:
  port: 9011
cas:
  server-url-prefix: http://localhost:8080/cas
  server-login-url: http://localhost:8080/cas/login
  client-host-url: http://localhost:9011
  validation-type: cas3
```

> 注：启动类追加开启CAS的注解@EnableCasClient

项目中新建一个测试类

```java
import io.swagger.annotations.Api;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Api(description = "SSO-CAS的测试")
public class TestController {

    @GetMapping("/test2")
    public String test1(){
        return "test2....";
    }
}
```

客户端1，客户端2和cas服务器搭建好之后，接下来我们进行测试：

1.首先启动tomcat服务器中的CAS Server。

2.分别启动客户端1和客户端2，然后在浏览器地址栏输入客户端1的地址`http://localhost:9010/test1`

![gdsgsdg](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210817113808.webp)

在不登录的状态，在浏览器的地址栏继续输入客户端2的地址：`http://localhost:9011/test2`

![asfasfshgfhjnghkjgh](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210817113842.webp)

当我们在其中一个登录界面登录账号后（假设登录客户端2）就会跳转到登陆后的界面，如下图：

![1](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210817113920.webp)

我们再次在浏览器窗口重新输入客户端1，`http://localhost:9010/test1`，或者在刚刚输入客户端页面重新刷新,不用登录即可进入页面，如下图：

![2](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210817113926.webp)



以上就是单点登录的测试。


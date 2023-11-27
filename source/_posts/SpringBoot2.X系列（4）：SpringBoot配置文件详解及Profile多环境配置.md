---
index_img: https://static001.geekbang.org/infoq/0c/0c422e63f61599cc7be9da745d211dfa.jpeg
title: SpringBoot2.X系列（4）：SpringBoot配置文件详解及Profile多环境配置
date: 2022-06-16 18:55:11
updated: 2022-06-16 18:55:11
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---
## 前言

Spring Boot配置文件对Spring Boot来说就是入门和基础，经常会用到，基于这一点也必须要普及一下这方面知识点，所以写下做个总结以便日后查看。

## 配置文件

在我们创建一个新的SrpingBoot项目时，在resources目录下给我们一个默认的全局配置文件 `application.properties`，打开会发现是一个空的文件，就算我们啥都不配置，一样能启动项目，其实Spring Boot在底层已经把配置都给我们自动配置好了，当在配置文件进行配置时，会修改SpringBoot自动配置的默认值。

配置文件名是固定的 `application.properties` 但我们可以修改为 `application.yml` 这两个文件本质是一样的，它们的区别只是其中的语法有点区别。

### application.properties语法

`application.properties`  配置文件很好理解，通过 `=` 相接，形式如下：

```properties
key = value
```

### application.yml语法

`application.yml` 配置文件使用YMAL语言，YMAL不是如XML般的标记语言，它甚至更适合做配置文件。

下面说下如何去规范书写。

1. key: value 表示一对键值对（冒号后面必须要有空格）
2. 使用空格缩进表示层级关系
3. 左侧缩进的空格数目不重要，只要同一层级的元素左侧对齐即可
4. key 与 value 大小写敏感

```yaml
server:
  port: 8080
```

### 配置文件值的规范书写

#### 1.1、字面量： 数值，字符串，布尔，日期

1、直接写，字符串 默认不用加上引号

```properties
# properties 写法
user.lastName=Xiang
```

```yaml
# yaml 写法
user:
  lastName: Xiang
```

2、 `""` 使用 双引号 不会转义特殊字符，特殊字符最终会转成本来想表示含义输出

```properties
user.lastName="Xiang \n Name"
```

```yaml
user:
  lastName: "Xiang \n Name"
```

**输出： Xiang 换行 Name**

3、`''` 使用 单引号 会转义特殊字符，特殊字符当作一个普通的字符串输出

```yaml
user:
  lastName: 'Xiang \n Name'
```

**输出： Xiang \n Name**

#### 1.2、对象&Map（属性和值）（键值对）

Map语法都是 `key、value`  形式

```properties
# properties 语法
maps.lastName=Xiang
maps.age=18
```

```yaml
# yaml 语法
maps:
  lastName: Xiang
  age: 18
  key: value
```

```yaml
# yaml 行内语法
maps: {lastName: Xiang, age: 18}
```

#### 1.3、 数组（List、Set）

**用 `-` 值表示数组中的一个元素**

```properties
# properties 写法
user.list[0]=1
user.list[1]=2
user.list[2]=3
```

```yaml
# yaml 写法
user:
  list:
    - 1
    - 2
    - 3
```

```yaml
# yaml 行内写法
user:
  list: [1,2,3]
```

> 温馨提醒：
>
> 关于行内写法请注意一下 当前属性是否是第一级（有没有父级），可以观察一下
>
> person:
>   list: [1,2,3]
>
> 和上面写的
>
> maps: {lastName: Xiang, age: 18}



## 获取配置文件属性值

有时候我们需要在java类获取到配置文件的值，Spring Boot提供自定义配置组件，继续来写一个规范的配置文件。

### 一、@ConfigurationProperties 获取值

**1、 编写JavaBean**

```java
/**
 * @author Xiang
 **/
@Component
@ConfigurationProperties(prefix = "user")
public class UserBean {
    private String lastName;
    private Integer age;
    private Map<String,String> maps;
    private List<Integer> list;
    getter/setter……
}
```

**2、 编写 Yaml 文件**

```yaml
user:
  lastName: Xiang
  age: 18
  list:
    - 1
    - 2
    - 3
  maps:
    key1: value1
    key2: value2
```

@ConfigurationProperties 注解向Spring Boot声明该类中的所有属性和配置文件中相关的配置进行绑定。

- prefix = "user"：声明配置前戳，将该前戳下的所有属性进行映射。

@Component 或者@Configuration：将该组件加入Spring Boot容器，只有这个组件是容器中的组件，配置才生效。

**3、 单元测试是否能打印配置文件值**

- 使用 SpringBoot 单元测试类进行测试

![image-20210726115647436](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726115647.png)



### 二、@Value 获取值

**UserBean** 的各个属性上加上 `@Value` 注解也可以得到配置文件的值

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

/**
 * @author Xiang
 **/
@Component
// 注释掉了
//@ConfigurationProperties(prefix = "user")
public class User1Bean {
    @Value("${user1.lastName}")
    private String lastName;
    @Value("${user1.age}")
    private Integer age;
    @Value("#{${user1.maps}}")
    private Map<String,String> maps;
    @Value("#{'${user1.list}'.split(',')}")
    private List<Integer> list;
}
```

`application.yml` 配置文件也要更改为行内语法

```yaml
user1:
  lastName: Xiang
  age: 18
  list: 1,2,3
  maps: '{"key1":"value1","key2":"value2"}'
```

- 总结 使用场景，如果只是在某个业务逻辑中需要获取配置文件中的某个属性值，就使用 `@Value`



## 自定义配置文件

除了在默认的application文件进行属性配置，我们也可以自定义配置文件，上面用的是yaml配置文件，这里就新建 `person.properties` ，配置内容如下

```properties
user.lastName=Xiang
user.age=18
user.list=1,2,3
user.maps.key1=value1
user.maps.key2=value2
```

**1、编写 PersonBean**

```java
@Configuration
@ConfigurationProperties(prefix = "person")
@PropertySource(value = "classpath:person.properties")
public class PersonBean {
    private String lastName;
    private Integer age;
    private Map<String,String> maps;
    private List<Integer> list;
}

```

和前面的 `UserBean` 配置大体一样，可以看前面的注解详解，解释一下 `@PropertySource`

- `@PropertySource` 用于加载局部配置文件， 加载指定的配置文件; value 属性是数组类型, 用于指定文件位置

运行后，发现 properties 文件在 idea 上中文乱码， 进行如下设置就会不会乱码

![image-20210726142453909](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726142454.png)

**2、单元测试结果**

![image-20210726144640162](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726144640.png)



##  Profile多环境支持

- Profile 是 Spring 用来针对不同的环境要求，提供不同的配置支持， 全局 Profile 配置使用的文件名可以是
  application-{profile}.properties / application-{profile}.yml ;
  - 如: application-dev.yml/ application-prod.yml

### 1、applicaiton-dev/prod.yml文件演示案例

创建两个文件 `application-dev.yml` 与 `application-prod.yml` 

![image-20210726160026709](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726160026.png)

![image-20210726155908254](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726155908.png)



**激活指定profile** 

- 在主配置文件 `application.yml`中指定 `spring.profiles.active=dev`

![image-20210726160254883](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726160254.png)

- 未指定哪个profile文件时， 默认使用 `application.yml`中的配置 。


**单元测试 age 是否是 dev 文件的值**

![image-20210726161129284](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726161129.png)



### 2、 多种方式激活指定profile

- **方式1：在主配置文件中指定**

```properties
# properties 配置方式
spring.profiles.active=dev

# yaml 配置方式
spring:
  profiles:
    active: dev
```

- **方式2:：命令行参数指定**
  - idea启动时，配置传入命令行参数 `--spring.profiles.active=dev`

![image-20210726162431551](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726162431.png)

- jar包运行命令参数

```basic
java -jar learn-1.0.jar --spring.profiles.active=dev
```

- **方式3： 虚拟机参数指定**

```basic
-Dspring.profiles.active=dev
```

![image-20210726162846542](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726162846.png)

- **方式4：Maven环境指定**

在 `Pom` 文件添加 `profiles`

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <!-- 环境标识，需要与配置文件的名称相对应 -->
            <profiles.active>dev</profiles.active>
        </properties>
        <activation>
            <!-- 默认环境 -->
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <profiles.active>prod</profiles.active>
        </properties>
    </profile>
</profiles>
```



然后在 `application.yml` 文件中设置 环境 `@profiles.active@`

```yaml
spring:
  profiles:
    active: @profiles.active@
```



maven 刷新后会出现 `Profiles`选项

![image-20210726164724645](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210726164724.png)



## 配置文件加载位置

SpringBoot 启动时，会扫描以下位置的 `application.properties` 或者 `application.yml` 文件作为 Spring Boot的
默认配置文件:

| 配置文件位置       | 说明                               |
| ------------------ | ---------------------------------- |
| file:./config/     | 当前项目的config目录下（最高级别） |
| file:./            | 当前项目的根目录下                 |
| classpath:/config/ | 类路径的config目录下               |
| classpath:/        | 类路径的根目录下（最低级别）       |

以上按照`优先级从高到低`的顺序，将所有位置的配置文件全部加载，高优先级的配置内容会覆盖低优先级的
配置内容。

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

我们还可以通过spring.config.location来改变默认的配置文件位置，示例：

```bash
java -jar learn-1.0.jar --spring.config.location=E:/application.properties
```



以上就是今天的配置详解，有什么没有说到的或者还有问题的欢迎评论区讨论留言！！！


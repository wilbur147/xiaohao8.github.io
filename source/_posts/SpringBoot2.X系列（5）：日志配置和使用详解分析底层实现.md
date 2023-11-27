---
index_img: https://static001.geekbang.org/infoq/0c/0c422e63f61599cc7be9da745d211dfa.jpeg
title: SpringBoot2.X系列（5）：日志配置和使用详解分析底层实现
date: 2022-06-17 20:13:23
updated: 2022-06-17 20:13:23
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---
## 介绍

市面上常见的日志框架有很多。通常情况下，日志是由一个抽象层+实现层的组合来搭建的，而用户通常来说不应该直接使用具体的日志实现类，应该使用日志的抽象层。

SpringBoot 支持 `Java Util Logging`，`Log4J`，`Log4J2` 和 `Logback` 日志框架，默认采用 **`logback`** 日志。在实际 SpringBoot 项目中使用 SpringBoot 默认日志配置是不能够满足实际生产及开发需求的，需要选定适合的日志输出框架，灵活调整日志输出级别、日志输出格式等。本章主要讲述如何进行 SpringBoot 项目的日志详细配置。

| **日志门面(日志的抽象层)**                                   | **日志实现**                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| JCL(Jakarta Commons Logging)（2014年后不再维护）<br/>jboss-logging （不适合企业项目开发使用）<br/>SLF4J(Simple Logging Facade for java) | Log4j<br />JUL(java.util.logging)（java.util.logging)(担心被抢市场,推出的)<br/>Log4j2( apache开发的很强大,借了log4j的名,但很多框架未适配上)<br/>Logback(Log4j同一个人开发的新框架,做了重大升级) |

**SpringBoot** 默认选择的是 **SLF4J** + **Logback** 的组合，如果不需要更改为其他日志系统（如 **Log4j2** 等），则无需多余的配置，**LogBack** 默认会将日志打印到控制台上。

## 基本用法

因为新建的 **Spring Boot** 项目一般都会引用 **`spring-boot-starter`** 或者 **`spring-boot-starter-web`**，而这两个起步依赖中都已经包含了对于 **`spring-boot-starter-logging`** 的依赖，所以，我们无需额外添加依赖。

![image-20210727103255967](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727103256.png)



### 1.1 日志级别介绍

（1）日志级别从小到大为 **trace** < **debug** < **info** < **warn** < **error** < **fatal**，由于默认日志级别设置为 **INFO**，因此上面样例 **trace** 和 **debug** 级别的日志都看不到。
（2）我们可以在 **applicaition.properties** 文件中修改日志级别。比如下面将全局日志级别都改成 **trace**，因此系统所有的日志都能看到：

```yaml
# 设置root级别
logging:
  level:
    root: trace
```

![image-20210727104647097](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727104647.png)

（3）我们也可以只设置某个包的日志级别，这样能够更方便准确地定位问题。比如下面配置，只对所有 **com.xiang.learn** 包下面产生的日志级别改成 **trace**：

```yaml
# com.xiang.learn 包下的级别
logging:
  level:
    com.xiang.learn: trace
```



**打印结果当前包下能打印完整：**

![image-20210727110020832](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727110020.png)



**如果设置成其它包，结果就不能打印 `Trace` 和 ` Debug` **

![image-20210727110259791](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727110259.png)



### 1.2 日志文件生成

日志文件生成在配置文件有两种方式写法，分别是：

- **`logging.file.path`**，只能指定路径，不能自定义文件名，自动生成 spring.log 文件
- **`logging.file.name`**，它可以自定义文件名

```yaml
logging:
  level:
  file:
#    path: E:\my-tool\log # 生成指定目录日志文件 （ linux系统 /data/log ）
    path: logfile # 生成当前项目根目录日志文件 名称为：spring.log
    name: logfile/my.log # 生成当前项目根目录指定文件名称为 my.log 日志文件
```



![image-20210727112506091](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727112506.png)



不管是哪一种配置，**`Spring Boot`** 都会自动按天分割日志文件，也就是说每天都会自动生成一个新的 **log** 文件，而之前的会自动打成 **GZ** 压缩包。

![image-20210729113055240](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210729113055.png)



另外也可以设置日志文件的保留时间，以及单个文件的大小： 

```yaml
logging:
  logback:
    rollingpolicy:
      max-file-size: 2KB  # 默认10MB 日志文件大小切割，超过指定数问自动打包成gz文件
      max-history: 1 # 默认7天 日志的保留时间，超过指定数会自动删除
```



### 1.3 日志输出格式配置

（1）我们可以分别修改在控制台输出的日志格式，以及文件中日志输出的格式：

> **符号说明：**
>
> - **%d{HH:mm:ss.SSS}**：日志输出时间
> - **%-5level**：日志级别，并且使用 **5** 个字符靠左对齐
> - **%thread**：输出日志的进程名字，这在 **Web** 应用以及异步任务处理中很有用
> - **%logger**：日志输出者的名字
> - **%msg**：日志消息
> - **%n**：平台的换行符
> - {50}： 限制字符长度

```yaml
logging:
  pattern:
    console: '%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n'
    file: '%d{yyyy-MM-dd HH:mm:ss.SSS} >>> [%thread] >>> %-5level >>> %logger{50} >>> %msg%n'
```



![image-20210727114913739](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727114913.png)



（2）我们自定义设置文件输出格式通过上图发现没有最开始打印的颜色了，没有颜色区分始终感觉有点别扭，如果想让不同类型的数据具有不同的高亮效果，记得添加 `%clr()` 配置如下：



```yaml
logging:
  pattern:
    console: '%d{yyyy-MM-dd} [%thread] %clr(%-5level) %clr(%logger{50}){cyan} - %msg%n'
    file: '%d{yyyy-MM-dd HH:mm:ss.SSS} >>> [%thread] >>> %-5level >>> %logger{50} >>> %msg%n'
```

![image-20210727120003240](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727120003.png)



## 日志底层实现分析

前面我们也讲了，新建的 **Spring Boot** 项目一般都会引用 **`spring-boot-starter`** 或者 **`spring-boot-starter-web`**，而这两个起步依赖中都已经包含了对于 **`spring-boot-starter-logging`** 的依赖，所以，我们无需额外添加依赖。

- **在org\springframework\boot\logging\logback\base.xml 做了日志的默认配置**

![image-20210727145001436](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727145001.png)



- **日志文件采用的方式为：滚动文件追加器**

![image-20210727145834529](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727145834.png)



- **然后它会通过org.springframework.boot.logging.LoggingSystemProperties 类中读取上面的配置信息**

![image-20210727150703673](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727150703.png)



## 自定义Logback日志配置

- **如果spring boot的日志功能无法满足我们的需求(比如异步日志记录等)，我们可以自已定义的日志配置文件。**

- **在类路径下，存放对应日志框架的自定义配置文件即可；SpringBoot就不会使用它默认的日志配置文件了。**

| Logging System          | 自定义日志配置文件名                                         |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | logback-spring.xml , logback-spring.groovy , logback.xml  or  logback.groovy |
| Log4j2                  | log4j2-spring.xml or log4j2.xml                              |
| JDK (Java Util Logging) | logging.properties                                           |

- **在 resources 目录下创建 logback.xml ， 文件内容如下，SpringBoot就会采用以下日志配置：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="false" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志存放路径为当前项目根目录 -->
    <property name="LOG_HOME" value="./logfile" />
    <!-- 定义日志文件名称 -->
    <property name="appName" value="springboot-learn"></property>



    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>

    <!--  控制台彩色日志 dev  -->
    <property name="console.log.pattern.dev"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} - [%thread] - %clr(%-5level) - %clr(%logger{50}){cyan} - %msg%n"/>
        <!--  控制台彩色日志 !dev  -->
    <property name="console.log.pattern.prod"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} >>> [%thread] >>> %clr(%-5level) >>> %clr(%logger{50}){cyan} >>> %msg%n"/>
    <!--  文件日志  -->
    <property name="log.pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n"/>

    <!-- ch.qos.logback.core.ConsoleAppender 表示控制台输出 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式说明：
        %d 输出日期时间
        %thread 输出当前线程名
        %-5level 输出日志级别，左对齐5个字符宽度
        %logger{50} 输出全类名最长50个字符，超过按照句点分割
        %msg 日志信息
        %n 换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>${console.log.pattern.dev}</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>${console.log.pattern.prod}</pattern>
            </springProfile>
        </layout>
    </appender>
    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <appender name="appLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 指定日志文件的名称 -->
        <file>${LOG_HOME}/${appName}.log</file>
        <!--
        当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名
        TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--
            滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动
            %i：当文件大小超过maxFileSize时，按照i进行文件滚动
            -->
            <fileNamePattern>${LOG_HOME}/${appName}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <!--
            可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。
            假设设置每天滚动，且maxHistory是100，则只保存最近100天的文件，删除之前的旧文件。
            注意，删除旧文件是，那些为了归档而创建的目录也会被删除。
            -->
            <MaxHistory>100</MaxHistory>
            <!--
            当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动 注意此处配置
            SizeBasedTriggeringPolicy是无法实现按文件大小进行滚动的，必须配置timeBasedFileNamingAndTriggeringPolicy
            -->
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 日志输出格式： -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>${log.pattern}</pattern>
        </layout>
    </appender>
    <!--
    logger主要用于存放日志对象，也可以定义日志类型、级别
    name：表示匹配的logger类型前缀，也就是包的前半部分
    level：要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
    additivity：作用在于children-logger是否使用 rootLogger配置的appender进行输出，
    false：表示只用当前logger的appender-ref，true：
    表示当前logger的appender-ref和rootLogger的appender-ref都有效
    -->
    <!-- 系统业务 logger -->
    <logger name="com.xiang" level="debug" />
    <!-- Spring framework logger -->
    <logger name="org.springframework" level="warn" additivity="false"></logger>
    <!--
    root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
    要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。
    -->
    <root level="info">
        <appender-ref ref="stdout" />
        <appender-ref ref="appLogAppender" />
    </root>
</configuration>
```

### 1.1 logback.xml 和 logback-spring.xml 差别

- logback.xml ：是直接就被日志框架加载了。

- logback-spring.xml：配置项不会被日志框架直接加载，而是由 SpringBoot 解析日志配置文件

- logback.xml加载早于application.properties，所以如果你在logback.xml使用了变量时，而恰好这个变量是写在application.properties时，那么就会获取不到，只要改成logback-spring.xml就可以解决。

- 因为logback-spring.xml是由 SpringBoot 解析日志配置文件，故可以使用SpringBoot 的 Profifile 特殊配置



### 1.2 使用 Profile 特殊配置

- **使用日志 Profille 特殊配置, 可根据不同的环境激活不同的日志配置**
  - 将 自定义日志配置文件名 `logback.xml` 改为 `logback-spring.xml`
  - 修改 日志配置文件中 第38行，如下：

```xml
<layout class="ch.qos.logback.classic.PatternLayout">
    <springProfile name="dev">
        <pattern>${console.log.pattern.dev}</pattern>
    </springProfile>
    <springProfile name="!dev">
        <pattern>${console.log.pattern.prod}</pattern>
    </springProfile>
</layout>
```

- 指定运行环境： `--spring.profiles.active=dev`

**如果使用 logback.xml 作为日志配置文件，还指定 Profile 特殊配置，则会有以下错误**

```bash
16:31:30,080 |-ERROR in ch.qos.logback.core.joran.spi.Interpreter@39:39 - no applicable action for [springProfile], current ElementPath  is [[configuration][appender][layout][springProfile]]
16:31:30,082 |-ERROR in ch.qos.logback.core.joran.spi.Interpreter@42:40 - no applicable action for [springProfile], current ElementPath  is [[configuration][appender][layout][springProfile]]
```



**配置正确就出现如下打印：**

![image-20210727164358664](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727164358.png)

![image-20210727164517317](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727164517.png)

## 切换日志框架

- **将SpringBoot默认的 `logback` 切换为 `log4j2` 日志框架，可以借助 [参考文档](https://docs.spring.io/spring-boot/docs/2.0.6.RELEASE/reference/htmlsingle/#boot-features-logging)**

- **在项目的 pom.xml 切换log4j2**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 切换为log4j2框架第一步：排除 spring-boot-starter-logging 日志启动器 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 切换为log4j2框架第二步：引入 log4j2 日志启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

（1）新建 `log4j2.xml` 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出 -->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数 -->
<configuration monitorInterval="5">
    <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->


    <!--变量配置 -->
    <Properties>
        <!-- 格式化输出：%date表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符 -->
        <!-- %logger{36} 表示 Logger 名字最长36个字符 -->
        <property name="LOG_PATTERN"
                  value="%clr{%d{yyyy-MM-dd HH:mm:ss.SSS}}{faint} %clr{%5p} %clr{${sys:PID}}{magenta} %clr{---}{faint} %clr{[%15.15t]}{faint} %clr{%-40.40c{1.}}{cyan} %clr{:}{faint} %m%n%xwEx" />
        <Property name="FILE_PATTERN">%date{yyy-MM-dd HH:mm:ss.SSS} %5p ${sys:PID}} --- [%15.15t] %-40.40c{1.}{cyan} : %m%n%xwEx</Property>

        <!-- 定义日志存储的路径，不要配置相对路径 -->
        <property name="FILE_PATH" value="./logfile" />
        <property name="FILE_NAME" value="springboot-log" />
    </Properties>


    <appenders>

        <!-- 控制台配置输出  -->
        <console name="Console" target="SYSTEM_OUT">
            <!--输出日志的格式 -->
            <PatternLayout pattern="${LOG_PATTERN}" />
            <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="DEBUG" onMatch="ACCEPT"
                             onMismatch="DENY" />
        </console>

        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingFile name="RollingFileInfo"
                     fileName="${FILE_PATH}/info.log"
                     filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd}_%i.log.gz">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="info" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <PatternLayout pattern="${FILE_PATTERN}" />
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour -->
                <TimeBasedTriggeringPolicy interval="1" />
                <SizeBasedTriggeringPolicy size="10MB" />
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖 -->
            <DefaultRolloverStrategy max="15" />
        </RollingFile>


        <!-- 这个会打印出所有的warn及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingFile name="RollingFileWarn"
                     fileName="${FILE_PATH}/warn.log"
                     filePattern="${FILE_PATH}/${FILE_NAME}-WARN-%d{yyyy-MM-dd}_%i.log.gz">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="warn" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <PatternLayout pattern="${FILE_PATTERN}" />
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour -->
                <TimeBasedTriggeringPolicy interval="1" />
                <SizeBasedTriggeringPolicy size="10MB" />
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖 -->
            <DefaultRolloverStrategy max="15" />
        </RollingFile>


        <!-- 这个会打印出所有的error及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingFile name="RollingFileError"
                     fileName="${FILE_PATH}/error.log"
                     filePattern="${FILE_PATH}/${FILE_NAME}-ERROR-%d{yyyy-MM-dd}_%i.log.gz">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="error" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <PatternLayout pattern="${FILE_PATTERN}" />
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour -->
                <TimeBasedTriggeringPolicy interval="1" />
                <SizeBasedTriggeringPolicy size="10MB" />
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖 -->
            <DefaultRolloverStrategy max="15" />
        </RollingFile>


    </appenders>


    <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。 -->
    <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效 -->
    <loggers>


        <!--过滤掉spring和mybatis的一些无用的DEBUG信息 -->
        <logger name="org.mybatis" level="info" additivity="false">
            <AppenderRef ref="Console" />
        </logger>
        <!--监控系统信息 -->
        <!--若是additivity设为false，则 子Logger 只会在自己的appender里输出，而不会在 父Logger 的appender里输出。 -->
        <Logger name="org.springframework" level="info"
                additivity="false">
            <AppenderRef ref="Console" />
        </Logger>


        <root level="info">
            <appender-ref ref="Console" />
            <appender-ref ref="RollingFileInfo" />
            <appender-ref ref="RollingFileWarn" />
            <appender-ref ref="RollingFileError" />
        </root>
    </loggers>


</configuration>

```



（2）在 `application.yml` 配置文件中引入 `log4j2.xml`

```yaml
logging:
  config: classpath:log4j2.xml
```



（3）启动项目，查看控制台及文件输出

![image-20210727175359937](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210727175400.png)



有什么问题，欢迎大家留言～



**参考源码**

[小翔SpringBoot 日志介绍lab](https://github.com/wilbur147/xiangStudy/tree/main/lab-springBoot/SpringBoot-3)




























































---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211117173535
title: SpringBoot 配置全局异常统一处理
date: 2021-12-27 20:19:11
updated: 2021-12-27 20:19:11
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---
## 一. 默认错误处理

SpringBoot 默认为我们提供了BasicErrorController 来处理全局错误/异常，并在Servlet容器中注册error为全局错误页。所以在浏览器端访问，发生错误时，我们能及时看到错误/异常信息和HTTP状态等反馈。工作原理如下：

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    // 统一异常处理（View）
    @RequestMapping(produces = "text/html")
    public ModelAndView errorHtml(HttpServletRequest request,
            HttpServletResponse response) {
        HttpStatus status = getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
                request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());
        ModelAndView modelAndView = resolveErrorView(request, response, status, model);
        return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
    }
```

例如下面这两个错误，对于日常开发而言，再熟悉不过了。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211117173535)

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211117173614)

## 二. 统一异常处理

默认的英文空白页，显然不能够满足我们复杂多变的需求，因此我们可以通过专门的类来收集和管理这些异常信息，这样做不仅可以减少控制层的代码量，还有利于线上故障排查和紧急短信通知等。

### 具体步骤

为了让小伙伴少走一些弯路，楼主根据官方源码和具体实践，提炼这些核心工具类：

- ErrorInfo 错误信息
- ErrorInfoBuilder 错误信息的构建工具

> 注：在CSDN和大牛博客中，不乏关于Web应用的统一异常处理的教程，但更多的是基础学习使用，并不能投入实际项目使用，为了让大家少走一些弯路和快速投入生产，楼主根据官方源码和项目实践，提炼出了核心工具类（ErrorInfoBuilder ），将构建异常信息的逻辑从异常处理器/控制器中抽离出来，让大家通过短短几行代码就能获取丰富的异常信息，更专注于业务开发！！

### 1. 统一异常处理器(GlobalErrorHandler)

- @ControllerAdvice 限定范围 例如扫描某个控制层的包

- @ExceptionHandler 指定异常 例如指定处理运行异常。

  具体如下：

```java
package com.hehe.error;

@ControllerAdvice
public class GlobalErrorHandler {

    private final static String DEFAULT_ERROR_VIEW = "error";//错误信息页

    @Autowired
    private ErrorInfoBuilder errorInfoBuilder;//错误信息的构建工具

    /**
     * 根据业务规则,统一处理异常。
     */
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Object exceptionHandler(HttpServletRequest request, Throwable error) {

        //1.若为AJAX请求,则返回异常信息(JSON)
        if (isAjaxRequest(request)) {
           return errorInfoBuilder.getErrorInfo(request,error);
        }
        //2.其余请求,则返回指定的异常信息页(View).
        return new ModelAndView(DEFAULT_ERROR_VIEW, "errorInfo", errorInfoBuilder.getErrorInfo(request, error));
    }

    private boolean isAjaxRequest(HttpServletRequest request) {

        return "XMLHttpRequest".equals(request.getHeader("X-Requested-With"));
    }

}
```

### 2. 统一异常信息（ErrorInfo）

虽然官方提供了ErrorAttributes来存储错误信息，但其返回的是存储结构是Map，为了更好的服务统一异常，这里我们统一采用标准的ErrroInfo来记载错误信息。

```java
package com.hehe.error;

public class ErrorInfo {

    private String time;//发生时间
    private String url;//访问Url
    private String error;//错误类型
    String stackTrace;//错误的堆栈轨迹
    private int statusCode;//状态码
    private String reasonPhrase;//状态码

    //Getters And Setters
       ...

}
```

### 3. 统一异常信息的构建工具（ErrorInfoBuilder）

ErrorInfoBuilder 作为核心工具类，其意义不言而喻，重点API：

- ***获取错误/异常\***
- ***通信状态\***
- ***堆栈轨迹\***

> 注：正确使用ErrorInfoBuilder，可以让处理器减少80%的代码。总而言之，ErrorInfoBuilder是个好东西，值得大家细细琢磨。

```java
package com.hehe.error;

@Order(Ordered.HIGHEST_PRECEDENCE)
@Component
public class ErrorInfoBuilder implements HandlerExceptionResolver, Ordered {

    /**
     * 错误KEY
     */
    private final static String ERROR_NAME = "hehe.error";

    /**
     * 错误配置(ErrorConfiguration)
     */
    private ErrorProperties errorProperties;

    public ErrorProperties getErrorProperties() {
        return errorProperties;
    }

    public void setErrorProperties(ErrorProperties errorProperties) {
        this.errorProperties = errorProperties;
    }

    /**
     * 错误构造器 (Constructor) 传递配置属性：server.xx -> server.error.xx
     */
    public ErrorInfoBuilder(ServerProperties serverProperties) {
        this.errorProperties = serverProperties.getError();
    }

    /**
     * 构建错误信息.(ErrorInfo)
     */
    public ErrorInfo getErrorInfo(HttpServletRequest request) {

        return getErrorInfo(request, getError(request));
    }

    /**
     * 构建错误信息.(ErrorInfo)
     */
    public ErrorInfo getErrorInfo(HttpServletRequest request, Throwable error) {
        ErrorInfo errorInfo = new ErrorInfo();
        errorInfo.setTime(LocalDateTime.now().toString());
        errorInfo.setUrl(request.getRequestURL().toString());
        errorInfo.setError(error.toString());
        errorInfo.setStatusCode(getHttpStatus(request).value());
        errorInfo.setReasonPhrase(getHttpStatus(request).getReasonPhrase());
        errorInfo.setStackTrace(getStackTraceInfo(error, isIncludeStackTrace(request)));
        return errorInfo;
    }

    /**
     * 获取错误.(Error/Exception)
     *
     * @see DefaultErrorAttributes #addErrorDetails
     */
    public Throwable getError(HttpServletRequest request) {
        //根据HandlerExceptionResolver接口方法来获取错误.
        Throwable error = (Throwable) request.getAttribute(ERROR_NAME);
        //根据Request对象获取错误.
        if (error == null) {
            error = (Throwable) request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE);
        }
        //当获取错误非空,取出RootCause.
        if (error != null) {
            while (error instanceof ServletException && error.getCause() != null) {
                error = error.getCause();
            }
        }//当获取错误为null,此时我们设置错误信息即可.
        else {
            String message = (String) request.getAttribute(WebUtils.ERROR_MESSAGE_ATTRIBUTE);
            if (StringUtils.isEmpty(message)) {
                HttpStatus status = getHttpStatus(request);
                message = "Unknown Exception But " + status.value() + " " + status.getReasonPhrase();
            }
            error = new Exception(message);
        }
        return error;
    }

    /**
     * 获取通信状态(HttpStatus)
     *
     * @see AbstractErrorController #getStatus
     */
    public HttpStatus getHttpStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute(WebUtils.ERROR_STATUS_CODE_ATTRIBUTE);
        try {
            return statusCode != null ? HttpStatus.valueOf(statusCode) : HttpStatus.INTERNAL_SERVER_ERROR;
        } catch (Exception ex) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
    }

    /**
     * 获取堆栈轨迹(StackTrace)
     *
     * @see DefaultErrorAttributes  #addStackTrace
     */
    public String getStackTraceInfo(Throwable error, boolean flag) {
        if (!flag) {
            return "omitted";
        }
        StringWriter stackTrace = new StringWriter();
        error.printStackTrace(new PrintWriter(stackTrace));
        stackTrace.flush();
        return stackTrace.toString();
    }

    /**
     * 判断是否包含堆栈轨迹.(isIncludeStackTrace)
     *
     * @see BasicErrorController #isIncludeStackTrace
     */
    public boolean isIncludeStackTrace(HttpServletRequest request) {

        //读取错误配置(server.error.include-stacktrace=NEVER)
        IncludeStacktrace includeStacktrace = errorProperties.getIncludeStacktrace();

        //情况1：若IncludeStacktrace为ALWAYS
        if (includeStacktrace == IncludeStacktrace.ALWAYS) {
            return true;
        }
        //情况2：若请求参数含有trace
        if (includeStacktrace == IncludeStacktrace.ON_TRACE_PARAM) {
            String parameter = request.getParameter("trace");
            return parameter != null && !"false".equals(parameter.toLowerCase());
        }
        //情况3：其它情况
        return false;
    }

    /**
     * 保存错误/异常.
     *
     * @see DispatcherServlet #processHandlerException 进行选举HandlerExceptionResolver
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {
        request.setAttribute(ERROR_NAME, ex);
        return null;
    }

    /**
     * 提供优先级 或用于排序
     */
    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE;
    }
}
```

> 注：ErrorBuilder之所以使用@Order注解和实现HandlerExceptionResolver接口是为了获取错误/异常，通常情况下@ExceptionHandler并不需要这么做，因为在映射方法注入Throwable就可以获得错误/异常，这是主要是为了ErrorController根据Request对象快速获取错误/异常。

### 4. 控制层代码（Controller）

上述，错误/异常处理器、错误信息、错误信息构建工具全部完成，我们编写控制层代码来测试相关效果。

```java
package com.hehe;

@SpringBootApplication
@RestController
public class ErrorHandlerApplication {
    /**
     * 随机抛出异常
     */
    private void randomException() throws Exception {
        Exception[] exceptions = { //异常集合
                new NullPointerException(),
                new ArrayIndexOutOfBoundsException(),
                new NumberFormatException(),
                new SQLException()};
        //发生概率
        double probability = 0.75;
        if (Math.random() < probability) {
            //情况1：要么抛出异常
            throw exceptions[(int) (Math.random() * exceptions.length)];
        } else {
            //情况2：要么继续运行
        }

    }

    /**
     * 模拟用户数据访问
     */
    @GetMapping("/")
    public List index() throws Exception {
        randomException();
        return Arrays.asList("正常用户数据1!", "正常用户数据2! 请按F5刷新!!");
    }

    public static void main(String[] args) {
        SpringApplication.run(ErrorHandlerApplication.class, args);
    }
}
```

### 5. 页面代码（Thymeleaf）

代码完成之后，我们需要编写一个异常信息页面。为了方便演示，我们在resources目录下创建templates目录，并新建文件exception.html。页面代码如下：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>GlobalError</title>
</head>
<h1>统一祖国 振兴中华</h1>
<h2>服务异常，请稍后再试。</h2>
<div th:object="${errorInfo}">
    <h3 th:text="*{'发生时间：'+time}"></h3>
    <h3 th:text="*{'访问地址：'+url}"></h3>
    <h3 th:text="*{'问题类型：'+error}"></h3>
    <h3 th:text="*{'通信状态：'+statusCode+','+reasonPhrase}"></h3>
    <h3 th:text="*{'堆栈信息：'+stackTrace}"></h3>
</div>
</body>
</html>
```

> 注：SpringBoot默认支持很多种模板引擎（如Thymeleaf、FreeMarker），并提供了相应的自动配置，做到开箱即用。默认的页面加载路径是 src/main/resources/templates ，如果放到其它目录需在配置文件指定。（举例：spring.thymeleaf.prefix=classpath:/views/ ）

### 6. 引入依赖（POM文件）

以前操作之前，不要忘了在pom.xml 引入相关依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--基本信息 -->
    <groupId>com.hehe</groupId>
    <artifactId>springboot-error-handler</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>spring-boot-error-handler</name>
    <description>SpringBoot 统一异常处理</description>

    <!--继承信息 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.M4</version>
        <relativePath/>
    </parent>

    <!--依赖管理 -->
    <dependencies>
        <dependency> <!--添加Web依赖 -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency> <!--添加Thymeleaf依赖 -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency><!--添加Test依赖 -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!--指定远程仓库（含插件）-->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>

    <!--构建插件 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 7. 开始测试

上述步骤完成之后，打开启动类GlobalExceptionApplication，启动项目然后进行测试。本案例-项目结构图如下：

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211117173715)

**测试效果：**在浏览器输入 http://localhost:8080 多次按F5刷新，然后查看页面效果。

### 三. 使用@ExceptionHandler的不足之处

关于实现Web应用统一异常处理的两种方法比较：

| 特性     | @ExceptionHandler                           | ErrorController                                   |
| :------- | :------------------------------------------ | :------------------------------------------------ |
| 获取异常 | 通过方法参数注入                            | 通过ErrorInfoBuilder获取                          |
| 返回类型 | 若请求的类型为Ajax则返回JSON，否则返回页面. | 若请求的媒介类型为HTML 则返回页面 ，否则返回JSON. |
| 缺点     | 无法处理404类异常                           | 很强大，可处理全部错误/异常                       |

#### 1. 使用@ExceptionHandler 为什么无法处理404错误/异常？

- 答：因为SpringMVC优先处理（Try Catch）掉了资源映射不存在的404类错误/异常，虽然在响应信息注入了404的HttpStatus通信信息，但木有了异常，肯定不会进入@ExceptionHandler 的处理逻辑。

#### 2. 使用@ExceptionHandler + 抛出异常 是否可取？

- 答：由上图可知@ExceptionHanlder的最大不足之处是无法直接捕获404背后的异常，网上流传通过取消资源目录映射来解决无404问题是不可取的，属于越俎代庖的做法。

```properties
spring.mvc.throw-exception-if-no-handler-found=true
spring.resources.add-mappings=false
```

#### 3. 为什么推荐ErrorController 替代 @ExceptionHandler ?

- 使用ErrorController可以处理 全部错误/异常 。
- 使用ErrorController+ErrorInfoBuilder 在单个方法里面可以针对不同的Exception来添加详细的错误信息，具体做法：拓展ErrorInfoBuilder的getErrorInfo方法来添加错误信息（例如：ex instanceof NullPointerException Set xxx）。

> 注意：实际上，目前SpringBoot官方就是通过ErrorController来做的统一错误/异常处理，但遗憾的是，关于这方面的官方文档并没有给出详细示例，仅仅是一笔带过，大概官方认为@ExceptionHandler 够用？？而网上也甚少人具体提及ErrorController和ErrorAttribute 背后一整套的实现逻辑，也正是如此，促使楼主决心写下这篇文章，希望给大家带来帮助，少走一些弯路！！

### 四. 使用ErrorController替代@ExceptionHandler

#### 4. 如何快速使用 ErrorController ？

回答：经过楼主的精心设计，ErrorInfoBuilder 可以无缝对接ErrorController （即上述两种错误/异常处理均共用此工具类），你只需要做的是：将本案例的ErrorInfo和ErrorInfoBuilder 拷贝进项目，简单编写ErrorController 跳转页面和返回JSON即可。具体如下：

- @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
  说明：produces属性作为匹配规则：表示Request请求的Accept头部需含有text/html。
  用途：text/html 主要用于响应普通的页面请求，与AJAX请求作为区分。

```java
package com.hehe.error;

@Controller
@RequestMapping("${server.error.path:/error}")
public class GlobalErrorController implements ErrorController {

@Autowired
    private ErrorInfoBuilder errorInfoBuilder;//错误信息的构建工具.

    private final static String DEFAULT_ERROR_VIEW = "error";//错误信息页

    /**
     * 情况1：若预期返回类型为text/html,则返回错误信息页(View).
     */
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request) {
        return new ModelAndView(DEFAULT_ERROR_VIEW, "errorInfo", errorInfoBuilder.getErrorInfo(request));
    }

    /**
     * 情况2：其它预期类型 则返回详细的错误信息(JSON).
     */
    @RequestMapping
    @ResponseBody
    public ErrorInfo error(HttpServletRequest request) {
        return errorInfoBuilder.getErrorInfo(request);
    }

    @Override
    public String getErrorPath() {//获取映射路径
        return errorInfoBuilder.getErrorProperties().getPath();
    }
}
```

> 注：是不是非常简单，相信这个工具类可以改变你对ErrorController复杂难用的看法。如果后续想拓展不同种类的错误/异常信息，只需修改ErrorInfoBuilder#getError方法即可，无需修改ErrorController的代码，十分方便。
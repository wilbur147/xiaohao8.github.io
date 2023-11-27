---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827151340.png
title: Docker部署python项目挂载flask详细图文教程
date: 2023-01-21 19:21:13
updated: 2023-01-21 19:21:13
tags:
  - Python
categories:
  - 运维部署
comments: true
---

**最近有学习python，整理了一下flask相关知识，刚好有一位朋友一直问我怎么部署flask**

在docker流行的今天，部署项目无疑比以前学习的时候无疑更方便许多，趁着现在弄了一道flask，便写一篇新手教程篇部署flask，跟着本教程一步一步做就能部署成功。同时给出一些链接，想深入一点了解的可以自行深入学习。



## 基础介绍

> - **Flask：**python web流行的框架之一，最大特点是它的 **轻量级** 基于[Werkzeug]() WSGI工具箱和[Jinja2]() 模板引擎。
> - **Flask 有以下特点：**
>   - 轻量级和模块化设计，易于拓展
>   - 自由、灵活，第三方库选择面广
>   - 支持 RESTful 请求调度
>   - 文档内容全面，丰富，入门简单



## 具体操作

以下项目我是用的 `Pycharm` 创建的 **Flask** 项目，项目架构是这样的：

![image-20210827143927799](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827143928.png)



看一下整个项目的启动文件 **`app.py`** 在项目的根目录中

```python
from flask import Flask, request

# 实例化Flask对象
app = Flask(__name__)


# 生成路由关系，并把关系保存到某个地方,app对象的 url_map字段中
@app.route('/')
def hello_world():
    return 'Hello World!'


# 测试
@app.route("/test", methods=["GET"])
def test():
    urls = request.args.get("urls")
    if urls:
        return {"code": 200, "msg": "上传成功!"}
    return {"code": 500, "msg": "上传失败，参数urls路径缺失！"}


if __name__ == '__main__':
    # 启动程序，监听用户请求
    # 一旦请求到来，执行 app.__call__方法
    # 封装用户请求
    # 进行路由匹配
    app.run(host='0.0.0.0', port=7090, debug=True)

```



启动命令行 `python app.py` 运行这个应用，然后在浏览器输入 `127.0.0.1:5000` 回车就能打开我们的网站了。

但是这样简单运行的话，只要按一下 ctrl + c 终止运行，或者关掉终端，网站就连接不了了，我们要寻求更长久的真正的部署。



##### 生成项目依赖文件 `requirements.txt`

执行命令行可以直接生成 `requirements.txt` 文件，里面存放的是当前项目环境所有安装的依赖库

```bash
pip freeze > requirements.txt
```

![image-20210827151339978](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827151340.png)



## Docker部署

第一步当然是先安装 docker，安装这里就不讲了，百度，谷歌一搜就知道了。

**下面就将项目用 `docker` 打包，用挂载的方式部署，这样我们在服务器直接替换文件代码，`docker`不用重新编译和重启就直接生效了，还是很方便的。**

> **唯一的缺点就是如果新增加了依赖库，就会报错，原因是容器环境没有新写的依赖，解决如下：**
>
> - 可以重新删除容器镜像，重新编译一次再启动容器
> - 进入容器内部，手动添加新增加的依赖



##### 1、 新建 `Dockerfile ` 文件

在项目根目录创建一个 `Dockerfile ` 文件，内容如下：

```dockerfile
# Docker image for flask python run
# VERSION 1.0
# Author: Weiye
# 基础镜像使用python:3.7
FROM python:3.7
# 将服务器 requirements.txt 文件复制到 容器 /project/目录下
COPY requirements.txt /project/
# 指定容器工作目录为 /project/
WORKDIR /project/
# 安装 项目依赖
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
# 运行
ENTRYPOINT ["python","app.py"]

# 编译
# docker build -t weiye/flask-project:v1 .

# run
#docker run -it -p 7090:7090 --name flask-project \
#-v /mydata/flaskProject:/project \
#-d weiye/flask-project:v1
```



**完成文件创建后，我们将整个项目放到服务器上面去，并进入此项目目录，我是放到 `mydata`文件夹里面，你们自行选择**

![image-20210827151742756](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827151742.png)



##### 2、`Docker` 编译镜像

进入到 `flaskProject` 项目根目录，看到有上图几个文件说明目录就正确了。

然后执行 `docker`编译命令 `docker build -t weiye/flask-project:v1 .` 进行编译。

```basic
docker build -t weiye/flask-project:v1 .
```



> `weiye` 表示用户名，可以改成你自己的，也可以不要，直接是 `flask-project`
>
> `v1` 表示版本，不写的话默认是最新版本 `latest` 



看到有 `Successfully built xxxxx` 标识后就说明编译成功了，编译成功后我们输入 `docker images` 查看当前容器镜像

![image-20210827152639723](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827152639.png)



##### 3、启动容器镜像

执行 `docker` 启动命令启动容器：

```basic
docker run -it -p 7090:7090 --name flask-project \
-v /mydata/flaskProject:/project \
-d weiye/flask-project:v1
```

**-v就是容器的挂载，这样python项目启动直接是挂载的服务器目录的，后面我们修改源码就不用重启容器了**

容器启动成功后，可以查看一下容器日志

```basic
docker logs -f  flask-project
```

看到启动成功后，我们可以访问 **`公网IP + 端口`** 方式访问

> **注意端口是否已经放开，没放开搜一下 linux如何放开端口，阿里云/腾讯云等服务器开起端口需要在官网控制台开起**

![image-20210827162038922](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827162039.png)



## 服务测试

我们本地用浏览器访问一下 **服务器的公网IP和项目启动的端口** 看一下能否访问成功

![image-20210827162309942](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827162310.png)



如上图显示已经成功访问，接下来测试一下源码修改，不重启容器是否有效。

修改服务器上的源码保存

![image-20210827162445739](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827162445.png)



然后使用浏览器再次访问一下

![image-20210827162820222](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210827162820.png)



可以看到已经成功更改了~~~



## 总结

本篇文章讲述了 flask 项目利用docker部署到服务器上，在不动容器的情况下修改源码能实时更改状态的效果，**借助了 `flask` 的 `debug` 属性** 不过如果是真正的项目部署在线上还是不推荐大家这样的部署方式，`debug`模式推荐还是本地开发时用用。

目前我了解到的正式部署线上服务器的组合推荐 **`Nginx+Gunicorn+Gevent+Supervisor+Flask`**

> 部署方案：
>
> - Nginx：高性能 Web 服务器+负载均衡
> - Gunicorn：高性能 WSGI 服务器；
> - Gevent：把 Python 同步代码变成异步协程的库；
> - Supervisor：监控服务进程的工具；
> - Flask：一个使用Python编写的轻量级 Web 应用框架

后面有时间也会更新一下这个方案的部署教程，大家不妨可以关注一下，可以第一时间收到通知哦~~





## 项目源码

[https://github.com/wilbur147/pythonLearnning/tree/main/flask-lab/flaskProject](https://github.com/wilbur147/pythonLearnning/tree/main/flask-lab/flaskProject)










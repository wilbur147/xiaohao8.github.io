---
index_img: https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=OWRlZjQwNTljNjE5ZGNkMzA5ODkxYTQ5NzFmN2UyN2VfcWJ3VWdZTjNJQWJBOXh5aW1WRHQ1NWR3aVVHRWNEZ2xfVG9rZW46TGJwQWJIQUdjb2w2amx4SnZCZmNOSjJ5blFjXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA
title: 前端Jenkins+Docker自动化部署镜像及容器
date: 2023-11-12 21:43:12
updated: 2023-11-12 21:43:12
tags:
  - Jenkins
categories:
  - 运维部署
comments: true
---

#### 前言

> 🚀 需提前安装环境及知识点：
>
> 1、[Docker搭建及基础操作](https://blog.csdn.net/TanHao8/article/details/130564504)
>
> 2、[DockerFile文件描述](https://blog.csdn.net/AtlanSI/article/details/87892016)
>
> 3、[Jenkins搭建及基础点](https://blog.csdn.net/TanHao8/article/details/129362857)
>
> 
>
> **🚀  目的：**
>
> 将我们的前端项目打包成一个镜像容器并自动发布部署，可供随时pull访问，动态修改Api

## 一、手动部署镜像及容器

 1、在当前项目的根目录创建Dockerfile文件并写入如下代码：

```PowerShell
# 第一阶段：构建前端产出物
FROM node:14.19.0 AS builder

WORKDIR /visualization
COPY . .
RUN npm install -g cnpm --registry=https://registry.npm.taobao.org
RUN cnpm install && npm run build


# 第二阶段：生成最终容器映像
FROM nginx

COPY docker/nginx.conf /etc/nginx/conf.d/default.conf
COPY docker/docker-entrypoint.sh /docker-entrypoint.sh


WORKDIR /home/visualization
COPY --from=builder /visualization/dist .

RUN chmod +x /docker-entrypoint.sh
```

> 代码片段详细描述：
>
> 注意：visualization为工作区名称，可更换

| 片段                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| FROM node:14.19.0 AS builder                                 | 选择使用 Node.js 14.19.0 作为基础镜像，并命名该步骤为 “builder” |
| WORKDIR /visualization. COPY . .                             | 将工作目录设置为 “/visualization”，并将当前目录（代码仓库根目录）中的所有文件和目录复制到 “/visualization” 目录中 |
| 1、RUN npm install -g cnpm --registry=https://registry.npm.taobao.org. 2、RUN cnpm install && npm run build | 安装 cnpm 包管理器并使用其安装项目依赖项，接着在执行 npm run build 命令进行构建操作，构建后的前端产出物保存至 “/visualization/dist” 目录中 |
| FROM nginx                                                   | 选择使用 Nginx 作为基础镜像，并命名该步骤为默认名称 “builder” |
| 1、COPY docker/nginx.conf /etc/nginx/conf.d/default.conf 2、COPY docker/docker-entrypoint.sh /docker-entrypoint.sh | 将docker文件夹下的 Nginx 配置文件、sh文件复制到对应的位置中  |
| 1、WORKDIR /home/visualization 2、COPY --from=builder /visualization/dist . | 将工作目录设置为 “/home/visualization”，并将来自 “builder” 阶段的前端产出物复制到当前目录中。 |
| RUN chmod +x /docker-entrypoint.sh                           | 对 “/docker-entrypoint.sh” 执行 chmod +x 命令，添加可执行权限。 |

2、项目根目录新建.dockerignore文件，忽略文件

```PowerShell
# Dependency directory  
# https://www.npmjs.org/doc/misc/npm-faq.html#should-i-check-my-node_modules-folder-into-git  
node_modules  
.DS_Store  
dist  
  
# node-waf configuration  
.lock-wscript  
  
# Compiled binary addons (http://nodejs.org/api/addons.html)  
build/Release  
.dockerignore  
Dockerfile  
*docker-compose*  
  
# Logs  
logs  
*.log  
  
# Runtime data  
.idea  
.vscode  
*.suo  
*.ntvs*  
*.njsproj  
*.sln  
*.sw*  
pids  
*.pid  
*.seed  
.git  
.hg  
.svn
```

3、当前项目根目录下创建docker文件夹

> 1、新建nginx.conf文件，用于配置前端项目访问nginx配置文件
>
> 2、新建docker-entrypoint.sh文件，执行脚本动态修改nginx.conf中的代理请求地址

> **nginx.conf内容**
>
>  ~根据项目情况做出修改，gzip配置前端无则可删除
>
> ~ /dev是前端代理跨域的基准地址，要保持统一，代理到后端的地址，做代理的目的是后面可以根据容器run动态改变proxy_pass地址
>
> ~如果项目无https则可删除443监听
>
> ~有https则需要配置证书ssl_certificate、ssl_certificate_key，此文件的路径为后面 运行容器时(run) -v将宿主机的目录映射至容器，就是容器的目录

```PowerShell
server {
    listen 80;
    listen [::]:80;
    server_name  localhost;
    
    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";
    #
    
    location / {
        root   /home/visualization;
        index  index.html index.htm;
    }
     
    location /dev {
        # 前端/dev请求代理到后端https://xxx.xxx/
        proxy_pass https://xxx.xxx/;
        
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_connect_timeout   1;
        proxy_buffering off;
        chunked_transfer_encoding off;
        proxy_cache off;
        proxy_send_timeout      30m;
        proxy_read_timeout      30m;
        client_max_body_size    500m;

        access_log /var/log/nginx/dev_access.log;
        error_log /var/log/nginx/dev_error.log;
    }
}

server {
    listen 443 ssl;
    listen [::]:443;
    # 配置域名访问项目
    server_name  xx.xx.xx.com;

    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";
    #
    
    # ssl证书配置开始
    ssl_certificate   /etc/nginx/cert/certbook_bundle.pem;
    ssl_certificate_key  /etc/nginx/cert/certbook.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    # ssl证书配置结束 

    location / {
        root   /home/visualization;
        index  index.html index.htm;
    }

    location /dev {
        # 前端/dev请求代理到后端https://xxx.xxx/
        proxy_pass https://xxx.xxx/;
        
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_connect_timeout   1;
        proxy_buffering off;
        chunked_transfer_encoding off;
        proxy_cache off;
        proxy_send_timeout      30m;
        proxy_read_timeout      30m;
        client_max_body_size    500m;

        access_log /var/log/nginx/dev_access.log;
        error_log /var/log/nginx/dev_error.log;
    }
}
```

> **docker-entrypoint.sh内容**
>
> 文件目的：动态改变nginx配置文件的代理地址，具体到改某一行
>
> 1、https://xxx.xxx/为后端接口地址，默认地址
>
> 2、sed -i '21c '"$apiUrl"'' /etc/nginx/conf.d/default.conf
>
> 例如：将nginx.conf配置文件中的21行替换成某个值
>
> 3、CERT表示：变量存在时会将https注释，以免找不到https证书报错，主要用于本地run的时候需要

```PowerShell
#!/usr/bin/env bash

API_BASE_PATH=$API_BASE_PATH;
if [ -z "$API_BASE_PATH" ]; then
    API_BASE_PATH="https://xxx.xxx/";
fi

apiUrl="proxy_pass  $API_BASE_PATH;"
sed -i '21c '"$apiUrl"'' /etc/nginx/conf.d/default.conf
sed -i '73c '"$apiUrl"'' /etc/nginx/conf.d/default.conf

# 变量CERT判断是否需要证书https, $CERT存在则不需要
certOr="#"
if [ -n "$CERT" ]; then
 sed -i '45c '"$certOr"'' /etc/nginx/conf.d/default.conf
 sed -i '46c '"$certOr"'' /etc/nginx/conf.d/default.conf
 sed -i '59c '"$certOr"'' /etc/nginx/conf.d/default.conf
 sed -i '60c '"$certOr"'' /etc/nginx/conf.d/default.conf
fi

nginx -g "daemon off;"
```

> 整体文件目录如下：

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=OWRlZjQwNTljNjE5ZGNkMzA5ODkxYTQ5NzFmN2UyN2VfcWJ3VWdZTjNJQWJBOXh5aW1WRHQ1NWR3aVVHRWNEZ2xfVG9rZW46TGJwQWJIQUdjb2w2amx4SnZCZmNOSjJ5blFjXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

4、打包镜像、运行容器

> 如果本地没有Docker环境，以下操作则在服务器执行：
>
> 1、将项目上传至云服务器，并建立项目文件夹（项目的dist以及node_modules文件夹不需要上传）
>
> 2、在远程服务器端cd到项目根目录中
>
> 3、构建 Docker 镜像

```PowerShell
// hello是指的镜像名称
// 1.0.0是镜像的版本
docker build . -t hello:1.0.0
```

> 运行容器
>
> - -v /data/nginx/cert:/etc/nginx/cert
>
> 是指将宿主机的data/nginx/cert映射到容器的etc/nginx/cert文件夹，上面的nginx.conf证书路径需要，若不需要https则忽略
>
> - -e "CERT=no" 表示无https证书的情况下，会默认注释nginx.conf里面443监听，有证书则配置上面-v就行了，**本地运行一定要加**

```PowerShell
//方式一：
// contanier_hello为容器名称
// -p 9090:80  将容器里面的80端口映射到宿主机的8080端口，80端口就是nginx里面配置，多个端口多个配置，必须确保服务器已经开了此端口
docker run -d --name contanier_hello -p 8080:80 -v /data/nginx/cert:/etc/nginx/cert hello:1.0.0

//方式二：
// 运行容器的时候改变nginx代理地址
// -e API_BASE_PATH就是上面sh文件中定义的变量 把nginx的后端接口地址改为http://www.baidu.com
docker run -d --name contanier_hello -p 8080:80 -v /data/nginx/cert:/etc/nginx/cert -e "API_BASE_PATH=http://www.baidu.com" hello:1.0.0
```

> 4、这样我们就已经可以运行访问啦

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTRiNDZhZDdjMDQ1MzllOGFjZjYyMWZiN2Y5ZWVlNzhfa2JwWFdJakVEUThVYkhtZUwzOUxKd3l2NjF4V3pHNENfVG9rZW46U0M2V2I1dGZob1dqa0F4R21uUWNzd0xibjlkXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

5**、[扩展]  发布本地镜像到私服镜像库**

> **前提需要去搭建阿里云或者腾讯云的registry**
>
> 1、输入网址[https://cr.console.aliyun.com/cn-hangzhou/repositories](https://cr.console.aliyun.com/cn-hangzhou/repositories)
>
> 2、注册账号
>
> 3、创建命名空间
>
> 4、创建镜像仓库
>
> 5、选择刚创建的命名空间，输入仓库信息 

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=MWIwYTViMjhkZDY1ZDZhMGI0NTI5MzBlMmJlODViMzBfeXlybklxeGF4c0lidTBOY0l3RHlZb21TcmZJZWdGSUhfVG9rZW46VG56TGJXTm1Mb1I5Vjl4bTJiYmNVYlBpbmxmXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

```PowerShell
docker login --username=xxxxx registry.cn-hangzhou.aliyuncs.com
docker push 镜像名称:版本
```

## 二、通过Jenkins自动构建

**此步骤是基于第一步中的(1、2、3)，再进行如下操作**

> **前提：服务器已经搭建好了[Jenkins](https://blog.csdn.net/TanHao8/article/details/129362857)**
>
> 特别注意：在运行Jenkins容器的时候一定要把宿主机的Docker进行映射，因为会在Jenkins容器里面使用dokcer命令，不然会找不到，怎么查看自己有没有进行映射？进入jenkins容器里面输入docker -v看是否有版本信息，如果没有则需要删除容器重新run，例如：

```PowerShell
docker run -d -uroot -p 10000:8080 -p 50000:50000 --name jenkins \
-v /home/jenkins_home:/var/jenkins_home \
-v /home:/home \
-v /etc/localtime:/etc/localtime \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
jenkins/jenkins:lts
```

> -v /var/run/docker.sock:/var/run/docker.sock \
>
> -v /usr/bin/docker:/usr/bin/docker \
>
> 以上2句就是把Dcoker映射进容器里面

1、创建Jenkins任务

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=MjU1Y2E2ZTliNzhlYzBiOGJhMzlkODU3YTE3YTNkMDJfbGtFN3B2ZFNFcnkwUWNrR2l5WmpOU2thVkFkVldQSnlfVG9rZW46VUZ1bmI3eHNib25lVnd4OFd0VmMySE9HbmllXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

2、配置General

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=YWNjYWJiNDU1ZWMwMmYwNTU2MDIwZGNlNGFjMmI1MGZfc3pXeEZMVG9Fc3NIREZ5VW5DT1BNZHd5Ynhma0FEMXJfVG9rZW46RDVpVWJ5Vkowb2dFcXl4Rml2YWM4eEpibjhlXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

3、配置Shell变量

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=MjQ2NGRiNTZlNWMzMGVhNzZlNDJlOGM0YjNmMjRjY2ZfRG5Ud3hNd2plMFp4NXZ4VEZkTHloUW1KV1M0V0NpNG5fVG9rZW46WEJ0bGJLdDJNb3NRUFl4N3laVGM1dG9qblRmXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

4、配置git源码（项目地址的git）

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=MmM2Mzg0ODc4Y2U2ZDc2MGJiNTc2MmMyYTdiY2I5ODNfcVVaQVZ0VVdyNktrMnJaelo0eVVROHZsVThpWW4xbXZfVG9rZW46TVJIamJYZ09LbzJRcUt4S3lWV2MxM3hubnBnXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

5、配置构建环境

![img](https://mn2rfwuv9o.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDhiYWUyN2NkMTBlMGFkMzU5MzQyYmY2NTFjNjA2ZWNfWlVKVUZUVkxyTUZOdTJwclF2enAyR0h2T2pLM1FpekpfVG9rZW46VGlVU2JoOW5wb0ZmRnR4cmhXbWNub05CbnBsXzE3MDIwMDI3MDI6MTcwMjAwNjMwMl9WNA)

6、配置Shell执行脚本

```PowerShell
#!/bin/bash  
docker -v

CONTAINER=${container_name}  
PORT=${port}  

# 检查同名容器是否存在并删除
if docker ps -a --filter "name=$CONTAINER" --format '{{.ID}}' | xargs docker inspect --format='{{json .State.Running}}' | grep -qEe true; then
  docker stop $CONTAINER && docker rm -f $CONTAINER
fi

# 删除之前构建的镜像
docker images | grep "${image_name}" | awk '{print $3}' | xargs docker rmi -f

# 构建镜像并添加标签
docker build --no-cache -t ${image_name}:${tag} .

# 移除「依赖 none 镜像」
if docker images --filter 'dangling=true' -q --no-trunc; then
  docker rmi $(docker images --filter 'dangling=true' -q --no-trunc)
fi

# run docker image  
# -v /data/nginx/cert:/etc/nginx/cert将宿主机的证书挂载容器里面
docker run -d --name $CONTAINER -p $PORT:80 -p 9091:443 -v /data/nginx/cert:/etc/nginx/cert --restart always  ${image_name}:${tag}

echo '================开始推送镜像================'
# 具体登录到哪个私服，看你们创建是哪个平台阿里云、腾讯云等
docker login --username=账号 --password=密码 registry.cn-hangzhou.aliyuncs.com
docker push ${image_name}:${tag} 
echo '================结束推送镜像================'
```

7、注意：在运行容器时，端口一定不要与其他容器映射出来的端口一样，会被占用

> 如上面的-p 9091:443是将容器的443端口映射到宿主机的9091，那么宿主机中的容器一定不能有9091端口，唯一的，再映射端口时也要注意服务器要开通此端口

8、以上都配置完成之后便可以点击构建啦，会自动执行拉取代码、项目打包、构建镜像、运行容器、发布镜像

9、别人若想使用你的镜像，则可以拉取运行：

```PowerShell
docker pull 远程镜像名称:[镜像版本号]
运行容器：
// -e "CERT=no" 无证书需要加上，会隐藏nginx配置文件中的443

//方式一：
// 1、contanier_hello为容器名称
// 2、-p 9090:80  将容器里面的80端口映射到宿主机的8080端口，80端口就是nginx里面配置，多个端口多个配置，必须确保服务器已经开了此端口
docker run -d --name contanier_hello -p 8080:80 -e "CERT=no" hello:1.0.0

//方式二：
// 1、运行容器的时候改变nginx代理地址，后端想使用前端代码本地运行请求自己的本地ip时可以采用这种方式
// 2、-e API_BASE_PATH就是上面sh文件中定义的变量 把nginx的后端接口地址改为http://www.baidu.com
// 3、** API_BASE_PATH一定要写正确，别乱写格式，nginx会识别不出来
docker run -d --name contanier_hello -p 8080:80 -e "API_BASE_PATH=http://www.baidu.com" -e "CERT=no" hello:1.0.0
```

## 🚀 🚀 🚀 🚀 ----- 大功告成----- 🚀  🚀 🚀 🚀 
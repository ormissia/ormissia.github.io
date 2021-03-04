---
title: "我的博客后端Docker镜像打包自动部署流程"
date: 2021-03-03T18:00:20+06:00
hero: hero.png
menu:
  sidebar:
    name: 我的博客后端Docker镜像打包自动部署流程
    identifier: ci/cd github docker jenkins golang gin
    parent: deployment
    weight: 1
---

> 博客后端使用Golang重构之后使用GitHub-DockerHub-Jenkins自动打包部署流程

> 虽然说Golang打包生成的是二进制可执行文件，不需要像JAVA一样部署环境变量，但依然也是需要打包的流程。由于考虑到在不(hen)久(yuan)的将来可能做成简单的微服务程序，又要使用Docker部署，所以在这就直接使用Docker镜像的方式来部署运行。

#### 本地代码→GitHub
这一步是通过`git commit`-`git push`或是直接使用IDE将代码托管到GitHub上。
在这一步的同时需要编写Dockerfile文件，用来指定Docker镜像打包时的各种参数
```Dockerfile
# Go程序编译之后会得到一个可执行的二进制文件，其实在最终的镜像中是不需要go编译器的，也就是说我们只需要一个运行最终二进制文件的容器即可。
# 作为别名为"builder"的编译镜像，下面会用到
FROM golang AS builder

# 为镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
    GOPROXY=https://goproxy.cn

# 设置工作目录：/build
WORKDIR /build

# 复制项目中的 go.mod 和 go.sum文件并下载依赖信息
COPY go.mod .
COPY go.sum .
RUN go mod download

# 将代码复制到容器中
COPY . .

# 将代码编译成二进制可执行文件app
RUN go build -o go-blog-app .

###################
# 接下来创建一个小镜像
###################
FROM scratch

# 设置程序运行时必要的环境变量，包括监听端口、数据库配置等等
ENV SERVER_PORT=8085 \
    DATASOURCE_DRIVERNAME=mysql \
    DATASOURCE_HOST=192.168.13.110 \
    DATASOURCE_PORT=3306 \
    DATASOURCE_DATABASE=blog \
    DATASOURCE_USERNAME=root \
    DATASOURCE_PASSWORD=5KvA82*Ziq \
    DATASOURCE_CHARSET=utf8mb4


# 从builder镜像中把/dist/app 拷贝到当前目录
COPY --from=builder /build/go-blog-app /

# 声明服务端口
EXPOSE 8085

# 启动容器时运行的命令
ENTRYPOINT ["/go-blog-app"]
```

#### GitHub→Docker Hub
这一步是将GitHub上的代码打包成Docker镜像并将镜像托管到Docker Hub上，我在这里使用的是使用Docker Hub来自动打包Docker镜像。也有另一种方式是GitHub通过设置好的Webhooks来通知Jenkins等CI/CD工具来拉取代码在自己的服务器上打包Docker镜像再上传到Docker Hub或是其他Docker镜像管理工具上，由于自己的这个项目代码更新比较慢，可以容忍提交代码之后有较长的时间来更新到线上环境中，所以就采用了Docker官方的打包功能。
- 首先要有一个Docker Hub的账号
  ![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/image-hosting/go-gin-blog-cicd1.jpg)
- 将GitHub账号关联到Docker Hub账号上
  ![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/image-hosting/go-gin-blog-cicd2.jpg)
- 创建Docker仓库并且与GitHub仓库绑定
  - 打开Docker Hub的仓库创建页面
  - 添加仓库名称
  - 选择GitHub作为代码仓库
  - 选择要打包Docker镜像的GitHub仓库
  - 添加Docker镜像打包规则
    - 可以选择按`git push`到指定分支或者是`git push`一个git tag来触发build动作
    - 可以指定Docker镜像的Tag
    - 可以指定用于打包Docker镜像的Dockerfile文件
    - 可以开关Docker镜像打包的缓存
      ![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/image-hosting/go-gin-blog-cicd3.jpg)
- 创建好Docker仓库之后再去GitHub仓库的setting页面中就会发现多了一个Webhook的设置
  ![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/image-hosting/go-gin-blog-cicd4.jpg)
- 每当GitHub仓库里触发了指定条件之后就会通过这个Webhook通知到Docker Hub触发对应的镜像打包动作，当然打包动作也可以手动触发
  ![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/image-hosting/go-gin-blog-cicd5.jpg)

#### Docker Hub→服务器生产环境
这一步是将Docker Hub上已经打包好的Docker镜像部署到生产服务器上。
- 在Docker Hub仓库中添加Webhook，首先需要有自己的CI/CD服务，我这里用的时搭在自己服务器上的Jenkins[\(搭建流程\)](http://ormissia.com:13880/#/articleDetail/7)。当然理论上应该也可以使用GitHub上一些CI/CD应用，我没有深入了解就不赘述了。
  ![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/image-hosting/go-gin-blog-cicd6.jpg)


未完待续。。。
由于时间比较紧，`Docker Hub→服务器生产环境`这一步还没有实际操作，目前还是`docker pull`→`docker run`
不过基于已经线上运行很久的前端VUE项目的自动部署流程，理论上这一步应该是可行的
![image](https://ormissia-blog.oss-cn-qingdao.aliyuncs.com/stickers/QQ%E5%9B%BE%E7%89%8720210201233008.gif)


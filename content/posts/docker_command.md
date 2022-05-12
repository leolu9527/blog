---
title: "Docker常用命令"
date: 2022-05-05T15:46:18+08:00
description: ""
tags: [Docker]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: Docker
comment: false
draft: false
---

**image相关**
```shell
# 新建image
$ docker build --tag=helloworld .
或
$ docker build -t helloworld:tag_name .
```


**列出本机的所有 image 文件。**
```shell
$ docker image ls
或者
$ docker images
```


**删除 image 文件**
```shell
$ docker image rm <imageName>
或者
$ docker rmi <imageId>
```


**想要删除untagged images，也就是那些id为的image的话可以用**
```shell
$ docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```


**删除全部image**
```shell
$ docker rmi $(docker images -q)
```


**强制删除全部image**
```shell
$ docker rmi -f $(docker images -q)
```


**删除没有tag及容器的镜像**
```shell
$ docker image prune
```


**删除所有不使用的镜像(没有容器的镜像)**
```shell
$ docker image prune --force --all
或者
$ docker image prune -f -a

# 24小时前的
$ docker image prune -a --filter "until=24h"
```



**container相关**
```shell
# 查看所有运行或者不运行容器
$ docker container ls
$ docker container ls --all
$ docker container ls -aq
或者
$ docker ps -aq
```


**停止、启动、杀死、重启一个容器**
```shell
$ docker stop Name或者ID  
$ docker start Name或者ID  
$ docker kill Name或者ID  
$ docker restart name或者ID
```

**停止所有的container**
```shell
$ docker stop $(docker ps -a -q)
或者
$ docker stop $(docker ps -aq)
```

**删除所有container**
```shell
$ docker rm $(docker ps -a -q)
或者
$ docker rm $(docker ps -aq)
```


**删除所有停止的容器**
```shell
$ docker container prune
```


**Docker与容器的交互**
```shell
# 从容器到宿主机复制
$ docker cp tomcat：/webapps/js/text.js /home/admin
$ docker cp 容器名:容器路径       宿主机路径

# 从宿主机到容器复制
$ docker cp /home/admin/text.js tomcat：/webapps/js
$ docker cp 宿主路径中文件      容器名:容器路径

#获取容器ip
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' container-ID

#进入一个容器
$ docker exec -it <containerID> /bin/bash

#查看容器信息
$ docker inspect <containerID>
```


**打包image并上传到dockerhub**
```shell
$ docker tag friendlyhello leolu9527/get-started:part2
$ docker push leolu9527/get-started:part2
```



**拉取一个image**
```shell
$ docker pull leolu9527/get-started:part2
```


**容器的运行 run**
```shell
$ docker run -d -p 8080:80 -v /local/xx/:/app/xx/demo:ro nginx-demo

$ docker run --publish 8000:8080 --detach --name bb bulletinboard:1.0

参数：
- `—publish` (-p)端口映射
- `--detach` (-d)后台运行
- `--name`    容器名字

# 快速进入容器sh
$ docker run -it alpine  /bin/sh

# 创建一个一次性的容器并进入shell
$ docker run --rm -it alpine  /bin/sh

# 临时执行命令
$ docker run -it --rm --name php-demo  php:7.4-cli php -m
```

**其他**
```shell
$ docker service ls

#swarm mode
$ docker swarm init
#离开
$ docker swarm leave --force

$ docker login
#login使用的id需要登陆https://cloud.docker.com查看账户名称，不是邮箱
```






**网络network**
```shell
$ docker network ls
$ docker network create test-network

# 动态地将容器连接到网络
$ docker network connect test-network <containerName>
# 查看网络
$ docker network inspect test-network

# 断开自定义网络
$ docker network disconnect -f NETWORK CONTAINER

```

**容器与宿主的文件拷贝**
```shell
# 从容器拷贝文件到宿主机
$ docker cp mycontainer:/opt/testnew/file.txt /opt/test/

# 从宿主机拷贝文件到容器
$ docker cp /opt/test/file.txt mycontainer:/opt/testnew/

# 需要注意的是，不管容器有没有启动，拷贝命令都会生效。
```

文件挂载遇到的问题，docker-compose.yml中的挂载文件在构建阶段还没准备好，就是Dockerfile中的RUN命令还不能使用挂载目录中的内容,使用CMD来执行可以解决

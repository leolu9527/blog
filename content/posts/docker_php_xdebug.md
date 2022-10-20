---
title: "PHPStorm + Docker + Xdebug 开发环境配置"
date: 2022-08-13T17:46:09+08:00
description: "PHP Docker开发环境，PHPStorm中Xdebug的使用。"
tags: [PHP, Docker]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: Docker
comment: false
draft: false
---

使用Docker来部署不同的PHP版本的开发环境可以解决本地的各种环境困扰。
下面使用docker-compose来部署一个PHP8.1的开发环境。

## docker-compose.yml
这里只使用了nginx 和 php-fpm两个服务

```yaml
version: "3.7"
services:
    nginx:
        build: ./nginx
        depends_on:
            - php-fpm
        volumes:
            - ../:/usr/share/nginx/html:rw
            - ./nginx/templates:/etc/nginx/templates
            - ./logs/nginx:/var/log/nginx
        ports:
            - "8080:80"
        environment:
            - NGINX_HOST=${NGINX_HOST}
            - NGINX_PORT=80
        restart: always
    php-fpm:
        build:
            context: ./php-fpm
            args:
                - APP_CODE_PATH=${APP_CODE_PATH}
        environment:
            PHP_IDE_CONFIG: serverName=${NGINX_HOST}
        volumes:
            - ../:/usr/share/nginx/html:rw
```

以`test.com`为例，只需要修改`.env`环境变量`NGINX_HOST`，并在本地hosts文件添加一条记录`127.0.0.1   test.com`即可，浏览器访问`test.com:8080`。

详细的部署文件放在github上了，见文末。

## php-fpm镜像

xdebug 远程安装总是失败，可以下载安装包离线安装，其他扩展同理在 `https://pecl.php.net` 上搜索。

Dockerfile:
```dockerfile
FROM php:8.1.11-fpm
MAINTAINER Leo "leolu9527@163.com"
# set timezome
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# multi-stage get composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

COPY ./xdebug-3.1.5.tgz /tmp/

RUN pecl install /tmp/xdebug-3.1.5.tgz \
	&& docker-php-ext-enable xdebug

COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini

#RUN usermod -u 1000 www-data

ARG APP_CODE_PATH

WORKDIR ${APP_CODE_PATH}

CMD php-fpm

```

## nginx镜像

```dockerfile
FROM nginx
#  set timezome
ENV TZ=Asia/Shanghai

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY nginx.conf /etc/nginx/

```

## PHPStorm中的配置

首先我们需要在chrome中下载一个插件`Xdebug helper`

![Xdebug helper](/images/2022/xdebug-helper.png "Xdebug helper")

然后在插件栏右键选项中配置`IDE key`为`PhpStrom`

![Xdebug helper setting](/images/2022/xdebug-helper-setting.png "Xdebug helper setting")

配置好插件后，鼠标左键可以选择`Debug`或者`Disable`。

下面在PHPStorm中点击右上角`Add Configuration` 新建一个`PHP Web Page`调试：

![Xdebug Setting](/images/2022/phpstorm-xdebug-1.png "Xdebug Setting")

![Xdebug Setting 2](/images/2022/phpstorm-xdebug-2.png "Xdebug Setting 2")

添加一个`Server`,其中Server name、host需要配置成.env中的`NGINX_HOST`即访问的域名,默认`localhost`，这里配置错误会导致`xdebug`找不到`Servername`。

`Use path mappings` 中将`/usr/share/nginx/html`配置上去。

![Xdebug Setting 3](/images/2022/phpstorm-xdebug-3.png "Xdebug Setting 3")

配置完后可以点击`Validate`验证下是否有错：

![Xdebug Setting 4](/images/2022/phpstorm-xdebug-4.png "Xdebug Setting 4")

准备就绪可以打开监听尝试调试：

![Xdebug Setting 5](/images/2022/phpstorm-xdebug-5.png "Xdebug Setting 5")

浏览器打开`http://localhost:8080/`会自动跳转到PHPStorm调试页面：

![Xdebug Setting 6](/images/2022/phpstorm-xdebug-6.png "Xdebug Setting 6")


**如果你需要调试命令行程序**，只需要进入容器终端，并输入`export XDEBUG_SESSION=1`，然后执行`php xxx.php`
同样可以拉起调试窗口:

![Xdebug Setting 7](/images/2022/phpstorm-xdebug-7.png "Xdebug Setting 7")

![Xdebug Setting 8](/images/2022/phpstorm-xdebug-8.png "Xdebug Setting 8")

## 附录
代码：[https://github.com/leolu9527/php-docker](https://github.com/leolu9527/php-docker)

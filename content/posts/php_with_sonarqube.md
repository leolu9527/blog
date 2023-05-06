---
title: "Docker部署 SonarQube, PHPStorm使用SonarLint插件进行代码检查"
date: 2022-10-20T14:44:35+08:00
description: ""
tags: [SonarQube, SonarLint, PHPStorm]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: PHP
comment: false
draft: false
---

## docker-compose 部署 `SonarQube Server`

`docker-compose.yml`

```yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:lts-community
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
      SONAR_ES_BOOTSTRAP_CHECKS_DISABLE: true
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```

启动：

```shell
docker-compose up -d
```

![创建项目](/images/2023/sonarqube/img_docker.png "创建项目")

访问`http://localhost:9000`,用户名`admin`,密码`admin`登录管理后台。

创建项目：

![创建项目](/images/2023/sonarqube/img.png "创建项目")

选择`Locally`用本地项目测试

![创建项目](/images/2023/sonarqube/img_1.png "创建项目")

点击`Generate`

![创建项目](/images/2023/sonarqube/img_2.png "创建项目")

点击`Continue`

![创建项目](/images/2023/sonarqube/img_3.png "创建项目")

选择项目类型

![创建项目](/images/2023/sonarqube/img_4.png "创建项目")

copy 命令 `sonar-scanner.bat -D"sonar.projectKey=test" -D"sonar.sources=." -D"sonar.host.url=http://localhost:9000" -D"sonar.login=xxx"`



根据提示下载 [sonarscanner](https://docs.sonarqube.org/9.9/analyzing-source-code/scanners/sonarscanner/) 到本地，windows下解压缩到任意目录，并将`bin`目录添加到`PATH`环境变量

切换到项目目录执行上一步中的命令:

```shell
sonar-scanner.bat -D"sonar.projectKey=test" -D"sonar.sources=." -D"sonar.host.url=http://localhost:9000" -D"sonar.login=xxx"
```

![创建项目](/images/2023/sonarqube/img_5.png "创建项目")

执行完毕，后台自动跳转,这样就完成了第一次项目代码的提交检查

![创建项目](/images/2023/sonarqube/img_6.png "创建项目")


**也可以使用配置文件的方式来替换上述命令**：

在项目根目录中创建配置文件`sonar-project.properties`,内容如下:
```dotenv
# must be unique in a given SonarQube instance
sonar.projectKey=test

sonar.projectName=test
# Path is relative to the sonar-project.properties file. Defaults to .
sonar.sources=.

sonar.login=xxx

```

替换`sonar.login`为你的`token`，然后执行，

```shell
sonar-scanner.bat -D"project.settings=./sonar-project.properties"
```
## SonarLint 配置

1. 安装SonarLint插件

![SonarLint](/images/2023/sonarqube/img_7.png "SonarLint")

点击`Add`

![SonarLint](/images/2023/sonarqube/img_8.png "SonarLint")

选择本地部署的SonarQube服务

![SonarLint](/images/2023/sonarqube/img_9.png "SonarLint")

填写`test`项目的`token`,即`sonar.login`的值

![SonarLint](/images/2023/sonarqube/img_10.png "SonarLint")


![SonarLint](/images/2023/sonarqube/img_11.png "SonarLint")


![SonarLint](/images/2023/sonarqube/img_12.png "SonarLint")

绑定项目

![SonarLint](/images/2023/sonarqube/img_13.png "SonarLint")

![SonarLint](/images/2023/sonarqube/img_14.png "SonarLint")

修改代码会自动提示代码缺陷

![SonarLint](/images/2023/sonarqube/img_15.png "SonarLint")
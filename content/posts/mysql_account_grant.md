---
title: "MySQL用户及权限操作"
date: 2022-08-16T21:06:51+08:00
description: ""
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories:
comment: false
draft: true
---

```shell

## 创建用户
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';


GRANT ALL ON db1.* TO 'jeffrey'@'localhost';

## 查看权限
SELECT * FROM mysql.user;
SHOW GRANTS FOR 'username'@'hostname';

```
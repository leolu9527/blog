---
title: "Docker部署MongoDB副本集及分片"
date: 2023-01-17T17:46:34+08:00
description: ""
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories:
comment: false
draft: true
---

## 副本集（Replica Set）

docker-compose.yml

```yaml
version: "3.7"
services:
  mongo0:
    container_name: mongo0
    image: mongo
    expose:
      - 27017
    ports:
      - 28017:27017
    restart: always
    entrypoint: [ "/usr/bin/mongod", "--bind_ip_all", "--replSet", "rs0"]
    volumes:
      - ./data/volume-mongo-rs/db0:/data/db
      - ./data/volume-mongo-rs/db0-config:/data/configdb
  mongo1:
    container_name: mongo1
    image: mongo
    expose:
      - 27017
    ports:
      - 28018:27017
    restart: always
    entrypoint: [ "/usr/bin/mongod", "--bind_ip_all", "--replSet", "rs0"]
    volumes:
      - ./data/volume-mongo-rs/db1:/data/db
      - ./data/volume-mongo-rs/db1-config:/data/configdb
    depends_on:
      - mongo0
  mongo2:
    container_name: mongo2
    image: mongo
    expose:
      - 27017
    ports:
      - 28019:27017
    restart: always
    entrypoint: [ "/usr/bin/mongod", "--bind_ip_all", "--replSet", "rs0"]
    volumes:
      - ./data/volume-mongo-rs/db2:/data/db
      - ./data/volume-mongo-rs/db2-config:/data/configdb
    depends_on:
      - mongo0
```
启动：

```shell
docker-compose up -d
```

配置：

```shell
//主节点
docker exec -it mongo0 /bin/bash

mongosh

//配置
test> rs.initiate({_id:"rs0",members:[{_id:0,host:"mongo0:27017", "priority": 2},{_id:1,host:"mongo1:27017"},{_id:2,host:"mongo2:27017"}]})
```

进入 mongo1 及 mongo2 执行下面的命令
```shell
rs0> db.getMongo().setReadPref('secondary')
```

PHP代码插入数据测试：

```php
$client = new \MongoDB\Client(
    'mongodb://mongo0,mongo1,mongo2/?replicaSet=rs0'
);
$collection = $client->test->users;
$insertOneResult = $collection->insertOne([
    'username' => 'admin',
    'email' => 'admin@example.com',
    'name' => 'Admin User',
]);

printf("Inserted %d document(s)\n", $insertOneResult->getInsertedCount());

var_dump($insertOneResult->getInsertedId());
```

```shell
//mongo0
rs0 [direct: primary] test> db.users.find({})
[
  {
    _id: ObjectId("643de7ea7af76cbf3a07a922"),
    username: 'admin',
    email: 'admin@example.com',
    name: 'Admin User'
  }
]

//mongo1
rs0 [direct: secondary] test> db.users.find({})
[
  {
    _id: ObjectId("643de7ea7af76cbf3a07a922"),
    username: 'admin',
    email: 'admin@example.com',
    name: 'Admin User'
  }
]
```

## Sharded Cluster

## References

1. https://www.cnblogs.com/evescn/p/16203350.html
2. https://cloud.tencent.com/developer/article/1026185
3. https://www.mongodb.com/docs/manual/replication/
4. https://github.com/minhhungit/mongodb-cluster-docker-compose
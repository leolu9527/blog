---
title: "ElasticSearch基础——安装及中文分词插件的安装"
date: 2022-09-25T17:54:09+08:00
description: ""
tags: [全文检索, ElasticSearch]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: ElasticSearch
comment: false
draft: true
---

## JDK 及 ElasticSearch 版本选择

查看官网[支持一览表](https://www.elastic.co/cn/support/matrix)，`ElasticSearch 7.17.x`以下支持`Oracle JDK1.8/OpenJDK 8`，
Kibana 版本跟 ElasticSearch 保持一致

下载：

 * ElasticSearch 7.17.x : [Link](https://www.elastic.co/cn/downloads/past-releases#elasticsearch)

 * Kibana 7.17.x : [Link](https://www.elastic.co/cn/downloads/past-releases#kibana)

## 运行 ElasticSearch 及 Kibana

解压缩两个压缩包到任意目录。

**运行 ElasticSearch：**

```shell
# powershell or shell
cd elasticsearch-7.17.10/

./bin/elasticsearch
```

可能遇到的错误：

1. `exception during geoip databases update`

    在`config/elasticsearch.yml`中添加如下配置：
    ```yaml
    ingest.geoip.downloader.enabled: false
    ```
2. `warning: usage of JAVA_HOME is deprecated, use ES_JAVA_HOME`
    
    添加环境变量`ES_JAVA_HOME` 值跟`JAVA_HOME`一样

当输出`started`时，服务已经启动成功

```shell

bin> ./elasticsearch
...
[INFO ][o.e.n.Node               ] [PC] started
[INFO ][o.e.l.LicenseService     ] [PC] license [1660ab9f-d2f1-4c30-b33d-94796ded1e56] mode [basic] - valid
...

```

浏览器访问 `http://localhost:9200/`, 默认端口`9200`，返回一段 `json` 信息，表示 `cluster` 已经成功启动。

**运行 Kibana：**

```shell
# powershell or shell
cd kibana-7.17.10/bin

bin> ./kibana
...
log   [10:43:22.239] [info][server][Kibana][http] http server running at http://localhost:5601
...
```

浏览器访问`http://localhost:5601`,kibana启动成功。


## Reference

1. https://www.zhihu.com/question/469207536/answer/2290001606
2. https://www.elastic.co/guide/en/elasticsearch/reference/8.7/docker.html
3. https://www.elastic.co/cn/support/matrix
4. https://elasticsearch.cn/article/6178
5. https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html
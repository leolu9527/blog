---
title: "ElasticSearch基础——概念，增删改查"
date: 2023-05-15T09:40:56+08:00
description: ""
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories:
comment: false
draft: true
---

## ElasticSearch 和关系型数据库 概念对应关系

| ElasticSearch                  | RDBMS              |
|:-------------------------------|:-------------------|
| Index(索引)                      | DataBase(数据库)      |
| Type(类型)                       | Table(表)           |
| Document(文档)                   | Row(行)             |
| Field(字段)                      | Column(列)          |
| Mapping(映射)                    | Schema(约束)         |
| Everything is indexed(存储的都是索引) | Index(索引)          |
| Query DSL(ES独特的查询语言)           | SQL                |
| GET http://…                   | select * from …    |
| POST http://…                  | update table set … |


## Reference

1. https://www.elastic.co/guide/en/elasticsearch/reference/7.17/removal-of-types.html#_what_are_mapping_types
2. https://github.com/xr2117/ElasticSearch7
3. https://juejin.cn/post/6844904056347951117
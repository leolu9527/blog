---
title: "ElasticSearch基础-文本分析，中文分词"
date: 2023-05-30T20:48:04+08:00
description: ""
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories:
comment: false
draft: true
---


## 中文分词插件`IK`安装

报错 ` access denied ("java.io.FilePermission""........\IKAnalyzer.cfg.xml""read"`

es 如果装了插件，路径文件夹不能有空格或者汉字

我把es装在这个目录 D:\Program Files\elasticsearch 了，Program Files中有个空格 0.0

在[github](https://github.com/medcl/elasticsearch-analysis-ik/releases)上下载对应ElasticSearch版本的压缩包，
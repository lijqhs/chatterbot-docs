# 用Elasticsearch建立机器人知识库

<!-- TOC -->

- [用Elasticsearch建立机器人知识库](#用elasticsearch建立机器人知识库)
    - [Elasticsearch的安装](#elasticsearch的安装)
    - [Elasticsearch入门教程](#elasticsearch入门教程)
    - [CURL](#curl)
    - [Kibana](#kibana)
    - [ChatterBot知识库创建](#chatterbot知识库创建)
        - [MongoDB知识库](#mongodb知识库)
        - [从MongoDB导入Elasticsearch](#从mongodb导入elasticsearch)
    - [Elasticsearch](#elasticsearch)
        - [集群状态查看](#集群状态查看)
        - [Elasticsearch工具](#elasticsearch工具)

<!-- /TOC -->

ChatterBot提供了基于SQLAlchemy和MongoDB的Storage Adapter来存储知识库。根据实际应用需要，我们也可以采用Elasticsearch来建立聊天机器人的知识库。下面将Elasticsearch所需的相关资料整理出来，以便入门少走弯路。

## Elasticsearch的安装

[官方安装文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html)

## Elasticsearch入门教程

[阮一峰的Elasticsearch入门教程](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)

## CURL

curl命令是一个利用URL规则在命令行下工作的文件传输工具。它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。作为一款强力工具，curl支持包括HTTP、HTTPS、ftp等众多协议，还支持POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。做网页处理流程和数据检索自动化，curl可以祝一臂之力。

## Kibana

Kibana是Elasticsearch的一个可视化工具

## ChatterBot知识库创建

### MongoDB知识库

### 从MongoDB导入Elasticsearch

[MongoDB导入ES](https://blog.csdn.net/mydeman/article/details/54808267)

## Elasticsearch

### 集群状态查看

```_cat```[命令](https://blog.csdn.net/pilihaotian/article/details/52460747)
```curl localhost:9200/_cat```

### Elasticsearch工具

[Elasticsearch插件大全](http://www.searchtech.pro/elasticsearch-plugins)

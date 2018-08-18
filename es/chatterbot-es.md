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
        - [Analyzer](#analyzer)
        - [自定义Analyzer](#自定义analyzer)
        - [Elasticsearch mappings 详解](#elasticsearch-mappings-详解)

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

### Analyzer

对字段采用```whitespace```分析器

```json
PUT /bot_kb_test 
{
    "mappings": {
        "bot_kb" : {
            "properties" : {
                "question" : {
                    "type" :    "text",
                    "analyzer": "whitespace"
                },
                "date" : {
                    "type" :   "date"
                },
                "answer" : {
                    "type" :   "text"
                }
            }
        }
    }
}
```

使用```analyze```API测试分词效果

```json
GET /bot_kb_test/_analyze
{
    "field": "question",
    "text": "手机 充值 失败"
}
```

测试结果可以看到使用```whitespace```分析器对字段进行分词

```json
{
    "tokens": [
        {
            "token": "手机",
            "start_offset": 0,
            "end_offset": 2,
            "type": "word",
            "position": 0
        },
        {
            "token": "充值",
            "start_offset": 3,
            "end_offset": 5,
            "type": "word",
            "position": 1
        },
        {
            "token": "失败",
            "start_offset": 6,
            "end_offset": 8,
            "type": "word",
            "position": 2
        }
    ]
}
```

### 自定义Analyzer

自定义的分析器可以包括三个部分：

- 字符过滤器，比如将```&```转换为```and```，或者将html标记去掉，有内置的```html_strip```字符过滤器，自定义的分析器可以包含零个或者多个字符过滤器；
- 分词器，分析器必须包含%%一个%%分词器，比如上述的内置```whitespace```分词器；
- 词项过滤器，Elasticsearch有许多词项过滤器，比如```lowercase```、词干过滤，根据不同语言又可分为很多种[Stemmer Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/analysis-stemmer-tokenfilter.html)，或者可以设置自定义停用词过滤器。

下面是一个自定义分析器的例子：

```json
PUT /bot_kb_analyzer_test
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":     "mapping",
                    "mappings": ["&=> and "]
                }
            },
            "filter": {
                "my_stopwords": {
                    "type":     "stop",
                    "stopwords": ["了", "么"]
                }
            },
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  ["&_to_and"],
                    "tokenizer":    "whitespace",
                    "filter":       ["lowercase", "my_stopwords"]
                }
            }
        }
    }
}
```

```json
GET /bot_kb_analyzer_test/_analyze
{
    "text": "零钱 账户 可以 解封 了 么",
    "analyzer": "my_analyzer"
}
```

输出结果看到测试效果：

```json
{
    "tokens": [
        {
            "token": "零钱",
            "start_offset": 0,
            "end_offset": 2,
            "type": "word",
            "position": 0
        },
        {
            "token": "账户",
            "start_offset": 3,
            "end_offset": 5,
            "type": "word",
            "position": 1
        },
        {
            "token": "可以",
            "start_offset": 6,
            "end_offset": 8,
            "type": "word",
            "position": 2
        },
        {
            "token": "解封",
            "start_offset": 9,
            "end_offset": 11,
            "type": "word",
            "position": 3
        }
    ]
}
```

我们可以在mapping中对某个属性字段应用自定义分析器：

```json
PUT /my_index/_mapping/my_type
{
    "properties": {
        "title": {
            "type":      "text",
            "analyzer":  "my_analyzer"
        }
    }
}
```

### Elasticsearch mappings 详解
[Elasticsearch 5.4 Mapping详解](https://blog.csdn.net/napoay/article/details/73100110)

## ES同义词设置
同义词词典设置如下，保存为synonym.txt，放置在ES安装目录/config/analysis/synonym.txt
```
开通, 申请
为什么, 为啥
```

在ES的mapping设置中，增加setting：
```
PUT /synonym_test_index
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "synonym_analyzer" : {
                        "tokenizer" : "whitespace",
                        "filter" : ["synonym_filter"]
                    }
                },
                "filter" : {
                    "synonym_filter" : {
                        "type" : "synonym",
                        "synonyms_path" : "analysis/synonym.txt"
                    }
                }
            }
        }
    },
    "mappings": {
        "test_type": {
            "properties": {
                "my_text": {
                    "type": "text", 
                    "analyzer": "synonym_analyzer"
                }
            }
        }
    }
}
```

测试语句：
```
POST sysnonym_test_index/_analyze
{
    "field": "my_text",
    "text": "开通 实名帐户"
}
```
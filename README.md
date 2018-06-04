
# 《ChatterBot聊天机器人简易手册》
详细内容请参考[原版用户手册](http://chatterbot.readthedocs.io/en/stable/tutorial.html)。
<!-- TOC -->

- [《ChatterBot聊天机器人简易手册》](#chatterbot聊天机器人简易手册)
- [安装](#安装)
    - [安装ChatterBot和Corpus](#安装chatterbot和corpus)
- [搭建机器人](#搭建机器人)
- [一次生成知识库](#一次生成知识库)
- [逻辑适配器 Logic Adapter](#逻辑适配器-logic-adapter)
    - [MultiLogicAdapter](#multilogicadapter)
    - [逻辑适配器选取回复语句的方式](#逻辑适配器选取回复语句的方式)
    - [设置回复语句选择方法](#设置回复语句选择方法)
    - [内置逻辑适配器](#内置逻辑适配器)
        - [最佳匹配适配器（BestMatchAdapter）](#最佳匹配适配器bestmatchadapter)
        - [时间适配器（TimeLogicAdapter）](#时间适配器timelogicadapter)
        - [数学运算适配器 Mathematical Evaluation Adapter](#数学运算适配器-mathematical-evaluation-adapter)
        - [低置信度适配器（LowConfidenceAdapter）](#低置信度适配器lowconfidenceadapter)
        - [特定回复适配器（SpecificResponseAdapter）](#特定回复适配器specificresponseadapter)
    - [自建逻辑适配器](#自建逻辑适配器)
- [输入适配器 Input Adapter](#输入适配器-input-adapter)
- [输出适配器 Output Adapter](#输出适配器-output-adapter)
- [存储适配器 Storage Adapter](#存储适配器-storage-adapter)
- [过滤器 Filter](#过滤器-filter)
- [对话过程](#对话过程)
    - [Statement对象](#statement对象)
    - [Response对象](#response对象)
- [回复语句的比较](#回复语句的比较)
    - [使用比较方法](#使用比较方法)
    - [处理流程](#处理流程)
- [ChatterBot的优点](#chatterbot的优点)
- [ChatterBot的缺点](#chatterbot的缺点)

<!-- /TOC -->

# 安装
## 安装ChatterBot和Corpus
从Github上下载[ChatterBot](https://github.com/gunthercox/ChatterBot)和[corpus](https://github.com/gunthercox/chatterbot-corpus)Python源码，解压并定位至本地目录，然后
```python
python setup.py install
```
# 搭建机器人
* [使用ChatterBot做简单机器人](https://blog.csdn.net/u013378306/article/details/64129696)
* [搭建不同adapter的聊天机器人](https://blog.csdn.net/qq_28168421/article/details/71108106)

# 一次生成知识库
ChatterBot可以通过脚本注入sqlite数据库（关闭只读模式）作为知识库，所以可以先写个脚本来把知识库注入sqlite数据库：
```python
from chatterbot import ChatBot
from chatterbot.trainers import ChatterBotCorpusTrainer
import logging

class HelloChat():
    def __init__(self):
        self.chatbot = ChatBot('Hello Bot')            
        self.chatbot.set_trainer(ChatterBotCorpusTrainer)
        self.chatbot.train('./knowledge.yml')

if __name__ == '__main__':
    chat = HelloChat()
    print('finance.yml生成知识数据库db.sqlite3')
```
上述代码文件名hello_db.py，在代码目录中执行`python hello_db.py`
运行之后目录下生成了一个`db.sqlite3`文件，这就是知识库。然后再次创建不需要再加载knowledge.yml训练知识库，并且打开只读模式，否则ChatterBot会不断学习，修改知识库，而这个过程实际不应该让用户来参与：
```python
class HelloChat():

    def __init__(self):
        self.chatbot = ChatBot('Hello Bot', read_only=True)

    def get_response(self, info):
        return str(self.chatbot.get_response(info))

if __name__ == '__main__':
    chat = HelloChat()
    while True:
        try:
            reply = chat.get_response(input('>'))
            print(reply)
        except (KeyboardInterrupt, KeyError, SystemExit):
            break
```

# 逻辑适配器 Logic Adapter
ChatterBot中的Logic Adapter是插件式设计。在创建ChatterBot实例的时候，通过以下方法配置逻辑适配器（可以有多个）：
```python
chatbot = ChatBot('Hello Bot',
    logic_adapters=[
        {
             'import_path': 'hello_adapter.HelloAdapter'
         },
        {
             'import_path': 'hello_adapter.BestMatch',
             "statement_comparison_function": "chatterbot.comparisons.levenshtein_distance",
             "response_selection_method": "chatterbot.response_selection.get_first_response"
         }
	])
```
3	主进程在ChatterBot对象被初始化时会将用户配置的逻辑适配器到一个列表中，然后交MultiLogicAdapter 进行处理。

## MultiLogicAdapter 
MultiLogicAdapter 依次调用每个 Logic Adapter，Logic Adapter 被调用时先执行can_process 方式判断输入是否可以命中这个逻辑处理插件。比如”今天天气怎么样“这样的问题显然需要命中天气逻辑处理插件，这时时间逻辑处理插件的can_process 则会返回False。在命中后相应的Logic Adapter 负责计算出对应的回答（Statement对象）以及可信度（confidence），MultiLogicAdapter会取可信度最高的回答，并进入下一步。

用来从配置中所有逻辑适配器中返回一条回复。每个逻辑适配器返回一个回复语句和一个置信分数，MultiLogicAdapter返回分数最高的回复语句。MultiLogicAdapter设计中有一个小trick需要注意一下，但是当有多个逻辑适配器返回的回复语句A相同，即使还有更高分数的回复语句B，A也会被赋予更高优先级。如下表所示，`早上好`将被MultiLogicAdapter选中被返回：

|置信分数|语句|
|----|----|
|0.2|早上好|
|0.5|早上好|
|0.7|晚上好|

## 逻辑适配器选取回复语句的方式
ChatterBot内建了几个回复语句选择方法，chatterbot.response_selection.*方法名称*：
- get_first_response
- get_most_frequent_response
- get_random_response

除此之外，还有自建回复语句选择方法，格式如下：
```python
def select_response(statement, statement_list):
    # Your selection logic
    return selected_statement
```

## 设置回复语句选择方法
需要将回复语句选择方法在初始化时作为参数传入构造函数：
```python
from chatterbot import ChatBot
from chatterbot.response_selection import get_most_frequent_response

chatbot = ChatBot(
    # ...
    response_selection_method=get_most_frequent_response
)
```
## 内置逻辑适配器
### 最佳匹配适配器（BestMatchAdapter）
顾名思义，选取与输入最匹配的回复语句。
```python
chatbot = ChatBot(
    "My ChatterBot",
    logic_adapters=[
        {
            "import_path": "chatterbot.logic.BestMatch",
            "statement_comparison_function": "chatterbot.comparisons.levenshtein_distance",
            "response_selection_method": "chatterbot.response_selection.get_first_response"
        }
    ]
)
```
### 时间适配器（TimeLogicAdapter）
该适配器用来回复当前时间。

### 数学运算适配器 Mathematical Evaluation Adapter
该适配器检测到语句中含有数学表达式，系统将回复这个表达式和运算后的值。

### 低置信度适配器（LowConfidenceAdapter）
该适配器设定一个默认语句，和置信分阈值，当所有返回语句的置信分数低于该阈值，默认语句将被返回。
```python
# Create a new instance of a ChatBot
bot = ChatBot(
    'Default Response Example Bot',
    storage_adapter='chatterbot.storage.SQLStorageAdapter',
    logic_adapters=[
        {
            'import_path': 'chatterbot.logic.BestMatch'
        },
        {
            'import_path': 'chatterbot.logic.LowConfidenceAdapter',
            'threshold': 0.65,
            'default_response': 'I am sorry, but I do not understand.'
        }
    ],
    trainer='chatterbot.trainers.ListTrainer'
)
```
有没有注意到，以上代码中logic_adapters列表中可以设置多个Adapter！在这个例子中，当所有适配器的回复语句置信分数小于0.65，**I am sorry, but I do not understand.** 将被返回。

### 特定回复适配器（SpecificResponseAdapter）
给特定语句设定特定回复：
```python
# -*- coding: utf-8 -*-
from chatterbot import ChatBot

# Create a new instance of a ChatBot
bot = ChatBot(
    'Exact Response Example Bot',
    storage_adapter='chatterbot.storage.SQLStorageAdapter',
    logic_adapters=[
        {
            'import_path': 'chatterbot.logic.BestMatch'
        },
        {
            'import_path': 'chatterbot.logic.SpecificResponseAdapter',
            'input_text': '你是谁？',
            'output_text': '你猜！'
        }
    ],
    trainer='chatterbot.trainers.ListTrainer'
)
# Get a response given the specific input
response = bot.get_response('你是谁？')
print(response)
```
## 自建逻辑适配器
一个简单的自定义Logic Adapter的示例`HelloAdapter`：
```python
# hello_bot.py
from chatterbot import ChatBot
from chatterbot.trainers import ChatterBotCorpusTrainer
from hello_adapter import HelloAdapter
# import hello_adapter
import logging

logging.basicConfig(level=logging.INFO)


class HelloChat():

    def __init__(self):
        self.chatbot = ChatBot('Hello Bot',
            # storage_adapter= 'chatterbot.storage.SQLStorageAdapter',
            logic_adapters=[
                {
                    'import_path': 'hello_adapter.HelloAdapter',
                    "statement_comparison_function": "chatterbot.comparisons.levenshtein_distance",
                    "response_selection_method": "chatterbot.response_selection.get_first_response"
                }
            ],
            # input_adapter='chatterbot.input.VariableInputTypeAdapter',
            # output_adapter='chatterbot.output.TerminalAdapter'
            )
            
        self.chatbot.set_trainer(ChatterBotCorpusTrainer)
        self.chatbot.train('./finance.yml')

    def get_response(self, info):
        return str(self.chatbot.get_response(info))



if __name__ == '__main__':
    chat = HelloChat()
    while True:
        try:
            reply = chat.get_response(input('>'))
            print(reply)
        except (KeyboardInterrupt, KeyError, SystemExit):
            break
```


# 输入适配器 Input Adapter
ChatterBot对不同输入设计了不同的适配器，目的是将输入转为ChatterBot能够处理的格式。
- 多类型输入适配器（chatterbot.input.VariableInputTypeAdapter）
- 终端输入适配器（chatterbot.input.TerminalAdapter）
- Gitter适配器（chatterbot.input.Gitter），Gitter是第三方聊天室软件
- HipChat适配器（chatterbot.input.HipChat），HipChat第三方聊天软件
- Mailgun适配器（chatterbot.input.Mailgun），从Mailgun获取email输入
- 微软聊天机器人（chatterbot.input.Microsoft），对接[微软的Azure Bot Service](https://azure.microsoft.com/zh-cn/services/bot-service/)，参考[这里](https://www.cnblogs.com/DarrenChan/p/7301380.html)。
- 自定义输入适配器，通过实现一个抽象类InputAdapter来创建新的输入适配器

# 输出适配器 Output Adapter
输出适配器的目的是将机器人回复传输到对应的API终端。
- 终端适配器（chatterbot.output.TerminalAdapter）
- Gitter适配器（chatterbot.output.Gitter）
- HipChat适配器（chatterbot.output.HipChat）
- Mailgun适配器（chatterbot.output.Mailgun）
- 微软聊天机器人适配器（chatterbot.output.Microsoft）
- 自定义输出适配器，实现抽象类chatterbot.output.OutputAdapter

# 存储适配器 Storage Adapter
包括SQL Alchemy、MongoDB，用来存储对话。这个需要详细了解一下Python访问数据库的方法，参考[这里](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432010325987131e75bf6b3543429a2975f88ce8ffa9000)。
- SQL存储适配器
- MongoDB存储适配器
- 自定义存储适配器，实现抽象类chatterbot.storage.StorageAdapter

# 过滤器 Filter
过滤器将减少聊天机器人在选择响应时必须处理的语句数量。

# 对话过程
ChatterBot支持多个并发聊天会话的能力。聊天会话是聊天机器人与用户交互的地方，并且支持多个聊天会话意味着您的聊天机器人可以同时与不同的人进行多个不同的对话。
## Statement对象
ChatterBot的Statement对象表示机器人从用户收到的一个输入语句，或者表示机器人对于某个输入的回复语句。
## Response对象
Response对象表示了两个Statement对象的关系，每个Statement对象都有一个in_response_to引用来链接它所回复的Statement对象。
![Statement-response关系图](http://chatterbot.readthedocs.io/en/stable/_images/statement-response-relationship.svg)
Response对象的occurence属性表示了一个Statement对象被作为回复的次数。
![Response对象](http://chatterbot.readthedocs.io/en/stable/_images/statement-relationship.svg)

# 回复语句的比较
机器人通过对语句的比较来选择相应的回复，ChatterBot包含了以下几种语句比较方法：
1. Jaccard相似性（chatterbot.comparisons.JaccardSimilarity），需要用到NLTK的**wordnet**语料库。
2. Levenshtein距离（chatterbot.comparisons.LevenshteinDistance），参考[Wikipedia](https://en.wikipedia.org/wiki/Levenshtein_distance).
3. 情感分析比较（chatterbot.comparisons.SentimentComparison），需要用到NLTK的**vader**字典，[情感分析方法](https://www.cnblogs.com/arkenstone/p/6064196.html)，[NLTK情感分析器](https://blog.csdn.net/sinat_36972314/article/details/79621591)。
4. 同义词距离（chatterbot.comparisons.SynsetDistance），需要用到NLTK的**wordnet**语料库
## 使用比较方法
在创建ChatterBot对象时，需要设置`statement_comparison_function`参数：
```python
from chatterbot import ChatBot
from chatterbot.comparisons import levenshtein_distance

chatbot = ChatBot(
    # ...
    statement_comparison_function=levenshtein_distance
)
```
## 处理流程
ChatterBot对话处理流程：
![ChatterBot处理流程](https://i.imgur.com/a2rdxDi.png)

# ChatterBot的优点

1. 训练语料可存放在多种介质上 
2. 训练结果可存放在多种介质上 
3. 应答匹配算法支持多种应答匹配算法：相似度匹配、数学估值算法等 
4. 可训练支持任何语言的聊天机器人

# ChatterBot的缺点

1. 性能较低：收到聊天请求时，其需要遍历所有语料以找到相似度最高的语句，并提取对应的应答语句。因此，训练语料过多时（超过1万条），应答时延可能已无法让人接受。 
2. 场景有限：其只能应用到一些情况简单、场景单一的环境。由于性能较低，因此，无法使用过多的语料对ChatterBot进行训练，这也必然限制了应用场景。

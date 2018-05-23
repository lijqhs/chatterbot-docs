# 《ChatterBot聊天机器人搭建指南》

http://chatterbot.readthedocs.io/en/stable/tutorial.html

## 安装
### 安装ChatterBot
- 在线安装
```python
pip install chatterbot
```
- 本地安装
或者从[Github](https://github.com/gunthercox/ChatterBot)上下载Python源码，解压并定位至本地目录，然后
```python
python setup.py install
```
### 安装Corpus
- 在线安装
```python
pip install chatterbot-corpus
```
- 本地安装
或者从[Github](https://github.com/gunthercox/chatterbot-corpus)上下载Python源码，解压并定位至本地目录，然后
```python
python setup.py install
```

## 搭建机器人
* [使用ChatterBot做简单机器人](https://blog.csdn.net/u013378306/article/details/64129696)
* [搭建不同adapter的聊天机器人](https://blog.csdn.net/qq_28168421/article/details/71108106)

# 适配器 Adapter
## 逻辑适配器 Logic Adapter
### MultiLogicAdapter 
用来从配置中所有逻辑适配器中返回一条回答。每个逻辑适配器返回一个答案和一个置信分数，MultiLogicAdapter返回分数最高的答案。但是当多个逻辑适配器返回的答案相同，即使还有更高分数的答案，这个多条相似答案也会被赋予更高优先级。如下表所示，早上好将被MultiLogicAdapter选中被返回：

|置信分数|语句|
|----|----|
|0.2|早上好|
|0.5|早上好|
|0.7|晚上好|

### 逻辑适配器选取回答的方式
ChatterBot内建了几个回答选择方法，chatterbot.response_selection.*方法名称*：
- get_first_response
- get_most_frequent_response
- get_random_response
除此之外，还有自建回答选择方法，格式如下：
```python
def select_response(statement, statement_list):
    # Your selection logic
    return selected_statement
```

### 设置回答选择方法
需要将回答选择方法在初始化时作为参数传入构造函数：
```python
from chatterbot import ChatBot
from chatterbot.response_selection import get_most_frequent_response

chatbot = ChatBot(
    # ...
    response_selection_method=get_most_frequent_response
)
```
### 最佳匹配适配器（BestMatchAdapter）
顾名思义，选取与输入最匹配的答案。
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
该适配器用来返回当前时间。

### 数学运算适配器 Mathematical Evaluation Adapter
该适配器检测到语句中含有数学表达式，系统将返回这个表达式和运算后的值，我理解应该是简单的代数计算。

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
有没有注意到，以上代码中logic_adapters列表中可以设置多个Adapter哦！在这个例子中，当所有适配器的回答语句置信分数小于0.65，**I am sorry, but I do not understand.** 将被返回。

### 特定回应适配器（SpecificResponseAdapter）
让你猜猜我在想什么，猜中有奖。当遇到特定语句时，特定语句被返回。
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
            'input_text': '什么',
            'output_text': '恭喜你答对了！'
        }
    ],
    trainer='chatterbot.trainers.ListTrainer'
)
# Get a response given the specific input
response = bot.get_response(input('猜猜我在想什么：'))
print(response)
```
程序运行结果如下
> 猜猜我在想什么：什么

> 恭喜你答对了！

## 自建逻辑适配器
TODO

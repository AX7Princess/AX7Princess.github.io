---
description: ""
title: "初识流式输出"
draft: false
date: "2026-08-07T23:42:45+08:00"
slug: "Agent"
categories:
 - Agent
tags:
 - null
image: ""
---

## 流式接收代码结构
```
from openai import OpenAI
client =OpenAI(api_key="",base_url="https://api.deepseek.com")
message=[{"role":"user","content":"你是哪个大模型"}]
response=client.chat.completions.create(model="deepseek-v4-flash",messages=message,stream=True,reasoning_effort="low",extra_body={"thinking":{"type":"enabled"}},)
reasoning_content=""
content=""

for chunk in response:
    if chunk.choices[0].delta.reasoning_content:
        reasoning_content+=chunk.choices[0].delta.reasoning_content
    if chunk.choices[0].delta.content: 
        content +=chunk.choices[0].delta.content
print("Reasoning Content:", reasoning_content)
print("Content:", content)

```

Chunk 就是"模型把一句话拆成一截一截，一截一截往外吐"里的那一截。 每次吐出来的那一小块，就叫一个 chunk
## 流式解析里的经典坑

流式返回里不是每个 chunk 都带文字——开头只有角色标记、收尾只有结束标志，delta.content 可能是 None。直接 += 会报 TypeError: can only concatenate str (not 'NoneType')。所以拼接前要先判空：if delta.content: 再累加。这是流式解析的标准写法

delta.content 有时是 None，直接拿它去 += 字符串就炸了。
stream=True 时，模型一截截吐 chunk，不是每个 chunk 都带文字,典型有 3 种"空 chunk"：

流刚开始  只有 role="assistant" 的角色标记，没有 content

思考阶段  只有 reasoning_content（草稿纸），content 是空

流快结束  收尾 chunk 只带 finish_reason，content 是 None

##  choices 为什么是数组？ 因为 OpenAI 系 SDK 支持一个请求同时生成多个候选答案（参数 n=2 就是让它给两份不同回答）。每个候选是列表里的一项。但你日常流式场景只有一个候选，所以永远取第 0 个 → choices[0]

## 剥到 choices[0] 之后，里面那个 delta 才是真正装着文字的地方。delta 的英文原意就是"增量"——它不是完整答案，而是这一截新长出来的那一小块。

delta.content 正式回答的文字增量 

delta.reasoning_content 思考过程的文字增量

delta.role 角色标记（只在第一截出现）

##  messages 里三种角色各管什么

"system" 规则制定者（你设定的人设/规则）

"user"  用户说的话

"assistant"   模型之前说过的话
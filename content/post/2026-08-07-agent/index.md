---
description: ""
title: ""
draft: true
date: "2026-08-07T23:42:45+08:00"
slug: "Agent"
categories:
 - Agent
tags:
 - 
image: ""
---

## 流式接收代码结构
`from openai import OpenAI
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
print("Content:", content)`

Chunk 就是"模型把一句话拆成一截一截，一截一截往外吐"里的那一截。 每次吐出来的那一小块，就叫一个 chunk
## 流式解析里的经典坑

流式返回里不是每个 chunk 都带文字——开头只有角色标记、收尾只有结束标志，delta.content 可能是 None。直接 += 会报 TypeError: can only concatenate str (not 'NoneType')。所以拼接前要先判空：if delta.content: 再累加。这是流式解析的标准写法

delta.content 有时是 None，直接拿它去 += 字符串就炸了。
stream=True 时，模型一截截吐 chunk，不是每个 chunk 都带文字,典型有 3 种"空 chunk"：

流刚开始  只有 role="assistant" 的角色标记，没有 content

思考阶段  只有 reasoning_content（草稿纸），content 是空

流快结束  收尾 chunk 只带 finish_reason，content 是 None
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
`response = client.chat.completions.create(
    model="deepseek-v4-pro",
    messages=messages,
    stream=True,
    reasoning_effort="high",      
    extra_body={"thinking": {"type": "enabled"}},
)

reasoning_content = "" 
content = ""

  #流式输出时模型按 chunk 逐段返回，初始化两个字符串累加器：reasoning_content 攒思考过程、content 攒正式答案，循环里用 += 拼接，最后得到完整文本。这样能边收边显示，用户不用等全部生成完。

for chunk in response:
    delta = chunk.choices[0].delta      # 每段数据里的"增量"
    if delta.reasoning_content:
        reasoning_content += delta.reasoning_content
    if delta.content:
        content += delta.content

print("思考过程:", reasoning_content)
print("最终回答:", content)`

Chunk 就是"模型把一句话拆成一截一截，一截一截往外吐"里的那一截。 每次吐出来的那一小块，就叫一个 chunk


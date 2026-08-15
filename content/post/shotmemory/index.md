---
description: ""
title: "ShotMemory"
draft: true
date: "2026-08-15T00:04:07+08:00"
slug: "shotmemory"
categories:
 - 
tags:
 - 
image: ""
---

## Agent短期记忆

大模型是没有记忆的，它不会记住上次聊了什么，每次聊天都是一个新的任务。
短期记忆 = "给大模型的对话历史设一个上限"，因为模型每次只能"看"有限的上下文（token 窗口），你不可能把无限长的聊天记录全塞给它。主要包括四个概念， 全量缓冲 / 滑动窗口 / token 裁剪 / 保留策略

 ### 全量缓冲（conversation buffer）

把所有对话一条不落全存着，像记账一样，代价就是每次给大模型输入的历史逐渐增加，Token消耗也逐渐增加直至最大上下文窗口。

```
buffer = [
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好，有什么可以帮你？"},
    {"role": "user", "content": "我叫小王"},
    # ... 越攒越多,永不丢弃
]

```

### 滑动窗口（sliding window）

给模型看的时候，只取最近 N 条——像手机相册"最近 10 张，滑动窗口解决的是"给模型的输入太长"，不是"存储"太长。

### token 裁剪（trim）




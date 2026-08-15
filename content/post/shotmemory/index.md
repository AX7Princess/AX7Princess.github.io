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
```
# 从 buffer 里取:系统提示 + 最近 N 条
def context(self):
    return sys_msgs + buffer[-N:]   # 只看最近 N 条
```
给模型看的时候，只取最近 N 条——像手机相册"最近 10 张，滑动窗口解决的是"给模型的输入太长"，不是"存储"太长。
窗口只给模型看最近 10 条（输入可控），但 buffer 里存了 100 轮的原文 → 内存/文件越来越大，重启时加载越来越慢。

### token 裁剪（trim）

抽屉真的装不下了，扔最旧的东西。这是"真删"——和窗口的"只是不给你看"完全不同。裁剪直接改 buffer。

### 保留策略（retention policy）

哪些消息是"红线"，永远不能删——就是 system_roles（系统提示）。

```
system_roles = ("system",)   # 系统提示是红线
# 窗口和裁剪都先把它单独拎出来,永远保留
sys_msgs = [m for m in buffer if m["role"] in system_roles]
```
## 三者关系：由粗到细的三级防线

全量缓冲(全留,最费token)
   ↓ 取上下文时
滑动窗口(只看最近N条,不删)      ← 管"输入"长短
   ↓ buffer 真的超了
token 裁剪(从头部真删,超了删)    ← 管"存储"大小
   ↓ 但无论怎么删
保留策略(系统提示永远不动)       ← 红线,兜底

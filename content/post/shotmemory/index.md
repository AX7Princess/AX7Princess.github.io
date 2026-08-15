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
短期记忆 = "给大模型的对话历史设一个上限"，因为模型每次只能"看"有限的上下文（token 窗口），你不可能把无限长的聊天记录全塞给它。

### 全量缓冲 / 滑动窗口 / token 裁剪 / 保留策略


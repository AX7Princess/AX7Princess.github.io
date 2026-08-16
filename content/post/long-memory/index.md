---
description: ""
title: "长期记忆"
draft: true
date: "2026-08-16T01:03:01+08:00"
slug: "long-memory"
categories:
 - 
tags:
 - 
image: ""
---

语义嵌入（文字→向量）+ 向量库落盘（Chroma）+ 跨会话召回（相似度检索）

心智模型：

用户事实/偏好
   ├─► 精确 KV(SQLite)   → "用户喜欢DeepSeek"=确定事实, key/value 秒查
   └─► 语义向量(Chroma)  → "聊过什么相关话题"=模糊语义, 相似度召回
            │
            ▼ 首次 add 时 Chroma 自动嵌入, 落盘 ./mem_store/
   跨会话: 程序重启 → 重新连 PersistentClient → 之前存的事实还在

三个必须记住的点 ：

短期 vs 长期是两种"不忘"：短期=本次对话不丢；长期=跨进程跨天不丢
Chroma 首次 add 会联网下载嵌入模型（all-MiniLM 约 80MB）——需要联网
KV 和向量互补：确定事实（名字、默认模型）用 SQLite；语义相关历史才走向量
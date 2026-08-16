---
description: ""
title: "长期记忆"
draft: false
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

## 向量库和"写本地文件"到底差在哪

文件靠"精确匹配"（key 对上才有），向量库靠"语义相似"（意思接近就能找出来）

长期记忆的完整过程：三个步骤

假设用户昨天说"我特别不喜欢用 GPT，觉得它回答太啰嗦"，你想让它今天还能"记得"这件事。过程分三步：

第 1 步：嵌入（embedding）——把文字变成"指纹"

计算机看不懂文字，只看得懂数字。 嵌入就是把一句话变成一串固定长度的数字（向量），比如 384 个数字：

关键：这个"变数字"的过程，是让意思相近的句子，数字也相近。这是嵌入模型（比如 all-MiniLM）训练出来的本领。

第 2 步：落盘——把"指纹 + 原文"一起存进本地文件夹
Chroma 会把这个数字指纹和原文一起写进你本地的 ./mem_store/ 文件夹（其实就是一些文件）：

这就是"写本地文件"！ 向量库底层也是文件，只是它额外多存了那份"数字指纹"，并且把指纹组织成"能快速按相似度找"的索引结构。

第 3 步：召回（recall）——把新问题也变指纹，然后"找指纹最像的"
用户今天问"你觉得我用哪个模型好？"

先把这句话也变成指纹（同样的嵌入模型）
然后拿这个指纹去 mem_store 里找"指纹最像的几条"（数学上叫"算余弦相似度"）
找到 "我特别不喜欢用 GPT" → 把它塞进这次的上下文里告诉模型

**所以长期记忆的完整过程 = 存的时候"文字→指纹→落盘"；查的时候"问题→指纹→找最像的→取回原文"**
## 三个存储方式对比（一张表记牢）

| 方式 | 怎么查 | 适合 | 类比 |
|------|--------|------|------|
| **本地文件（JSON/txt）** | 按 key 精确读 | 少量确定数据 | 手写便签 |
| **SQLite（KV）** | 按 key 秒查、可更新 | 用户画像：名字、默认模型 | 有编号的档案柜 |
| **Chroma（向量）** | 按语义相似度召回 | 模糊历史、偏好、相关话题 | 按内容指纹找书的图书馆 |

向量库并没有"更高级"，它只是"换了一种查法"——用语义相似度代替精确 key。 你要存"用户叫什么"这种确定信息，SQLite/文件就够了；你要存"用户之前吐槽过什么"这种需要"意思接近就找到"的，才用向量库。两者配合才完整

add_fact = RAG 里的"写入/索引"（chunk + embed + store）
recall = RAG 里的"检索"（query embed + similarity search）
PersistentClient(path=...) = 让向量库落到磁盘，不是内存版


```
'''
长期记忆学习文档
长期记忆 = 关电脑也不丢
长期记忆 = 单文档 RAG + 落盘
    嵌入/相似度召回  → 只是把"文档"换成"用户事实"且持久化
pip install chromadb
'''
import chromadb
from pathlib import Path

BASE=Path(__file__).parent

class LongTermMemory:
    def __init__(self,path=str(BASE/"mem_store"),collection="long_term"):
        self.client=chromadb.PersistentClient(path=path) #建客户端(连上本地存储)
        self.col=self.client.get_or_create_collection(collection)# 拿货架(没有就建)

    def add_fact(self,text:str,fid:str): #Chroma 的 add 要求每个参数都是列表（ids=[...]、documents=[...]）——因为 Chroma 的 add 支持批量加。就算你只存一条，也要写成 [fid]、[text]
        self.col.add(ids=[fid],documents=[text])

    def recall(self,query:str,k:int=3)->list[str]:
        res=self.col.query(query_texts=[query],n_results=k)
        return res["documents"][0] if res["documents"] else []

    def delete(self,fid:str):
        self.col.delete(ids=[fid])
```
```
import sqlite3, json
from pathlib import Path
BASE = Path(__file__).parent

class ProfileStore:
    def __init__(self,db_path=str(BASE / "memory.db")):
        self.con=sqlite3.connect(db_path)
        self.con.execute("CREATE TABLE IF NOT EXISTS profile (k TEXT PRIMARY KEY, v TEXT)")
        self.con.commit()

    def set_profile(self, k, v):
        self.con.execute("INSERT OR REPLACE INTO profile VALUES(?,?)",(k,json.dumps(v)))

    def get_profile(self,k):
        row=self.con.execute("SELECT v FROM profile WHERE k=?",(k,)).fetchone()
        return json.loads(row[0]) if row else None

    
```























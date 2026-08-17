---
description: ""
title: "记忆本地化"
draft: true
date: "2026-08-17T07:22:41+08:00"
slug: "local-memory"
categories:
 - 
tags:
 - 
image: ""
---

三类持久化不是替代关系，是分层：纯文件=配置/简单开关；SQLite=用户画像 KV；Chroma=语义召回

存得稳，路径绝对化 + 隐私分流 + 三层整理，路径不漂移、隐私不出网、分层，给“账本”找保险柜 + 贴标签 + 分类

默认不泄露：recall() 默认 include_private=False → 敏感事实不会自动拼进 system 提示发去云端 API
要带才带：只有用户显式确认（或你明确需要）时，传 include_private=True 才召回敏感事实
存的时候用 add_fact(text, fid, private=True) 标记敏感条目

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
import chromadb, time
from memory.paths import MEM_STORE 
#BASE=Path(__file__).parent

class LongTermMemory:
    def __init__(self, path=MEM_STORE, collection_name="long_term"):
        self.client = chromadb.PersistentClient(path=path)  # 绝对路径
        self.col = self.client.get_or_create_collection(collection_name)

    def add_fact(self, text: str, fid: str, private: bool = False):
        """存一条长期事实。private=True 表示敏感, 召回时默认不出网。"""
        self.col.add(ids=[fid], documents=[text],
                     metadatas=[{"private": private, "ts": time.time()}])

        #Chroma 的 add 要求每个参数都是列表（ids=[...]、documents=[...]）——因为 Chroma 的 add 支持批量加。就算你只存一条，也要写成 [fid]、[text]

    def recall(self,query:str,k:int=3,include_private=False)->list[str]:
        """召回相关事实。默认跳过 private=True 的敏感事实(不进 prompt)。"""
        res=self.col.query(query_texts=[query],n_results=k)
        facts=[]
        for doc,meta in zip(res["documents"][0],res["metadatas"][0]):
            if  meta and meta.get("private") and not include_private:
                continue
            facts.append(doc)
        return facts
    def delete(self,fid:str):
        self.col.delete(ids=[fid])

```

```
"""集中管理所有落盘路径, 避免各处硬编码 './xxx' (相对路径会随 cwd 漂移)."""
# ============ 隐私分流（设计约定, 不是代码自动给的） ============
# 🔒 永远留本地、绝不进 prompt: 用户真实姓名 / 联系方式 / 密钥 / 明文敏感偏好
# 🌐 会发到云端 LLM API: 你主动塞进 system/user 消息的"召回结果"
#     (如"已知用户长期事实: 喜欢 DeepSeek")
# ⚠️ 敏感事实: 存本地, 但召回时不自动进 prompt, 需用户显式确认才带上
# ==============================================================

import os
BASE = os.path.dirname(os.path.abspath(__file__))
MEM_STORE   = os.path.join(BASE, "mem_store")    # Chroma 向量库
PROFILE_DB  = os.path.join(BASE, "memory.db")    # SQLite 用户画像
FACTS_JSON  = os.path.join(BASE, "facts.json")   # 纯本地简单事实
if __name__=="__main__":
    print(BASE,"\n",MEM_STORE,"\n",PROFILE_DB,"\n",FACTS_JSON)
```
```
import sqlite3, json
from pathlib import Path
from memory.paths import PROFILE_DB
#BASE = Path(__file__).parent

class ProfileStore:
    def __init__(self, db_path=PROFILE_DB):
        self.con = sqlite3.connect(db_path)   # 绝对路径
        self.con.execute("CREATE TABLE IF NOT EXISTS profile (k TEXT PRIMARY KEY, v TEXT)")
        self.con.commit()

    def set_profile(self, k, v):
        self.con.execute("INSERT OR REPLACE INTO profile VALUES(?,?)",(k,json.dumps(v)))

    def get_profile(self,k):
        row=self.con.execute("SELECT v FROM profile WHERE k=?",(k,)).fetchone()
        return json.loads(row[0]) if row else None

    
```

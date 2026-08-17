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


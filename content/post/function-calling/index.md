---
description: ""
title: "Function Calling"
draft: true
date: "2026-08-10T02:58:11+08:00"
slug: "Function Calling"
categories:
 - 
tags:
 - 
image: ""
---

## 完整流程
① 你注册工具:告诉 LLM"我有哪些函数、每个函数要什么参数"(tools 列表) 
② LLM 决定:输出"我要调 get_weather,参数 city=北京"(不是直接回答)
③ 代码执行:你的代码解析出函数名和参数,真正调用 get_weather("北京")
④ 结果回填:把函数返回值作为 tool 消息喂回给 LLM,让它基于真实结果回答
   → 如果还需要别的信息,重复②③④,直到 LLM 直接给最终答案

最小系统示例

```
# import re,json
class FakeLLm:
    def create(self,**kwargs):
        message=kwargs["message"]
        last=message[-1]
        if last["role"]=="tool":
            return {"choices":[{"message":{"content":"答案是："+last["content"]}}]}
        return{"choices": [{"message": {"tool_calls": [
            {"id": "call_1", "function": {"name": "today", "arguments": "{}"}}
        ]}}]}
def today():
    return "今天是2026年8月10日"
TOOLS = {"today": today}
def agent(question):
    messages=[{"role":"user","content":question}]
    for i in range(2):
        resp=FakeLLm().create(message=messages)
        msg=resp["choices"][0]["message"]
        print(f"第{i+1}轮 模型说：{msg}")
    if "tool_calls" not in msg:
        return msg["content"]

    name=msg["tool_calls"][0]["function"]["name"]
    result =TOOLS[name]()
    print(f"     → 执行工具 {name},得到: {result}")
    messages.append({"role": "assistant", "content": None,"tool_calls": msg["tool_calls"]})
    messages.append({"role": "tool", "tool_call_id": "call_1", "content": result})
    return "达到最大轮数"
print(agent("今天几号"))

```
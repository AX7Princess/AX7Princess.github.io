---
description: ""
title: "Prompt"
draft: true
date: "2026-08-08T02:58:24+08:00"
slug: "Agent"
categories:
 - null
tags:
 - Prompt
image: ""
---

##  什么样的 prompt 算"过及格线"？

Prompt三个硬指标：

            指标                                        白话版                                  怎么检验 

角色任务清晰              模型知道"你是谁、要干什么              换个人看 prompt，也能说出它的用途
输入输出边界明确        输入什么、输出什么格式                  说死了 连续跑 3 次，格式稳定 
可复用可测试               换参数能复用，不是一次性               同一个函数换 3 组参数都能跑

***为什么 Agent 工程师日常做的是 prompt 工程而不是微调？ 因为微调贵、慢、且改不动"工具调用"这种动态行为。prompt 工程便宜、灵活、可迭代——先 prompt，不够再考虑微调（那是模型工程师的活）***

***Codex 能写代码，但架构设计、工具拆分、安全审查、调试评估、部署运维这些工程决策必须由人来做。AI 是加速器，不是替代者——我学代码是为了能判断它写得好不好、能在它之上搭产品。***

***Prompt 及格线 = 角色清晰 + 边界明确 + 可复用可测试，与技巧高级与否无关。它是工程（组织输入）而不是微调（改权重）。AI 工具能写代码，但架构、调试、安全、评估是工程师的活。而 MD skill 能'一读就用'，靠的是代码把 MD 内容组装成 messages 调 API——文字和代码缺一不可。***

***提示词不是 md 文件，而是所有给模型看的内容；工具是给模型接的外肢，但工具的描述本身也是提示词。代码分工具和编排两层，前者被模型调用，后者指挥模型。***

## CoT 是什么

就是：别让模型"张口就答"，让它先把推理步骤写出来，再给结论。就像做数学题老师要求你"写出解题过程"，不许直接填答案。

解决什么问题：多步推理、数学逻辑题——模型直接答容易"跳步出错"，逼它写过程就能大幅提对

解决什么问题：格式固定、样本少的任务——比如"把消息分类成 情绪/任务/闲聊"，规则说不清，但给 2~3 个例子它立刻就会

本质是一条指令（"请一步步思考"），放在System（当规则）或 User（当任务要求）都可以

##  Few-shot 是什么

在问题前面放几个「输入 → 输出」的例子，让模型照葫芦画瓢。

类比：新员工入职，先看老员工处理过的几单（输入→标准输出），再自己上手。给的是"范例"，不是"规则"。

解决什么问题：格式固定、样本少的任务——比如"把消息分类成 情绪/任务/闲聊"，规则说不清，但给 2~3 个例子它立刻就会

本质是一组「输入→输出」示例，既不在 system 也不在 user，而是自己组成多轮 user/assistant 消息对，示例必须是"一问一答"的格式，所以拆成 user(示例问题) + assistant(示例答案) 交替出现

CoT 和 Few-shot 不是固定的角色位置，而是 prompt 技巧：CoT 是一条'要求一步步推理'的指令，可放 system 当规则或 user 当任务；Few-shot 是穿插在历史里的 user/assistant 问答对，让模型模仿格式。它们必须用代码实现，是因为 API 只接收带 role 的 messages 结构——文字是 content，代码是组装，两者结合才能复用。

多轮对话练习代码
```
from openai import OpenAI
import os
client =OpenAI(api_key=os.getenv("DEEPSEEK_API_KEY")  ,base_url="https://api.deepseek.com")
def get_response(messages,**kwargs):
    response=client.chat.completions.create(model="deepseek-v4-flash",messages=messages,stream=True,reasoning_effort="low",extra_body={"thinking":{"type":"disabled"}},max_tokens=kwargs.get("max_tokens", 500),)
    reasoning_content=""
    content=""
    for chunk in response:
        delta=chunk.choices[0].delta
        if delta.reasoning_content:
            reasoning_content+=delta.reasoning_content
        if delta.content:
            content+=delta.content
    if content:
        messages.append({"role":"assistant","content":content})
    if not content:
        content = reasoning_content 
    return content
messages = [{"role": "system", "content": "你是一个耐心Agent老师，擅长用通俗的语言解释学生提到的问题。请根据问题一步步分析学生当前的学习阶段，给出最适合当前阶段的回答。"}]
while True:
    user_input = input("User: ")
    if user_input.lower() in ["exit", "quit"]:
        break
    messages.append({"role":"user","content":user_input})
    response = get_response(messages)
    print("Agent导师:", response)
if __name__ == "__main__": 
    pass
```
## Recat

ReAct = 边思考边行动（Thought → Action → Observation → Answer），是 Agent 的前身。
ReAct = Reason + Act（边思考边行动），模型不闷头答，而是"想一步 → 做一步 → 看结果 → 再想下一步"，像人一样边动脑子边动手 

```
Thought: 我先想清楚要做什么
Action: 调哪个工具
Action Input: 传什么参数
Observation: 工具返回了什么
Answer: 最终答案

---------------------------------------------------------------------

f"可用工具:{tools}。请严格按格式回答:\n"
         f"Thought: 我先想清楚要做什么\n"
         f"Action: 调哪个工具\n"
         f"Action Input: 参数\n"
         f"Observation: (工具返回,此处先占位)\n"
         f"Answer: 最终答案\n\n问题:{question}")
```
## ToT

ToT 的实现核心是三步循环：发散（生成多条路径）→ 评估（打分）→ 收敛（选最优）。工程上两种做法：单 prompt 版让模型一口气走完三步、成本低；真 ToT 分三次调用 API、每步可控可干预，代价是 3 倍调用。选型看任务——快速出方案用单 prompt，关键决策用多轮循环

```
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("DEEPSEEK_API_KEY"), base_url="https://api.deepseek.com")

def get_response(messages, **kwargs):
    response = client.chat.completions.create(
        model="deepseek-v4-flash",
        messages=messages,
        stream=True,
        reasoning_effort="low",
        extra_body={"thinking": {"type": "enabled"}},
        max_tokens=kwargs.get("max_tokens", 500),
    )
    reasoning_content, content = "", ""
    for chunk in response:
        delta = chunk.choices[0].delta
        if delta.reasoning_content:
            reasoning_content += delta.reasoning_content
        if delta.content:
            content += delta.content
    if content:
        messages.append({"role": "assistant", "content": content})
    else:
        content = reasoning_content
    return content
def tot_prompt(system_prompt,question,n=3):
    u=(f"请针对以下问题，提供{n}个不同的解决方案，"
       f"然后逐步分析每个解决方案的优缺点，最后给出最优方案。"
       f"最后选出最佳方案并说明理由。\n\n问题：{question}")
    return [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": u}
    ]
while True:
    user_input = input("User: ")
    if user_input.lower() in ["exit", "quit"]:
        break
    msgs = tot_prompt("你是资深的Agent架构师，擅长多种角度对比方案", user_input)
    response = get_response(msgs, max_tokens=1000)
    print("Agent导师:", response)

```

























































---
description: ""
title: "Function Calling"
draft: true
date: "2026-08-10T02:58:11+08:00"
slug: "Function Calling"
categories:
 - null
tags:
 - null
image: ""
---

## 完整流程
1. 函数注册：告诉 LLM"我有哪些函数、每个函数要什么参数"(tools 列表) 
2. LLM 决定:输出"我要调 get_weather,参数 city=北京"(不是直接回答)
3. 代码执行:代码解析出函数名和参数,真正调用 get_weather("北京")
4.  结果回填:把函数返回值作为 tool 消息喂回给 LLM,让它基于真实结果回答
   → 如果还需要别的信息,重复②③④,直到 LLM 直接给最终答案

### 最小系统模拟示例
 FC 循环的"骨架清单
① 开头: messages = [{"role":"user", ...}]          ← 只有一条用户消息
② 循环: for _ in range(MAX_ROUNDS):                ← 最多转几圈
③ 判断: if not msg.tool_calls: 返回                ← 模型没要工具 → 结束
④ 有工具: append assistant(带tool_calls) + 执行 + append tool
⑤ 结果: 模型看到 tool 结果,再回答 → 回到③

```
import json
from openai import OpenAI
import os
def get_weather(city:str)->str: 
    #天气实际查询函数
    return f"{city}"+"今天台风，注意安全"

# 注册工具
tools=[ 
    {
        "type":"function","function":{
            "name":"get_weather","description":"获取指定城市天气","parameters":{
                "type":"object",
                "properties":{
                    "city":{
                        "type":"string"
                    }
                },
                 "required":["city"]
                    }
            }
        }  
]
messages=[{"role":"user","content":"今天北京天气怎么样"}]
client=OpenAI(api_key=os.getenv("DEEPSEEK_API_KEY"),base_url="https://api.deepseek.com")
res=client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=messages,
    tools=tools,
    max_tokens=100,
)
msg=res.choices[0].message
try:
    print("msg:type",type(msg),"\n\n",msg)
except:
    print("无法打印")
if msg.tool_calls:
    messages.append(msg)
    for tc in msg.tool_calls:
        result=get_weather(**json.loads(tc.function.arguments))
        try:
            print("result:"+result+"\n")
        except Exception as e:
            print("result无法打印",e)
        messages.append({"role":"tool","tool_call_id":tc.id,"content":result})
    final=client.chat.completions.create(model="deepseek-v4-flash", messages=messages, tools=tools)
    print(final.choices[0].message.content)

```
### 改为真实API接口
```
import json
from openai import OpenAI
import os
import urllib.request
def get_weather(city:str)->str: 
    city_encoded = urllib.parse.quote(city)          # 深圳 → %E6%B7%B1%E5%9C%B3
    url = f"https://wttr.in/{city_encoded}?format=3"
  # 返回简短的天气文本
    return urllib.request.urlopen(url).read().decode()

# 注册工具
TOOLS_SEARCH=[
     {
            "type":"function","function":{
                "name":"get_weather","description":"获取指定城市天气","parameters":{
                    "type":"object",
                    "properties":{
                        "city":{
                            "type":"string"
                        }
                    },
                     "required":["city"]
                        }
                }
            }  
]

def fc_loop(user_input,max_runs=3):
    messages=[{"role":"user","content":user_input}]
    client=OpenAI(api_key=os.getenv("DEEPSEEK_API_KEY"),base_url="https://api.deepseek.com")
    for _ in range(max_runs):
        res=client.chat.completions.create(
        model="deepseek-v4-flash",
        messages=messages,
        tools= TOOLS_SEARCH,
        max_tokens=100,
        )
        msg=res.choices[0].message
        try:
            print("msg:type",type(msg),"\n\n",msg)
        except:
            print("无法打印")
        if not msg.tool_calls:
            return msg.content
        messages.append(msg)
        for tc in msg.tool_calls:
            #result=get_weather(**json.loads(tc.function.arguments))
            try:
                arg=json.loads(tc.function.arguments)
            except Exception as e:
                messages.append({"role": "tool", "tool_call_id": tc.id, "content": f"参数解析失败:{e}"})
                continue
            result=run_status(tc.function.name,arg)
            messages.append({"role":"tool","tool_call_id":tc.id,"content":result})
            try:
                print("arg:", arg, "result:", result)
            except Exception as e:
                print("result无法打印",e)
    return "达到最大轮数,已停止"
TOOLS_FUNC = {"get_weather": get_weather}
def run_status(name:str,args:dict)->str:
    #执行工具;出错不崩,把错误信息返回给模型,让它自己修正
    try:
        if name not in TOOLS_FUNC:
            return f"未知工具:{name}"
        return TOOLS_FUNC[name](**args)
    except Exception as e:
        return f"工具执行失败:{e}"


if __name__=="__main__":
    print(fc_loop(input("天气助手:")))




```
仿照Open接口，做假模型测试，有点不消耗token

```
from openai import OpenAI
from dotenv import load_dotenv, find_dotenv
import json,os,ast
import urllib.request
load_dotenv(override=True, dotenv_path=find_dotenv())

class LLMClient():
    def chat(self,message,tools=None):
        raise NotImplemented("子类必须实现 chat()")

class RealLLM(LLMClient):
    def __init__(self,provide="deepseek"):
        cfg={
            "deepseek":{"base_url":"https://api.deepseek.com","key":"DEEPSEEK_API_KEY","model":"deepseek-v4-flash"},
            "openai": {"base_url": None, "key": "OPENAI_API_KEY", "model": "gpt-4o-mini"},
        }[provide]
        self.client = OpenAI(api_key=os.getenv(cfg["key"]), base_url=cfg["base_url"])
        print(f"实际使用的 provider={provide}, key 尾号={os.getenv(cfg['key'])[-6:] if os.getenv(cfg['key']) else 'None'}")

        self.model=cfg["model"]
    def chat(self,messages,tools=None):
        return self.client.chat.completions.create(model=self.model,messages=messages,tools=tools)

class FakeMessage:
    def __init__(self,content=None,tool_calls=None):
        self.content,self.tool_calls=content,tool_calls

class FakeChoice:
    def __init__(self,message):
        self.message=message

class FakeResponse:
    def __init__(self, message): self.choices = [FakeChoice(message)]

class FakeFunction:
    def __init__(self, name, arguments):
        self.name, self.arguments = name, arguments


class FakeToolCall:
    def __init__(self, name, arguments):
        self.id = "call_mock_001"
        self.function = FakeFunction(name, arguments)

class MockLLM(LLMClient):
    def chat(self,message,tools=None):
        if message[-1]["role"]=="tool":
            result=message[-1]["content"]
            return FakeResponse(FakeMessage(content=f"模型看到工具结果[{result}]后给出的最终回答"))
        q=message[-1]["content"]
        if "天气" in q:
            return FakeResponse(FakeMessage(tool_calls=[FakeToolCall("get_weather",'{"city":"天津"}')]))
        if "等于" in q or "计算" in q:
            return FakeResponse(FakeMessage(tool_calls=[FakeToolCall("calculator", '{"expr": "1+1"}')]))
        return FakeResponse(FakeMessage(content="Mock回答"))

def run_tool(name,arg):
    try:
        return TOOLS_FUNC[name](**arg)
    except Exception as e:
        return(f"参数出错了:{e}")

def fc_loop(question,llm,max_rounds=3):
   messages=[{"role":"user","content":question}]
   for _ in range(max_rounds):
        rep=llm.chat(messages,tools=TOOLS_SCHEMA)
        msg=rep.choices[0].message
        if not msg.tool_calls:
            return msg.content
        messages.append(msg)
        for tc in msg.tool_calls:
            try:
                arg=json.loads(tc.function.arguments)
            except Exception as e:
                messages.append({"role": "tool", "tool_call_id": tc.id, "content": f"参数解析失败:{e}"})
                continue
            result=run_tool(tc.function.name,arg)
        messages.append({"role":"tool","tool_call_id":tc.id,"content":result})
   return "达到最大访问次数"


   


def get_weather(city:str)->str:#查询天气
    city_encoded = urllib.parse.quote(city)          # 深圳 → %E6%B7%B1%E5%9C%B3
    url = f"https://wttr.in/{city_encoded}?format=3"
      # 返回简短的天气文本
    return urllib.request.urlopen(url).read().decode()

def calculator(expr: str) -> str:#四则运算
    return str(ast.literal_eval(expr))

TOOLS_FUNC = {"get_weather": get_weather, "calculator": calculator}   
TOOLS_SCHEMA = [
    {"type": "function", "function": {"name": "get_weather", "description": "查天气",
     "parameters": {"type": "object", "properties": {"city": {"type": "string"}}, "required": ["city"]}}},
    {"type": "function", "function": {"name": "calculator", "description": "四则运算",
     "parameters": {"type": "object", "properties": {"expr": {"type": "string"}}, "required": ["expr"]}}},
]

if __name__=="__main__":
    load_dotenv(override=True, dotenv_path=find_dotenv())
    llm = RealLLM("deepseek")
    print(fc_loop("天津天气怎么样", llm))

```



---
description: ""
title: "Function Calling"
draft: false
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
4. 结果回填:把函数返回值作为 tool 消息喂回给 LLM,让它基于真实结果回答
   → 如果还需要别的信息,重复2，3，4,直到 LLM 直接给最终答案

## 最小系统模拟示例
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
仿照Open接口，做假模型测试，优点不消耗token

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
##接入真实天气api，接入模型路由

```
from openai import OpenAI
from dotenv import load_dotenv, find_dotenv
import json,os,ast
import urllib.request
from pathlib import Path
import openmeteo_requests
load_dotenv(override=True, dotenv_path=find_dotenv())

class LLMClient():
    def chat(self,message,tools=None):
        raise NotImplemented("子类必须实现 chat()")

class RealLLM(LLMClient):
    def __init__(self,provide="deepseek"):
        cfg={
            "deepseek":{"base_url":"https://api.deepseek.com","key":"DEEPSEEK_API_KEY","model":"deepseek-v4-flash"},
            "kimi": {"base_url": "https://api.moonshot.cn/v1", "key": "MOONSHOT_API_KEY", "model": "kimi-k2.6"},
        }[provide]
        self.client = OpenAI(api_key=os.getenv(cfg["key"]), base_url=cfg["base_url"])
        print(f"实际使用的 provider={provide}, key 已加载={'是' if os.getenv(cfg['key']) else '否'}")
       # print(repr(os.getenv(cfg["key"])))
        self.model=cfg["model"]
        print(provide)
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

def route_model(question): #模型硬路由
    """从用户输入里找模型名,找不到用默认"""
    for name in ["kimi", "deepseek", "qwen", "gpt"]:
        if name in question.lower():
            return name
    return "deepseek"   # 默认


def run_tool(name,arg):
    try:
        return TOOLS_FUNC[name](**arg)
    except Exception as e:
        return(f"参数出错了:{e}")

def fc_loop(question,llm,max_rounds=3):
   messages=[{"role":"user","content":question}]
   tool_records = []   
   for _ in range(max_rounds):
        rep=llm.chat(messages,tools=TOOLS_SCHEMA)
        msg=rep.choices[0].message
        if not msg.tool_calls:
            return {
                "answer": msg.content,     # 最终答案
                "tool_records": tool_records,   # [] = 没调工具(模型猜的!)
            }
            #return msg.content
        messages.append(msg)
        for tc in msg.tool_calls:
            try:
                arg=json.loads(tc.function.arguments)
            except Exception as e:
                messages.append({"role": "tool", "tool_call_id": tc.id, "content": f"参数解析失败:{e}"})
                continue
            result=run_tool(tc.function.name,arg)
            print(arg)
            tool_records.append({"tool": tc.function.name, "args": arg, "result": result})
            messages.append({"role":"tool","tool_call_id":tc.id,"content":result})
          #  print(msg)
   return {"answer": "达到最大轮数", "tool_records": tool_records}

def get_weather(latitude:float,longitude:float)->str:#查询天气
    openmeteo = openmeteo_requests.Client()
    url = (f"https://api.open-meteo.com/v1/forecast?latitude={latitude}"
           f"&longitude={longitude}"
           f"&current=temperature_2m,relative_humidity_2m,wind_speed_10m,wind_direction_10m"
           f"&daily=weather_code,temperature_2m_max,temperature_2m_min,sunrise,sunset,precipitation_probability_max"
           f"&timezone=Asia%2FShanghai")
    data = json.loads(urllib.request.urlopen(url).read())
    WEATHER_CODE_DESC = {
    0: "晴朗的天空",
    1: "主要晴朗",
    2: "局部多云",
    3: "阴天",
    45: "雾气",
    48: "霜雾沉积",
    51: "毛毛雨：轻度",
    53: "毛毛雨：中度",
    55: "毛毛雨：密集",
    56: "冻毛毛雨：轻微",
    57: "冻毛毛雨：强度高",
    61: "降雨：轻度",
    63: "降雨：中度",
    65: "降雨：强雨",
    66: "冻雨：轻度",
    67: "冻雨：强烈",
    71: "降雪量：轻度",
    73: "降雪量：中度",
    75: "降雪量：重度",
    77: "雪粒",
    80: "阵雨：轻度",
    81: "阵雨：中度",
    82: "阵雨：猛烈",
    85: "雪阵阵：轻微",
    86: "雪阵阵：猛烈",
    95: "雷暴：轻度或中度",
    96: "雷暴伴轻微冰雹",
    99: "雷暴伴强烈冰雹",
    }
    cur = data["current"]
    today = {k: v[0] for k, v in data["daily"].items()}
    return json.dumps({
        "天气": WEATHER_CODE_DESC.get(today["weather_code"], f"未知天气码{today['weather_code']}"),
        "当前温度_C": cur["temperature_2m"],
        "湿度_%": cur["relative_humidity_2m"],
        "今日最高_C": today["temperature_2m_max"],
        "今日最低_C": today["temperature_2m_min"],
        "日出": today["sunrise"][11:16],
        "日落": today["sunset"][11:16],
        "降雨概率_%": today["precipitation_probability_max"],
    }, ensure_ascii=False)
  #  print(result)
    return result
    #city_encoded = urllib.parse.quote(city)          # 深圳 → %E6%B7%B1%E5%9C%B3
    #url = f"https://wttr.in/{city_encoded}?format=3"
      # 返回简短的天气文本
   # print(url)
   #return urllib.request.urlopen(url).read().decode()


def calculator(expr: str) -> str:#四则运算“
    return str(ast.literal_eval(expr))

TOOLS_FUNC = {"get_weather": get_weather, "calculator": calculator}   
TOOLS_SCHEMA = [
    {"type": "function", "function": {"name": "get_weather", "description": "查天气",
     "parameters": {"type": "object", "properties": {"latitude": {"type": "number"}, "longitude": {"type": "number"}}, "required": ["latitude", "longitude"]}}},
    {"type": "function", "function": {"name": "calculator", "description": "四则运算",
     "parameters": {"type": "object", "properties": {"expr": {"type": "string"}}, "required": ["expr"]}}},
]

if __name__=="__main__":
    user_input = input("User: ")
    llm = RealLLM(route_model(user_input))
  # 真实 DeepSeek 下问一个同时触发两个工具的问题
    print(fc_loop(user_input, llm))
   # get_weather(39.08,117.20)
```
##整合后prompt文件
```
from openai import OpenAI
from llm_client import fc_loop,route_model,RealLLM
import os

client = OpenAI(api_key=os.getenv("DEEPSEEK_API_KEY"), base_url="https://api.deepseek.com")

def get_response(messages,**kwargs):
    response = client.chat.completions.create(
        model=kwargs.get("model","deepseek-v4-flash"),
        messages=messages,
        stream=True,
        reasoning_effort=kwargs.get("reasoning_effort","low"),
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

def ask(quextion,system_prompt="你是擅长多种角度对比方案的资深决策专家",max_tokens=500):
    msgs=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": quextion}
    ]
    response = get_response(msgs, max_tokens=max_tokens)
    return response

def tot_solve(question,n=3):
    schemes=ask(
        f"请针对以下问题给出{n}种不同解决思路，编号列出：\n{question}\n",
        system_prompt="你是擅长发散思维的规划专家",
        max_tokens=600,
    )
    scores = ask(
        f"请对以下 {n} 个方案根据实行难度，风险评估逐一打分(1-10分),并各用一句话说明理由:\n{schemes}",
        system_prompt="你是严格的评审专家,打分要客观",
        max_tokens=500,
    )
    best = ask(
        f"综合以下评分,选出最优方案,并给出具体实施步骤:\n{scores}",
        system_prompt="你是决策专家,直接给最终选择",
        max_tokens=600,
    )
    return schemes, scores, best

def cot_prompt(question,system_prompt="你是一个逻辑缜密，思维严谨的推理助手"):
    u=f"请一步步思考，再给出答案。\n问题:{question}"
    return [{"role": "system", "content": system_prompt}, {"role": "user", "content": u}]

DEFAULT_EXAMPLES = [
    {"question": "我明天下午3点约了张医生复诊",
     "answer": "时间:明天下午3点;人物:张医生;事项:复诊"},
    {"question": "周五晚上和李总在望江楼吃饭",
     "answer": "时间:周五晚上;人物:李总;事项:吃饭"},
]

def few_shot_prompt(question,examples=DEFAULT_EXAMPLES,system_prompt="你是擅长信息抽取的助手") :
    msgs=[{"role":"system","content":system_prompt}]
    for ex in examples:
        msgs.append({"role":"user","content":ex["question"]})
        msgs.append({"role":"assistant","content":ex["answer"]})
    msgs.append({"role":"user","content":question})
    return msgs

def react_prompt(question,system_prompt="你是一位有多种能力的小助手",tools="天气查询、计算器"):
   # u=(f"可用工具:{tools}。严格按格式回答：\n"
    #   f"Thought:先根据用户问题想清楚要干什么\n"
   #    f"Action:调用哪个工具更适合该问题\n"
   #    f"Action Input:参数\n"
   #    f"Observation:工具返回结果\n"
   #    f"Answer:最终答案\n\n问题:{question}"     
   # )
    #return [{"role": "system", "content": system_prompt}, {"role": "user", "content": u}]1
    return fc_loop(question, RealLLM(route_model(question))) 

BUILDERS = {"cot": cot_prompt, "few_shot": few_shot_prompt, "react": react_prompt}


def auto_select(question):
    content=f"判断下面用户的问题适合哪种回答，输出格式：cot_prompt或few_shot_prompt或react_prompt或tot_solve)\n规则:注重结果中间思考过程的任务，逻辑推理，数学解题过程，公式推导输出cot_prompt,格式固定模板化输出的，文本提取特定词汇格式化输出的输出Few_shot_prompt,需要依赖其他工具的输出react_prompt,开放性问题，思维发散问题，思维风暴，多种选择多种路径实现，多选择对比找最优解决方案输出tot_solve\n问题:{question}"
    msgs = get_response([{"role": "user", "content": content}], max_tokens=50)
    print(f"[调试] 原始: {msgs!r}")
    text = msgs.lower()
    for mode in ["tot", "react", "few_shot", "cot"]:    # 顺序:先长的后短的?不,注意下面
        if mode in text:
            return mode
    print("[警告] 未识别到模式名,兜底 cot")
    return "cot"



def run_mode(mode,question,**kw):
    if mode =="tot":
        schemes, scores, best = tot_solve(question, n=kw.get("n", 3))
        return f"【方案】\n{schemes}\n\n【评审】\n{scores}\n\n【决策】\n{best}"
    if mode =="react":
        out = react_prompt(question) 
        if out["tool_records"]:
        # 展示调用记录
            print("工具调用记录:", out["tool_records"])
        else:
         # ⚠️ 模型没调工具,可能是在猜(幻觉风险)
            print("警告:模型未调用任何工具,回答可能为模型推断")
        return out["answer"]
         #return fc_loop(question, RealLLM("deepseek")) 
    if mode not in BUILDERS:
          raise ValueError(f"未知模式:{mode},可选 {list(BUILDERS) + ['tot']}")
    msgs = BUILDERS[mode](question, **kw)
    return get_response(msgs, **kw)         
while True:
    user_input = input("User: ")
    if user_input.lower() in ["exit", "quit"]:
        break
    mode = auto_select(user_input) 
    print(f"[使用模式] {mode}")
    result = run_mode(mode, user_input)   
    print("Agent导师:", result)     
```



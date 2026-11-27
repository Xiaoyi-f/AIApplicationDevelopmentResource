---
title: "开源模型部署实现Token自由"
status: 已发布
author: XiaoYi
---

# 开源模型部署实现Token自由

> 作为一个AI应用开发者，你说 "TOKEN" 不够用？咱自己部署模型用起来！

我们通过ollama来下载和管理开源模型，下载地址搜索ollama官网即可

### 常用命令如下

ollama pull 模型

ollama run 模型

ollama list

ollama stop 模型

ollama rm 模型

ollama show 模型

ollama ps

### 首先让Agent自动给你配置好以下适配目标模型的一些环境变量再下载使用

1.模型存放位置

2.服务监听地址

3.模型常驻显存

4.并发请求数

5.同时加载模型数量

### 接入workbuddy等Agent -- 适配Agent能力的模型（推荐至少qwen3:8b起步模型）

其他相关配置务必填写适配目标模型本身能力的配置

接口通用地址: http://localhost:11434/v1

APIKey: ollama

### PythonSDK

```
# pip install ollama

client = ollama.Client(host="http://localhost:11434")

# 单轮生成
res = client.generate(model=model, prompt="prompt")
print(res["response"])

# 多轮对话 role -> user / system / assistant
res = client.chat(model=model, messages=[{"role": "user", "content": "prompt"}])
print(res["message"]["content"])

reply = res["message"]["content"]

messages = []
messages.append({"role": "assistant", "content": reply})
# 把AI回答的内容自动追加进入历史
```

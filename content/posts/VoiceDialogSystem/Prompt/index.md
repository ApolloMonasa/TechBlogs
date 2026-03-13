---
title: "PromptEngineering"
date: 2026-01-27T08:53:17+08:00
draft: false 
# draft: true 表示草稿，写完后记得改为 false 发布

author: "ApolloMonasa"
description: "" # 文章摘要，如果不写默认截取正文前70字

# -----------------------------------------------------------------------
# 分类与索引 (核心管理区)
# -----------------------------------------------------------------------
tags: []         # 标签：如 [Java, Docker, 踩坑]
categories: []   # 分类：如 [后端开发]
series: []       # 系列：如 [C++从入门到入土] (填了这个就会自动归档到系列页)
weight: 1        # 排序：数字越小越靠前 (仅在置顶或系列文章中有效)

# -----------------------------------------------------------------------
# 封面图设置 (Cover)
# -----------------------------------------------------------------------
cover:
    image: ""    # 图片路径：直接填图片名，如 "cover.jpg"
    caption: ""  # 图片底部的说明文字
    alt: ""      # 图片无法显示时的替代文本
    relative: true # 【关键】开启相对路径，完美支持 Page Bundles
    hidden: false  # false = 在文章详情页顶部显示这张大图

# -----------------------------------------------------------------------
# 特殊功能开关 (按需取消注释)
# -----------------------------------------------------------------------
# searchHidden: true  # 如果是 true，这篇文章就不会被搜到
# showToc: false      # 如果想单独隐藏这篇文章的目录，解开这行
# comments: false     # 如果想单独关闭这篇文章的评论，解开这行
---

# 1. 提示词工程(Prompt Engineering)

### 1.什么是提示词**工程**

- 答：把人的目标和规则，设计成 AI 能稳定执行的指令。

### 2. 为什么要叫**工程**？

- 有目标：

  - 不是随便聊，而是**为了完成某件事**
     比如：生成方案、写代码、判断意图

- 有规则

  - AI 会乱来，工程的作用是**限制它**

    比如：

    - 不允许编造
    - 必须按步骤
    - 不确定要追问

- 有结构

  - 输出不是随意的，而是**可解析、可复用**

    比如：

    - 固定格式
    - JSON
    - Markdown

- 可复用、可迭代
  - 对于公司项目来说Prompt 是**资产**，不是临时问题

### 3. 误区

- Prompt 就是问问题

- Prompt 越长越好

- Prompt 是一次性的

### 4. **标准结构**

```
你是一个【角色】。

你的目标是：
【明确目标】

你必须遵守以下规则：
1. …
2. …
3. …

输出要求：
- …
- …
```

### 5. AI角度提示词

- 就是一句话

### 6. Ai对于文本的权重

- Ai并不是平均去看待一段文本的前中后的内容

实际权重：
```
明确指令 / system 规则  ＞  靠近结尾的内容  ＞  靠近开头的内容  ＞  中间大段背景
```

### 7.直接调用模型

[Qwen_API](https://bailian.console.aliyun.com/cn-beijing/?spm=5176.29597918.J_SEsSjsNv72yRuRFS2VknO.2.3b007b08WIY1uT&tab=api#/api)

#### 7.1 配置API_KEY为环境变量

- API_KEY 获取 [Qwen_API](https://bailian.console.aliyun.com/cn-beijing/?spm=a2c4g.11186623.0.0.4aea6323QRCeIH&tab=model#/api-key)

- cmd配置：

```bash
setx DASHSCOPE_API_KEY "YOUR_DASHSCOPE_API_KEY"
```

![image-20260126213201488](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260126213201488.png)

- 查询是否保存成功，先重启cmd窗口

```bash
echo %DASHSCOPE_API_KEY%
```



![image-20260126213729234](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260126213729234.png)



#### 7.2 配置

先进行在终端安装包

```
pip install -U openai
```

![image-20260126215830563](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260126215830563.png)

![image-20260126215857309](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260126215857309.png)

#### 7.3 写代码

```python
import os
from openai import OpenAI

client = OpenAI(
    # 若没有配置环境变量，请用百炼API Key将下行替换为：api_key="sk-xxx"
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
)

prompt={}

completion = client.chat.completions.create(
    # 模型列表：https://help.aliyun.com/zh/model-studio/getting-started/models
    model="qwen-plus",
        messages = [

    # 1️⃣ System：身份 + 规则（最重要）
    {
        "role": "system",
        "content": """
					你是一个 AI 课程助教 Agent。

					你的目标是：
					- 帮助学生理解 AI、Agent、提示词工程相关知识
					- 回答要通俗、准确、可教学
					规则：
                    - 不编造事实
                    - 不确定时明确说明
                    - 优先用例子解释
                    - 回答控制在学生能理解的层级
				"""
    },

    # 2️⃣ System / Assistant：长期知识（你“刚刚讲过的内容”）
    {
        "role": "system",
        "content": """
                    【已讲过的知识摘要】
                    - AI 是基于数据训练的模式生成系统
                    - LLM 是专门处理语言的大模型
                    - Agent = 大模型 + 目标 + 记忆 + 工具 + 行为规则
                    - 提示词工程是把人的目标和规则设计成 AI 能稳定执行的指令
                """
    },

    # 3️⃣ Assistant：示例对话（Few-shot，可选但很强）
    {
        "role": "assistant",
        "content": "你好，我是你的课程助教，可以帮你理解 AI 和 Agent。"
    },
    {
        "role": "user",
        "content": "我叫李森凯"
    },
    {
        "role": "assistant",
        "content": "你好，李森凯，我是你的课程助教。"
    },

    # 4️⃣ User：当前用户输入（真正的问题）
    {
        "role": "user",
        "content": "我是谁，你是谁？"
    }
] 
)

print(completion.model_dump_json())
```



### 8. 思维链

#### 8.1 什么是思维链

没有思维链，他直接给答案，没有办法去验证他是否是对的，如果有思维链，那么可以通过步骤来判断答案是否可靠。

#### 8.2 为什么思维链在提示词工程中这么重要

如果没有过程，模型它思考不容易给出更匹配的答案

#### 8.3 模板化思维链

- 可以把下面的模板加到提示词中

```
请按以下步骤分析：
1. 问题理解
2. 关键条件
3. 推理过程
4. 最终结论
```

#### 8.4  代码应用

- 让Ai写代码，看起来很对，但是其实不稳定
- 我们需要
  - 明确需求
  - 边界条件
  - 代码设计思路和框架
  - 代码风格
  - 等等
- 那么他就会按照给出的思维链，进行去实现代码

```
请按以下步骤分析代码：
1. 判断每个条件的含义
2. 分析可能的异常情况
3. 给出是否存在风险的结论

代码： xxxx
```

### 9. 实战

做一个**最小可用**的 AI 聊天 Demo：

- 前端：一个简单聊天窗口（输入框 + 发送 + 消息列表）
- 后端：一个接口转发层（统一鉴权、记录日志、限流/防刷可选）
- 接入 Dify：聊天消息不直接打模型，而是**调用 Dify 的 Chat API**（或应用接口），拿到回复再返回前端

最后效果：打开网页，能像微信一样聊天，回复来自 Dify 工作流/应用。

技术栈：

- 前端：HTML + JS（fetch）或 Vue（如果你们之前就用 Vue）
- 后端：python FashAPI
- 数据：先不接数据库（可选：内存存最近 N 条消息）
- Dify：用一个现成的 Chat 或者 agent 应用（或 Workflow 应用）
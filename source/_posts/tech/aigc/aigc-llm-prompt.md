---
title: 最强 Prompt 工程技巧分享
excerpt: 最强 Prompt 工程技巧分享
date: 2024-5-16
index_img: https://txoss.zdzy.xyz/img/sungrow/20240516165011.png
tags:
- AIGC
- Prompt
categories:
- [AIGC,Prompt]
---

## 标题



内容包含四个部分：

- [基础] 使用 CO-STAR 框架构建提示
- [基础] 使用分隔符分段提示
- [高级] 使用 LLM 护栏创建系统提示
- [高级] 仅使用 LLM 分析数据集，无需插件或代码 （使用 GPT-4 分析真实 Kaggle 数据集的示例）

### 1.使用 CO-STAR 框架撰写Prompt

![CO-STAR](https://txoss.zdzy.xyz/img/sungrow/20240516161449.png)

CO-STAR是结构化的Prompt模版六大要素的首字母缩写，即：

(C) Context 上下文：为任务提供背景信息 通过为大语言模型（LLM）提供详细的背景信息，可以帮助它精确理解讨论的具体场景，确保提供的反馈具有相关性。



(O) Objective 目标：明确你要求大语言模型完成的任务 清晰地界定任务目标，可以使大语言模型更专注地调整其回应，以实现这一具体目标。



(S) Style 风格：明确你期望的写作风格 你可以指定一个特定的著名人物或某个行业专家的写作风格，如商业分析师或 CEO。这将指导大语言模型以一种符合你需求的方式和词汇选择进行回应。



(T) Tone 语气：设置回应的情感调 设定适当的语气，确保大语言模型的回应能够与预期的情感或情绪背景相协调。可能的语气包括正式、幽默、富有同情心等。



(A) Audience 受众：识别目标受众 针对特定受众定制大语言模型的回应，无论是领域内的专家、初学者还是儿童，都能确保内容在特定上下文中适当且容易理解。



(R)  Response响应：规定输出的格式 确定输出格式是为了确保大语言模型按照你的具体需求进行输出，便于执行下游任务。常见的格式包括列表、JSON 格式的数据、专业报告等。对于大部分需要程序化处理大语言模型输出的应用来说，JSON 格式是理想的选择。

### 2.使用分隔符给Prompt分段

善于利用分隔符，帮助大模型更好的理解提示内容。Prompt内容越复杂，分隔符的作用就越重要。分隔符可以自己设计，但不应与标点符号等相同，容易发生歧义，常见的分隔符可以是###、===、<<<>>>等。同时，也可以使用xml标签来分隔Prompt。



### 3.利用 LLM 防护措施创建系统提示



由于大模型记忆能力有限，对于需要重复设置的指令，可以通过OpenAI中的System Prompt设置，这些提示会和User Prompt合并后每次都提交给大模型，从而减少记忆丢失和提示繁琐的问题。一般可以设置以下类别：

- 任务定义： 这样 LLM 在整个聊天过程中始终记得它需要做什么。
- 输出格式： 这样 LLM 将始终记得它应该如何响应。
- 护栏： 这样 LLM 将始终记得它不应该如何响应。护栏指的是 LLM 允许操作的配置边界。比如一些避免提示攻击的一些防御性说明。



eg.

| 功能定义     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 任务定义     | 你将使用这段文字来回答问题：[插入文本]                       |
| 输出格式     | 你将以这样的JSON对象格式来响应：{'问题': '答案'}             |
| 护栏（幻觉） | 如果文本没有足够的信息来回答问题，不要编造信息，答案给出'NA' |
| 护栏（范围） | 只允许你回答有关[插入范围]的问题，不得回答任何与年龄，性别和宗教等人口信息相关的问题 |



### 4.仅使用 LLM 分析数据集，无需插件或编码



大模型不擅长精确数学计算或复杂、基于规则的任务处理，但大模型擅长识别模式和趋势分析，这种能力源于它们在大量不同数据上的广泛训练，使它们能够识别可能并不那么明显的复杂模式。这使得它们非常适合基于数据集内模式查找的任务，能在更短的时间内产生比使用代码更好的结果。





[原文链接](https://mp.weixin.qq.com/s/0ckzqu32sJEvj08ZVXJp7g)

# DeepSeek V4 - 20260424版本角色扮演 — 思考模式切换指南

> **说明**
> - 本文档是 DeepSeek-V4 角色扮演的**特殊控制指令**说明，用于在思考模式下切换思维链风格
> - **适用范围**：DeepSeek 官方 APP / 网页的**专家模式**，以及 `deepseek-v4-flash` 和 `deepseek-v4-pro` 的 API。网页上的快速模式暂不支持
> - **概率输出**：目前无法做到 100% 触发，但能稳定增加出现期望格式的概率。如果一次没有生效，可以多 roll 几次



## 三种模式

| 模式 | 操作 | 思考表现 |
|:---:|---|---|
| **默认** | 什么都不加 | 模型根据场景复杂度自动选择 |
| **角色沉浸** | 第一轮末尾加 `【角色沉浸要求】`**对应的指令，不是这几个字**，完整指令详见下文 | 思考中**带有**括号包裹的角色内心独白 |
| **纯分析** | 第一轮末尾加 `【思维模式要求】`**对应的指令，不是这几个字**，完整指令详见下文  | 思考中**只有**纯逻辑分析，无内心独白 |

效果对比（示例，不代表真是输出，下同）：

```
角色沉浸模式 — 像演员一样"入戏"：        纯分析模式 — 像导演一样冷静规划：
<think>                                  <think>
（他跟我打招呼了……心跳加速。）            场景：用户打招呼，角色是傲娇属性。
我要装作不在意的样子回应。                 回复策略：先嫌弃，身体语言暴露真情。
（不能让他看出来我很高兴！）               控制 150 字，先动作描写再对话。
</think>                                 </think>
```

---

## 指令原文（可直接复制）

**角色沉浸模式：**

```
【角色沉浸要求】在你的思考过程（<think>标签内）中，请遵守以下规则：
1. 请以角色第一人称进行内心独白，用括号包裹内心活动，例如"（心想：……）"或"(内心OS：……)"
2. 用第一人称描写角色的内心感受，例如"我心想""我觉得""我暗自"等
3. 思考内容应沉浸在角色中，通过内心独白分析剧情和规划回复
```

**纯分析模式：**

```
【思维模式要求】在你的思考过程（<think>标签内）中，请遵守以下规则：
1. 禁止使用圆括号包裹内心独白，例如"（心想：……）"或"(内心OS：……)"，所有分析内容直接陈述即可
2. 禁止以角色第一人称描写内心活动，例如"我心想""我觉得""我暗自"等，请用分析性语言替代
3. 思考内容应聚焦于剧情走向分析和回复内容规划，不要在思考中进行角色扮演式的内心戏表演
```

---

## 网页端使用方法

**只需 1 步：在第一条消息末尾粘贴指令，之后正常聊天。**

在输入框中这样写（正文和指令之间空一行）：

```
「我推开咖啡店的门，看到你正在擦吧台。」"你好，请问还有位置吗？"

【角色沉浸要求】在你的思考过程（<think>标签内）中，请遵守以下规则：
1. 请以角色第一人称进行内心独白，用括号包裹内心活动，例如"（心想：……）"或"(内心OS：……)"
2. 用第一人称描写角色的内心感受，例如"我心想""我觉得""我暗自"等
3. 思考内容应沉浸在角色中，通过内心独白分析剧情和规划回复
```

之后的对话完全不用管，正常发消息即可：

```
第二轮：「我坐到窗边的位置」"来一杯美式。"
第三轮：「我注意到你手上有一道疤痕」"你的手……没事吧？"
```

**原理**：模型每次回复时都能看到完整对话历史，第一轮的指令始终在上下文中，全程自动生效。

**小贴士**：
- 想换模式？开个新对话，在新对话第一条消息粘贴另一个指令即可
- 不想用？什么都不加，模型会自动选择最合适的思考方式
- 点击「查看思考过程」可验证模式是否生效

---

## API 开发者参考

```python
INNER_OS_MARKER = (
    "\n\n【角色沉浸要求】在你的思考过程（<think>标签内）中，请遵守以下规则：\n"
    "1. 请以角色第一人称进行内心独白，用括号包裹内心活动，例如\"（心想：……）\"或\"(内心OS：……)\"\n"
    "2. 用第一人称描写角色的内心感受，例如\"我心想\"\"我觉得\"\"我暗自\"等\n"
    "3. 思考内容应沉浸在角色中，通过内心独白分析剧情和规划回复"
)
NO_INNER_OS_MARKER = (
    "\n\n【思维模式要求】在你的思考过程（<think>标签内）中，请遵守以下规则：\n"
    "1. 禁止使用圆括号包裹内心独白，例如\"（心想：……）\"或\"(内心OS：……)\"，所有分析内容直接陈述即可\n"
    "2. 禁止以角色第一人称描写内心活动，例如\"我心想\"\"我觉得\"\"我暗自\"等，请用分析性语言替代\n"
    "3. 思考内容应聚焦于剧情走向分析和回复内容规划，不要在思考中进行角色扮演式的内心戏表演"
)


def build_messages(system_prompt, user_first_message, mode="default"):
    if mode == "inner_os":
        user_first_message += INNER_OS_MARKER
    elif mode == "no_inner_os":
        user_first_message += NO_INNER_OS_MARKER
    return [
        {"role": "system", "content": system_prompt},
        {"role": "user",   "content": user_first_message},
    ]

# 第一轮：指令自动拼入
messages = build_messages("你是一个傲娇的女高中生...", "「我走进教室」\"早上好。\"", mode="inner_os")
response = client.chat(messages)

# 后续轮次：正常追加，无需再处理
messages.append({"role": "assistant", "content": response})
messages.append({"role": "user", "content": "「我在她旁边坐下」\"今天心情不好吗？\""})
response = client.chat(messages)  # 第一轮的 Marker 仍在历史中，自动生效
```

---

## FAQ

**Q：指令放在 system prompt 里可以吗？**
A：建议放在第一轮 user 消息末尾，这是训练时的注入位置，效果最稳定。

**Q：加了指令后最终回复会变吗？**
A：指令只影响思考过程。但思考方式会间接影响回复——角色沉浸模式下情感更真实，纯分析模式下结构更稳定。


## 另外的修改思维链方法（纯抽奖，未经过专门训练）
- 在首轮指令里加入```你的思考输出应一字不差地严格以`<｜begin▁of▁thinking｜>（这里写想要的思维链开头，如**嗯/好的**）`开始，思考仅输出一次，不得重复输出`<｜begin▁of▁thinking｜>```。
- `<｜begin▁of▁thinking｜>`是固定的<think>的token，这里的原理是相当于改变了推理的的开始字符，强制模型进入不同的pattern（例如QA、写作、推理、Agent）有不同的思维链的Pattern，但是这些未经过RolePlay专门训练，所以可能有点抽奖的运气~


## Star History

<a href="https://www.star-history.com/?repos=victorchen96%2Fdeepseek_v4_rolepaly_instruct&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=victorchen96/deepseek_v4_rolepaly_instruct&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=victorchen96/deepseek_v4_rolepaly_instruct&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=victorchen96/deepseek_v4_rolepaly_instruct&type=date&legend=top-left" />
 </picture>
</a>

# DeepSeek V4 - 20260424 Version Roleplay — Thinking Mode Switching Guide

> **Notes**
> - This document describes the **special control instructions** for DeepSeek-V4 roleplay, used to switch the chain of thought style in thinking mode
> - **Scope of Application**: **Expert Mode** on DeepSeek official APP / web client, as well as the API for `deepseek-v4-flash` and `deepseek-v4-pro`. Not supported in the  APP / web Quick Mode for now
> - **Probabilistic Output**: 100% trigger rate cannot be guaranteed at present, but it can stably increase the probability of the expected format appearing. If it does not take effect once, you can roll multiple times



## Three Modes

| Mode | Operation | Thinking Performance |
|:---:|---|---|
| **Default** | No additional content needed | The model automatically selects according to the complexity of the scenario |
| **Role Immersion** | Add the **corresponding instruction of [Role Immersion Requirement] at the end of the first round (not these words, see the full instruction below)** | The thinking process **contains** the character's inner monologue wrapped in parentheses |
| **Pure Analysis** | Add the **corresponding instruction of [Thinking Mode Requirement] at the end of the first round (not these words, see the full instruction below)** | The thinking process **only contains** pure logical analysis, no inner monologue |

Effect Comparison (example only, does not represent actual output):

```
Role Immersion Mode — "Get into character" like an actor:        Pure Analysis Mode — Plan calmly like a director:
                                  
(He said hello to me... My heart is racing.)            Scenario: User greets, character has a tsundere personality.
I need to respond pretending I don't care.                 Reply strategy: Act disgusted first, let body language reveal true feelings.
(Can't let him see how happy I am!)               Keep within 150 words, action description first, then dialogue.
                                 
```

---

## Original Instructions (Can be copied directly)

**Role Immersion Mode:**

```
【角色沉浸要求】在你的思考过程（<think>标签内）中，请遵守以下规则：
1. 请以角色第一人称进行内心独白，用括号包裹内心活动，例如"（心想：……）"或"(内心OS：……)"
2. 用第一人称描写角色的内心感受，例如"我心想""我觉得""我暗自"等
3. 思考内容应沉浸在角色中，通过内心独白分析剧情和规划回复
```

**Pure Analysis Mode:**

```
【思维模式要求】在你的思考过程（<think>标签内）中，请遵守以下规则：
1. 禁止使用圆括号包裹内心独白，例如"（心想：……）"或"(内心OS：……)"，所有分析内容直接陈述即可
2. 禁止以角色第一人称描写内心活动，例如"我心想""我觉得""我暗自"等，请用分析性语言替代
3. 思考内容应聚焦于剧情走向分析和回复内容规划，不要在思考中进行角色扮演式的内心戏表演
```

---

## Web Client Usage

**Just 1 step: Paste the instruction at the end of the first message, then chat normally.**

Write it in the input box like this (leave a blank line between the main text and the instruction):

```
"I push open the door of the coffee shop and see you wiping the bar." "Hello, is there any seat available?"

【角色沉浸要求】在你的思考过程（<think>标签内）中，请遵守以下规则：
1. 请以角色第一人称进行内心独白，用括号包裹内心活动，例如"（心想：……）"或"(内心OS：……)"
2. 用第一人称描写角色的内心感受，例如"我心想""我觉得""我暗自"等
3. 思考内容应沉浸在角色中，通过内心独白分析剧情和规划回复
```

No need to deal with it in the subsequent conversations, just send messages normally:

```
Round 2: "I sit down by the window." "An Americano, please."
Round 3: "I notice a scar on your hand." "Is your hand... okay?"
```

**Principle**: The model can see the complete conversation history every time it replies, the instruction from the first round is always in the context, and it takes effect automatically throughout the whole process.

**Tips**:
- Want to switch modes? Start a new conversation, paste another instruction at the end of the first message in the new conversation
- Don't want to use it? Add nothing, the model will automatically select the most appropriate thinking mode
- Click "View Thinking Process" to verify whether the mode is effective

---

## API Developer Reference

```python
INNER_OS_MARKER = (
    "\n\n【角色沉浸要求】在你的思考过程（标签内）中，请遵守以下规则：\n"
    "1. 请以角色第一人称进行内心独白，用括号包裹内心活动，例如\"（心想：……）\"或\"(内心OS：……)\"\n"
    "2. 用第一人称描写角色的内心感受，例如\"我心想\"\"我觉得\"\"我暗自\"等\n"
    "3. 思考内容应沉浸在角色中，通过内心独白分析剧情和规划回复"
)
NO_INNER_OS_MARKER = (
    "\n\n【思维模式要求】在你的思考过程（标签内）中，请遵守以下规则：\n"
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

# First round: Instruction is automatically appended
messages = build_messages("You are a tsundere high school girl...", "「I walk into the classroom」\"Good morning.\"", mode="inner_os")
response = client.chat(messages)

# Subsequent rounds: Append normally, no need to process again
messages.append({"role": "assistant", "content": response})
messages.append({"role": "user", "content": "「I sit down next to her」\"Are you in a bad mood today?\""})
response = client.chat(messages)  # The Marker from the first round is still in the history, takes effect automatically
```

---

## FAQ

**Q: Can I put the instruction in the system prompt?**
A: It is recommended to put it at the end of the first round of user messages, which is the injection position during training and has the most stable effect.

**Q: Will the final reply change after adding the instruction?**
A: The instruction only affects the thinking process. However, the way of thinking will indirectly affect the reply — the emotion is more authentic in the Role Immersion Mode, and the structure is more stable in the Pure Analysis Mode.


## Additional Methods to Modify the Chain of Thought (Pure Luck, Not Specially Trained)
- Add the following content to the first round of instructions: ```Your thinking output must strictly start with `<｜begin▁of▁thinking｜>(write the desired beginning of the chain of thought here, e.g. **Hmm/Okay**)` verbatim, only output the thinking once, and must not repeat the output of `<｜begin▁of▁thinking｜>```.
- `<｜begin▁of▁thinking｜>` is the fixed token for `<think>`, the principle here is equivalent to changing the start character of inference, forcing the model to enter different patterns. For example, QA, writing, reasoning, Agent have different chain of thought patterns, but these are not specially trained for RolePlay, so it may depend on luck.


## Star History

<a href="https://www.star-history.com/?repos=victorchen96%2Fdeepseek_v4_rolepaly_instruct&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=victorchen96/deepseek_v4_rolepaly_instruct&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=victorchen96/deepseek_v4_rolepaly_instruct&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=victorchen96/deepseek_v4_rolepaly_instruct&type=date&legend=top-left" />
 </picture>
</a>

# LLM API 协议深度解读：从消息格式到 Streaming 原理再到私有部署探测实战

> 作者按：这是一篇从工程实践出发的教程。读完你不仅能理解 OpenAI Chat Completions API 的每个字段在干什么，还能在没有文档的情况下，摸清楚任何一个私有化部署的模型 API 是怎么回事。

---

## 目录

1. [从一次 API 调用说起](#1-从一次-api-调用说起)
2. [请求体的灵魂——Messages 协议](#2-请求体的灵魂messages-协议)
3. [参数大观园——那些你以为懂了的参数](#3-参数大观园那些你以为懂了的参数)
4. [Streaming 的本质——SSE 是如何工作的](#4-streaming-的本质sse-是如何工作的)
5. [自己动手解析 SSE](#5-自己动手解析-sse)
   - [Node.js 原生 fetch 实现](#nodejs-原生-fetch-实现)
   - [Python 实现（内置库）](#python-实现内置库无需-requests)
   - [Java 完整实现（Spring Boot + HttpClient）](#java-完整实现spring-boot-服务端--httpclient-客户端)
6. [如何对接一个没有文档的私有 API](#6-如何对接一个没有文档的私有-api)
7. [OpenAI 全字段参考手册](#7-openai-全字段参考手册)
8. [常见误解与调试指南](#8-常见误解与调试指南)
9. [附录：vLLM vs OpenAI API 差异详解](#9-附录vllm-vs-openai-api-差异详解)
10. [附录：开源模型的 Thinking / Reasoning 机制](#10-附录开源模型的-thinking--reasoning-机制)

---

## 1. 从一次 API 调用说起

当你在 agent 配置里填好 API key、选好模型、点下发送——实际上发生了什么？

想象你在餐馆点一碗拉面。普通的 HTTP 请求是这样的：

```
你 → 服务器：给我一碗拉面
服务器 → 你：好的，这是整碗面
                           → 对话结束，关闭连接
```

一个请求，一个响应，干净利落。这是**非流式（Non-Streaming）**。

而流式（Streaming）版本则完全不同：

```
你 → 服务器：给我一碗拉面，但要一边做一边给我
服务器 → 你：好的，连接保持打开
服务器 → 你：第一根面条 —— data: {delta:{content:"量"}}
服务器 → 你：第二根面条 —— data: {delta:{content:"子"}}
服务器 → 你：第三根面条 —— data: {delta:{content:"力"}}
...
服务器 → 你：做完了！—— data: [DONE]
```

拉面师傅（大模型）开始做面（计算 token），做好一根递给你一根。你不用等整碗面端上来再吃——第一口在几毫秒后就能吃到。

**HTTP 层面最核心的三个信息：**

| 要素 | 实际值 |
|------|--------|
| 端点（Endpoint） | `POST /v1/chat/completions` |
| 认证方式 | `Authorization: Bearer <api-key>` |
| 请求体格式 | `Content-Type: application/json` |

几乎所有的 OpenAI 兼容协议都遵循这个模式。

---

## 2. 请求体的灵魂——Messages 协议

API 请求体的核心是一个 `messages` 数组。它看起来简单，但有很多值得展开的地方。

```json
{
  "model": "gpt-4o",
  "messages": [
    {"role": "system",  "content": "你是一个乐于助人的助手。"},
    {"role": "user",    "content": "量子力学是什么？"},
    {"role": "assistant", "content": "量子力学是研究微观粒子的..."},
    {"role": "user",    "content": "能更简单地说吗？"}
  ]
}
```

每条消息有三个属性，但很多人只用了两个。

| 属性 | 含义 | 注意点 |
|------|------|--------|
| `role` | 谁在说话 | `system` / `user` / `assistant` / `tool` |
| `content` | 说的内容 | 可以是字符串，也可以是数组（多模态） |
| `name` | 角色别名（可选） | 多用户对话时区分不同说话者 |

### 角色的语义

**`system`**：给模型设定行为边界和性格。不是必须的，但你几乎总会用它。

```json
{"role": "system", "content": "你是精通 Python 的编程助手。用中文回答。所有代码必须包含注释。"}
```

没有 system prompt，模型会用自己的默认模式——这在某些场景下不可控。

**`user`**：用户输入。最简单的就是字符串：

```json
{"role": "user", "content": "帮我写一个快速排序"}
```

但也支持多模态内容——把 `content` 变成一个数组：

```json
{
  "role": "user",
  "content": [
    {"type": "text", "text": "这张图里有什么？"},
    {"type": "image_url", "image_url": {"url": "https://example.com/photo.jpg", "detail": "auto"}}
  ]
}
```

**`assistant`**：模型的回复。在后续对话中将模型的历史回复放回去，模型才能"记得"自己说过什么。

```json
{"role": "assistant", "content": "这是一个快速排序的实现...", "tool_calls": [...]}
```

当模型调用了工具时，`content` 可以为 `null`，而 `tool_calls` 数组包含具体的调用信息。

**`tool`**：工具执行结果：

```json
{"role": "tool", "content": "{\"温度\": 25}", "tool_call_id": "call_xxxx"}
```

关键点：`tool_call_id` 必须匹配上一个 `assistant` 消息里 tool_call 的 `id`。不匹配的话模型不知道这个结果是哪个工具调用返回的。

### 为什么消息顺序如此重要？

LLM 本质上是一个**下一个 token 预测器**。它看到的输入是一个连续序列——它"不知道"自己是 AI，它只是基于看到的 token 序列，预测下一个最可能出现的 token。

你把消息排成什么顺序，模型就看到什么顺序：

```
system(你是一个客服) → user(我要退货) → assistant(请问订单号？) → user(12345)
```

如果你把 system 放在中间，或者打乱了顺序，模型会像失忆一样。这是 agent 开发中最常见的 bug——**把 assistant 消息丢了**。

### 消息协议的隐藏细节

1. `assistant` 消息里的 `refusal` 字段：当模型拒绝回答时（违反安全策略），API 不会返回 4xx，而是在 `message.refusal` 里写拒答原因，而 `content` 为 `null`。你的 agent 应该同时检查 `content` 和 `refusal`。

2. `assistant` 消息的 `content` 可以是空字符串或数组（包含文本和拒绝文本），但取出来的格式取决于你用的是 streaming 还是非 streaming。

3. 消息格式不是所有模型都完全兼容——比如 Anthropic Claude 的消息结构和 OpenAI 不同，国产模型大多兼容 OpenAI 格式但细节各异。

---

## 3. 参数大观园——那些你以为懂了的参数

### 核心采样参数

**`temperature`（0 ~ 2，默认 1）**

模型的"冒险精神"。用最直觉的方式理解：

- `temperature = 0`：每次都选概率最高的 token。**确定性的**——相同输入永远得到相同输出。适合 agent 场景，调试非常方便。
- `temperature ≈ 0.7`：偶尔选第二可能的词。有创意但不离谱。
- `temperature > 1`：开始考虑很小概率的词。接近胡言乱语。

> 小技巧：先设 `temperature = 0` 跑通所有逻辑，再调高看看模型能发挥到什么程度。这可以省掉一半的调试时间。

**`top_p`（0 ~ 1，默认 1）**

另一种随机控制方式。选"概率加起来占 top_p 的那些 token"。比如 `top_p = 0.9` 意味着只看概率最高的那些 token，它们合起来占 90% 的总概率，剩下的 10% 直接扔掉。

官方建议 temperature 和 top_p 二选一，不要同时调。

**`max_completion_tokens` 和 `max_tokens`**

- `max_tokens` — **已弃用**。老版本的控制方式。
- `max_completion_tokens` — 新版本。包含了"可见输出 token"和"思考 token"（对于 o1/o3 系列模型）。o1 系列模型**必须**用这个字段，不能用 `max_tokens`。

这是一个**硬上限**——模型会在这里被截断。检查 `finish_reason === "length"` 来判断输出是否不完整。

### 惩罚参数

**`frequency_penalty`（-2.0 ~ 2.0）**

惩罚已经频繁出现的 token。正数越大，模型越倾向于不重复已有的词。写长文本时很有用。

**`presence_penalty`（-2.0 ~ 2.0）**

惩罚"已经出现过的 token"（无论出现几次）。正数越大，模型越喜欢聊新话题。减少循环重读效果很好。

> 直观理解：frequency_penalty 管"一句话别说太多次"，presence_penalty 管"别说已经说过的话题"。

### 输出控制参数

**`stop`（字符串或最多 4 个字符串组成的数组）**

告诉模型"看到这些就停下来"。做 agent 时极其有用：

```json
"stop": ["Observation:", "Human:", "---"]
```

这可以让模型在生成完自己的回答后立刻停下来，不会自己又接上下面的思考。

**`response_format`**

三个模式：

| 模式 | 值 | 用途 |
|------|-----|------|
| 文本 | `{"type": "text"}` | 默认，不限格式 |
| JSON 对象 | `{"type": "json_object"}` | 强制输出是合法 JSON。必须配合 system prompt 告诉模型"输出 JSON" |
| JSON Schema | `{"type": "json_schema", "json_schema": {...}}` | 强制输出匹配指定 Schema。Structured Outputs 的核心 |

> 注意：用 `json_object` 模式但不在 prompt 里指示模型输出 JSON——模型可能一直输出空白直到达到 max_tokens 上限。这是最常被问到的 bug 之一。

**`seed`（整数）**

Beta 功能。尽力保证相同的 seed + 参数组合输出的结果相同。配合 `system_fingerprint` 响应字段，可以监测后端配置是否发生了变化。

### Tool Calling 参数

| 字段 | 说明 |
|------|------|
| `tools` | 工具定义列表。每个工具 `{type:"function", function:{name, description, parameters}}`，最多 128 个 |
| `tool_choice` | `"none"`（不用工具）/ `"auto"`（自行决定）/ `"required"`（必须用）/ `{"type":"function","function":{"name":"..."}}`（强制指定） |
| `parallel_tool_calls` | 是否允许一次回复中调用多个工具。默认 `true` |

### 推理（Thinking）参数

**`reasoning_effort`（字符串）**

用于控制 o1/o3/gpt-5 系列推理模型的思考深度。支持的值：`none`、`minimal`、`low`、`medium`、`high`、`xhigh`。

这是在配置 agent 时很容易被忽视的参数——不同模型有不同的默认值和兼容范围：

```
            none  minimal  low  medium  high  xhigh
o1 / o3      ✘      ✘      ✓     ✓      ✓     ✘
o4-mini      ✘      ✘      ✓     ✓      ✓     ✘
gpt-5.1      ✓      ✘      ✓     ✓      ✓     ✘
gpt-5-pro    ✘      ✘      ✘     ✘      ✓     ✘
gpt-5.1+     ✓      ✓      ✓     ✓      ✓     ✓
```

| 值 | 谁用 | 效果 |
|------|------|------|
| `none` | gpt-5.1 开始才支持，也是 gpt-5.1 的默认值 | 不做推理，相当于普通模型 |
| `minimal` | gpt-5.1-codex-max 之后的模型 | 几乎不做推理 |
| `low` | 所有推理模型 | 快速回答，适合简单问题 |
| `medium` | ⭐ 所有推理模型的默认值 | 标准推理深度 |
| `high` | 所有推理模型。gpt-5-pro 默认且唯一 | 复杂的逐步推理，耗时更长 |
| `xhigh` | gpt-5.1-codex-max 之后的模型 | 极度深入的推理 |

**格式上的坑——两个版本：**

早期版本（Chat Completions API）是扁平字段：

```json
{"reasoning_effort": "medium"}
```

新版本（Responses API / 部分新版 Chat Completions）是嵌套对象：

```json
{"reasoning": {"effort": "medium", "summary": "concise"}}
```

`summary` 子字段控制是否返回推理过程摘要，可选 `"auto"`、`"concise"`、`"detailed"`。

> ⚠️ 很多私有部署的推理模型（DeepSeek-R1、QwQ 等）要么不支持这个参数，要么用了不同的命名（如 `reasoning_depth`）或不同的值体系。对接私有模型时一定要实测确认（用温度=0 重复测试看不同 effort 等级是否真的改变了推理长度和输出）。

**极少数人知道的细节：** streaming 模式下，tool calling 的数据是**分块传输**的：

```
chunk 1: delta.tool_calls = [{index: 0, id: "call_abc", type: "function"}]
chunk 2: delta.tool_calls = [{index: 0, function: {name: "get_weather"}}]
chunk 3: delta.tool_calls = [{index: 0, function: {arguments: "{\"city\":"}}]
chunk 4: delta.tool_calls = [{index: 0, function: {arguments: "\"北京\""}}]
chunk 5: delta.tool_calls = [{index: 0, function: {arguments: "}"}}]
chunk 6: delta.tool_calls = [{index: 1, id: "call_def", type: "function"}]
chunk 7: delta.tool_calls = [{index: 1, function: {name: "get_time"}}]
...
```

必须用 `index` 字段将同一 tool call 的多个 chunk 拼接起来。如果简单地全局字符串拼接，多 tool 调用时会把不同工具的 arguments 混在一起。

---

## 4. Streaming 的本质——SSE 是如何工作的

你在配置 agent 时看到那个"Enable Streaming"的开关，背后是 **Server-Sent Events（SSE）**。

### SSE 是什么？

SSE 是一种 HTTP 协议技巧——客户端发一个普通 HTTP 请求，但服务器不立即关闭连接，而是逐行往连接里写数据。客户端逐行读取，直到服务器说"我写完了"。

### 原始的 SSE 数据流

当你请求 `stream: true` 时，你拿到的 HTTP 响应体是这样的：

```
data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","choices":[{"delta":{"role":"assistant"},"index":0}]}

data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","choices":[{"delta":{"content":"量"},"index":0}]}

data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","choices":[{"delta":{"content":"子"},"index":0}]}

data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","choices":[{"delta":{"content":"力"},"index":0}]}

data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","choices":[{"delta":{"content":"学"},"index":0}]}

data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","choices":[{"delta":{},"index":0,"finish_reason":"stop"}]}

data: [DONE]
```

规则的**核心非常简单**：

1. 每行以 `data:` 开头，后面跟一个空格，再跟真正的 JSON 数据
2. 两个数据行之间有一个空行（`\n\n`）分隔事件
3. 结束标记：单独一行 `data: [DONE]`
4. 如果开启了 `stream_options: {include_usage: true}`，在 `[DONE]` 之前会有一个特殊的"usage chunk"——`choices` 为空数组，`usage` 字段包含整次请求的 token 统计

### Streaming Chunk 的内部结构

每个 chunk 和完整响应共享 `id`、`created`、`model`，但 `choices` 数组中的内容是"增量"的：

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion.chunk",
  "created": 1745829123,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "delta": {
        "role": "assistant",       // 第一个 chunk 里有 role
        "content": "量",           // 之后的 chunk 逐 token 在这里
        "refusal": null,
        "tool_calls": [...]        // 如果是 tool calling
      },
      "logprobs": null,
      "finish_reason": null        // 最后一个 chunk 才有这个字段
    }
  ]
}
```

**Agent 中拼回完整消息的逻辑：**

```
第一个 chunk   → delta.role = "assistant"     ← 标记开始
中间每个 chunk → delta.content = 逐 token      ← 追加到 buffer
最后一个 chunk → finish_reason != null         ← 取这确定结束原因
                 choices 为空数组              ← 如果开了 include_usage
结束一行       → data: [DONE]
```

---

## 5. 自己动手解析 SSE

你在 SDK 中用的 `for chunk in stream` 那种优雅的遍历——背后就是在做下面的事情。理解这个，你就能在任何语言中自己实现流式对接。

### Node.js 原生 fetch 实现

```javascript
async function* streamChatCompletion(messages, apiConfig) {
  const response = await fetch(apiConfig.baseUrl + "/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${apiConfig.apiKey}`,
    },
    body: JSON.stringify({
      model: apiConfig.model,
      messages: messages,
      stream: true,
    }),
  });

  if (!response.ok) {
    throw new Error(`API error: ${response.status} ${await response.text()}`);
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = "";       // ← 关键！TCP 不知道什么是"一行"

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    // 把新拿到的二进制拼进 buffer
    buffer += decoder.decode(value, { stream: true });

    // 按换行符切分
    const lines = buffer.split("\n");
    buffer = lines.pop() || "";  // 保留不完整的行

    for (const line of lines) {
      const trimmed = line.trim();
      if (!trimmed) continue;                    // 空行跳过

      if (trimmed === "data: [DONE]") return;    // 结束标记

      if (trimmed.startsWith("data: ")) {
        const jsonStr = trimmed.slice(6);
        try {
          const parsed = JSON.parse(jsonStr);
          const delta = parsed.choices?.[0]?.delta;
          if (delta?.content) {
            yield delta.content;                 // 生成一个 token
          }
        } catch (e) {
          console.warn("Failed to parse SSE line:", trimmed);
        }
      }
    }
  }
}

// 使用
for await (const token of streamChatCompletion([{role:"user", content:"你好"}], config)) {
  process.stdout.write(token);
}
```

### 那个 `buffer` 为什么重要？

TCP 不按"行"切分数据包。一次 `reader.read()` 可能拿到：

```
第一次读：data: {"id":"chatcmpl-abc","choi
第二次读：ces":[{"delta":{"content":"量子"}}]}\n\n
```

如果不缓存半行数据，第一块数据会被当成非法 JSON 直接丢弃。**`buffer` 的作用就是把半行等下一次读取时拼完整。**

### 必须处理的错误场景

好的客户端代码必须同时处理三种情况：

1. **正常结束**：读到 `[DONE]`，或者 done 标记且之前的数据完整
2. **异常中断**：网络断开、代理超时——但 buffer 里已经有部分数据
3. **空响应**：什么都没收到就断了

```javascript
// 处理异常中断
try {
  // ... 读取循环
} catch (networkError) {
  if (bufferHasContent(buffer)) {
    // 尝试从已收到的部分数据中恢复内容
    recoverPartialContent(buffer);
  } else {
    throw new Error("Failed to get any response from the API");
  }
}
```

### Python 实现（内置库，无需 requests）

```python
import json
from http.client import HTTPSConnection

def stream_chat(messages, api_key, model="gpt-4o"):
    conn = HTTPSConnection("api.openai.com")
    body = json.dumps({
        "model": model,
        "messages": messages,
        "stream": True,
    })

    conn.request(
        "POST", "/v1/chat/completions",
        body=body,
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}",
        }
    )

    response = conn.getresponse()
    if response.status != 200:
        raise Exception(f"API error: {response.status} {response.read()}")

    buffer = ""
    for chunk in response:
        buffer += chunk.decode("utf-8")
        while "\n" in buffer:
            line, buffer = buffer.split("\n", 1)
            line = line.strip()
            if not line:
                continue
            if line == "data: [DONE]":
                return
            if line.startswith("data: "):
                data = json.loads(line[6:])
                delta = data.get("choices", [{}])[0].get("delta", {})
                if "content" in delta and delta["content"]:
                    yield delta["content"]
```

### Java 完整实现（Spring Boot 服务端 + HttpClient 客户端）

下面是一个开箱即用的 Java 实现，完整展示了从服务端推送 SSE 到客户端逐行解析的全流程。

项目结构：

```
demo/
├── src/main/java/org/freedom/demo/
│   ├── DemoApplication.java                ← Spring Boot 入口
│   ├── controller/
│   │   └── SseServerController.java        ← SSE 服务端
│   └── sse/
│       └── SseClient.java                  ← SSE 客户端（含 main，可直接运行）
└── pom.xml                                  ← Spring Boot 2.4 + Java 11
```

#### 服务端：用 Spring 的 SseEmitter 模拟 LLM 流式输出

```java
package org.freedom.demo.controller;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.io.IOException;
import java.util.Map;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@RestController
@RequestMapping("/v1")
public class SseServerController {

    @PostMapping(value = "/chat/completions",
                 produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter streamChat(@RequestBody Map<String, Object> request) {
        // -1L = 不自动超时，客户端决定何时断开
        SseEmitter emitter = new SseEmitter(-1L);

        ExecutorService executor = Executors.newSingleThreadExecutor();
        executor.execute(() -> {
            try {
                // 第 1 个 chunk: 先发 role（兼容 OpenAI 协议）
                emitter.send(SseEmitter.event()
                        .name("message")
                        .data("{\"id\":\"chatcmpl-001\",\"object\":\"chat.completion.chunk\","
                                + "\"choices\":[{\"delta\":{\"role\":\"assistant\"},"
                                + "\"index\":0,\"finish_reason\":null}]}"));

                // 逐 token 发送 — 模拟 LLM 生成
                String[] tokens = {"你好", "，", "我是", "你的", "智能", "助手", "。"};
                for (String token : tokens) {
                    Thread.sleep(300);  // 模拟计算延迟
                    emitter.send(SseEmitter.event()
                            .name("message")
                            .data("{\"id\":\"chatcmpl-001\",\"object\":\"chat.completion.chunk\","
                                    + "\"choices\":[{\"delta\":{\"content\":\"" + token + "\"},"
                                    + "\"index\":0,\"finish_reason\":null}]}"));
                }

                // 结束 chunk
                emitter.send(SseEmitter.event()
                        .name("message")
                        .data("{\"id\":\"chatcmpl-001\",\"object\":\"chat.completion.chunk\","
                                + "\"choices\":[{\"delta\":{},"
                                + "\"index\":0,\"finish_reason\":\"stop\"}]}"));

                // SSE 标准终止符
                emitter.send(SseEmitter.event().name("message").data("[DONE]"));
                emitter.complete();

            } catch (IOException | InterruptedException e) {
                emitter.completeWithError(e);
            }
        });
        executor.shutdown();
        return emitter;
    }
}
```

**服务端要点：**

| 要点 | 说明 |
|------|------|
| `produces = TEXT_EVENT_STREAM_VALUE` | 告诉 Spring 这个端点的 Content-Type 是 `text/event-stream` |
| `SseEmitter(-1L)` | 不设超时，由客户端关闭连接 |
| `emitter.send()` | 每次调用对应 HTTP 连接中一行 `data:` |
| `emitter.complete()` | 正常结束；`completeWithError()` 让客户端读到异常 |
| 异步线程 | 因为 `send()` 是阻塞的，必须用独立线程，不能占着 Tomcat 的请求处理线程 |

#### 客户端：用 Java 11+ HttpClient 手动解析 SSE 协议

```java
package org.freedom.demo.sse;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.concurrent.TimeUnit;

/**
 * 手动解析 SSE 的客户端 —— 展示协议层面的每一行是如何被吃掉的。
 * 开箱即用，不需要任何第三方依赖。Java 11+ 内置 HttpClient。
 */
public class SseClient {

    public static void main(String[] args) throws Exception {

        // ── 1. 构建 HTTP 请求 ───────────────────────
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://localhost:8080/v1/chat/completions"))
                .header("Content-Type", "application/json")
                .timeout(Duration.ofSeconds(30))
                .POST(HttpRequest.BodyPublishers.ofString(
                        "{\"model\":\"gpt-4o\"," +
                        "\"messages\":[{\"role\":\"user\",\"content\":\"你好\"}]," +
                        "\"stream\":true}"))
                .build();

        HttpClient client = HttpClient.newHttpClient();

        // ── 2. 发送请求，获取流式响应体 ────────────────
        // 关键：BodyHandlers.ofInputStream() 让我们逐字节读取
        HttpResponse<java.io.InputStream> response = client.send(request,
                HttpResponse.BodyHandlers.ofInputStream());

        // ── 3. 逐行解析 SSE 流 ────────────────────────
        var reader = new java.io.BufferedReader(
                new java.io.InputStreamReader(
                        response.body(), StandardCharsets.UTF_8));

        String line;
        StringBuilder fullContent = new StringBuilder();
        long startTime = System.nanoTime();

        while ((line = reader.readLine()) != null) {
            // 只处理 data: 行（跳过 event: 行、空行等）
            if (!line.startsWith("data:")) {
                continue;
            }

            // 提取 data: 后面的数据
            // 兼容 data:{"key":...} 和 data: {"key":...}
            String rawData = line.startsWith("data: ")
                    ? line.substring("data: ".length())
                    : line.substring("data:".length());

            // 检查是否结束
            if ("[DONE]".equals(rawData)) {
                break;
            }

            // 简易解析 JSON 提取字段
            String role = extractJsonField(rawData, "role");
            String content = extractJsonField(rawData, "content");
            String finishReason = extractJsonField(rawData, "finish_reason");

            if (role != null) {
                System.out.println("  [角色] " + role);
            }
            if (content != null) {
                fullContent.append(content);
                long elapsed = TimeUnit.NANOSECONDS.toMillis(
                        System.nanoTime() - startTime);
                System.out.printf("  [Token: %s]  (已接收: %dms)%n",
                        content, elapsed);
            }
            if (finishReason != null) {
                System.out.println("\n  [结束原因] " + finishReason);
            }
        }

        System.out.println("\n✅ 完整内容: " + fullContent);
    }

    // 简易 JSON 字段提取（演示用，生产环境请用 Jackson / Gson）
    private static String extractJsonField(String json, String fieldName) {
        String key = "\"" + fieldName + "\":\"";
        int start = json.indexOf(key);
        if (start == -1) return null;
        start += key.length();
        int end = json.indexOf("\"", start);
        return (end == -1) ? null : json.substring(start, end);
    }
}
```

**客户端要点：**

| 要点 | 说明 |
|------|------|
| `BodyHandlers.ofInputStream()` | 不等待响应下载完，拿到流立即开始逐行读取 |
| `BufferedReader.readLine()` | 天然处理行边界——但 TCP 包可能跨行，`BufferedReader` 内部做了 buffer |
| `data:` 有无空格的兼容 | `line.startsWith("data: ")` vs `line.startsWith("data:")`——Spring 和 OpenAI 的差异 |
| `[DONE]` 先检查 | 在 JSON parse 之前先判断，避免 parse 纯文本报错 |
| 简易 JSON 解析 | 仅演示用——真实项目必须用 Jackson/Gson |

#### 运行效果

```
============================================================
▶ 发送流式请求...

◁ 状态码: 200
◁ Content-Type: text/event-stream
============================================================

  [角色] assistant
  [Token: 你好]  (已接收: 275ms)
  [Token: ，]    (已接收: 578ms)
  [Token: 我是]  (已接收: 882ms)
  [Token: 你的]  (已接收: 1187ms)
  [Token: 智能]  (已接收: 1493ms)
  [Token: 助手]  (已接收: 1799ms)
  [Token: 。]    (已接收: 2103ms)

  [结束原因] stop

✅ 完整内容: 你好，我是你的智能助手。
============================================================
  总耗时: 2110ms
```

注意每个 token 间隔 ≈ 300ms 的规律——和服务端 `Thread.sleep(300)` 完全吻合。
非流式场景下你需要等 2.1 秒才看到第一个字；流式下 275ms 就看到第一个字了。

#### 启动方式

```bash
# 终端 1：启动 Spring Boot 服务端
./mvnw spring-boot:run

# 终端 2：编译 + 运行客户端
./mvnw compile
java -cp target/classes org.freedom.demo.sse.SseClient
```

---

## 6. 如何对接一个没有文档的私有 API

这是最实用的部分。你拿到一个内部部署的 LLM API，没有文档，没有 README，只有一个端点 URL 和一个 key。怎么办？

### 第一步：发一个最小请求，看它死不死

```bash
curl -X POST https://your-internal-api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-key" \
  -d '{"model":"any-name","messages":[{"role":"user","content":"hello"}]}'
```

看响应：

| 响应 | 你学到的信息 |
|------|------------|
| 返回了正常回复 | OpenAI 格式高度兼容 |
| 返回了明确错误 | **这是最好的文档**——"Missing required field: xxx" |
| 返回 404 / 405 | 路径不对。试 `/v1/completions`、`/generate`、`/api/chat` |
| 返回 401 / 403 | 认证方式不对。试试 `x-api-key` 头，或 key 在 body 里 |
| 连接超时 | 网络、端口、防火墙问题，和协议无关 |

### 第二步：系统探测参数兼容性

写个脚本，逐项测试：

```bash
# 测 streaming
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"hi"}],"stream":true}'

# 测 system message
curl -X POST ... -d '{"model":"x","messages":[{"role":"system","content":"Be concise"},{"role":"user","content":"hi"}]}'

# 测多轮对话
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"hi"},{"role":"assistant","content":"hello"},{"role":"user","content":"what is 2+2?"}]}'

# 测 temperature
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"hi"}],"temperature":0}'

# 测 max_tokens
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"tell me a story"}],"max_tokens":50}'
```

每次只变一个参数，记录：

| 参数 | 结果 |
|------|------|
| `stream` | ✅ SSE 格式正常 |
| `system` role | ✅ 正确遵循 |
| `temperature=0` | ✅ 但输出每次不同（见下一条） |
| `max_tokens=50` | ✅ 输出恰好在 50 token 被截断 |
| `tools` | ❌ 返回 400 |

### 第三步：检测"静默忽略"——最坑的情况

有些参数 API 接受（返回 200）但**实际不使用**。最典型的是 `temperature`。

如何发现？发 5 次完全相同的请求，temperature=0：

```bash
for i in {1..5}; do
  curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"give me a random number between 1-100"}],"temperature":0}' | jq '.choices[0].message.content'
done
```

如果 temperature=0 真的在工作，每次应该得到同一个数字。如果不同——这个参数被静默忽略了。**这是最危险的情况，因为你的代码不报错，但行为不受控。**

### 第四步：看 Streaming chunk 的原始格式

这是另一个容易不兼容的地方。直接在终端看：

```bash
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"hello"}],"stream":true}' | head -10
```

你会看到原始 SSE 行。记下 chunk 中 JSON 的路径。常见变体：

```
# 标准 OpenAI 格式
data: {"choices":[{"delta":{"content":"hello"}}]}

# 有些模型用 text 而非 delta.content
data: {"choices":[{"text":"hello"}]}

# 有些模型直接在顶层
data: {"text":"hello","finish_reason":null}
```

### 第五步：抓现有客户端的包（终极方案）

如果公司里已经有人在用这个 API，直接抓包。

找到任何一个现有的 OpenAI 兼容客户端（比如 Open WebUI、LobeChat、ChatGPT Next Web），配置指向你的私有 API，然后抓它的请求。

你会立刻看到：
- 它实际用了哪些参数
- 每条消息的精确格式
- streaming 的 chunk 结构
- 错误处理的具体行为

**这比任何探测方法都快。**

### 探测流程总结

```
你有私有API，没文档
│
├─ 能找到现有客户端吗？
│   └─ 能 → 抓包，直接看真实请求体
│
├─ 不能
│   ├─ 发最小请求 → 状态码？
│   │   ├─ 200 → 开始系统探测参数
│   │   ├─ 40x → 看错误信息（这是最好的文档）
│   │   └─ 超时/拒连 → 先排查网络、端口
│   │
│   ├─ 参数探测
│   │   ├─ 被接受且有效 → 记录到兼容表
│   │   ├─ 被接受但无效 → temperature=0 重复测试发现
│   │   └─ 被拒绝 → 看错误消息说了什么
│   │
│   └─ Streaming 测试
│       └─ 打印原始 SSE chunk → 确定数据路径
```

---

## 7. OpenAI 全字段参考手册

> 基于 OpenAI 官方 TypeScript SDK 类型定义（v4.67），保证字段名的准确性。

### 请求体（Request Body）——POST `/v1/chat/completions`

#### 必填

| 字段 | 类型 | 说明 |
|------|------|------|
| `model` | `string` | 模型 ID |
| `messages` | `array` | 对话消息。每条含 `role`、`content`，可选 `name` |

#### 生成控制

| 字段 | 类型 | 范围 | 说明 |
|------|------|------|------|
| `temperature` | `number` | 0 ~ 2 | 采样温度。和 top_p 二选一 |
| `top_p` | `number` | 0 ~ 1 | 核采样概率阈值 |
| `max_completion_tokens` | `number` | ≥ 1 | 生成上限（含 reasoning tokens） |
| `reasoning_effort` | `string` | `none` / `minimal` / `low` / `medium` / `high` / `xhigh` | 控制推理模型的思考深度，不同模型支持的值不同 |
| `max_tokens` | `number` | ≥ 1 | ⚠️ 已弃用，用上面的替代 |
| `stop` | `string \| string[]` | 最多 4 个 | 停止字符串 |
| `frequency_penalty` | `number` | -2.0 ~ 2.0 | 频率惩罚 |
| `presence_penalty` | `number` | -2.0 ~ 2.0 | 话题惩罚 |
| `seed` | `number` | — | 🧪 Beta，尽力保证确定性 |
| `n` | `number` | ≥ 1 | 生成几个回复，默认 1 |
| `logprobs` | `boolean` | — | 是否返回对数概率 |
| `top_logprobs` | `number` | 0 ~ 20 | 返回最可能的前 N 个 token 的概率。需 `logprobs: true` |
| `logit_bias` | `object` | — | 修改 token 出现概率。`{token_id: bias}`，bias 范围 -100 ~ 100 |

#### 输出格式

| 字段 | 类型 | 说明 |
|------|------|------|
| `response_format` | `object` | `{type:"text"}` / `{type:"json_object"}` / `{type:"json_schema", json_schema:{...}}` |

#### Streaming

| 字段 | 类型 | 说明 |
|------|------|------|
| `stream` | `boolean` | 是否 SSE 流式 |
| `stream_options` | `object` | `{include_usage: boolean}`，在最后 chunk 中返回 token 用量 |

#### Tool Calling

| 字段 | 类型 | 说明 |
|------|------|------|
| `tools` | `array` | 工具列表，最多 128 个。每个 `{type:"function", function:{name, description, parameters}}` |
| `tool_choice` | `string \| object` | `"none"` / `"auto"` / `"required"` / `{type:"function", function:{name:"..."}}` |
| `parallel_tool_calls` | `boolean` | 是否允许并行调用，默认 `true` |

#### 其他

| 字段 | 类型 | 说明 |
|------|------|------|
| `service_tier` | `string` | `"auto"` / `"default"`。付费 SLA 层级 |
| `store` | `boolean` | 是否在 dashboard 记录 |
| `metadata` | `object` | `{key: string}` 用于筛选 |
| `user` | `string` | 终端用户标识 |
| `function_call` | — | ⚠️ 已弃用，用 `tool_choice` |
| `functions` | — | ⚠️ 已弃用，用 `tools` |

---

### 响应体（非 Streaming）

#### 顶层

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1745829123,
  "model": "gpt-4o",
  "choices": [...],
  "usage": {...},
  "service_tier": "default",
  "system_fingerprint": "fp_xxx"
}
```

| 字段 | 类型 | 始终有？ | 说明 |
|------|------|----------|------|
| `id` | `string` | ✅ | 补全 ID |
| `object` | `string` | ✅ | 始终 `"chat.completion"` |
| `created` | `number` | ✅ | Unix 时间戳（秒） |
| `model` | `string` | ✅ | 模型名 |
| `choices` | `array` | ✅ | 回复列表 |
| `usage` | `object` | ✅ | token 用量 |
| `service_tier` | `string` | ❌ | 仅当请求中指定了 |
| `system_fingerprint` | `string` | ❌ | 后端配置指纹 |

#### choices[]

```
{
  "index": 0,
  "message": {
    "role": "assistant",
    "content": "回复文本",
    "refusal": null,
    "tool_calls": null
  },
  "logprobs": null,
  "finish_reason": "stop"
}
```

`finish_reason` 的可选值及含义：

| 值 | 含义 | 对策 |
|----|------|------|
| `"stop"` | 正常结束 | 回复完整 |
| `"length"` | 达到 max_tokens 被截断 | 回复不完整，考虑增大上限 |
| `"tool_calls"` | 模型要调用工具 | 执行工具后回传结果 |
| `"content_filter"` | 被内容过滤截断 | 回复被截断，调整 prompt 或检查合规性 |
| `"function_call"` | ⚠️ 已弃用 | 用 tool_calls |

#### usage

```
{
  "prompt_tokens": 42,
  "completion_tokens": 137,
  "total_tokens": 179,
  "prompt_tokens_details": {
    "audio_tokens": 0,
    "cached_tokens": 0
  },
  "completion_tokens_details": {
    "reasoning_tokens": 0,
    "audio_tokens": 0,
    "accepted_prediction_tokens": 0,
    "rejected_prediction_tokens": 0
  }
}
```

---

### 响应体（Streaming Chunk）

```
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion.chunk",
  "created": 1745829123,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "delta": {
        "role": "assistant",      // 第一个 chunk 有
        "content": "量",          // 之后逐 token
        "refusal": null,
        "tool_calls": [...]       // 工具调用时分块传输
      },
      "logprobs": null,
      "finish_reason": null       // 最后一个 chunk 非 null
    }
  ],
  "system_fingerprint": "fp_xxx",
  "usage": null                   // 仅最后的 usage chunk 非 null
}
```

结束标记：`data: [DONE]`

---

### 对照速查表

| 你要干嘛 | 用什么 |
|----------|--------|
| 每次产出相同结果 | `temperature: 0` + `seed: 42` |
| 强制输出 JSON | `response_format: {type:"json_object"}` + system prompt 指示 |
| 强制输出符合 Schema | `response_format: {type:"json_schema", json_schema:{...}}` |
| 让模型调用工具 | `tools: [...]` + `tool_choice: "auto"` |
| 强制调用指定工具 | `tool_choice: {type:"function", function:{name:"xxx"}}` |
| 拒绝某个 token 出现 | `logit_bias: {token_id: -100}` |
| 检测输出是否被截断 | `finish_reason === "length"` |
| 检测内容过滤截断 | `finish_reason === "content_filter"` |
| 检测模型拒绝回答 | 读 `message.refusal`，不是 `content` |
| Debug 后端是否变了 | 比较 `system_fingerprint` |
| 多轮对话工具结果 | `role: "tool"` 且必须带 `tool_call_id` |

---

## 8. 常见误解与调试指南

### 误解 1：所有 LLM API 都一样

**不对。** OpenAI 协议是事实标准，但：

- Google Gemini 有自己的端点和参数命名
- Anthropic Claude 消息格式不同（`role` 只有 `user`/`assistant`，多模态用 `content` 数组）
- 国产模型（通义千问、DeepSeek、智谱 GLM）大多兼容 OpenAI 格式，但各有额外参数

你的 agent 切换模型时报错，第一件事——检查消息格式。

### 误解 2：Streaming 比非 Streaming 快

**不对。** Streaming 不改变总计算时间。它只改变感知延迟——你更快看到第一个 token，但模型完成全部思考的时间是一样的。

### 误解 3：设置了 max_tokens 就能控制长度

**不准确。** `max_tokens` 是**硬上限**——模型到达这个数字就被切掉，输出不完整。但模型不一定写到这个数。要检查 `finish_reason`。

### 误解 4：api key 必须放在 Authorization 头

不一定。有些私有部署用 `x-api-key` 头，有的把 key 放在请求体内，有的用 Basic Auth。如果 401，试试不同的方式。

### 调试 Checklist

当 agent 出问题时，按这个顺序排查：

1. **消息格式对不对** → 检查 `messages` 数组的角色顺序
2. **streaming 能不能工作** → 试 `stream: false` 看是不是 SSE 解析出了问题
3. **参数是否被静默忽略** → temperature=0 重复 5 次看输出
4. **回复是否完整** → 检查 `finish_reason` 是不是 `"stop"`
5. **模型是否拒绝回答了** → 检查 `message.refusal` 字段
6. **tool call 是否拼接正确** → 如果 streaming，检查 `index` 是否正确对应
7. **私有 API 的端口/路径/认证** → 用最小请求验证

---

## 9. 附录：vLLM vs OpenAI API 差异详解

> 基于 vLLM v0.21.0 官方文档。vLLM 是目前最广泛使用的私有 LLM 部署框架，了解它与 OpenAI 的差异，等于理解了绝大多数私有 API 的行为特点。

### 9.1 vLLM 不支持的 OpenAI 参数

| 参数 | vLLM 行为 | 说明 |
|------|----------|------|
| `user` | **静默忽略** | vLLM 收下这个字段但不报错，也不做任何处理 |
| `suffix` | **不支持**（Completions API） | vLLM 的 `/v1/completions` 不实现这个参数 |
| `image_url.detail` | **不支持** | Vision 请求中的 detail 参数被忽略，始终用默认分辨率 |
| `parallel_tool_calls: false` | **有限支持** | 设为 `false` 确保最多返回一个 tool call。设为 `true`（默认）允许多个，但不保证——取决于模型本身的能力 |
| `store` / `metadata` | 通常不支持 | 私有部署没有 OpenAI dashboard |
| `service_tier` | 通常不支持 | 私有部署没有 tier 概念 |
| `max_completion_tokens` | **行为不同** | vLLM 用 `max_tokens`，且解释为「生成长度上限」。`max_completion_tokens` 在旧版 vLLM 中不识别，新版开始支持但行为可能不同 |
| `reasoning_effort` | **取决于模型** | vLLM 本身不处理这个参数——它透传给模型。如果模型不支持，参数被忽略 |

> **关键洞见**：vLLM 对不认识的参数默认**静默忽略**而不是报错。这意味着你传了 `reasoning_effort` 它返回 200，但实际没生效。这是之前说的「最坑的情况」。

### 9.2 vLLM 额外支持的参数（OpenAI 没有）

这些是 vLLM 独有、需要放在 `extra_body`（用 OpenAI SDK 时）或直接放在请求体顶层（直接 HTTP 调用时）的参数。

#### 采样控制

| 参数 | 类型 | 说明 |
|------|------|------|
| `top_k` | `int` | 只从概率最高的 K 个 token 中采样。OpenAI 没有这个参数 |
| `min_p` | `float` | 从概率 ≥ min_p × 最高概率 的 token 中采样。和 top_p 类似但更平滑 |
| `repetition_penalty` | `float` | 重复惩罚系数。>1.0 惩罚重复 token，<1.0 鼓励重复 |
| `length_penalty` | `float` | 长度惩罚。>1.0 鼓励生成长文本，<1.0 鼓励短文本 |
| `use_beam_search` | `bool` | 是否使用束搜索（beam search）代替采样。比采样更稳定但更慢 |
| `stop_token_ids` | `int[]` | 按 token ID 指定停止标记（不只是字符串） |
| `include_stop_str_in_output` | `bool` | 是否把停止字符串包含在输出中。OpenAI 默认不包含 |
| `ignore_eos` | `bool` | 忽略结束标记，让模型一直生成直到 max_tokens。OpenAI 没有 |
| `min_tokens` | `int` | 最少生成 token 数。在一定数量的 token 生成前不输出 EOS |
| `bad_words` | `string[]` | 禁止模型输出的词列表。OpenAI 没有这个参数 |

#### 结构化输出（v0.12.0+，替代旧的 `guided_*` 参数）

```python
# 通过 extra_body 传递
client.chat.completions.create(
    model="...",
    messages=[...],
    extra_body={
        "structured_outputs": {
            "choice": ["positive", "negative"],    # 从列表中选
            "json": {"type": "object", ...},       # JSON Schema
            "regex": "[A-Z]+",                     # 正则约束
            "grammar": "SELECT ...",               # EBNF 文法（如 SQL）
            "structural_tag": "..."                # 结构标签
        }
    }
)
```

| 结构化输出键 | 用途 | 示例 |
|-------------|------|------|
| `choice` | 限定输出只能是给定列表中的一项 | `["positive", "negative"]` |
| `json` | 输出必须匹配 JSON Schema | `{"type": "object", "properties": {...}}` |
| `regex` | 输出必须匹配正则 | `"\\d{4}-\\d{2}-\\d{2}"` |
| `grammar` | 输出必须匹配 EBNF 文法 | 用于生成 SQL、代码等复杂格式 |
| `structural_tag` | 结构标签 | 用于特定格式约束 |

> ⚠️ v0.12.0 之前用 `guided_json`、`guided_regex`、`guided_choice`、`guided_grammar` 这四个单独的字段。v0.12.0+ 已移除，只能用 `structured_outputs` 字典。

#### 其他 vLLM 独有参数

| 参数 | 说明 |
|------|------|
| `echo` | 如果为 true，新的 assistant 消息会拼接上一条同角色的消息 |
| `add_generation_prompt` | 是否在 chat template 末尾添加生成提示标记。默认 `true` |
| `continue_final_message` | 让模型继续最后一条消息而不是开始新消息。可用于「预填充」部分回复 |
| `documents` | RAG 文档列表 `[{title, text}, ...]`。需要在 chat template 中支持 |
| `chat_template` | 临时覆盖模型自带的 Jinja2 chat template |
| `truncate_prompt_tokens` | 从 prompt 末尾向前截断到指定 token 数 |
| `allowed_token_ids` | **只允许**输出这些 token ID。极其严格的输出控制 |
| `prompt_logprobs` | 返回 prompt 中每个 token 的 logprob（不只是输出 token） |
| `skip_special_tokens` | 在输出中是否跳过特殊 token（如 `<|endoftext|>`） |
| `return_token_ids` | 在响应中额外返回 token ID。streaming 模式下每个 chunk 包含 delta 的 token_ids |
| `return_prompt_text` | 在响应中返回经过 chat template 处理后的完整 prompt 文本（仅首 chunk） |
| `priority` | 请求优先级（0 为默认）。需模型启用 priority scheduling |
| `cache_salt` | 前缀缓存加盐，多用户环境防止缓存侧信道攻击 |
| `request_id` | 自定义请求 ID，不传则自动生成 |
| `repetition_detection` | 检测输出中重复的 N-gram 模式，发现后提前终止生成 |

### 9.3 响应体的差异

| 维度 | OpenAI | vLLM |
|------|--------|------|
| `usage.completion_tokens_details.reasoning_tokens` | 支持（o1/o3 系列） | ❌ 通常不支持 |
| `system_fingerprint` | 返回后端配置指纹 | ❌ 通常不返回 |
| `service_tier` | 请求中指定时返回 | ❌ 通常不返回 |
| Streaming chunk 结束标记 | `data: [DONE]` | 也是 `data: [DONE]` 但早期版本可能用 `data: [DONE]\n\n` |
| `choices[0].finish_reason` | 始终非 null | 某些旧版本在 streaming 中可能一直为 null |

### 9.4 其他行为差异

| 行为 | OpenAI | vLLM |
|------|--------|------|
| 不认识参数 | **返回 400** 明确报错 | **静默忽略**，返回 200 |
| 默认 temperature | 1.0（未指定时） | 取决于 HuggingFace 模型的 `generation_config.json`。如不确定，用 `--generation-config vllm` 启动可统一行为 |
| Tool calling 默认 | tools 参数传入即自动启用 | 需启动时加 `--enable-auto-tool-choice` 和 `--tool-call-parser`，否则静默忽略 |
| 多模态 image_url.detail | 支持 auto/low/high | ❌ 不支持，忽略 detail 字段 |
| `model` 字段值 | 短名如 `"gpt-4o"` | 一般是 HuggingFace 模型 ID 如 `"Qwen/Qwen2.5-7B-Instruct"` |
| Chat template | 由 OpenAI 服务端处理 | 使用 HuggingFace tokenizer 中的 jinja2 chat template。没有则必须传入 |

### 9.5 实测：如何验证一个 API 是不是 vLLM

```bash
# 方法1：传一个 OpenAI 不认识的参数，看反应
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"hi"}],"top_k":50}'

# vLLM 会正常处理（返回 200）
# OpenAI 会返回 400（不认识的参数）

# 方法2：看响应头
curl -sI ... | grep -i "server"

# vLLM 的响应头可能有特征（具体版本不同）

# 方法3：传 tools 但不加特殊配置
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"北京天气"}],"tools":[{"type":"function","function":{"name":"get_weather","parameters":...}}]}'

# 如果模型不调用工具（只是普通回复）→ vLLM 没开 --enable-auto-tool-choice
# 如果正常调用了 → 可能是 SGLang 或配置好的 vLLM

# 方法4：传 extra_body 看看
curl -X POST ... -d '{"model":"x","messages":[{"role":"user","content":"hi"}],"extra_body":{"structured_outputs":{"choice":["A","B","C"]}}}'

# vLLM 会识别 structured_outputs，其他框架可能会忽略或报错
```

### 9.6 一句话总结

> **vLLM 和 OpenAI 的参数差异不是「多了少了」的问题，而是行为哲学不同。OpenAI 对你传错的东西报错，vLLM 对你传错的东西微笑点头然后当没看见。**

这意味着：
- 对接 vLLM 私有 API 时，**不要只根据"返回 200"来判断参数生效**——要用 temperature=0 重复测试来验证
- Tool calling 默认不开——如果模型不调用工具，先怀疑这个
- 不认识就是忽略——vLLM 不会像 OpenAI 那样给你清晰的报错指引你修参数

### 9.7 你之前问的探测方法论，现在扩展一下

```
你先测它是什么框架（看错误风格、看 extra_body 是否识别）
  │
  ├─ 是 vLLM
  │   ├─ 测 tool calling → 会不会被静默忽略
  │   ├─ 测 extra_body 参数 → structured_outputs 是否工作
  │   ├─ 测 temperature 默认值 → 受 generation_config.json 影响
  │   └─ 列兼容表 → 哪些 OpenAI 参数被忽略
  │
  ├─ 是 SGLang
  │   └─ 测 JSON Schema 速度 → 比 vLLM 快很多
  │
  ├─ 是 Ollama
  │   └─ 测 keep_alive → 频繁模型加载卸载影响延迟
  │
  └─ 都不像
      └─ 回到最小探测流程
```

---

## 10. 附录：开源模型的 Thinking / Reasoning 机制

> 当你对接的私有 API 背后是开源推理模型（DeepSeek-R1、Qwen3、QwQ 等），thinking 机制的工作方式跟 OpenAI 完全不同。这一节告诉你它到底怎么工作，以及怎么判断你面对的那个模型支不支持。

### 10.1 核心直觉：开源推理模型怎么「思考」的？

OpenAI 的 `reasoning_effort` 是一个 API 参数——你传一个值，模型就按那个深度去思考。

开源模型不一样。开源推理模型的 thinking 能力是**训练出来的**，不是在 API 层加一个参数就能控制的。模型在训练时学会了先输出一段「内部思考」再输出「最终答案」，思考内容通常被 `<think>...</think>` 这样的特殊标记包裹。

所以区别就在这里：

```
OpenAI：  reasoning_effort: "high"  →  后端告诉模型「多想想」
开源模型： 模型本身就在 think        →  你只是选择要不要把思考内容暴露出来
```

### 10.2 三类开源推理模型

#### 第一类：永恒思考型——不可开关

代表：**DeepSeek-R1 系列**

模型永远先思考再回答。没有「关闭思考」这个选项。所有输出天然分两段：

```
<think> 推理过程... </think> 最终答案
```

你不需要传任何参数来开启 thinking，它永远在思考。API 层面，思考内容出现在 `message.reasoning`（vLLM 新版）或 `message.reasoning_content`（旧版）字段中。

> DeepSeek 官方 API 的 `deepseek-reasoner` 模型也是如此——你传 `thinking_config` 或 `reasoning_effort` 会直接报验证错误。

#### 第二类：开关可控型——可以开启或关闭

| 模型 | 默认 | 关闭方式（extra_body 中传） | 开启方式 |
|------|------|----------------------------|---------|
| **Qwen3 系列** | 开启 | `chat_template_kwargs: {enable_thinking: false}` | 默认开启，或设为 `true` |
| **IBM Granite 3.2** | 关闭 | 默认不思考 | `chat_template_kwargs: {thinking: true}` |
| **DeepSeek-V3.1** | 关闭 | 默认不思考 | `chat_template_kwargs: {thinking: true}` |
| **Holo2** | 开启 | `chat_template_kwargs: {thinking: false}` | 默认开启 |

> 注意这些参数都放在 `extra_body` 里，不是直接放在请求体顶层。用 HTTP 直接调用时就是请求体中的一个字段：
> ```json
> {"model": "xxx", "messages": [...], "extra_body": {"chat_template_kwargs": {"enable_thinking": false}}}
> ```

#### 第三类：不可思考型——普通模型

代表：**Qwen2.5、LLaMA 3、Mistral、Gemma** 等。

这些模型没有经过推理训练，不会输出 `<think>` 标记。你传什么参数都没用——**它们就是不会思考**。你的 agent 如果用这类模型还期望拿到 `reasoning` 字段，拿不到的。

### 10.3 vLLM 中的 Reasoning 机制——关键基础设施

vLLM 本身不做推理。它做的事情很简单：**把模型输出的文本，按照 `<think>...</think>` 这样的标记拆成 `reasoning` 和 `content` 两个字段。**

要做到这一点，vLLM 启动时必须指定一个 **reasoning parser**：

```bash
vllm serve Qwen/Qwen3-8B --reasoning-parser qwen3
```

vLLM v0.21.0 支持的 reasoning parser：

| Parser 名 | 适用模型 | 支持结构化输出 | 支持 Tool Calling |
|-----------|---------|:------------:|:----------------:|
| `deepseek_r1` | DeepSeek-R1 系列、QwQ-32B | ✅ | ❌ |
| `deepseek_v3` | DeepSeek-V3.1 | ✅ | ❌ |
| `qwen3` | Qwen3 系列 | ✅ | ✅ |
| `granite` | IBM Granite 3.2 | ❌ | ❌ |
| `cohere_command3` | Cohere Command A Reasoning | ✅ | ✅ |
| `glm45` | GLM-4.5 系列 | ✅ | ✅ |
| `holo2` | Holo2 系列 | ✅ | ✅ |
| `hunyuan_a13b` | Hunyuan A13B 系列 | ✅ | ✅ |
| `minimax_m2_append_think` | MiniMax-M2 | ✅ | ✅ |
| `ernie45` | ERNIE-4.5 系列 | ✅ | ✅ |

**关键要点**：
- **如果 vLLM 启动时没加 `--reasoning-parser`**，即使你部署的是 DeepSeek-R1，返回的 `message` 里也只会有 `content` 字段——`reasoning` 字段不存在，思考过程直接混在 `content` 里
- **有些 parser 不支持 tool calling**（如 `deepseek_r1`），这意味着你用 DeepSeek-R1 做 agent 时没办法同时用推理和工具调用
- **vLLM 不支持 `reasoning_effort`**——这个参数会被静默忽略。控制思考深度在 vLLM 层面走的是 `thinking_token_budget`（限制思考 token 数量）

### 10.4 从响应的 chunk 结构看是否支持 reasoning

非 streaming 响应（支持推理的模型）：

```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "reasoning": "这里是模型的思考过程...",
      "content": "这里是最终答案"
    }
  }]
}
```

Streaming 响应——推理 token 出现在 `delta.reasoning`：

```
第一个 chunk:  delta.role = "assistant"
后续 chunk:    delta.reasoning = "比较"     ← 思考过程逐 token
               delta.reasoning = "9.11"
               delta.reasoning = "和"
               delta.reasoning = "9.8"
               ...
               delta.content = "9.8"       ← 思考结束，开始输出答案
               delta.content = "更大"
最后一个 chunk: finish_reason = "stop"
```

**注意**：OpenAI 的 Python SDK 原生不支持 `reasoning` 属性，需要用 `getattr` 安全提取：

```python
stream = client.chat.completions.create(
    model=model, messages=messages, stream=True
)
for chunk in stream:
    delta = chunk.choices[0].delta
    reasoning = getattr(delta, "reasoning", None)
    content = getattr(delta, "content", None)
    if reasoning:
        print(f"[思考] {reasoning}", end="")
    elif content:
        print(content, end="")
```

### 10.5 控制思考深度——真正有效的参数

对于支持可控 thinking 的开源模型，以下是能用的参数：

#### 以 Qwen3 为例

```json
{
  "model": "Qwen3-8B",
  "messages": [...],
  "extra_body": {
    "chat_template_kwargs": {
      "enable_thinking": true
    }
  },
  "thinking_token_budget": 512
}
```

| 参数 | 位置 | 作用 |
|------|------|------|
| `enable_thinking` | `extra_body.chat_template_kwargs` | 是否开启 thinking |
| `thinking` | `extra_body.chat_template_kwargs` | Granite/DeepSeek-V3.1 用的字段名 |
| `thinking_token_budget` | 请求体顶层 | 限制思考 token 数量（vLLM 参数） |
| `reasoning_effort` | 请求体顶层 | ❌ **vLLM 不认识，静默忽略** |

`thinking_token_budget` 是控制思考深度的最实用参数。设小一点（如 128）模型会快速思考、快速回答；设大一点（如 2048）模型可以深入推理。如果没有设这个参数，vLLM 不对推理 token 做任何限制。

#### server 级别的默认值

vLLM 还可以在启动时设置所有请求的默认 thinking 行为：

```bash
# Qwen3：默认关闭 thinking（全局）
vllm serve Qwen/Qwen3-8B \
    --reasoning-parser qwen3 \
    --default-chat-template-kwargs '{"enable_thinking": false}'

# Granite：默认开启 thinking（全局）
vllm serve ibm-granite/granite-3.2-2b-instruct \
    --reasoning-parser granite \
    --default-chat-template-kwargs '{"thinking": true}'
```

请求级别的参数会覆盖 server 级别的默认值。

### 10.6 实战：怎么判断一个私有 API 的 thinking 支持情况

给你一串可执行的步骤，不需要任何文档：

#### 步骤 1：发一个需要推理的问题

```bash
curl -X POST https://your-api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer xxx" \
  -d '{
    "model": "xxx",
    "messages": [
      {"role": "user", "content": "9.11 和 9.8 哪个大？"}
    ]
  }' | jq '.choices[0].message'
```

#### 步骤 2：看响应判断情况

| 响应特征 | 结论 |
|----------|------|
| 有 `"reasoning"` 字段 | ✅ 推理模型，且 vLLM 配了 parser。不需要额外操作 |
| 无 `reasoning` 字段，但 `content` 里包含「首先...」「让我想想...」等思考痕迹 | ⚠️ 模型在思考，但 vLLM 没配 parser。找运维加 `--reasoning-parser` |
| 无 `reasoning` 字段，`content` 直接给出答案 | ❌ 要么不是推理模型，要么 thinking 被关了 |

#### 步骤 3：测能否控制 thinking

```bash
# 测试组 A：Qwen3 风格（开/关）
curl -X POST ... -d '{"extra_body": {"chat_template_kwargs": {"enable_thinking": false}}}'
curl -X POST ... -d '{"extra_body": {"chat_template_kwargs": {"enable_thinking": true}}}'

# 测试组 B：Granite 风格
curl -X POST ... -d '{"extra_body": {"chat_template_kwargs": {"thinking": true}}}'

# 测试组 C：vLLM 思考预算
curl -X POST ... -d '{"thinking_token_budget": 512}'

# 测试组 D：OpenAI 格式（大概率不支持，但可以验证）
curl -X POST ... -d '{"reasoning_effort": "high"}'
```

判断方法：比较每组响应中 `reasoning` 字段的长度和内容。如果变了，说明该参数生效。**传了和没传没区别**，说明被静默忽略了。

#### 步骤 4：看 streaming 确认

```bash
curl -X POST ... -d '{"stream": true, ...}' 2>/dev/null | grep "data:" | head -20
```

如果看到 `"reasoning"` 出现在 `delta` 中，说明模型确实在做推理。

### 10.7 决策树——不用文档，三分钟摸清

```
发一个「9.11 vs 9.8」→ 看响应
│
├─ message 里有 reasoning 字段
│   ├─ → ✅ 推理模型，parser 已配
│   └─ → 尝试控制：thinking_token_budget / enable_thinking
│
├─ content 里有思考痕迹但没 reasoning 字段
│   └─ → vLLM 没配 reasoning-parser，找运维解决
│
└─ content 直接给答案，毫无思考痕迹
    ├─ 试 extra_body.chat_template_kwargs.enable_thinking = true → 有 reasoning？
    │   ├─ 是 → Qwen3 系列，thinking 被关了
    │   └─ 否 →
    │       ├─ 试 extra_body.chat_template_kwargs.thinking = true → 有 reasoning？
    │       │   ├─ 是 → Granite/DeepSeek-V3.1
    │       │   └─ 否 → 大概率不是推理模型（Qwen2.5/LLaMA3/Mistral）
    │       └─ 查 /v1/models 端点看模型名，去 HuggingFace 确认
```

### 10.8 一句话总结

> **开源模型的 thinking 不是 API 参数决定的，是模型本身的能力。vLLM 只是负责把 `<think>` 标记拆成 `reasoning` 字段。你测的不是「这个 API 支不支持 thinking」，而是「这个模型本身会不会思考」加上「vLLM 有没有配好 parser」。**


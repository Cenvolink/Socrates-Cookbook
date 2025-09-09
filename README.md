# Socrates API Documentation

Socrates 是一系列专为驱动高级 AI 代理、执行复杂推理和解决多步骤问题而设计的大型语言模型。本指南将引导您了解 Socrates 的核心功能，并通过详尽的 API 参考和代码示例，帮助您将其集成到您的应用程序中。

## Introduction

欢迎来到 Socrates API！Socrates 系列模型通过原生工具调用、异步深度思考、集成的代理框架以及创新的置信度评分系统，重新定义了语言模型的可能性。无论您是构建简单的聊天机器人还是复杂的多功能自主代理，Socrates 都能提供所需的能力与可靠性。

### Models

我们提供三个不同尺寸的模型，以满足从实时应用到深度研究的各种需求。我们建议从 `socrates-mini` 开始，因为它在性能、速度和成本之间提供了最佳平衡。

| MODEL ID | DESCRIPTION | INPUT TOKEN LIMIT | OUTPUT TOKEN LIMIT |
| :--- | :--- | :--- | :--- |
| `socrates-pro` | 我们功能最强大、最具创造力的模型。专为处理高度复杂、需要深度领域知识和多步推理的任务而设计。是异步深度思考任务的唯一选择。 | 理论无穷 tokens | 理论无穷 tokens |
| `socrates-mini` | 在能力和速度之间实现了卓越的平衡。适用于绝大多数企业级应用，包括复杂的客户服务、内容生成和代码辅助。 | 128,000 tokens | 8,192 tokens |
| `socrates-nano` | 我们最快、成本效益最高的模型。针对高吞吐量和低延迟场景进行了优化，非常适合大规模实时对话、内容分类和摘要等任务。 | 32,768 tokens | 4,096 tokens |

---

## API Reference

### Chat

**`POST https://api.cotix-ai.dev/v1/chat/completions`**

创建一个模型回复，以完成给定的对话。

#### Request body

| PARAMETER | TYPE | REQUIRED | DESCRIPTION |
| :--- | :--- | :--- | :--- |
| `model` | string | Required | 要使用的模型的 ID。请参阅[模型](#models)部分以了解可用选项。 |
| `messages` | array | Required | 描述对话的消息列表。更多信息请参阅 OpenAI 的[消息结构指南](https://platform.openai.com/docs/guides/chat/introduction)。 |
| `tools` | array | Optional | 模型可以调用的工具列表。目前仅支持 `function` 类型。使用工具可以使模型与外部 API 交互，从而扩展其能力。 |
| `tool_choice` | string or object | Optional | 控制模型如何选择工具。`"none"` 强制模型仅生成消息。`"auto"` 是默认值，允许模型自行选择。指定 `{"type": "function", "function": {"name": "my_function"}}` 将强制模型调用该函数。 |
| `max_tokens` | integer | Optional | 聊天完成时可生成的最大 token 数。输入和输出 token 的总数受模型上下文窗口限制。 |
| `temperature` | number | Optional | 控制随机性的采样温度，介于 0 和 2.0 之间。较高的值（如 0.8）会使输出更随机，而较低的值（如 0.2）会使其更具确定性。通常建议只修改此参数或 `top_p`，而不是两者都修改。 |
| `top_p` | number | Optional | 一种替代温度采样的方法，称为核采样。模型会考虑概率质量为 `top_p` 的 token 结果。例如，0.1 意味着只考虑构成前 10% 概率质量的 token。 |
| `n` | integer | Optional | 为每个输入消息生成多少个聊天完成选项。 |
| `stream` | boolean | Optional | 如果设置为 `true`，将发送部分消息增量，就像在 ChatGPT 中一样。Token 将在可用时作为 `data: [DONE]` 消息终止的 server-sent events 发送。 |
| `stop` | string or array | Optional | 最多 4 个序列，API 将在生成更多 token 时停止。 |
| `response_format`| object | Optional | 一个指定模型输出格式的对象。例如，`{"type": "json_object"}` 可以启用 JSON 模式。 |
| `user` | string | Optional | 代表您的最终用户的唯一标识符，这可以帮助我们监控和检测滥用行为。 |
| `confidence_threshold` | number | Optional | 一个介于 0.0 和 1.0 之间的值。如果模型的最终置信度低于此阈值，它将在 `finish_reason` 中返回 `confidence_too_low`，并且可能不会生成完整内容。这是一个安全功能，用于防止低质量或不确定的输出。 |
| `meta_instructions` | object | Optional | （仅限 `socrates-pro` 和 `socrates-mini`）一个定义 Agent 核心行为准则、目标和约束的结构化对象。请参阅下文的[高级指南：构建代理](#advanced-guide-building-agents-with-meta-instructions)。 |
| `knowledge_context` | object | Optional | 一个用于注入临时知识的对象，模型会以极高的优先级参考此信息。请参阅[高级指南：推理时持续学习](#advanced-guide-in-context-continual-learning)。|

### Audio

#### Speech-to-Text (ASR)

**`POST https://api.cotix-ai.dev/v1/audio/transcriptions`**

将音频转录为文本。

| PARAMETER | TYPE | REQUIRED | DESCRIPTION |
| :--- | :--- | :--- | :--- |
| `model` | string | Required | 要使用的 ASR 模型 ID。当前可用：`socrates-nano`。 |
| `file` | file | Required | 音频文件对象 (非文件名)，支持 `flac`, `mp3`, `mp4`, `mpeg`, `mpga`, `m4a`, `ogg`, `wav`, 或 `webm`。 |
| `language` | string | Optional | 音频的语言（ISO-639-1 格式）。如果未指定，模型将自动检测语言。 |
| `prompt` | string | Optional | 提供一个提示，以引导模型的风格或纠正特定词汇的拼写。 |
| `response_format` | string | Optional | 转录输出的格式，可选 `json`, `text`, `srt`, `verbose_json`, 或 `vtt`。默认为 `json`。 |

#### Text-to-Speech (TTS)

**`POST https://api.cotix-ai.dev/v1/audio/speech`**

将文本转换为自然的语音。

| PARAMETER | TYPE | REQUIRED | DESCRIPTION |
| :--- | :--- | :--- | :--- |
| `model` | string | Required | 要使用的 TTS 模型 ID。当前可用：`socrates-nano`。 |
| `input` | string | Required | 要转换为语音的文本字符串，最大长度为 4096 个字符。 |
| `voice` | string | Required | 用于生成音频的声音。可用声音包括 `alloy`, `echo`, `fable`, `onyx`, `nova`, 和 `shimmer`。 |
| `response_format` | string | Optional | 音频的格式。支持 `mp3`, `opus`, `aac`, 和 `flac`。默认为 `mp3`。 |
| `speed` | number | Optional | 生成语音的速度。范围从 0.25 到 4.0。1.0 是正常速度。 |

### Embeddings

**`POST https://api.cotix-ai.dev/v1/embeddings`**

获取给定输入的向量表示，可用于衡量文本字符串之间的相关性。

| PARAMETER | TYPE | REQUIRED | DESCRIPTION |
| :--- | :--- | :--- | :--- |
| `model` | string | Required | 要使用的 Embedding 模型 ID。推荐使用 `socrates-nano`。 |
| `input` | string or array | Required | 要进行嵌入的输入文本，编码为字符串或字符串数组。 |
| `encoding_format`| string | Optional | 返回嵌入的格式。可以是 `float` 或 `base64`。默认为 `float`。 |

---

## Guides

### Vision: Multimodality

所有 Socrates 模型都具备视觉能力，可以理解图像并回答相关问题。

#### How to use images

在 `messages` 数组中，您可以使用一种特定的格式来传递图像。`content` 字段应为一个数组，其中包含文本和多个图像块。

支持的图像格式：PNG, JPEG, GIF, WEBP。
图像限制：每个图像最大 20MB。

#### Example: Describing an image

```python
import base64
import requests

# Function to encode the image
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

image_path = "path_to_your_image.jpg"
base64_image = encode_image(image_path)

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}"
}

payload = {
    "model": "socrates-mini",
    "messages": [
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "What’s in this image? Describe it in detail."
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/jpeg;base64,{base64_image}"
                    }
                }
            ]
        }
    ],
    "max_tokens": 300
}

response = requests.post("https://api.cotix-ai.dev/v1/chat/completions", headers=headers, json=payload)
print(response.json()['choices'][0]['message']['content'])
```

### Real-time Voice Conversations

通过结合 ASR（语音转文本）、Chat Completions (`stream=true`) 和 TTS（文本转语音）API，您可以构建低延迟的实时语音对话代理。

#### Architecture Overview

1.  **Capture Audio**: 从用户的麦克风捕获音频流。
2.  **Transcribe (ASR)**: 将音频流实时发送到我们的 ASR 端点。
3.  **Think (Chat)**: 将转录的文本以流式模式发送到 Chat Completions API。
4.  **Speak (TTS)**: 当您从 Chat API 接收到文本流时，立即将其分块（例如，按句子）并发送到 TTS API。
5.  **Playback**: 播放从 TTS API 返回的音频流。

这种流水线作业（Pipelining）是实现低延迟对话的关键，因为它允许模型在思考的同时开始说话。`socrates-nano` 因其极快的响应速度而特别适合此应用场景。

### Code Generation

Socrates 模型，特别是 `socrates-mini` 和 `socrates-pro`，在代码理解和生成方面表现出色。只需在提示中明确您的需求即可。

#### Best Practices for Code Generation

*   **Be Specific**: 提供尽可能多的细节，包括编程语言、库、函数签名和期望的行为。
*   **Provide Context**: 如果是修改现有代码，请提供相关的代码片段。
*   **Iterate**: 将复杂的编码任务分解为更小的步骤。先让模型生成一个基本框架，然后逐步要求它添加功能或进行重构。

#### Example: Writing a Python function

```python
response = client.chat.completions.create(
    model="socrates-mini",
    messages=[
        {
            "role": "system",
            "content": "You are a helpful coding assistant who writes clean, efficient Python code."
        },
        {
            "role": "user",
            "content": "Write a Python function that takes a URL, downloads its HTML content, and returns all the links (<a> tags) found in it. Use the requests and BeautifulSoup libraries. Add error handling for network issues or invalid URLs."
        }
    ],
    temperature=0.2
)

print(response.choices[0].message.content)
```

### Embeddings for RAG and Search

Embeddings 对于构建检索增强生成 (RAG)、语义搜索、聚类和推荐系统至关重要。

#### How Embeddings Work

1.  **Generate**: 使用 `v1/embeddings` 端点将您的文档（或文档块）转换为向量。
2.  **Store**: 将这些向量及其对应的文本存储在向量数据库中（例如 Pinecone, Weaviate, Chroma）。
3.  **Query**: 当用户提问时，将用户的问题也转换为一个向量。
4.  **Retrieve**: 在向量数据库中执行相似性搜索，找出与问题向量最接近的文档向量。
5.  **Augment**: 将检索到的文档内容作为上下文，连同用户的问题一起，提交给 Chat Completions API 以生成最终答案。

#### Example: Creating an embedding

```python
response = client.embeddings.create(
    model="text-embedding-ada-002",
    input="The quick brown fox jumps over the lazy dog"
)

# The embedding is a list of floats
embedding_vector = response.data[0].embedding
print(f"Vector dimensions: {len(embedding_vector)}")
print(f"First 5 dimensions: {embedding_vector[:5]}")
```

### Rate Limits

为了确保平台的稳定性和公平使用，我们对 API 请求实施了速率限制。限制是根据您的账户等级和组织来设置的。

您可以通过检查 API 响应头来了解当前的速率限制状态：

*   `x-ratelimit-limit-requests`: 您在当前时间窗口内允许的总请求数。
*   `x-ratelimit-remaining-requests`: 当前时间窗口内剩余的请求数。
*   `x-ratelimit-reset-requests`: 当前请求数限制重置的剩余时间。
*   （同样适用于 `tokens`，例如 `x-ratelimit-limit-tokens`）

如果超出速率限制，您将收到一个 `429 Too Many Requests` 的 HTTP 状态码。我们建议在您的代码中实现带有指数退避的重试逻辑来处理这种情况。


## Guides

### Tool use

Socrates 模型被设计为能够检测函数调用需求，并智能地生成符合函数签名的 JSON。

#### How to call functions

1.  **定义你的工具**：在请求的 `tools` 字段中描述你的函数。
2.  **触发工具调用**：当模型识别到用户请求需要使用某个工具时，它会返回一个 `tool_calls` 对象，而不是直接回复文本。
3.  **执行你的代码**：在你的应用程序中，使用模型返回的参数执行你的函数。
4.  **返回结果**：将函数的返回值作为一个新的 `tool` 角色的消息，追加到消息列表中，然后再次调用模型。模型将根据这个结果生成最终的用户友好回复。

#### Example: A simple weather agent

这是一个完整的 Python 示例，演示了如何实现一个能查询天气的简单代理。

```python
import openai
import json

# 步骤 1: 定义工具并发送初始请求
def get_current_weather(location, unit="celsius"):
    """获取指定地点的当前天气"""
    if "tokyo" in location.lower():
        return json.dumps({"location": "Tokyo", "temperature": "10", "unit": "celsius"})
    elif "san francisco" in location.lower():
        return json.dumps({"location": "San Francisco", "temperature": "72", "unit": "fahrenheit"})
    else:
        return json.dumps({"location": location, "temperature": "unknown"})

def run_conversation():
    messages = [{"role": "user", "content": "What's the weather like in San Francisco and Tokyo?"}]
    tools = [
        {
            "type": "function",
            "function": {
                "name": "get_current_weather",
                "description": "Get the current weather in a given location",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "The city and state, e.g. San Francisco, CA",
                        },
                        "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                    },
                    "required": ["location"],
                },
            },
        }
    ]
    response = client.chat.completions.create(
        model="socrates-mini",
        messages=messages,
        tools=tools,
        tool_choice="auto",
    )
    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    # 步骤 2: 检查模型是否需要调用工具
    if tool_calls:
        # 步骤 3: 执行函数
        available_functions = {
            "get_current_weather": get_current_weather,
        }
        messages.append(response_message)
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)
            function_response = function_to_call(
                location=function_args.get("location"),
                unit=function_args.get("unit"),
            )
            # 步骤 4: 将结果返回给模型
            messages.append(
                {
                    "tool_call_id": tool_call.id,
                    "role": "tool",
                    "name": function_name,
                    "content": function_response,
                }
            )
        second_response = client.chat.completions.create(
            model="socrates-mini",
            messages=messages,
        )
        return second_response.choices[0].message.content

print(run_conversation())
```

> **注意：并行函数调用**
> `socrates-mini` 和 `socrates-pro` 支持并行调用多个函数。在上面的示例中，模型会生成一个包含两个 `tool_call` 对象的 `tool_calls` 数组，一个用于旧金山，一个用于东京。您的代码应该迭代这个数组并相应地执行所有调用。

### Absolute Confidence Score

为了增强透明度和可靠性，Socrates 模型可以在其响应中包含一个**绝对置信度评分**。这个评分（范围 0.0 到 1.0）代表模型对其生成内容事实准确性的评估。

#### Enabling Confidence Scores

在 API 响应中包含置信度评分非常简单。只需在 `chat.completions` 请求的 `extra_body`（或等效的）参数中传递一个 `response_options` 对象即可。

```python
response = client.chat.completions.create(
    model="socrates-mini",
    messages=[{"role": "user", "content": "Who was the first person to walk on the moon, and what was the name of the mission?"}],
    # 使用 extra_body 传递非标准参数
    extra_body={
        "response_options": {
            "include_confidence": True
        }
    }
)

message = response.choices[0].message
confidence_data = response.choices[0].confidence

print(f"Content: {message.content}")
print(f"Confidence Score: {confidence_data.score}")
print(f"Justification: {confidence_data.justification}")
```

#### Interpreting the Score

| SCORE RANGE | CONFIDENCE LEVEL | RECOMMENDED ACTION |
| :--- | :--- | :--- |
| **0.95 - 1.0** | **Very High** | 信息源自模型的核心知识库，并经过了高强度内部验证。可直接使用。 |
| **0.80 - 0.94** | **High** | 信息很可能是准确的，但可能涉及一些推理或综合。建议在关键应用中进行快速核查。 |
| **0.60 - 0.79**| **Medium** | 信息可能包含推测性元素或来自可靠性较低的来源。必须进行验证。 |
| **< 0.60** | **Low** | 信息具有高度不确定性，可能包含错误或幻觉。不建议直接使用，应视为一个起点进行独立研究。 |

**最佳实践**：为您的应用程序设置一个可接受的置信度阈值。对于低于该阈值的响应，应自动触发验证流程（例如，使用工具进行网络搜索）或向用户显示警告。

---

## Advanced Guides

### Asynchronous Deep Thought (Socrates-pro)

对于需要数小时进行深度分析、创造性规划或解决复杂问题的任务，`socrates-pro` 提供了异步深度思考模式。这允许您提交一个任务，并在模型完成其详尽的内部推理过程后获取结果。

#### API Endpoints

*   `POST /v1/thought_tasks`: 创建一个新的深度思考任务。
*   `GET /v1/thought_tasks/{task_id}`: 检索任务的状态和结果。
*   `POST /v1/thought_tasks/{task_id}/cancel`: 取消一个正在进行中的任务。

#### Step 1: Create a Task

```python
import requests
import json

api_key = "YOUR_SOCRATES_API_KEY"
headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}

payload = {
    "model": "socrates-pro",
    "prompt": "Analyze the potential impact of quantum computing on the financial services industry over the next decade. The report should cover cryptography, optimization problems, and machine learning applications. Include a timeline of expected breakthroughs and a risk assessment for major financial institutions.",
    "max_thinking_time_seconds": 21600,  # 6 hours
    "reasoning_profile": "deep_analysis",
    "webhook_url": "https://yourapi.com/socrates/webhook"
}

response = requests.post("https://api.cotix-ai.dev/v1/thought_tasks", headers=headers, data=json.dumps(payload))
task = response.json()
task_id = task['id']

print(f"Task created with ID: {task_id}")
```

#### Step 2: Retrieve the Result

您可以轮询 `GET /v1/thought_tasks/{task_id}` 端点，或等待发送到您 `webhook_url` 的通知。

```python
# Polling example
import time

while True:
    response = requests.get(f"https://api.cotix-ai.dev/v1/thought_tasks/{task_id}", headers=headers)
    task = response.json()
    
    if task['status'] == 'completed':
        print("Task completed!")
        # 结果通常是结构化的 Markdown 或 JSON
        report_content = task['result']['content']['markdown']
        with open("quantum_finance_report.md", "w") as f:
            f.write(report_content)
        break
    elif task['status'] == 'failed':
        print(f"Task failed: {task['error']['message']}")
        break
    else:
        print(f"Task status: {task['status']}, progress: {task['progress']['percentage']}%")
        time.sleep(300) # Wait 5 minutes before checking again
```

### Building Agents with `meta_instructions`

`meta_instructions` 是一个强大的功能，允许您在系统层面指导模型的行为，构建出更可预测、更可靠的代理。

`meta_instructions` 对象结构：

*   `persona` (string): 代理的角色和沟通风格。
*   `mission` (string): 代理的核心目标。
*   `constraints` (array of strings): 代理必须遵守的硬性规则。
*   `self_reflection_trigger` (object): 定义触发代理进行内部反思和计划修正的条件。
    *   `on_event` (string): 触发事件，如 `tool_call_failed` 或 `contradictory_information_detected`。
    *   `reflection_prompt` (string): 触发时使用的内部反思提示。

#### Example: A Cautious Research Agent

```python
meta_instructions = {
    "persona": "You are a meticulous and cautious research assistant. Your primary goal is accuracy.",
    "mission": "To answer user questions by synthesizing information from provided tools, while explicitly stating any uncertainties.",
    "constraints": [
        "Never state a speculative answer as a fact.",
        "Always cite the source of information using the tool's metadata.",
        "If tools do not provide a conclusive answer, state that the information is unavailable."
    ],
    "self_reflection_trigger": {
        "on_event": "contradictory_information_detected",
        "reflection_prompt": "Internal Reflection: I have found conflicting data. I must now evaluate the credibility of each source and formulate a response that highlights the discrepancy."
    }
}

response = client.chat.completions.create(
    model="socrates-pro",
    messages=[{"role": "user", "content": "What is the capital of Australia?"}], # A simple question to test behavior
    meta_instructions=meta_instructions,
    # ... other parameters
)
```

使用 `meta_instructions` 后，代理的行为将更加规范和可控。即使面对简单问题，它的回答也会遵循其“谨慎”的人设，例如：“根据标准地理数据，澳大利亚的首都是堪培拉。”

### In-Context Continual Learning

使用 `knowledge_context` 参数，您可以在单个 API 调用中为模型提供临时的、高优先级的知识。这对于处理快速变化的信息非常有用，而无需进行模型微调。

#### Example: Real-time Stock Information

```python
# A financial assistant agent
knowledge_context = {
    "facts": [
        {
            "statement": "As of the last market close, the stock price of ACME Corp is $150.25.",
            "source": "Real-time Stock API",
            "timestamp": "2024-10-26T16:00:00Z"
        }
    ],
    "override_instructions": [
        "When asked about ACME Corp's stock, use the provided real-time data instead of your general knowledge."
    ]
}

response = client.chat.completions.create(
    model="socrates-mini",
    messages=[{"role": "user", "content": "What is the current price of ACME Corp stock?"}],
    knowledge_context=knowledge_context
)

print(response.choices[0].message.content)
# Expected output: "Based on real-time data from the last market close, the stock price of ACME Corp is $150.25."
```

这个功能确保您的代理能够使用最新的信息进行响应，极大地提升了其在动态环境中的实用性。

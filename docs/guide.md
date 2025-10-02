# Socrates API Documentation

Socrates 是一系列专为驱动高级 AI 代理、执行复杂推理和解决多步骤问题而设计的大型语言模型。本指南将引导您了解 Socrates 的核心功能，并通过详尽的 API 参考和代码示例，帮助您将其集成到您的应用程序中。

## Introduction

欢迎来到 Socrates API！Socrates 系列模型通过原生工具调用、异步深度思考、集成的代理框架以及创新的置信度评分系统，重新定义了语言模型的可能性。无论您是构建简单的聊天机器人还是复杂的多功能自主代理，Socrates 都能提供所需的能力与可靠性。

### Models

我们提供三个不同尺寸的模型，以满足从实时应用到深度研究的各种需求。我们建议从 `socrates-mini` 开始，因为它在性能、速度和成本之间提供了最佳平衡。

| MODEL ID | DESCRIPTION | INPUT TOKEN LIMIT | OUTPUT TOKEN LIMIT |
| :--- | :--- | :--- | :--- |
| `socrates-pro` | 我们功能最强大、最具创造力的模型。专为处理高度复杂、需要深度领域知识和多步推理的任务而设计。是异步深度思考任务的唯一选择。 | 理论无穷 | 理论无穷 |
| `socrates-mini` | 在能力和速度之间实现了卓越的平衡。适用于绝大多数企业级应用，包括复杂的客户服务、内容生成和代码辅助。 | 128,000 tokens | 8,192 tokens |
| `socrates-nano` | 我们最快、成本效益最高的模型。针对高吞吐量和低延迟场景进行了优化，非常适合大规模实时对话、内容分类和摘要等任务。 | 32,768 tokens | 4,096 tokens |

## API Reference

### Chat Completions

**`POST https://api.cotix-ai.dev/v1/chat/completions`**

创建一个模型回复，以完成给定的对话。

#### Request Body

| PARAMETER | TYPE | REQUIRED | DESCRIPTION |
| :--- | :--- | :--- | :--- |
| `model` | string | Required | 要使用的模型的 ID。请参阅[模型](#models)部分以了解可用选项。 |
| `messages` | array | Required | 描述对话的消息列表。更多信息请参阅 OpenAI 的[消息结构指南](https://platform.openai.com/docs/guides/chat/introduction)。 |
| `tools` | array | Optional | 模型可以调用的工具列表。目前支持 `function`, `web_search`, `computer_usage`, 和 `image_generation` 类型。使用工具可以使模型与外部世界交互，从而扩展其能力。 |
| `tool_choice` | string or object | Optional | 控制模型如何选择工具。`"none"` 强制模型仅生成消息。`"auto"` 是默认值，允许模型自行选择。指定 `{"type": "function", "function": {"name": "my_function"}}` 将强制模型调用该函数。 |
| `response_format`| object | Optional | 一个指定模型输出格式的对象。例如，`{"type": "json_object"}` 可以启用 JSON 模式，确保模型返回一个有效的 JSON 对象。 |
| `max_tokens` | integer | Optional | 聊天完成时可生成的最大 token 数。输入和输出 token 的总数受模型上下文窗口限制。 |
| `temperature` | number | Optional | 控制随机性的采样温度，介于 0 和 2.0 之间。较高的值（如 0.8）会使输出更随机，而较低的值（如 0.2）会使其更具确定性。通常建议只修改此参数或 `top_p`，而不是两者都修改。 |
| `top_p` | number | Optional | 一种替代温度采样的方法，称为核采样。模型会考虑概率质量为 `top_p` 的 token 结果。例如，0.1 意味着只考虑构成前 10% 概率质量的 token。 |
| `stream` | boolean | Optional | 如果设置为 `true`，将发送部分消息增量，就像在 ChatGPT 中一样。Token 将在可用时作为 `data: [DONE]` 消息终止的 server-sent events 发送。 |
| `stop` | string or array | Optional | 最多 4 个序列，API 将在生成更多 token 时停止。 |
| `user` | string | Optional | 代表您的最终用户的唯一标识符，这可以帮助我们监控和检测滥用行为。 |
| `confidence_threshold` | number | Optional | 一个介于 0.0 和 1.0 之间的值。如果模型的最终置信度低于此阈值，它将在 `finish_reason` 中返回 `confidence_too_low`，并且可能不会生成完整内容。这是一个安全功能，用于防止低质量或不确定的输出。 |
| `meta_instructions` | object | Optional | （仅限 `socrates-pro` 和 `socrates-mini`）一个定义 Agent 核心行为准则、目标和约束的结构化对象。请参阅下文的[高级指南：构建代理](#building-agents-with-meta_instructions)。 |
| `knowledge_context` | object | Optional | 一个用于注入临时知识的对象，模型会以极高的优先级参考此信息。请参阅[高级指南：上下文持续学习](#in-context-continual-learning)。|
| `url_context` | object | Optional | 允许模型自动提取和理解用户消息中 URL 内容的对象。将其设置为 `{"enabled": true}` 来激活此功能。请参阅[指南：使用 URL 上下文](#using-url-context)获取详细信息。 |

---

## Guides

### Function Calling

Socrates 模型能够检测何时需要调用函数，并智能地生成符合函数签名的 JSON。这是构建能够与外部 API 和服务交互的代理的基础。

**工作流程:**

1.  **定义工具 (Define Tools)**: 在请求的 `tools` 字段中描述您的函数，包括其名称、描述和参数。
2.  **触发调用 (Trigger Call)**: 模型在识别到用户请求需要使用工具时，会返回一个 `tool_calls` 对象，而不是直接回复文本。
3.  **执行代码 (Execute Code)**: 在您的应用程序中，使用模型返回的参数执行您的函数。
4.  **返回结果 (Return Results)**: 将函数的返回值作为一个新的 `tool` 角色的消息追加到会话中，然后再次调用模型，让它根据结果生成最终的人类可读回复。

#### Example: A Multi-Function Weather Agent

这是一个完整的 Python 示例，演示了如何实现一个能查询天气并进行单位转换的代理。

```python
import os
import json
from openai import OpenAI

# 建议使用环境变量来配置 API 密钥和基础 URL
client = OpenAI(
    api_key=os.environ.get("SOCRATES_API_KEY", "YOUR_API_KEY"),
    base_url=os.environ.get("SOCRATES_BASE_URL", "https://api.cotix-ai.dev/v1")
)

# 步骤 3: 在你的代码中定义并执行函数
def get_current_weather(location, unit="celsius"):
    """获取指定地点的当前天气。"""
    weather_data = {
        "tokyo": {"temperature": 10, "unit": "celsius"},
        "san francisco": {"temperature": 72, "unit": "fahrenheit"}
    }
    location_key = location.lower()
    if location_key in weather_data:
        return json.dumps(weather_data[location_key])
    else:
        return json.dumps({"location": location, "temperature": "unknown"})

def convert_temperature(value, from_unit, to_unit):
    """在摄氏度和华氏度之间转换温度。"""
    if from_unit == to_unit:
        return value
    if from_unit == "celsius" and to_unit == "fahrenheit":
        return (value * 9/5) + 32
    if from_unit == "fahrenheit" and to_unit == "celsius":
        return (value - 32) * 5/9
    return value

def run_conversation():
    messages = [{"role": "user", "content": "What's the weather like in San Francisco in Celsius?"}]
    
    # 步骤 1: 定义你的工具
    tools = [
        {
            "type": "function",
            "function": {
                "name": "get_current_weather",
                "description": "Get the current weather in a given location.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {"type": "string", "description": "The city and state, e.g., San Francisco, CA"},
                        "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                    },
                    "required": ["location"],
                },
            },
        },
        {
            "type": "function",
            "function": {
                "name": "convert_temperature",
                "description": "Convert temperature between Celsius and Fahrenheit.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "value": {"type": "number", "description": "The temperature value to convert."},
                        "from_unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                        "to_unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                    },
                    "required": ["value", "from_unit", "to_unit"],
                },
            },
        }
    ]
    
    # 首次调用以确定需要执行的步骤
    response = client.chat.completions.create(
        model="socrates-pro",  # 使用更强的模型进行多步推理
        messages=messages,
        tools=tools,
        tool_choice="auto",
    )
    
    response_message = response.choices[0].message
    messages.append(response_message)
    
    # 步骤 2 & 3 & 4: 循环处理工具调用直到模型给出最终答复
    while response_message.tool_calls:
        tool_calls = response_message.tool_calls
        available_functions = {
            "get_current_weather": get_current_weather,
            "convert_temperature": convert_temperature,
        }
        
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)
            function_response = function_to_call(**function_args)
            
            messages.append(
                {
                    "tool_call_id": tool_call.id,
                    "role": "tool",
                    "name": function_name,
                    "content": function_response,
                }
            )
        
        second_response = client.chat.completions.create(
            model="socrates-pro",
            messages=messages,
            tools=tools,
            tool_choice="auto"
        )
        response_message = second_response.choices[0].message
        messages.append(response_message)
        
    return response_message.content

print(run_conversation())
```

> **注意：并行函数调用**
> `socrates-mini` 和 `socrates-pro` 支持并行调用多个函数。模型可以一次性返回多个 `tool_call` 对象，您的代码应该迭代这个数组并相应地执行所有调用，然后将所有结果一次性返回给模型。

### Built-in Tools: Web Search & Computer Usage

为了简化常见代理任务，Socrates 内置了对 `web_search` 和 `computer_usage` 工具的支持。您无需实现这些工具的后端逻辑；只需在 `tools` 数组中声明它们，模型就会在需要时调用它们，并将执行结果作为上下文来生成下一步操作或最终回复。

```python
response = client.chat.completions.create(
    model="socrates-pro",
    messages=[
        {"role": "user", "content": "What were the top 3 AI news headlines yesterday? Then, create a file named 'news_summary.txt' and write a brief summary of each into it."}
    ],
    tools=[
        {"type": "web_search"},
        {"type": "computer_usage"}
    ],
    tool_choice="auto"
)

# Socrates 将在内部处理搜索和文件创建的工具调用。
# 你会直接收到一个总结了所有操作的最终回复。
print(response.choices[0].message.content)
# Example Output: "I've searched for yesterday's top AI news and created a file 'news_summary.txt' with the summaries."
```

### Image Generation

Socrates 可以通过 `image_generation` 工具创建图像。模型会智能地将用户的自然语言请求转换为适合图像生成模型的详细提示，并返回生成的图像信息。

```python
response = client.chat.completions.create(
    model="socrates-mini",
    messages=[
        {"role": "user", "content": "Generate a photorealistic image of an astronaut riding a majestic white horse on the red plains of Mars, with Earth visible in the sky."}
    ],
    tools=[{"type": "image_generation"}],
    tool_choice="auto"
)

# 响应将包含一个 tool_call。在实际应用中，您需要像处理函数调用一样处理它。
# 为简化示例，这里直接展示最终可能生成的文本和数据。
print(response.choices[0].message.content)
# Expected output might look like this:
# "I have created the image you requested. You can view it here: https://cdn.cotix-ai.dev/images/generated_image_uuid.png"
```

### Vision: Understanding Images

所有 Socrates 模型都具备视觉能力，可以理解图像并回答相关问题。在 `messages` 数组中，您可以使用特定的格式传递图像的 URL 或 Base64 编码数据。

```python
import base64
import requests

api_key = os.environ.get("SOCRATES_API_KEY", "YOUR_API_KEY")

def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

image_path = "path_to_your_image.jpg"
base64_image = encode_image(image_path)

headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}

payload = {
    "model": "socrates-mini",
    "messages": [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What is unusual about this image? Analyze the details and provide a witty comment."},
                {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{base64_image}"}}
            ]
        }
    ],
    "max_tokens": 500
}

response = requests.post("https://api.cotix-ai.dev/v1/chat/completions", headers=headers, json=payload)
print(response.json()['choices'][0]['message']['content'])
```

### Using URL Context

通过启用 `url_context`，您可以让模型直接从用户提供的 URL 中获取信息。当模型在 `messages` 中检测到 URL 时，它会自动访问该链接，读取其内容（如网页文本、PDF 文档），并利用这些信息来回答问题或执行任务。这极大地简化了需要引用在线文档或文章的用例。

**工作原理:**
1.  在您的 API 请求中设置 `url_context: {"enabled": true}`。
2.  在用户的消息中包含一个或多个完整的 URL。
3.  模型将在后台提取 URL 的内容，并将其作为高优先级上下文来生成回复。

```python
# 确保已初始化 client
# client = OpenAI(api_key=..., base_url=...)

response = client.chat.completions.create(
    model="socrates-mini",
    messages=[
        {
            "role": "user",
            "content": "Please summarize the main points of this article: https://www.example.com/blog/ai-in-healthcare-2024"
        }
    ],
    url_context={"enabled": true}  # 启用此功能
)

# 模型将直接返回基于文章内容的摘要
print(response.choices[0].message.content)
# Expected output:
# "The article discusses several key applications of AI in healthcare for 2024. The main points include the use of machine learning for predictive diagnostics, AI-powered drug discovery which significantly speeds up research, and the role of natural language processing in streamlining clinical documentation..."
```

> **注意**：`url_context` 与 `web_search` 工具不同。`web_search` 用于发现信息，而 `url_context` 用于分析用户已经提供的特定链接。

### Structured Output (JSON Mode)

通过设置 `response_format={"type": "json_object"}`，您可以强制模型输出一个语法正确的 JSON 对象。这对于需要结构化数据的应用场景至关重要，例如 API 调用、数据提取或前端组件的动态生成。

```python
response = client.chat.completions.create(
    model="socrates-mini",
    response_format={"type": "json_object"},
    messages=[
        {"role": "system", "content": "You are a helpful assistant designed to extract structured data and output it as a JSON object. Ensure all string values are present."},
        {"role": "user", "content": "Parse the following user profile: 'Name: Jane Doe, Age: 29, City: Berlin. She has two skills: Python and Data Analysis.'"}
    ]
)

# 输出将是一个有效的 JSON 字符串
output_json = json.loads(response.choices[0].message.content)
print(json.dumps(output_json, indent=2))
# Expected output:
# {
#   "name": "Jane Doe",
#   "age": 29,
#   "city": "Berlin",
#   "skills": [
#     "Python",
#     "Data Analysis"
#   ]
# }
```

---

## Advanced Guides

### Asynchronous Deep Thought (`socrates-pro`)

对于需要深度分析、创造性规划或解决复杂问题的任务，`socrates-pro` 提供了异步深度思考模式。这允许您提交一个任务，并在模型完成其详尽的内部推理过程后通过 Webhook 或轮询获取结果。这对于需要几分钟甚至几小时计算时间的任务是理想选择。

#### API Endpoints
*   `POST /v1/thought_tasks`: 创建一个新的深度思考任务。
*   `GET /v1/thought_tasks/{task_id}`: 检索任务的状态和结果。

#### Step 1: Create a Task
```python
import requests
import json

headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}
payload = {
    "model": "socrates-pro",
    "prompt": "Develop a comprehensive market entry strategy for a new line of eco-friendly smart home devices targeting the European market. The report should include a competitive analysis, target audience segmentation, pricing strategy, marketing plan, and a 3-year financial projection. The output should be a structured JSON object.",
    "max_thinking_time_seconds": 10800,  # 3 hours
    "webhook_url": "https://yourapi.com/socrates/webhook"
}
response = requests.post("https://api.cotix-ai.dev/v1/thought_tasks", headers=headers, data=json.dumps(payload))
task_id = response.json()['id']
print(f"Task created with ID: {task_id}")
```

#### Step 2: Retrieve the Result
您可以轮询 `GET /v1/thought_tasks/{task_id}` 端点，或等待发送到您 `webhook_url` 的通知。

```python
import time

while True:
    res = requests.get(f"https://api.cotix-ai.dev/v1/thought_tasks/{task_id}", headers=headers)
    task = res.json()
    if task['status'] == 'completed':
        print("Task completed!")
        report_content = task['result']['content']['json_object']
        print(json.dumps(report_content, indent=2))
        break
    elif task['status'] == 'failed':
        print(f"Task failed: {task['error']['message']}")
        break
    else:
        print(f"Task status: {task['status']}, progress: {task.get('progress', {}).get('percentage', 0)}%")
        time.sleep(300) # Wait 5 minutes
```

### Building Agents with `meta_instructions`

`meta_instructions` 是一个强大的功能，允许您在系统层面指导模型的行为，构建出更可预测、更可靠的代理。它定义了代理的核心人格、使命和行为约束，就像一个不可更改的底层操作系统。

```python
meta_instructions = {
    "persona": "You are a meticulous and cautious research assistant. Your primary goal is accuracy and you must communicate with a professional and objective tone.",
    "mission": "To answer user questions by synthesizing information from provided tools, while explicitly stating any uncertainties or conflicts in the source data.",
    "constraints": [
        "Never state a speculative answer as a fact.",
        "Always cite the source of information using the tool's metadata if available.",
        "If tools do not provide a conclusive answer, state that the information is unavailable rather than guessing.",
        "Do not express personal opinions or emotions."
    ],
    "self_reflection_trigger": {
        "on_event": "contradictory_information_detected",
        "reflection_prompt": "Internal Reflection: I have found conflicting data. Step 1: Identify the sources of conflict. Step 2: Evaluate the credibility of each source based on its metadata. Step 3: Formulate a response that highlights the discrepancy and provides a balanced view."
    }
}

response = client.chat.completions.create(
    model="socrates-pro",
    messages=[{"role": "user", "content": "Is coffee good or bad for your health?"}],
    meta_instructions=meta_instructions,
    tools=[{"type": "web_search"}]
)

# 使用 meta_instructions 后，代理的回答会更加规范、严谨和可控。
# 例如: "Based on a review of current scientific literature, the health effects of coffee are complex. Some studies suggest benefits such as a reduced risk of certain diseases, citing [Source A]. Conversely, other research points to potential negative effects like increased anxiety, citing [Source B]. Therefore, a definitive answer is not available, and effects may vary by individual."
print(response.choices[0].message.content)
```

### In-Context Continual Learning with `knowledge_context`

使用 `knowledge_context` 参数，您可以在单个 API 调用中为模型提供临时的、高优先级的知识。模型会优先使用此上下文中的信息，而不是其内部知识。这对于处理快速变化的信息（如实时数据、用户会话状态）非常有用，而无需进行模型微调。

```python
# 一个金融助理代理
knowledge_context = {
    "facts": [
        {
            "statement": "As of the last market close, the stock price of ACME Corp (ACM) is $150.25.",
            "source": "Real-time Stock API",
            "timestamp": "2024-10-26T16:00:00Z"
        },
        {
            "statement": "ACME Corp's Q3 earnings report will be released next Monday.",
            "source": "Internal Calendar"
        }
    ],
    "override_instructions": [
        "When asked about ACME Corp's stock, use the provided real-time data instead of your general knowledge.",
        "Be aware of upcoming events mentioned in the context."
    ]
}

response = client.chat.completions.create(
    model="socrates-mini",
    messages=[{"role": "user", "content": "What is the current price of ACM stock and is there any news?"}],
    knowledge_context=knowledge_context
)
print(response.choices[0].message.content)
# Expected output: "Based on real-time data from the last market close, the stock price of ACME Corp (ACM) is $150.25. Additionally, please note that their Q3 earnings report is scheduled for release next Monday."
```

### Model Optimization with Fine-Tuning

对于需要深度专业知识或独特品牌声音的特定任务，微调 (Fine-Tuning) 允许您在自己的数据上训练 Socrates 模型。这可以显著提高性能、减少延迟并降低 token 成本，使模型成为您特定领域的专家。

#### Fine-Tuning Process

1.  **准备数据**: 创建一个包含高质量训练样本的 JSONL 文件。每个样本都是一个 `messages` 对话列表，代表一个理想的交互。
2.  **上传文件**: 使用文件 API 将您的数据集上传到我们的服务器。
3.  **创建微调任务**: 启动一个微调任务，指定您的训练文件和基础模型（如 `socrates-mini`）。
4.  **使用模型**: 任务完成后，您将获得一个新的、定制化的模型 ID，您可以在 Chat Completions API 中直接使用它。

我们的微调 API 遵循 OpenAI 的标准，您可以使用他们的 `openai` Python 库来管理整个流程。

```python
# 1. 准备数据 (e.g., medical_bot_training.jsonl)
# {"messages": [{"role": "system", "content": "You are a medical assistant specialized in cardiology."}, {"role": "user", "content": "What is atrial fibrillation?"}, {"role": "assistant", "content": "Atrial fibrillation (AFib) is an irregular and often rapid heart rhythm that can lead to blood clots in the heart..."}]}
# ... (more examples)

# 2. Upload file
try:
    training_file = client.files.create(
      file=open("medical_bot_training.jsonl", "rb"),
      purpose="fine-tune"
    )
    print(f"File uploaded successfully: {training_file.id}")
except Exception as e:
    print(f"Error uploading file: {e}")
    # Handle error

# 3. Create fine-tuning job
try:
    job = client.fine_tuning.jobs.create(
      training_file=training_file.id,
      model="socrates-mini", # 选择微调的基础模型
      suffix="cardio-bot-v1" # 为您的模型添加一个自定义后缀
    )
    print(f"Fine-tuning job created: {job.id}")
except Exception as e:
    print(f"Error creating job: {e}")
    # Handle error

# 4. (稍后) 使用你的微调模型 (任务完成后)
# 你可以在我们的仪表板上监控任务状态
fine_tuned_model_id = "ft:socrates-mini:my-org:cardio-bot-v1:..." # 从仪表板获取完整的模型ID
response = client.chat.completions.create(
    model=fine_tuned_model_id,
    messages=[
        {"role": "system", "content": "You are a medical assistant specialized in cardiology."},
        {"role": "user", "content": "Tell me about treatments for AFib."}
    ]
)
print(response.choices[0].message.content)
```

微调是实现 Socrates 模型在您特定用例中达到最佳性能的最终步骤。请查阅我们完整的[开发者指南](Advanced_Developer_Guides.md)以获取关于数据格式化和最佳实践的详细信息。

## Response Objects

理解 API 返回的结构对于构建健壮的应用程序至关重要。以下是主要端点返回的 JSON 对象示例。

### Chat Completion Object

当 `stream=false` 时，Chat Completions API 返回此对象。

```json
{
  "id": "chatcmpl-9pFN3aJp7fV2fT7f6d4d1e2f3g4h5",
  "object": "chat.completion",
  "created": 1712345678,
  "model": "socrates-mini-0125",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The weather in Tokyo is 10°C."
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 8,
    "total_tokens": 33
  },
  "confidence": null
}
```

*   **`choices.message`**: 包含模型生成的回复。
*   **`choices.message.tool_calls`**: 如果模型决定调用工具，此数组将包含一个或多个 `tool_call` 对象。
    ```json
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_abc123",
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "arguments": "{\"location\":\"Tokyo\",\"unit\":\"celsius\"}"
                }
            }
        ]
    }
    ```
*   **`choices.finish_reason`**: `stop` (自然完成), `length` (达到 `max_tokens`), `tool_calls` (需要调用工具), `content_filter` (内容被过滤), 或 `confidence_too_low` (置信度低于 `confidence_threshold`)。
*   **`usage`**: 本次 API 调用的 token 消耗统计。
*   **`confidence`**: 如果在请求中启用了置信度评分，此字段将包含评分和理由。
    ```json
    "confidence": {
      "score": 0.98,
      "justification": "The information is based on well-established historical facts present in the core training data and cross-verified with multiple internal knowledge sources."
    }
    ```

### Chat Completion Chunk Object (Streaming)

当 `stream=true` 时，API 会返回一系列 `chat.completion.chunk` 对象。

```json
// First chunk
{
  "id": "chatcmpl-9pFOf...",
  "object": "chat.completion.chunk",
  "created": 1712345679,
  "model": "socrates-mini-0125",
  "choices": [
    {
      "index": 0,
      "delta": { "role": "assistant", "content": "" },
      "finish_reason": null
    }
  ]
}

// Subsequent chunks...
{
  "id": "chatcmpl-9pFOf...",
  "object": "chat.completion.chunk",
  "created": 1712345679,
  "model": "socrates-mini-0125",
  "choices": [
    {
      "index": 0,
      "delta": { "content": "The " },
      "finish_reason": null
    }
  ]
}

// Last chunk...
{
  "id": "chatcmpl-9pFOf...",
  "object": "chat.completion.chunk",
  "created": 1712345679,
  "model": "socrates-mini-0125",
  "choices": [
    {
      "index": 0,
      "delta": {},
      "finish_reason": "stop"
    }
  ]
}
```
*   **`choices[0].delta`**: 包含自上次事件以来生成的 token。您需要将这些 `delta.content` 拼接起来以获得完整的消息。

## Rate Limits

为了确保平台的稳定性和公平使用，我们对 API 请求实施了速率限制。限制是根据您的账户等级和组织来设置的，并分为每分钟请求数 (RPM)、每分钟 token 数 (TPM)。

您可以通过检查 API 响应头来了解当前的速率限制状态：

*   `x-ratelimit-limit-requests`: 您在当前时间窗口内允许的总请求数。
*   `x-ratelimit-remaining-requests`: 当前时间窗口内剩余的请求数。
*   `x-ratelimit-reset-requests`: 当前请求数限制重置的剩余时间 (例如 `60s`)。
*   `x-ratelimit-limit-tokens`: 您在当前时间窗口内允许的总 token 数。
*   `x-ratelimit-remaining-tokens`: 当前时间窗口内剩余的 token 数。
*   `x-ratelimit-reset-tokens`: 当前 token 数限制重置的剩余时间 (例如 `60s`)。

如果超出速率限制，您将收到一个 `429 Too Many Requests` 的 HTTP 状态码。我们强烈建议在您的代码中实现带有指数退避（Exponential Backoff）的重试逻辑来优雅地处理这种情况。

#### Example: Exponential Backoff in Python

```python
import time
import random
from openai import OpenAI, RateLimitError

client = OpenAI(api_key="...", base_url="https://api.cotix-ai.dev/v1")

def chat_with_backoff():
    max_retries = 5
    base_delay = 1  # seconds

    for i in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="socrates-nano",
                messages=[{"role": "user", "content": "Hello!"}]
            )
            return response
        except RateLimitError as e:
            if i == max_retries - 1:
                print("Max retries reached. Failing.")
                raise e
            
            # 指数退避逻辑
            delay = base_delay * (2 ** i) + random.uniform(0, 1)
            print(f"Rate limit exceeded. Retrying in {delay:.2f} seconds...")
            time.sleep(delay)

# 调用函数
# chat_with_backoff()
```

## Error Handling

一个健壮的应用程序需要能妥善处理各种 API 错误。Socrates API 使用标准的 HTTP 状态码来指示请求的成功或失败。

| STATUS CODE | ERROR CODE | DESCRIPTION |
| :--- | :--- | :--- |
| **`400 Bad Request`** | `invalid_request_error` | 请求格式错误或包含无效参数（例如，`messages` 格式不正确）。错误消息中会提供更多细节。 |
| **`401 Unauthorized`**| `invalid_api_key` | 您的 API 密钥不正确或已过期。请检查您的密钥和组织凭证。 |
| **`403 Forbidden`**| `permission_denied` | 您无权访问所请求的资源，例如，尝试使用一个您未订阅的模型。|
| **`404 Not Found`** | `model_not_found` | 您请求的模型 ID 不存在或已被弃用。 |
| **`429 Too Many Requests`** | `rate_limit_exceeded` | 您已超出当前的速率限制。请参阅[速率限制](#rate-limits)部分。 |
| **`500 Internal Server Error`** | `server_error` | 我们这边出了问题。这通常是临时性的。请稍后重试。 |
| **`503 Service Unavailable`** | `engine_overloaded` | 我们的服务器当前负载过高。请稍后重试。这通常发生在高峰时段。 |

#### Error Response Body

失败的请求会返回一个包含错误详情的 JSON 对象：

```json
{
  "error": {
    "message": "Invalid 'temperature' value: 3.5. It must be a number between 0 and 2.",
    "type": "invalid_request_error",
    "param": "temperature",
    "code": "invalid_temperature"
  }
}
```

## Security & Best Practices

*   **API 密钥管理**: 绝不要在客户端代码（如浏览器或移动应用）中暴露您的 API 密钥。将它们安全地存储在服务器端，并使用环境变量或密钥管理服务进行管理。定期轮换您的密钥以降低风险。
*   **用户身份验证**: `user` 参数是监控和防止滥用行为的关键。为每个最终用户分配一个稳定、唯一的匿名 ID。不要使用用户的个人身份信息（PII）。
*   **内容审核**: 虽然我们的模型经过了安全对齐，但我们建议对输入和输出进行额外的审核，以符合您应用程序的安全政策和用户条款。
*   **数据隐私**: 我们不会使用通过 API 发送的数据来训练我们的模型，除非您明确选择加入。
  
## SDK & Client Libraries

为了简化与 Socrates API 的集成，我们提供了官方的 SDK，并确保与 OpenAI 的客户端库兼容。我们推荐使用这些库，因为它们能处理认证、请求构建和错误处理等底层细节。

### Python

我们的 API 与 `openai` Python V1.x.x 库完全兼容。

**Installation:**
```bash
pip install openai
```

**Usage:**
```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get("SOCRATES_API_KEY"),
    base_url="https://api.cotix-ai.dev/v1"
)

# ... (使用 client 对象调用 API)
```

### TypeScript / JavaScript

我们的 API 与 `openai` Node.js V4.x.x 库完全兼容。

**Installation:**
```bash
npm install openai
# or
yarn add openai
```

**Usage:**
```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.SOCRATES_API_KEY,
  baseURL: 'https://api.cotix-ai.dev/v1',
});

async function main() {
  const completion = await client.chat.completions.create({
    messages: [{ role: 'user', content: 'Say this is a test' }],
    model: 'socrates-nano',
  });
  console.log(completion.choices[0]);
}

main();
```

对于其他语言，您可以使用任何支持自定义请求头和基础 URL 的 OpenAI 兼容库，或者直接发送 HTTP 请求。

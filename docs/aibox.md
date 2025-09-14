# Cotix AI Box 用户手册 & 开发文档

**Cotix AI Box** 是一款革命性的边缘计算物联网设备，专为在无网络或网络不佳环境下安全、低延迟地运行大型语言模型而设计。它内置了高效的 `Socrates-nano` 模型，将强大的 AI 推理能力从云端带到您的指尖，实现了数据隐私、实时响应和完全离线的自主运行。

## 目录

*   [**产品介绍 (Introduction)**](#introduction)
    *   [核心优势](#core-advantages)
    *   [硬件规格](#hardware-specifications)
*   [**快速开始 (Quick Start)**](#quick-start)
    *   [步骤 1: 设备开箱与连接](#step-1-setup-and-connect)
    *   [步骤 2: 发送你的第一个请求](#step-2-send-your-first-request)
*   [**模型 (Model)**](#model)
*   [**API 参考 (API Reference)**](#api-reference)
    *   [Chat Completions](#chat-completions)
    *   [Request Body](#request-body)
*   [**功能指南 (Guides)**](#guides)
    *   [函数调用 (Function Calling)](#function-calling)
    *   [结构化输出 (JSON Mode)](#structured-output-json-mode)
    *   [视觉能力 (Vision)](#vision-understanding-images)
*   [**高级应用 (Advanced Usage)**](#advanced-usage)
    *   [与外部设备交互](#interacting-with-external-devices)
    *   [离线知识库更新](#offline-knowledge-updates)
*   [**SDK & 客户端库**](#sdk-client-libraries)
    *   [Python](#python)
    *   [TypeScript / JavaScript](#typescript--javascript)
*   [**错误处理 & 状态码**](#error-handling)
*   [**安全与隐私**](#security--privacy)

---

## <a name="introduction"></a>产品介绍 (Introduction)

欢迎使用 Cotix AI Box！这款设备将 Socrates 系列中最敏捷的 `Socrates-nano` 模型部署在高性能的边缘硬件上，为您提供前所未有的本地 AI 体验。无论是智能家居中枢、工业自动化控制、离线数据分析，还是注重隐私的个人助理，Cotix AI Box 都能提供稳定、可靠且无需依赖云端的智能解决方案。

### <a name="core-advantages"></a>核心优势

*   **完全离线运行**: 所有计算均在本地完成，无需连接互联网。确保了最高级别的数据隐私和商业机密安全。
*   **毫秒级延迟**: 摆脱网络延迟，为实时交互和控制应用提供即时响应。
*   **强大的边缘智能**: 具备函数调用、JSON 模式、视觉理解等先进的 LLM 功能，能够执行复杂任务。
*   **低功耗设计**: 经过优化的软硬件一体设计，适合 7x24 小时不间断运行。
*   **标准 API 接口**: 完全兼容 OpenAI API 格式，让您现有的应用程序可以无缝迁移到边缘设备。

### <a name="hardware-specifications"></a>硬件规格

*   **处理器**: 定制化 NPU (神经网络处理单元)，专为 LLM 推理优化
*   **内存**: 8GB LPDDR5
*   **存储**: 128GB 高速 eMMC
*   **连接性**: Wi-Fi (用于初始设置和固件更新), 千兆以太网口, USB 3.0 x2, GPIO 接口
*   **预装模型**: `Socrates-nano` (Edge-Optimized Version)
*   **功耗**: 典型负载 < 15W

## <a name="quick-start"></a>快速开始 (Quick Start)

### <a name="step-1-setup-and-connect"></a>步骤 1: 设备开箱与连接

1.  将 Cotix AI Box 连接到电源。
2.  使用网线将其连接到您的本地局域网 (LAN)。
3.  设备启动后会自动获取一个 IP 地址。您可以通过路由器管理界面查找名为 `cotix-ai-box` 的设备，以确定其 IP 地址 (例如 `192.168.1.108`)。

### <a name="step-2-send-your-first-request"></a>步骤 2: 发送你的第一个请求

设备开箱即用，无需 API Key。您只需将请求指向设备的本地 IP 地址即可。

打开您的终端，使用 `curl` 命令发送一个简单的请求：

```bash
curl http://192.168.1.108:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "socrates-nano",
    "messages": [
      {
        "role": "system",
        "content": "你是一个运行在 Cotix AI Box 上的边缘 AI 助手。"
      },
      {
        "role": "user",
        "content": "你好！介绍一下你自己。"
      }
    ]
  }'
```

您将收到来自 AI Box 的即时回复，证明设备已成功运行。

## <a name="model"></a>模型 (Model)

Cotix AI Box 内置并优化了 `Socrates-nano` 模型，使其在边缘硬件上发挥最佳性能。

| MODEL ID | DESCRIPTION | 输入 Token 限制 | 输出 Token 限制 |
| :--- | :--- | :--- | :--- |
| `socrates-nano` | 速度最快、效率最高的模型。专为高吞吐量、低延迟的边缘计算场景优化，非常适合实时对话、指令控制和自动化任务。 | 32,768 tokens | 4,096 tokens |

> **注意**: `socrates-pro` 和 `socrates-mini` 是云端模型，无法在 Cotix AI Box 上运行。

## <a name="api-reference"></a>API 参考 (API Reference)

### <a name="chat-completions"></a>Chat Completions

**POST** `http://<YOUR_BOX_IP>:8000/v1/chat/completions`

创建一个模型回复来完成给定的对话。

### <a name="request-body"></a>Request Body

| PARAMETER | TYPE | REQUIRED | DESCRIPTION |
| :--- | :--- | :--- | :--- |
| **`model`** | string | **Required** | 必须设置为 `"socrates-nano"`。 |
| **`messages`** | array | **Required** | 描述对话的消息列表。遵循 OpenAI 的消息结构。 |
| **`tools`** | array | Optional | 模型可以调用的本地函数列表。请参阅[函数调用](#function-calling)指南。 |
| **`tool_choice`** | string or object | Optional | 控制模型如何选择工具。`"none"` 强制生成消息，`"auto"` 是默认值。 |
| **`response_format`** | object | Optional | 设置 `{"type": "json_object"}` 以启用 [JSON 模式](#structured-output-json-mode)。 |
| **`max_tokens`** | integer | Optional | 生成回复的最大 token 数。 |
| **`temperature`** | number | Optional | 控制随机性，介于 0 和 2.0 之间。较低的值更具确定性。 |
| **`top_p`** | number | Optional | 核采样参数，通常与 temperature 二选一。 |
| **`stream`** | boolean | Optional | 设置为 `true` 以流式接收回复，实现打字机效果。 |
| **`stop`** | string or array | Optional | 最多 4 个序列，API 将在此处停止生成。 |

> **注意**: 云端特有的参数如 `user`, `confidence_threshold`, `meta_instructions`, `knowledge_context` 在 Cotix AI Box 上不适用。

## <a name="guides"></a>功能指南 (Guides)

### <a name="function-calling"></a>函数调用 (Function Calling)

`socrates-nano` 能够检测何时需要调用您在本地代码中定义的函数。这是将 AI Box 与其他硬件（如智能灯、传感器）或本地服务（如数据库）集成的关键。

**工作流程:**

1.  **定义工具**: 在 API 请求的 `tools` 字段中描述您的函数。
2.  **触发调用**: AI Box 返回一个 `tool_calls` 对象，指示需要调用哪个函数及参数。
3.  **执行代码**: 您的应用程序接收到 `tool_calls` 后，执行相应的本地代码。
4.  **返回结果**: 将函数的执行结果作为 `tool` 角色的消息再次发送给 AI Box，它将根据结果生成最终回复。

*示例代码请参考[云端文档的 Python 示例](...link-to-original-doc...)，只需将 `base_url` 指向您的 AI Box IP 地址。*

### <a name="structured-output-json-mode"></a>结构化输出 (JSON Mode)

通过设置 `response_format={"type": "json_object"}`，您可以强制模型输出一个语法正确的 JSON 对象。这对于设备控制、状态上报和结构化数据提取等物联网场景至关重要。

```python
# Python Example
response = client.chat.completions.create(
    model="socrates-nano",
    response_format={"type": "json_object"},
    messages=[
        {"role": "system", "content": "你是一个智能家居助手，以 JSON 格式输出指令。"},
        {"role": "user", "content": "把客厅的灯调到 80% 亮度，颜色设为暖白色。"}
    ]
)
output = json.loads(response.choices[0].message.content)
# Expected output:
# {
#   "device": "living_room_light",
#   "action": "set_state",
#   "parameters": {
#     "brightness": 0.8,
#     "color": "warm_white"
#   }
# }
```

### <a name="vision-understanding-images"></a>视觉能力 (Vision: Understanding Images)

`socrates-nano` 具备视觉能力。您可以将连接到 AI Box 的 USB 摄像头捕获的图像，或本地存储的图像以 Base64 编码格式发送给模型，进行分析和问答。

*示例请参考云端文档的 Vision 部分，将请求 URL 替换为 `http://<YOUR_BOX_IP>:8000/v1/chat/completions`。*

## <a name="advanced-usage"></a>高级应用 (Advanced Usage)

### <a name="interacting-with-external-devices"></a>与外部设备交互

利用 AI Box 的 GPIO 接口和函数调用功能，您可以构建强大的物理世界代理。例如，定义一个 `set_gpio_pin(pin_number, state)` 函数，然后通过自然语言指令 "打开 12 号引脚的继电器" 来控制连接的硬件。

### <a name="offline-knowledge-updates"></a>离线知识库更新

虽然无法进行在线微调，但您可以通过固件更新的方式，定期为您的 AI Box 推送包含特定领域知识的优化版 `Socrates-nano` 模型。请联系我们的企业支持以获取定制化模型服务。

## <a name="sdk-client-libraries"></a>SDK & 客户端库

我们与 OpenAI 的官方库完全兼容，让集成变得轻而易举。

### <a name="python"></a>Python

**安装:** `pip install openai`

**使用:**

```python
import os
from openai import OpenAI

# 将 base_url 指向你的 AI Box IP
client = OpenAI(
    api_key="not-needed-for-local-box", # 本地运行时 API Key 不是必需的
    base_url="http://192.168.1.108:8000/v1"
)

response = client.chat.completions.create(
    model="socrates-nano",
    messages=[{"role": "user", "content": "在边缘设备上运行感觉如何？"}]
)
print(response.choices[0].message.content)
```

### <a name="typescript--javascript"></a>TypeScript / JavaScript

**安装:** `npm install openai`

**使用:**

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'not-needed-for-local-box', // 本地运行时 API Key 不是必需的
  baseURL: 'http://192.168.1.108:8000/v1',
});

async function main() {
  const completion = await client.chat.completions.create({
    messages: [{ role: 'user', content: 'Say this is a test from the edge!' }],
    model: 'socrates-nano',
  });
  console.log(completion.choices[0]);
}

main();
```

## <a name="error-handling"></a>错误处理 & 状态码

Cotix AI Box 使用标准的 HTTP 状态码。

| STATUS CODE | ERROR CODE | DESCRIPTION |
| :--- | :--- | :--- |
| 400 Bad Request | `invalid_request_error` | 请求格式错误或参数无效。 |
| 404 Not Found | `model_not_found` | 您请求的模型 ID 不正确 (应为 `socrates-nano`)。 |
| 429 Too Many Requests | `rate_limit_exceeded` | Box 的处理能力已达上限。请降低请求频率。 |
| 500 Internal Server Error | `server_error` | 设备内部发生未知错误。请检查设备日志或尝试重启。 |
| 503 Service Unavailable | `engine_overloaded` | 模型推理引擎当前负载过高，请稍后重试。 |

## <a name="security--privacy"></a>安全与隐私

**Cotix AI Box 的核心设计理念是隐私优先。**

*   **数据不出本地**: 您的所有请求和数据都只在设备内部处理，永远不会发送到任何云服务器。
*   **物理安全**: 将设备部署在您信任的物理环境中，以确保最高级别的安全。
*   **网络隔离**: 您可以将 AI Box 部署在完全隔离的内部网络中，进一步杜绝任何潜在的外部访问风险。

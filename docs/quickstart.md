# 快速入门指南

欢迎来到 Socrates！本指南将带您在 **5分钟内** 完成您的第一个 API 调用，并逐步探索 Socrates API 独有的强大功能，包括如何构建更可靠、无幻觉的应用，以及如何利用我们旗舰级的“深度思考”模式解决复杂问题。

## 准备工作

在开始之前，您需要准备两样东西：

1.  **Socrates API 密钥**: 您可以在您的账户仪表板中找到并创建您的 API 密钥。
2.  **Python 环境**: 确保您的电脑已安装 Python 3.7+。

---

### 第1步: 安装 SDK 并配置密钥

我们将使用与 OpenAI V1.x 兼容的官方 Python 库来与 Socrates API 进行交互。打开您的终端并运行以下命令：

```bash
pip install openai
```

安装完成后，最佳实践是通过环境变量来配置您的 API 密钥，这样可以避免将密钥硬编码到您的代码中。

=== "Linux / macOS"
    ```bash
    export SOCRATES_API_KEY="sk-..."
    ```

=== "Windows (CMD)"
    ```bash
    set SOCRATES_API_KEY="sk-..."
    ```

=== "Windows (PowerShell)"
    ```powershell
    $Env:SOCRATES_API_KEY="sk-..."
    ```

!!! warning "保护您的密钥"
    您的 API 密钥是机密的。请勿在任何客户端代码或公共代码库（如 GitHub）中暴露它。

---

### 第2步: 您的第一个 API 调用

让我们从一个基础的聊天补全（Chat Completion）请求开始。我们将使用 `socrates-mini` 模型，因为它在性能、速度和成本之间取得了绝佳的平衡。

创建一个名为 `hello_socrates.py` 的文件，并粘贴以下代码：

```python
import os
from openai import OpenAI

# 初始化 Socrates 客户端
# SDK 会自动从环境变量读取 SOCRATES_API_KEY
client = OpenAI(
    api_key=os.environ.get("SOCRATES_API_KEY"),
    base_url="https://api.cenvolink.dev/v1"
)

try:
    response = client.chat.completions.create(
        model="socrates-mini",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "用一句话解释什么是大型语言模型。"}
        ]
    )
    print(response.choices[0].message.content)

except Exception as e:
    print(f"发生错误: {e}")

```

现在，运行这个文件：

```bash
python hello_socrates.py
```

您应该会看到类似下面的输出，简洁地解释了大型语言模型：

> 大型语言模型是一种经过海量文本数据训练的人工智能模型，能够理解和生成类似人类的自然语言。

恭喜！您已经成功完成了您的第一个 Socrates API 调用。但这仅仅是开始。

---

### 第3步: 构建更可靠的应用 (无幻觉特性)

AI 的“幻觉”（即生成不准确的信息）是构建可信赖应用的主要障碍。Socrates API 提供了独特的工具来主动对抗幻觉。

让我们来体验 **绝对置信度评分（Absolute Confidence Score）**。我们将向模型询问一个它可能知道但不完全确定的事实，并要求它评估自己答案的可信度。

修改您的代码如下：

```python
# ... (前面的 client 初始化代码保持不变) ...
import json

try:
    response = client.chat.completions.create(
        model="socrates-pro", # 使用更强大的模型以获得更准确的置信度评估
        messages=[
            {"role": "user", "content": "请问亚历山大大帝的爱马叫什么名字，它死于哪场战役？"}
        ],
        # 使用 extra_body 传递 Socrates 的独有参数
        extra_body={
            "response_options": {
                "include_confidence": True
            }
        }
    )

    # 分别获取回答内容和置信度数据
    answer = response.choices[0].message.content
    confidence_data = response.choices[0].confidence # 这是 Socrates 的特有字段

    print("--- 回答内容 ---")
    print(answer)
    print("\n--- 模型置信度评估 ---")
    print(f"置信度分数: {confidence_data.score:.2f}")
    print(f"评估理由: {confidence_data.justification}")

    # 在您的应用中，您可以根据分数来决定下一步操作
    if confidence_data.score < 0.9:
        print("\n[应用提示]：模型对此回答的置信度不是最高，建议在关键应用中进行核实。")

except Exception as e:
    print(f"发生错误: {e}")
```

运行这段代码，您将看到模型不仅给出了答案（马的名字是“布西发拉斯”，它死于海达斯佩斯河战役），还会给出一个高分（例如 0.98）和理由，因为它对此历史事实非常确定。

现在，尝试问一个更具推测性的问题，比如 `“2030年最主流的编程语言会是什么？”`，然后观察置信度分数的变化。这就是构建无幻觉应用的第一步：**让模型“知其所不知”**。

---

### 第4步: 解决真正复杂的问题 (深度思考)

对于需要数分钟甚至数小时进行深度分析、规划或创造的任务（例如，撰写一份完整的市场分析报告），传统的 API 调用会因为超时而失败。为此，我们创造了 `socrates-pro` 的 **异步深度思考（Asynchronous Deep Thought）** 模式。

您只需提交一个任务，我们的平台会在后台进行类似蒙特卡洛树搜索的深度推理，完成后再通知您或让您来获取结果。

!!! warning "注意"
    此功能为异步调用，以下代码将演示如何 **提交任务** 和 **轮询结果**。在生产环境中，我们强烈建议使用 **Webhook** 来接收完成通知。

这是一个使用 `requests` 库的示例，演示如何提交一个任务并获取结果：

```python
import os
import requests
import json
import time

SOCRATES_API_KEY = os.environ.get("SOCRATES_API_KEY")
API_BASE_URL = "https://api.cenvolink.dev/v1"

headers = {
    "Authorization": f"Bearer {SOCRATES_API_KEY}",
    "Content-Type": "application/json"
}

# 1. 提交一个需要深度思考的复杂任务
print("--- 步骤 1: 提交深度思考任务 ---")
task_payload = {
    "model": "socrates-pro",
    "prompt": "为一家传统书店制定一个新颖的、三管齐下的战略，使其在亚马逊和电子书时代不仅能生存下来，还能蓬勃发展。报告应包含具体、可行的步骤。",
    "max_thinking_time_seconds": 300,  # 允许思考5分钟 (生产中可设置更长)
}

try:
    # 提交任务
    submit_response = requests.post(
        f"{API_BASE_URL}/thought_tasks",
        headers=headers,
        data=json.dumps(task_payload)
    )
    submit_response.raise_for_status() # 如果请求失败则抛出异常
    task = submit_response.json()
    task_id = task['id']
    print(f"任务已成功提交，ID 为: {task_id}")

    # 2. 轮询任务状态直到完成
    print("\n--- 步骤 2: 等待任务完成 (将每30秒检查一次) ---")
    while True:
        status_response = requests.get(
            f"{API_BASE_URL}/thought_tasks/{task_id}",
            headers=headers
        )
        status_response.raise_for_status()
        task_status = status_response.json()

        status = task_status.get('status')
        progress = task_status.get('progress', {}).get('percentage', 0)
        
        if status == 'completed':
            print("\n--- 任务完成！---")
            result = task_status.get('result', {}).get('content', {}).get('markdown', '没有返回结果。')
            print("\n--- 最终报告 ---")
            print(result)
            break
        elif status == 'failed':
            print(f"\n任务失败: {task_status.get('error', {}).get('message', '未知错误')}")
            break
        else:
            print(f"当前状态: {status}, 进度: {progress:.1f}% ...")
            time.sleep(30) # 等待30秒后再次检查

except requests.exceptions.RequestException as e:
    print(f"API 请求发生错误: {e}")

```

运行此脚本，您将看到任务被提交，然后脚本会定期检查状态。几分钟后，当任务完成时，它将打印出一份由 `socrates-pro` 深度思考后生成的、结构完整的战略报告。

## 下一步

您已经掌握了 Socrates API 的基础和核心优势。现在，您可以：

*   深入阅读我们的 [**无幻觉指南**](Hallucination.md)，学习更多构建可信赖 AI 的高级策略。
*   探索我们的 [**API 参考**](README.md) 文档，了解所有可用的参数和功能。
*   尝试我们的 [**微调指南**](Advanced_Developer_Guides.md)，创建专属于您业务领域的专家模型。

我们很高兴能与您一同构建智能的未来。如果您有任何问题，欢迎加入我们的开发者社区！

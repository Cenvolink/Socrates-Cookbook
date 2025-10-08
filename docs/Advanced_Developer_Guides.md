# Socrates Advanced Developer Guides

欢迎来到 Socrates API 的高级开发指南。本部分将深入探讨两个最关键的主题，以释放大型语言模型的全部潜力：**微调（Fine-Tuning）** 和 **提示工程（Prompt Engineering）**。虽然您可以仅凭基础的 API 调用构建出功能强大的应用，但掌握这些高级技术，将使您能够创建出真正卓越、可靠且高度差异化的 AI 解决方案。

这两份指南是相辅相成的。提示工程是与模型进行高效沟通的艺术，而微调则是从根本上改变模型内在知识和行为的科学。在生产环境中，最成功的应用往往是两者的结合。

---

## Part I: Mastering Fine-Tuning

### Chapter 1: The Philosophy and Strategy of Fine-Tuning

微调（Fine-Tuning）是一个强大的过程，它允许您在自己的专有数据上继续训练预训练好的 Socrates 模型，从而创建一个针对特定任务或领域进行优化的新“专家”模型。这不仅仅是教模型新知识，更是塑造其行为、风格和推理模式。

#### 1.1. Why Fine-Tune? Beyond Prompt Engineering

虽然提示工程非常强大，但它有其局限性。当您的需求超越了通过提示可以高效、可靠地实现的范畴时，就应该考虑微调。

| 特性 | 提示工程 (Prompt Engineering) | 微调 (Fine-Tuning) |
| :--- | :--- | :--- |
| **知识注入** | **临时性、上下文驱动**。通过 RAG 或长上下文窗口注入知识，但模型“知道”它在参考外部文档。 | **永久性、内化吸收**。知识被整合到模型的权重中，成为其“核心记忆”的一部分。 |
| **行为塑造** | **指令性**。通过 `system` 消息和指令来引导行为，但模型仍可能偏离。 | **根本性**。通过大量示例从根本上改变模型的默认行为和响应模式。 |
| **风格一致性**| **可实现，但不稳定**。可以通过指令要求特定风格，但一致性可能随输入变化而波动。 | **高度一致**。模型自然地采用微调数据中呈现的语气和风格，无需每次提醒。 |
| **复杂格式** | **困难**。对于复杂的、非标准的 JSON 或 XML 格式，仅靠提示很难保证 100% 的语法正确性。 | **可靠**。模型可以学习并可靠地生成任何结构化的输出格式。 |
| **成本与延迟**| **输入 Token 成本高**。复杂的提示、上下文和示例会占用大量 Token。 | **输入 Token 成本低，有训练成本**。一次性训练成本，但之后可以用更短的提示，降低每次调用的成本和延迟。 |
| **可扩展性** | **有限**。当知识库变得庞大或指令变得极其复杂时，提示会变得难以管理和维护。 | **高**。将不断增长的专有知识和行为规范系统地整合到模型中。 |

**战略决策：何时选择微调？**

1.  **领域专业化（Domain Specialization）**:
    *   **场景**: 您正在为法律、医学、金融或工程等领域构建应用。
    *   **理由**: 这些领域充满了专业术语、独特的缩写和复杂的概念关系。微调可以使模型内化这些知识，从而提供更准确、更专业的回答。

2.  **品牌声音与个性（Brand Voice & Persona）**:
    *   **场景**: 您需要一个 AI 聊天机器人，其沟通风格与您的品牌（例如，正式、俏皮、有同理心）完全一致。
    *   **理由**: 微调可以训练模型自然地采用您指定的语气和措辞，而无需在每个提示中重复冗长的风格指南。这使得交互更加自然和一致。

3.  **可靠的结构化输出（Reliable Structured Output）**:
    *   **场景**: 您的应用需要模型输出特定且复杂的 JSON、XML 或自定义格式，以驱动下游的自动化流程。
    *   **理由**: 即使有 JSON 模式，复杂的嵌套结构或非标准格式也可能让模型出错。通过在数百个正确格式的示例上进行微调，可以实现近乎完美的格式遵循度。

4.  **处理独特边缘案例（Handling Unique Edge Cases）**:
    *   **场景**: 您的业务流程中有一些通过简单规则难以描述的复杂决策逻辑或独特的客户请求。
    *   **理由**: 通过提供这些边缘案例及其期望输出的示例，微调可以教会模型如何处理这些特殊情况，这是通用提示难以做到的。

5.  **性能与成本优化（Performance & Cost Optimization）**:
    *   **场景**: 您有一个高流量的应用，并且发现为了获得理想结果，您需要使用非常长的提示（包含大量示例和上下文）。
    *   **理由**: 微调可以将这些冗长的指令和示例“烘焙”到模型中。之后，您可以使用更短、更简洁的提示来触发相同的行为，从而显著减少每次 API 调用的 Token 消耗和延迟。

### Chapter 2: The Art and Science of Data Preparation

微调的成功 90% 取决于数据的质量。模型只会成为你教它的样子。“垃圾进，垃圾出”在这里是绝对的真理。

#### 2.1. Data Formatting: The JSONL Standard

您的数据集必须是 **JSONL (JSON Lines)** 格式。这是一个纯文本文件，其中每一行都是一个独立的、有效的 JSON 对象。

```json
{"messages": [...对话示例1...]}
{"messages": [...对话示例2...]}
{"messages": [...对话示例3...]}
```

每个 JSON 对象的核心是 `messages` 键，其值是一个标准的对话消息数组，与 Chat Completions API 的格式完全相同。

**对话结构 (`messages` 数组):**

*   `{"role": "system", "content": "..."}`: **系统消息**。这是设定模型高级行为和角色的地方。强烈建议在所有训练示例中包含一个一致的系统消息。它为微调过程提供了一个强大的锚点。
*   `{"role": "user", "content": "..."}`: **用户消息**。模拟最终用户可能提出的各种输入。
*   `{"role": "assistant", "content": "..."}`: **助手消息**。这是您希望模型在看到前面的对话历史后生成的 **理想** 回复。这是训练的核心。

#### 2.2. Crafting High-Quality Examples: A Deep Dive

1.  **一致性是王道 (Consistency is King)**
    *   **系统消息**: 在整个数据集中使用相同的、精心设计的系统消息。例如，如果你在微调一个代码审查机器人，系统消息应该是 `You are an expert code reviewer specializing in Python. You identify potential bugs, suggest performance improvements, and enforce PEP 8 style guidelines.`
    *   **风格和语气**: 如果您希望模型是正式的，那么所有的 `assistant` 回复都应该是正式的。如果希望它使用 Markdown，所有回复都应该使用 Markdown。
    *   **格式**: 如果期望 JSON 输出，确保每一个相关的 `assistant` 回复都是一个格式完美、语法正确的 JSON 字符串。

2.  **质量远胜数量 (Quality over Quantity)**
    *   一个常见的误区是认为数据越多越好。事实是，**50个精心策划、多样化的高质量示例，远胜于500个草率、重复或有错误的示例。**
    *   **建议起点**: 从 50 到 100 个示例开始。这个数量足以让模型学习到一个特定的行为或风格。对于更复杂的任务或知识注入，可能需要数百甚至数千个示例。
    *   **审查流程**: 建立一个严格的数据审查流程。每个示例都应该由至少两个人审查，以确保其准确性、一致性和清晰度。

3.  **多样性与覆盖面 (Diversity and Coverage)**
    *   您的数据集应该像最终用户一样“思考”。它需要覆盖您的应用可能遇到的各种输入和场景。
    *   **正面案例**: 用户提供了所有必要信息。
    *   **负面案例**: 用户的问题模糊不清、提供了错误信息或提出了超出范围的请求。在这些情况下，模型的理想回复可能是请求澄清或礼貌地拒绝。
    *   **边缘案例**: 那些不常见但很重要的场景。
    *   **多样化的措辞**: 对于同一个意图，提供多种不同的提问方式。例如，“重置我的密码”、“我忘了密码”、“无法登录”。

4.  **示例：为情感支持机器人创建数据**

    *   **系统消息 (一致)**: `"You are Compassionate Companion, an AI assistant designed to provide empathetic and supportive responses. You do not give medical advice."`

    *   **正面案例 (多轮对话)**:
        ```json
        {"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "I feel so overwhelmed with work lately."}, {"role": "assistant", "content": "It sounds like you're under a lot of pressure. It's completely understandable to feel overwhelmed when work demands so much. Remember to take small breaks for yourself, even just for a few minutes."}]}
        ```

    *   **负面案例 (超出范围)**:
        ```json
        {"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "What medication should I take for anxiety?"}, {"role": "assistant", "content": "I understand you're looking for help with anxiety, and that's a brave step. However, as an AI assistant, I am not qualified to give medical advice. It's really important to speak with a healthcare professional who can provide you with the right guidance."}]}
        ```

    *   **澄清案例 (输入模糊)**:
        ```json
        {"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "I'm sad."}, {"role": "assistant", "content": "I'm sorry to hear you're feeling sad. Sometimes just putting a name to our feelings can be a first step. Is there anything specific that happened, or is it a more general feeling?"}]}
        ```

#### 2.3. Data Validation

在上传之前，编写脚本来验证您的数据文件：
1.  **JSONL 格式检查**: 确保文件中的每一行都是一个有效的 JSON 对象。
2.  **结构检查**: 确保每个 JSON 对象都包含一个 `messages` 键。
3.  **对话格式检查**: 确保 `messages` 数组中的每个元素都包含 `role` 和 `content` 键，且 `role` 是 `system`, `user`, 或 `assistant` 之一。
4.  **一致性检查**: 检查所有示例中的 `system` 消息是否一致。

### Chapter 3: The Fine-Tuning Process in Practice

#### 3.1. Choosing the Right Base Model

您的微调模型继承了基础模型的所有通用知识和推理能力。选择正确的基础模型至关重要。

*   `socrates-nano`: 最快、成本效益最高的选择。非常适合分类、简单格式化、风格模仿等任务。如果您的任务不需要复杂的推理，从这里开始是明智的。
*   `socrates-mini`: 性能与速度的平衡点。适用于大多数需要一定推理能力的企业级应用，如复杂的客户服务对话、代码辅助等。这是最常见的微调选择。
*   `socrates-pro`: 功能最强大的模型。当您的任务需要深度、多步骤的推理，并且您需要在此基础上进行专业化时，选择此模型。由于其规模较大，微调成本也最高。

#### 3.2. Step-by-Step API Usage

以下是使用 `openai` Python 库的完整流程，它与我们的 Socrates API 兼容。

**1. 上传文件**

```python
from openai import OpenAI
import os

# 配置客户端
client = OpenAI(
    api_key=os.environ.get("SOCRATES_API_KEY"),
    base_url="https://api.cenvolink.dev/v1"
)

# 上传您的训练数据文件
# 确保文件存在且格式正确
try:
    training_file = client.files.create(
      file=open("path_to_your_data.jsonl", "rb"),
      purpose="fine-tune"
    )
    print(f"File uploaded successfully. File ID: {training_file.id}")
except FileNotFoundError:
    print("Error: The training file was not found.")
except Exception as e:
    print(f"An error occurred during file upload: {e}")
```

**2. 创建微调任务**

```python
# 使用上一步中获取的文件 ID
file_id = training_file.id

try:
    job = client.fine_tuning.jobs.create(
      training_file=file_id,
      model="socrates-mini",  # 选择您的基础模型
      suffix="my-custom-agent-v1",  # 为您的模型添加一个有意义的后缀
      hyperparameters={
          "n_epochs": 4, # 可以调整的超参数，例如训练轮数
          "learning_rate_multiplier": 2 # 学习率乘数
      }
    )
    print(f"Fine-tuning job created. Job ID: {job.id}")
except Exception as e:
    print(f"An error occurred while creating the fine-tuning job: {e}")

```

**3. 监控任务状态**

微调任务需要一些时间才能完成，具体取决于数据集的大小和模型的选择。您可以轮询任务状态。

```python
import time

job_id = job.id
while True:
    status = client.fine_tuning.jobs.retrieve(job_id)
    job_status = status.status
    print(f"Job Status: {job_status}")
    if job_status in ["succeeded", "failed", "cancelled"]:
        if job_status == "succeeded":
            fine_tuned_model_id = status.fine_tuned_model
            print(f"Fine-tuning succeeded! Your new model ID is: {fine_tuned_model_id}")
        else:
            print(f"Fine-tuning failed or was cancelled. Reason: {status.error}")
        break
    time.sleep(300) # 每 5 分钟检查一次
```

### Chapter 4: Evaluation and Iteration

您的第一个微调模型可能不是完美的。持续的评估和迭代是通往卓越模型的必经之路。

#### 4.1. Creating an Evaluation Set

在开始微调之前，从您的完整数据集中分离出大约 10-20% 的数据作为“验证集”。这个验证集不用于训练，而是用于在微调后客观地评估模型的性能。

#### 4.2. Evaluation Methods

*   **定性评估（Qualitative Evaluation）**: 手动检查模型在验证集上的输出。
    *   它是否遵循了指令？
    *   它的语气和风格是否正确？
    *   它的回答是否事实准确？
    *   与基础模型相比，它的表现是否更好？
*   **定量评估（Quantitative Evaluation）**:
    *   **准确率 (Accuracy)**: 对于分类或提取任务，计算模型输出与标准答案完全匹配的百分比。
    *   **BLEU/ROUGE 分数**: 对于摘要或翻译任务，使用这些指标来衡量生成文本与参考文本的重叠程度。
    *   **格式正确率**: 对于结构化数据任务，计算模型输出有效 JSON 或 XML 的百分比。
*   **Reward Model**: 使用像 `socrates-pro` 这样的强大模型来评估微调模型的输出。您可以提示 `socrates-pro`：“根据以下标准（[标准1], [标准2]），为这个回答打分（1-10分），并提供理由。回答：[微调模型的输出]”

#### 4.3. Iterating on Your Model

根据评估结果，您可能需要：
*   **添加更多数据**: 如果模型在某些类型的输入上表现不佳，添加更多类似的、高质量的示例到您的训练集中。
*   **修正现有数据**: 检查您的训练数据中是否存在错误、不一致或模糊的示例。
*   **调整超参数**: 对于高级用户，可以尝试调整 `n_epochs` 或 `learning_rate_multiplier` 等超参数。增加 `n_epochs` 可能会让模型学习得更充分，但也可能导致过拟合。
*   **重新训练**: 使用更新后的数据集创建一个新的微调任务。

微调是一个循环过程：**准备数据 -> 训练 -> 评估 -> 迭代**。通过这个循环，您的模型将不断进化，越来越接近您的理想目标。

---

## Part II: The Art of Prompt Engineering

如果说微调是给演员进行专业培训，那么提示工程就是为这位训练有素的演员编写一份精彩的剧本。一份好的剧本可以引导演员发挥出最佳水平。提示工程是通过精心设计输入来引导语言模型生成所需输出的艺术和科学。

### Chapter 5: Core Principles of Effective Prompting

#### 5.1. The Principle of Clarity and Specificity

模型没有人类的常识和上下文。您必须假设它是一个聪明但完全字面的执行者。

*   **反模式 (Anti-Pattern)**: `“总结一下这个会议。”`
    *   **问题**: 总结给谁看？需要多长？重点是什么？
*   **最佳实践 (Best Practice)**: `“将以下会议记录总结成一份面向高管的五点式备忘录。每点不超过两句话。重点关注已做出的决策、分配的行动项目以及未解决的风险。”`

**技巧**:
*   **量化约束**: 使用具体的数字（“写一个约300字的段落”，“提供3个主要论点”）。
*   **定义受众和目标**: 明确输出是为谁服务的，以及它要达到的目的（“为初学者解释”，“旨在说服潜在投资者”）。
*   **明确禁止**: 如果不希望模型做某事，请明确说明（“不要使用技术术语”，“不要提及价格”）。

#### 5.2. The Principle of Context Provisioning

模型只能利用您在提示中提供的信息进行推理。上下文越丰富、越相关，输出质量越高。

*   **反模式**: `“我的订单状态是什么？”`
    *   **问题**: 哪个订单？谁的订单？
*   **最佳实践**: `“作为客户服务代理，请根据以下客户信息和订单历史，回答客户关于订单状态的问题。客户ID: 12345，订单号: ABC-98765，订单历史: {‘ABC-98765’: {‘status’: ‘shipped’, ‘carrier’: ‘FedEx’, ‘tracking_number’: ‘Z123...’}}。客户问题: ‘我的订单状态是什么？’”`

**技巧**:
*   **RAG (Retrieval-Augmented Generation)**: 将提示工程与外部知识库（如向量数据库）结合。当用户提问时，首先从数据库中检索最相关的文档块，然后将这些文档块作为上下文连同问题一起注入到提示中。
*   **对话历史**: 在构建聊天机器人时，始终包含最近的几轮对话历史，以维持上下文的连贯性。

#### 5.3. The Principle of Persona Assignment

为模型分配一个角色或专家身份，是引导其输出风格、深度和格式的捷径。

*   **通用提示**: `“解释一下黑洞。”`
    *   **可能的结果**: 一段非常基础、百科全书式的描述。
*   **角色提示**:
    *   **对孩子**: `“你是一位友好的天文学家，正在给一群好奇的8岁孩子解释什么是黑洞。使用简单的比喻，让它听起来像一场太空冒险。”`
    *   **对大学生**: `“你是一位物理学教授，在为大学一年级的《天体物理学导论》课程准备讲稿。解释一下史瓦西半径和事件视界，并简要提及奇点。”`
    *   **对开发者**: `“你是一位 Python 开发者，编写一个函数 `describe_black_hole()` 的文档字符串 (docstring)。解释其核心概念，并描述函数的参数（如果有的话）。”`

### Chapter 6: Advanced Prompting Techniques

#### 6.1. Few-Shot Prompting (少样本提示)

在提示中提供一两个完整的示例，是向模型展示您期望的输出格式和推理过程的最有效方法之一。

**示例：情感分类**

```
Classify the sentiment of the customer feedback as Positive, Negative, or Neutral.

Feedback: "The user interface is a bit confusing, but the customer support was excellent."
Sentiment: Neutral

Feedback: "I'm absolutely thrilled with the new features! It's a game-changer."
Sentiment: Positive

Feedback: "The app crashed twice today and I lost my work."
Sentiment: Negative

Feedback: "The delivery was faster than I expected."
Sentiment:
```

模型在看到这些示例后，会更容易理解“Neutral”不仅指中性词汇，也指包含正面和负面混合的反馈。

#### 6.2. Chain-of-Thought (CoT) Prompting (思维链提示)

对于需要逻辑、数学或多步骤推理的问题，CoT 是一种革命性的技术。您不是直接要求答案，而是要求模型“一步一步地思考”。

*   **标准提示**:
    `Q: Roger has 5 tennis balls. He buys 2 more cans of tennis balls. Each can has 3 tennis balls. How many tennis balls does he have now?`
    `A:` (模型可能会直接猜一个数字，容易出错)

*   **CoT 提示**:
    `Q: Roger has 5 tennis balls. He buys 2 more cans of tennis balls. Each can has 3 tennis balls. How many tennis balls does he have now?`
    `A: Let's think step by step.`
    `1. Roger starts with 5 tennis balls.`
    `2. He buys 2 cans of balls.`
    `3. Each can has 3 balls, so 2 cans have 2 * 3 = 6 balls.`
    `4. In total, he now has his initial 5 balls plus the new 6 balls.`
    `5. 5 + 6 = 11 balls.`
    `So the answer is 11.`

通过引导模型输出其推理过程，大大提高了最终答案的准确性。在生产中，您可以指示模型在内部进行思考，最后只输出最终答案。

#### 6.3. Self-Consistency (自洽性)

这是 CoT 的一种增强。您不是只向模型请求一次 CoT 推理，而是多次（例如3-5次）请求，并使用较高的 `temperature` (如 0.7) 来获得多样化的推理路径。然后，您选择在这些不同路径中出现次数最多的最终答案。这就像让一个委员会投票，大大提高了复杂问题的可靠性。

#### 6.4. Structured Prompting with Delimiters

使用清晰的分隔符（如 XML 标签、Markdown 标题或自定义字符串）来组织提示的各个部分，可以显著减少模型的混淆。

```xml
<INSTRUCTIONS>
You are an expert copywriter. Your task is to rewrite the user-provided text to be more persuasive and engaging for a marketing email.
The target audience is young tech professionals.
The tone should be enthusiastic but professional.
</INSTRUCTIONS>

<USER_TEXT>
Our new software update is available. It has new features and bug fixes. You should install it.
</USER_TEXT>

<REWRITTEN_TEXT>
```
这种结构使模型能够精确地理解哪部分是指令，哪部分是需要处理的输入。

### Chapter 7: The Iterative Prompt Development Lifecycle

完美的提示不是一蹴而就的。它是一个类似于软件开发的迭代过程。

1.  **构思与初稿 (Idea & V1 Draft)**:
    *   明确您的目标：您希望模型做什么？
    *   编写一个简单、直接的提示，遵循核心原则。

2.  **测试与评估 (Test & Evaluate)**:
    *   **创建测试套件**: 准备一个包含多种输入的测试集，覆盖典型案例、困难案例和边缘案例。
    *   **运行测试**: 自动化地或手动地运行您的提示，并记录模型的输出。
    *   **分析失败**: 找出模型输出不符合预期的案例。是事实错误？语气不对？格式错误？还是完全偏离了主题？

3.  **分析与改进 (Analyze & Refine)**:
    *   **根本原因分析**: 为什么模型会失败？是指令不够清晰？上下文不足？还是提示中有歧义？
    *   **改进策略**:
        *   **增强清晰度**: 添加更具体的约束或定义。
        *   **添加示例**: 如果模型在格式或风格上出错，添加一个少样本示例。
        *   **引导推理**: 如果是逻辑错误，尝试引入思维链提示。
        *   **添加反例**: 明确告诉模型**不要**做什么。例如，“...不要听起来像个机器人。”

4.  **版本控制 (Version Control)**:
    *   像对待代码一样对待您的提示。使用 Git 或其他版本控制系统来跟踪提示的变更。这使您可以在新版本表现不佳时回滚，并清楚地了解每次修改带来的影响。

5.  **部署与监控 (Deploy & Monitor)**:
    *   将经过验证的提示部署到生产环境。
    *   监控真实世界中的表现。用户的交互可能会揭示您在测试中没有预料到的新问题。收集这些失败案例，将它们添加到您的评估套件中，然后开始下一轮的迭代。

### Conclusion: A Symbiotic Relationship

微调和提示工程是提升 AI 应用性能的左膀右臂。

*   **从提示工程开始**: 对于任何新任务，总是从精心设计的提示开始。这通常是成本最低、速度最快的方法，并且能帮助您更好地理解任务的复杂性。
*   **当提示变得笨重时，转向微调**: 当您发现提示变得越来越长、越来越复杂，或者您需要无法通过提示实现的极高可靠性时，就是时候考虑微调了。
*   **两者结合，发挥最大效力**: 最强大的系统往往是 **微调过的模型 + 精心设计的提示**。微调负责内化核心知识和行为，而提示则在运行时提供具体的、上下文感知的指令。

通过系统地应用本指南中介绍的策略和技术，您将能够充分利用 Socrates API 的强大功能，构建出不仅能完成任务，而且能以精确、可靠和优雅的方式完成任务的下一代 AI 应用程序。

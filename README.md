# Socrates Cookbook

<p align="center">
  欢迎来到 Socrates AI 官方文档的源文件仓库！
  <br>
  所有发布在 <a href="https://cotix-ai.github.io/Socrates-Cookbook/">https://cotix-ai.github.io/Socrates-Cookbook/</a> 上的内容都在这里进行维护。
  <br>
  <br>
  <a href="https://github.com/Cotix-AI/Socrates-Cookbook/actions/workflows/deploy-docs.yml">
    <img src="https://github.com/Cotix-AI/Socrates-Cookbook/actions/workflows/deploy-docs.yml/badge.svg" alt="Build Status">
  </a>
  <a href="https://cotix-ai.github.io/Socrates-Cookbook/">
    <img src="https://img.shields.io/badge/docs-latest-blue.svg" alt="Documentation">
  </a>
</p>

---

## 📖 关于此仓库

这个仓库包含了 [Socrates AI 官方文档网站](https://cotix-ai.github.io/Socrates-Cookbook/) 的所有 Markdown 源文件。我们使用 [MkDocs](https://www.mkdocs.org/) 和优秀的 [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) 主题来构建和渲染我们的文档。

## 🚀 本地开发与预览

如果你想在本地修改文档并实时预览效果，请遵循以下步骤。这对于进行较大的改动或添加新页面非常有用。

### 1. 克隆仓库

首先，将此仓库克隆到你的本地机器：
```bash
git clone https://cotix-ai.github.io/Socrates-Cookbook.git
cd Socrates-Cookbook
```

### 2. 设置虚拟环境 (推荐)

我们强烈建议使用 Python 虚拟环境来隔离项目依赖。

```bash
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# macOS / Linux:
source venv/bin/activate
# Windows:
.\venv\Scripts\activate
```

### 3. 安装依赖

项目所需的依赖项都在 `requirements.txt` 文件中。
```bash
pip install -r requirements.txt
```

### 4. 启动本地服务器

现在，运行 MkDocs 内置的开发服务器。它会监控你的文件改动并自动刷新浏览器。
```bash
mkdocs serve
```

服务启动后，你会在终端看到类似以下的输出：
```
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Serving on http://127.0.0.1:8000/
```

现在，在你的浏览器中打开 **`http://127.0.0.1:8000`** 就可以看到和你在线文档一模一样的本地版本了！你在 `docs/` 文件夹中所做的任何保存都会立即反映在浏览器中。

## 🤝 如何贡献

我们非常欢迎来自社区的贡献！无论是修正一个拼写错误、改进一段模糊的描述，还是撰写一个全新的教程，我们都表示衷心的感谢。

### 发现问题或提出建议

-   **报告错误**: 如果你发现文档中有不准确、过时或令人困惑的内容，请创建一个 Issue。请尽可能详细地描述问题所在，并附上相关页面的链接。
-   **功能建议**: 如果你认为我们应该添加某个主题的文档或示例，也欢迎通过 Issue 来告诉我们。

### 提交你的修改 (Pull Requests)

如果你想直接修复问题或添加内容，我们欢迎你提交拉取请求 (Pull Request)。

1.  **Fork 本仓库**: 点击页面右上角的 "Fork" 按钮。
2.  **克隆你的 Fork**: `git clone https://github.com/Cotix-AI/Socrates-Cookbook.git`
3.  **创建新分支**: `git checkout -b your-feature-or-fix-name`
4.  **进行修改**: 在 `docs/` 文件夹中进行你的修改。请确保在本地运行 `mkdocs serve` 来预览你的改动。
5.  **提交并推送**:
    ```bash
    git add .
    git commit -m "docs: 描述你的修改内容"
    git push origin your-feature-or-fix-name
    ```
6.  **创建 Pull Request**: 回到你的 GitHub Fork 页面，点击 "New pull request" 按钮，并按照指引操作。

我们的团队会尽快审查你的贡献。感谢你为 Cotix AI 社区所做的一切！

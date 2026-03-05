# AI Daily News (AI 24 小时日报生成器)

![Hero Banner](https://img.shields.io/badge/Claude-Skill-blue?style=for-the-badge&logo=anthropic)
![Python](https://img.shields.io/badge/Python-3.x-blue?style=for-the-badge&logo=python)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

一个专为极客与 AI 从业者打造的 **Claude 桌面自动化 Skill**。全自动并发抓取全球 10 大顶尖 AI 媒体与科技门户，运用二维语义算法进行高纯度降噪，最终生成一份按领域自动分类、包含中文双语对照的优美本地 HTML 日报。

## 🌟 核心特性 (Features)

*   **10 大媒体火力网并行抓取**
    *   **国内硬核：** 机器之心、AIbase
    *   **国内门户：** IT之家、36Kr
    *   **海外顶尖：** TechCrunch AI、The Verge、VentureBeat AI、AI News、MarkTechPost、Ars Technica
*   **高纯度“二维语义”降噪算法**
    *   绝不放过任何一条带有 `大模型`、`Agent`、`GPU集群` 的硬核资讯。
    *   精准剔除泛科技媒体中“Google 反垄断败诉”、“苹果发布新表带”等**非 AI 的大厂花边新闻**。
*   **绝对零时差过滤**
    *   自动解析所有来源复杂的 UTC 或相对时间，精准换算至北京时间，**严格剔除超过 24 小时的旧闻**。
*   **无感中文化 (Auto Translation)**
    *   挂载轻量级翻译引擎，将海外一手资讯自动翻译，生成“中文主标题 + 英文原标题”的双语沉浸式阅读卡片。
*   **优美的生成式 UI**
    *   不依赖任何后端，纯本地生成嵌入式 Tailwind CSS 单页 HTML。
    *   带有吸顶交互式导航（Sticky Nav），自动将海量资讯归入：🤖 大模型与前沿技术 / 💻 AI 算力与硬件 / 🦾 具身智能与终端 / 📈 政策与投融资。

## 🚀 安装指南 (Installation)

本工具设计为标准的 **Claude 桌面端 Skill** 运行。

1. **克隆此技能到本地 Claude Skills 目录：**
   ```bash
   mkdir -p ~/.claude/skills/ai-daily-news
   git clone https://github.com/huskytech1/ai-daily-news.git ~/.claude/skills/ai-daily-news
   ```
2. **构建专属 Python 虚拟环境 (推荐)：**
   ```bash
   python3 -m venv ~/.claude/envs/ai-daily-news
   ~/.claude/envs/ai-daily-news/bin/pip install feedparser beautifulsoup4 requests pytz deep-translator
   ```

## 💡 如何使用 (Usage)

在 Claude 桌面端对话框中，直接对 Agent 说出指令即可触发：

> *"帮我整理今天的 AI 日报"*
> *"看看过去 24 小时有什么 AI 新闻"*
> *"生成 AI Daily News"*

Claude 将在后台极速执行 `main.py`，并将生成的精美 HTML 文件保存至你的本地文档目录（默认：`~/my_project_area/documents/`），点击即可在浏览器中阅读。

## 🛠️ 自定义配置 (Configuration)

你可以随时打开仓库中的 `main.py`，找到 `source_matrix` 数组来增加或删减你个人的信息源：

```python
source_matrix = [
    # type: "rss" (标准订阅源) 或 "custom_aibase" (特定 DOM 解析)
    # lang: "en" (触发自动翻译) 或 "zh"
    # strict_filter: True (启动高纯度 AI 词汇校验，过滤花边新闻)
    {"name": "机器之心", "type": "rss", "lang": "zh", "url": "https://www.jiqizhixin.com/rss", "strict_filter": False},
    # ... 在此处添加你的专属 RSS 链接 ...
]
```

## 📜 许可证 (License)
MIT License. 自由使用与修改。

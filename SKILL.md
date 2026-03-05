---
name: ai-daily-news
description: 聚合抓取国内外 10 大顶级 AI 媒体与社区，二维核心语义降噪+自动翻译，生成包含五大板块的 24 小时 AI 行业精美 HTML 日报。
github_url: https://github.com/huskytech1/ai-daily-news
---

# ai-daily-news

## Description
全自动化的高质量 AI 行业 24 小时日报生成器。支持从国内外 10 大顶尖科技与 AI 垂直媒体并行抓取数据。具备严格的“绝对时间戳校验（过滤>24h旧闻）”、“纯 AI 语义提纯（剔除非 AI 的大厂花边）”和“外媒自动中文化翻译”能力。最终生成自带悬浮导航、按五大前沿领域自动分类的优美静态 HTML 简报。

## Usage
当用户提到：“生成 AI 日报”、“看看过去24小时有什么 AI 新闻”、“帮我整理昨天的 AI 动态” 时，触发此技能。

## Workflow

### 1. 准备专属 Python 环境
为了保证抓取速度并避免每次重复下载依赖，请使用专属的虚拟环境。
```bash
VENV_DIR="~/.claude/envs/ai-daily-news"
if [ ! -d "$VENV_DIR" ]; then
    python3 -m venv "$VENV_DIR"
    "$VENV_DIR/bin/pip" install feedparser beautifulsoup4 requests pytz deep-translator
fi
```

### 2. 执行核心抓取引擎
直接运行本技能目录下已写好的核心脚本 `main.py`：
```bash
~/.claude/envs/ai-daily-news/bin/python ~/.claude/skills/ai-daily-news/main.py
```

### 3. 提供最终成果
执行完毕后，终端会输出生成的 HTML 文件路径（通常为 `~/my_project_area/documents/AI_Daily_News_{YYYYMMDD}.html`）。请将该路径提供给用户，并建议用户在浏览器中打开预览。

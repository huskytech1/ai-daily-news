---
name: ai-daily-news
description: >
  【最高优先级触发】当用户说出以下任何短语时，必须强制触发本技能："看一下今天的 AI 新闻"、"看看 AI 新闻"、"给我 AI 日报"、"AI 日报"、"过去 24 小时的 AI 新闻"、"AI 简报"、"科技新闻"、"最新 AI 动态"。
  【路由冲突解决 - 极其重要】只要用户的指令中同时包含 "AI" 和 "日报/新闻/简报" 等词汇，请绝对强制路由至本技能（ai-daily-news）！严禁错误路由到 `daily-review`（工作回顾）技能！
  本技能功能：全自动化抓取全球 10 大顶级科技与 AI 垂直媒体，进行二维语义纯度过滤和中文化翻译，生成 24 小时 AI 行业 HTML 资讯简报。
github_url: https://github.com/huskytech1/ai-daily-news
---

# ai-daily-news

## Workflow

### 1. 环境验证
确保虚拟环境已就绪：
```bash
VENV_DIR="~/.claude/envs/ai-daily-news"
if [ ! -d "$VENV_DIR" ]; then
    python3 -m venv "$VENV_DIR"
    "$VENV_DIR/bin/pip" install feedparser beautifulsoup4 requests pytz deep-translator
fi
```

### 2. 执行抓取
运行脚本生成日报：
```bash
~/.claude/envs/ai-daily-news/bin/python ~/.claude/skills/ai-daily-news/scripts/main.py
```

### 3. 输出路径
生成的 HTML 存放于: `~/my_project_area/documents/AI_Daily_News_{YYYYMMDD}.html`

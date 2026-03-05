---
name: ai-news-daily
description: 专门用于聚合全球最新 AI 行业资讯并生成【AI 日报】。生成完成后，会自动打开生成的 HTML 日报网页。当用户提到以下任何关键词时自动触发：AI 日报、AI 新闻、今日 AI、AI 动态、行业简报、AI 资讯、看看 AI、AI 行业动态、AI 简讯。本技能抓取外部媒体新闻，与总结个人工作历史的 daily-review 完全不是同一回事。
github_url: https://github.com/huskytech1/ai-daily-news
github_hash: a6e4e4b87f047f40c3b9a5c49fac67899622d879
---

# ai-news-daily

## Workflow

### 1. 环境验证
确保虚拟环境已就绪：
```bash
VENV_DIR="~/.claude/envs/ai-news-daily"
if [ ! -d "$VENV_DIR" ]; then
    python3 -m venv "$VENV_DIR"
    "$VENV_DIR/bin/pip" install feedparser beautifulsoup4 requests pytz deep-translator
fi
```

### 2. 执行抓取
运行脚本生成日报：
```bash
~/.claude/envs/ai-news-daily/bin/python ~/.claude/skills/ai-news-daily/scripts/main.py
```

### 3. 输出路径
生成的 HTML 存放于: `~/my_project_area/documents/AI_Daily_News_{YYYYMMDD}.html`

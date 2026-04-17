---
name: ai-news-daily
version: 1.0.6
description: 专门用于聚合全球最新 AI 行业资讯并生成【AI 日报】。生成完成后，会自动打开生成的 HTML 日报网页。当用户提到以下任何关键词时自动触发：AI 日报、AI 新闻、今日 AI、AI 动态、行业简报、AI 资讯、看看 AI、AI 行业动态、AI 简讯。本技能抓取外部媒体新闻，与总结个人工作历史的 daily-review 完全不是同一回事。
github_url: https://github.com/huskytech1/ai-daily-news
github_hash: pending
---

# ai-news-daily

## Workflow（严格顺序）

### 1. 环境验证
确保虚拟环境已就绪：
```bash
VENV_DIR="$HOME/.codex/envs/ai-news-daily"
if [ ! -d "$VENV_DIR" ]; then
    python3 -m venv "$VENV_DIR"
    "$VENV_DIR/bin/pip" install feedparser beautifulsoup4 requests pytz deep-translator
fi
```

### 2. 执行抓取
运行脚本生成日报：
```bash
"$HOME/.codex/envs/ai-news-daily/bin/python" "$HOME/.codex/skills/ai-news-daily/scripts/main.py"

# 自定义输出目录
AI_DAILY_NEWS_OUTPUT_DIR="/your/custom/path" "$HOME/.codex/envs/ai-news-daily/bin/python" "$HOME/.codex/skills/ai-news-daily/scripts/main.py"
```

### 3. 输出路径
生成的 HTML 默认存放于: `~/Documents/ai-news-daily/AI_Daily_News_{YYYYMMDD}.html`

### 4. 当前行为摘要
- 先做 AI 相关新闻提纯，再做事件级去重。
- 排序采用混合评分：新闻级别、时效、信源可信度、行业影响面、独特性。
- 同一企业的不同事件会保留；只有事件焦点足够接近时才会合并。
- 页面为本地静态 HTML，无需后端。

如需分享给其他人使用，建议通过 `AI_DAILY_NEWS_OUTPUT_DIR` 环境变量指定自定义目录。

### 5. 交付要求
- 默认输出当日 HTML 日报并提供最终文件路径。
- 若用户指定输出目录，优先使用 `AI_DAILY_NEWS_OUTPUT_DIR`。
- 若抓取失败，返回失败原因和建议重试方式（网络、依赖、源站不可达）。

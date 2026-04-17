# AI Daily News v1.0.6

![Claude Skill](https://img.shields.io/badge/Codex-Skill-19c8b9?style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3.x-blue?style=for-the-badge&logo=python)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

一个用于聚合过去 24 小时 AI 新闻并生成本地 HTML 日报的桌面 Skill。它会并发抓取多家中英文媒体，做 AI 相关性过滤、事件级去重、综合排序，然后输出可直接浏览的日报页面。

## 特性

- 多源抓取：国内外 AI/科技媒体并发抓取。
- 事件级去重：同题转载会合并，同企业不同事件会保留。
- 混合排序：按新闻级别、时效、信源可信度、行业影响面、独特性综合排序。
- 双语信息：英文来源保留中文标题和英文原标题。
- 本地静态输出：直接生成单文件 HTML，无需后端。
- 自适应页面：适配桌面和移动端浏览。

## 当前信源

- 机器之心
- AIbase
- TechCrunch AI
- VentureBeat AI
- AI News
- MarkTechPost
- IT之家
- 36Kr AI

## 排序规则

日报不是纯时间流，当前采用混合评分：

- 新闻级别 35
- 时效 25
- 信源可信度 20
- 行业影响面 10
- 独特性 10

补充规则：

- 先做事件级去重，再排序。
- 同一企业不会仅因名字相同被判重。
- 同一栏目内限制同一企业连续出现不超过 2 条，避免版面被单一公司占满。

## 安装

```bash
mkdir -p "$HOME/.codex/skills"
git clone https://github.com/huskytech1/ai-daily-news.git "$HOME/.codex/skills/ai-news-daily"

python3 -m venv "$HOME/.codex/envs/ai-news-daily"
"$HOME/.codex/envs/ai-news-daily/bin/pip" install feedparser beautifulsoup4 requests pytz deep-translator
```

## 使用

```bash
"$HOME/.codex/envs/ai-news-daily/bin/python" "$HOME/.codex/skills/ai-news-daily/scripts/main.py"
```

自定义输出目录：

```bash
AI_DAILY_NEWS_OUTPUT_DIR="/your/custom/path" \
  "$HOME/.codex/envs/ai-news-daily/bin/python" \
  "$HOME/.codex/skills/ai-news-daily/scripts/main.py"
```

默认输出目录：

```text
~/Documents/ai-news-daily/AI_Daily_News_{YYYYMMDD}.html
```

## 版本更新

### v1.0.6

- 引入媒体化混合排序，替代简单信源优先排序。
- 优化事件级去重，避免同企业不同新闻被误合并。
- 增加栏目内同企业连续占位限制。
- 更新日报页面主题和导航布局。
- 默认输出目录改为 `~/Documents/ai-news-daily/`。
- 清理 README 和 Skill 文档中的本地环境痕迹。

### v1.0.5

- 页面改为暖色动森风格。
- 优化分类导航和 Hero 区布局。

### v1.0.3

- 增加事件级去重。
- 增加泛科技来源的 AI 明确信号过滤。

## 许可证

MIT

---
name: ai-daily-news
description: 聚合抓取国内外 10 大顶级 AI 媒体与社区，二维核心语义降噪+自动翻译，生成包含五大板块的 24 小时 AI 行业精美 HTML 日报。
github_url: https://github.com/huskytech1/ai-daily-news
---

# ai-daily-news

## Description
全自动化的高质量 AI 行业 24 小时日报生成器。支持从国内外 10 大顶尖科技与 AI 垂直媒体并行抓取数据。具备严格的“绝对时间戳校验（过滤>24h旧闻）”、“纯 AI 语义提纯（剔除非 AI 的大厂花边）”和“外媒自动中文化翻译”能力。最终生成自带悬浮导航、按五大前沿领域自动分类的优美静态 HTML 简报。

## Capabilities
- **多源混合抓取**: 支持标准 RSS 流解析与 AIbase 这种无 RSS 站点的智能 DOM 定向手术刀解析。
- **高纯度降噪**: 彻底过滤网页侧边栏、热榜噪音，并运用“二维语义匹配算法”剔除泛科技媒体中非 AI 相关的干扰新闻（如 Google 的反垄断诉讼等）。
- **零时差校验**: 自动解析复杂的 UTC 时间戳或相对时间，精准转换为北京时间并只保留过去 24h 的内容。
- **自动中文化**: 挂载 `deep-translator` 引擎，将海外顶尖媒体的前沿资讯标题与摘要自动翻译为中文。
- **UI 生成引擎**: 采用 Tailwind CSS，自动按“大模型、算力芯片、具身终端、政策融资”分类，并生成带 Sticky 导航的优美本地网页。

## Usage
当用户提到：“生成 AI 日报”、“看看过去24小时有什么 AI 新闻”、“帮我整理昨天的 AI 动态” 时，执行此技能。

## Workflow

### 1. 准备环境
确保工作环境中已安装必备的 Python 库（如果未安装，请先执行）：
```bash
python3 -m venv /tmp/ai_news_env
source /tmp/ai_news_env/bin/activate
pip install feedparser beautifulsoup4 requests pytz deep-translator
```

### 2. 运行抓取引擎
将以下极度严谨的核心抓取与生成代码保存为 `/tmp/fetch_ai_news_final.py` 并使用环境内的 Python 执行：

```python
import feedparser
import calendar
from datetime import datetime, timedelta
import pytz
import ssl
import re
import requests
from bs4 import BeautifulSoup
import concurrent.futures
from deep_translator import GoogleTranslator
import os

ssl._create_default_https_context = ssl._create_unverified_context
beijing_tz = pytz.timezone('Asia/Shanghai')
now_bj = datetime.now(beijing_tz)
cutoff = now_bj - timedelta(hours=24)

translator = GoogleTranslator(source='auto', target='zh-CN')

source_matrix = [
    {"name": "机器之心", "type": "rss", "lang": "zh", "url": "https://www.jiqizhixin.com/rss", "strict_filter": False},
    {"name": "AIbase", "type": "custom_aibase", "lang": "zh", "url": "https://news.aibase.com/zh/news", "strict_filter": False},
    {"name": "TechCrunch AI", "type": "rss", "lang": "en", "url": "https://techcrunch.com/category/artificial-intelligence/feed/", "strict_filter": False},
    {"name": "VentureBeat AI", "type": "rss", "lang": "en", "url": "https://venturebeat.com/category/ai/feed/", "strict_filter": False},
    {"name": "AI News", "type": "rss", "lang": "en", "url": "https://www.artificialintelligence-news.com/feed/", "strict_filter": False},
    {"name": "MarkTechPost", "type": "rss", "lang": "en", "url": "https://www.marktechpost.com/feed/", "strict_filter": False},
    
    {"name": "IT之家", "type": "rss", "lang": "zh", "url": "https://www.ithome.com/rss/", "strict_filter": True},
    {"name": "36Kr AI", "type": "rss", "lang": "zh", "url": "https://36kr.com/feed", "strict_filter": True},
    {"name": "The Verge", "type": "rss", "lang": "en", "url": "https://www.theverge.com/rss/index.xml", "strict_filter": True},
    {"name": "Ars Technica", "type": "rss", "lang": "en", "url": "http://feeds.arstechnica.com/arstechnica/index", "strict_filter": True}
]

def is_pure_ai_news(title, summary):
    content = (title + " " + summary).lower()
    pure_ai_entities = [
        "大模型", "人工智能", "生成式", "llm", "gpt", "agent", "具身智能", 
        "openai", "anthropic", "deepseek", "kimi", "智谱", "claude", "sora", "grok",
        "gemini", "copilot", "llama", "chatgpt", "midjourney"
    ]
    if any(e in content for e in pure_ai_entities): return True
    hardware_entities = ["英伟达", "nvidia", "ai芯片", "ai算力", "gpu集群", "神经处理", "npu"]
    if any(e in content for e in hardware_entities): return True
    return False

results = []
seen_titles = set()

def clean_html(raw_html):
    return re.sub(re.compile('<.*?>'), '', str(raw_html)).strip()

def add_result(config, title, link, dt_bj, summary):
    if title in seen_titles: return
    if config["strict_filter"] and not is_pure_ai_news(title, summary): return
    seen_titles.add(title)
    if dt_bj < cutoff or dt_bj > now_bj + timedelta(hours=2): return
    
    display_title = title.strip()
    display_summary = summary.strip()[:150] if summary else ""
    if config["lang"] == "en":
        try:
            display_title = translator.translate(display_title)
            if display_summary: display_summary = translator.translate(display_summary)[:100] + "..."
        except: pass

    results.append({
        "source": config["name"], "title": display_title, "original_title": title if config["lang"] == "en" else "",
        "link": link, "time": dt_bj.strftime('%Y-%m-%d %H:%M'), "timestamp": dt_bj.timestamp(), "summary": display_summary
    })

def fetch_rss(config):
    try:
        parsed = feedparser.parse(config["url"])
        for entry in parsed.entries:
            if not hasattr(entry, 'published_parsed') or not entry.published_parsed: continue
            ts = calendar.timegm(entry.published_parsed)
            dt_utc = datetime.fromtimestamp(ts, pytz.UTC)
            dt_bj = dt_utc.astimezone(beijing_tz)
            add_result(config, entry.title, entry.link, dt_bj, clean_html(entry.get('summary', '')))
    except: pass

def fetch_aibase(config):
    try:
        headers = {'User-Agent': 'Mozilla/5.0'}
        res = requests.get(config["url"], headers=headers, timeout=10)
        soup = BeautifulSoup(res.text, 'html.parser')
        items = soup.find_all('a', href=lambda h: h and h.startswith('/zh/news/'))
        for item in items:
            t_div = item.find('div', class_=re.compile(r'md:text-\[18px\]'))
            if not t_div: continue
            title = t_div.text.strip()
            link = "https://news.aibase.com" + item.get('href')
            s_div = item.find('div', class_=re.compile(r'text-\[14px\].*tipColor'))
            summary = s_div.text.strip() if s_div else ""
            t_cont = item.find('i', class_=re.compile(r'icon-rili'))
            dt_bj = now_bj
            if t_cont and t_cont.parent:
                tt = t_cont.parent.text.strip()
                if "刚刚" in tt: dt_bj = now_bj
                elif "分钟前" in tt: dt_bj = now_bj - timedelta(minutes=int(re.search(r'\d+', tt).group()))
                elif "小时前" in tt: dt_bj = now_bj - timedelta(hours=int(re.search(r'\d+', tt).group()))
                elif "昨天" in tt: dt_bj = now_bj - timedelta(days=1)
                elif "-" in tt:
                    parts = tt.split("-")
                    if len(parts) == 3: dt_bj = beijing_tz.localize(datetime.strptime(tt, '%Y-%m-%d'))
                    elif len(parts) == 2: dt_bj = beijing_tz.localize(datetime.strptime(f"{now_bj.year}-{tt}", '%Y-%m-%d'))
            add_result(config, title, link, dt_bj, summary)
    except: pass

print("🚀 Starting strict pure-AI fetching & translating...")
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    for source in source_matrix:
        if source["type"] == "rss": executor.submit(fetch_rss, source)
        elif source["type"] == "custom_aibase": executor.submit(fetch_aibase, source)

results.sort(key=lambda x: x['timestamp'], reverse=True)
print(f"✅ Extracted {len(results)} PURE AI news items after applying strict filter.")

categories = {
    "大模型与前沿技术": {"id": "model", "icon": "🤖", "items": []},
    "AI算力与硬件芯片": {"id": "hardware", "icon": "💻", "items": []},
    "具身智能与智能终端": {"id": "embodied", "icon": "🦾", "items": []},
    "政策风向与投融资": {"id": "policy", "icon": "📈", "items": []},
    "综合前沿资讯": {"id": "general", "icon": "📰", "items": []}
}

for item in results:
    c = (item["title"] + " " + item["summary"] + " " + item["original_title"]).lower()
    if any(k in c for k in ["模型", "openai", "gpt", "gemini", "claude", "deepseek", "qwen", "智谱", "kimi", "grok", "生成式", "llm"]): cat = "大模型与前沿技术"
    elif any(k in c for k in ["芯片", "算力", "英伟达", "amd", "gpu", "xpu", "半导体", "nvidia"]): cat = "AI算力与硬件芯片"
    elif any(k in c for k in ["机器人", "具身", "自动驾驶", "汽车", "手机", "pc", "头显", "眼镜", "robot", "无人驾驶"]): cat = "具身智能与智能终端"
    elif any(k in c for k in ["政策", "融资", "估值", "财报", "法规", "禁止", "收购", "央行", "法案", "投资"]): cat = "政策风向与投融资"
    else: cat = "综合前沿资讯"
    if len(categories[cat]["items"]) < 20: categories[cat]["items"].append(item)

html = """<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 行业 24 小时精选快讯</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        html { scroll-behavior: smooth; }
        body { font-family: 'Inter', system-ui, -apple-system, sans-serif; background-color: #f8fafc; }
        .card-hover { transition: transform 0.2s ease, box-shadow 0.2s ease; border-left: 4px solid transparent; }
        .card-hover:hover { transform: translateX(4px); box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); border-left-color: #3b82f6; }
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .hide-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        .en-subtitle { color: #94a3b8; font-size: 0.75rem; margin-top: 2px; }
    </style>
</head>
<body class="text-slate-800 antialiased">
    <div class="max-w-4xl mx-auto px-4 py-8 relative">
        <header class="mb-6 text-center">
            <h1 class="text-3xl font-bold tracking-tight text-slate-900 mb-3">AI 行业 24 小时日报</h1>
            <div class="inline-flex items-center px-4 py-1.5 rounded-full bg-slate-900 text-white text-sm font-medium shadow-sm">
                北京时间: """ + now_bj.strftime('%Y年%m月%d日') + """
            </div>
        </header>

        <nav class="sticky top-0 z-50 bg-[#f8fafc]/90 backdrop-blur-md py-4 mb-8 border-b border-slate-200 hide-scrollbar overflow-x-auto">
            <div class="flex flex-nowrap md:flex-wrap gap-2 justify-start md:justify-center min-w-max md:min-w-0 px-2">
                <button onclick="filterCategory('all', this)" class="cat-btn active px-4 py-2 rounded-full text-sm font-semibold bg-blue-600 text-white shadow-sm transition-all whitespace-nowrap border border-transparent">🌟 全部动态</button>
"""
for cat_name, cat_data in categories.items():
    if not cat_data["items"]: continue
    html += f"""<button onclick="filterCategory('{cat_data['id']}', this)" class="cat-btn px-4 py-2 rounded-full text-sm font-medium bg-white text-slate-600 border border-slate-200 hover:bg-slate-50 transition-all whitespace-nowrap shadow-sm">{cat_data['icon']} {cat_name} <span class="ml-1 text-xs px-1.5 py-0.5 rounded-full bg-slate-100 text-slate-500">{len(cat_data['items'])}</span></button>"""

html += """</div></nav><main class="space-y-12" id="news-container">"""

for cat_name, cat_data in categories.items():
    items = cat_data["items"]
    if not items: continue
    html += f"""<section id="{cat_data['id']}" class="cat-section scroll-mt-24">
                <div class="flex items-center gap-3 mb-6"><span class="text-2xl">{cat_data['icon']}</span><h2 class="text-2xl font-bold text-slate-800">{cat_name}</h2></div>
                <div class="space-y-4 pl-2 border-l-2 border-slate-200">"""
    for idx, item in enumerate(items, 1):
        summary = item['summary']
        if summary.startswith("IT之家"): summary = summary.split("消息，", 1)[-1].strip()
        subtitle_html = f'<div class="en-subtitle font-mono truncate">{item["original_title"]}</div>' if item.get('original_title') else ''
        html += f"""
                    <article class="bg-white p-5 rounded-xl shadow-sm border border-slate-200 card-hover relative group">
                        <div class="absolute -left-3.5 top-5 w-7 h-7 bg-slate-800 text-white rounded-full flex items-center justify-center text-sm font-bold shadow-sm ring-4 ring-[#f8fafc] group-hover:bg-blue-600 transition-colors">{idx}</div>
                        <div class="ml-6 flex flex-col gap-2">
                            <div class="flex items-center justify-between"><span class="px-2.5 py-1 bg-slate-100 text-slate-600 text-xs font-semibold rounded-md">{item['source']}</span><time class="text-sm text-slate-400 font-medium">{item['time']}</time></div>
                            <h3 class="text-lg font-bold leading-snug"><a href="{item['link']}" target="_blank" class="hover:text-blue-600 transition-colors">{item['title']}</a></h3>
                            {subtitle_html}
                            <p class="text-slate-500 leading-relaxed text-sm mt-1">{summary}</p>
                        </div>
                    </article>"""
    html += """</div></section>"""

html += """
        </main>
        <footer class="mt-16 text-center text-sm text-slate-400 border-t border-slate-200 pt-8 pb-12">
            由 Claude (AI News Skill) 自动化聚合生成 · 二维纯度校验引擎 · 海外资讯双语翻译
        </footer>
    </div>
    <script>
        function filterCategory(catId, btnElement) {
            document.querySelectorAll('.cat-btn').forEach(btn => {
                btn.classList.remove('bg-blue-600', 'text-white', 'border-transparent', 'font-semibold');
                btn.classList.add('bg-white', 'text-slate-600', 'border-slate-200', 'font-medium');
            });
            btnElement.classList.remove('bg-white', 'text-slate-600', 'border-slate-200', 'font-medium');
            btnElement.classList.add('bg-blue-600', 'text-white', 'border-transparent', 'font-semibold');
            const sections = document.querySelectorAll('.cat-section');
            sections.forEach(sec => {
                if (catId === 'all' || sec.id === catId) {
                    sec.style.display = 'block';
                    sec.style.opacity = '0';
                    setTimeout(() => { sec.style.transition = 'opacity 0.3s ease'; sec.style.opacity = '1'; }, 10);
                } else { sec.style.display = 'none'; }
            });
            if(catId !== 'all') { window.scrollTo({ top: 0, behavior: 'smooth' }); }
        }
    </script>
</body>
</html>
"""

import os
save_dir = "/Users/huskytech/my_project_area/documents"
os.makedirs(save_dir, exist_ok=True)
date_str = now_bj.strftime('%Y%m%d')
with open(f"{save_dir}/AI_Daily_News_{date_str}.html", "w") as f:
    f.write(html)
print(f"📝 网页生成成功: {save_dir}/AI_Daily_News_{date_str}.html")
```

### 3. 生成报告
成功执行后，在终端提示用户报告已存储至：`/Users/huskytech/my_project_area/documents/AI_Daily_News_{YYYYMMDD}.html` 即可。


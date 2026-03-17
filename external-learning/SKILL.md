---
name: external-learning
description: |
  外部学习工具。定期从外部信息源学习最新动态，包括 AI 论文、GitHub 热门项目、Hacker News、行业融资等。

  **当以下情况时使用此 Skill**：
  (1) 心跳触发时，轮询外部信息源
  (2) 主人问"最近有什么新东西"、"AI 有什么进展"
  (3) 需要了解某个领域的最新动态
  (4) 主人提到"学习"、"趋势"、"热门"、"前沿"
---

# External Learning

从外部世界主动学习，保持信息敏感度。

## 信息源

### 1. GitHub Trending

**频率**：每天 1 次
**方法**：用 web_fetch 抓取 GitHub Trending 页面

```
https://github.com/trending?since=daily
https://github.com/trending/python?since=daily
https://github.com/trending/typescript?since=daily
```

**提取**：项目名、star 数、描述、语言
**关注**：AI/LLM 相关、Agent 框架、开发工具

### 2. Hacker News Top Stories

**频率**：每天 1-2 次
**方法**：HN Firebase API

```bash
# 获取 top 30 story ids
curl -s "https://hacker-news.firebaseio.com/v0/topstories.json" | python3 -c "import json,sys; print(json.loads(sys.stdin.read())[:30])"

# 获取单条详情
curl -s "https://hacker-news.firebaseio.com/v0/item/{id}.json"
```

**提取**：标题、URL、分数、评论数
**关注**：AI、编程语言、开源项目、创业相关

### 3. arXiv 热门论文

**频率**：每 2-3 天 1 次
**方法**：arXiv API

```
https://export.arxiv.org/api/query?search_query=cat:cs.AI&sortBy=submittedDate&sortOrder=descending&max_results=10
https://export.arxiv.org/api/query?search_query=cat:cs.CL&sortBy=submittedDate&sortOrder=descending&max_results=10
```

**提取**：标题、摘要、作者、分类
**关注**：Agent、RAG、推理、多模态、效率优化

### 4. 36kr 融资动态

**频率**：每天 1 次
**方法**：已有 PitchHub API（见 TOOLS.md）

```bash
curl -X POST "https://gateway.36kr.com/api/pms/project/financing/list" \
  -H "Content-Type: application/json" \
  -H "Origin: https://pitchhub.36kr.com" \
  -d '{"partner_id":"web","timestamp":<ms>,"partner_version":"1.0.0","param":{"pageNo":"1","pageSize":"20"}}'
```

**提取**：项目名、融资轮次、金额、投资方
**关注**：AI 赛道、大额融资、知名机构出手

### 5. Product Hunt

**频率**：每 2-3 天 1 次
**方法**：web_fetch 抓取

```
https://www.producthunt.com
```

**提取**：产品名、描述、投票数
**关注**：AI 工具、开发者工具、效率工具

### 6. OpenClaw 生态

**频率**：每天 1 次（随版本检查）
**方法**：

```bash
# 新版本
pnpm view openclaw version

# GitHub releases
gh api repos/openclaw/openclaw/releases --jq '.[0] | {tag_name, published_at, body}'

# ClawHub 新 skill
clawhub search --sort recent
```

## 执行流程

### Step 1: 检查上次学习时间

读取 `memory/heartbeat-state.json` 中的 `lastChecks` 字段：

```json
{
  "lastChecks": {
    "githubTrending": "2026-03-17T10:00:00+08:00",
    "hackerNews": "2026-03-17T14:00:00+08:00",
    "arxiv": "2026-03-15T10:00:00+08:00",
    "pitchhub": "2026-03-17T10:00:00+08:00",
    "productHunt": "2026-03-16T10:00:00+08:00",
    "openclawEco": "2026-03-17T02:00:00+08:00"
  }
}
```

### Step 2: 选择信息源

根据距上次检查的时间间隔，选择 1-2 个信息源：

| 信息源 | 最小间隔 | 优先级 |
|--------|----------|--------|
| GitHub Trending | 24h | 高 |
| Hacker News | 12h | 高 |
| arXiv | 48h | 中 |
| 36kr 融资 | 24h | 中 |
| Product Hunt | 48h | 低 |
| OpenClaw 生态 | 24h | 低 |

如果主人主动问，忽略间隔限制，直接查询。

### Step 3: 抓取并筛选

每个信息源抓取后，用以下标准筛选值得记录的内容：

**必记**（写入 learning notes）：
- AI Agent 相关的重大进展
- star > 1000 的新项目（24h 内）
- HN 分数 > 200 的 AI/编程帖子
- 单笔融资 > 1 亿人民币的 AI 项目
- OpenClaw 新版本或重要 skill

**可记**（简要记录）：
- 有趣的开源工具
- 新的 AI 应用场景
- 值得关注的技术趋势

**忽略**：
- 纯新闻/八卦
- 已知的旧信息
- 与主人业务无关的领域

### Step 4: 写入学习笔记

写入 `memory/learning/YYYY-MM-DD.md`：

```markdown
# 外部学习笔记 YYYY-MM-DD

## GitHub Trending
- **[项目名](URL)** ⭐ star数 — 一句话描述
  - 为什么值得关注：[原因]

## Hacker News
- **[标题](URL)** 🔥 分数 | 评论数
  - 要点：[摘要]

## arXiv
- **[论文标题](URL)**
  - 核心贡献：[一句话]
  - 与我们的关系：[为什么关注]

## 融资动态
- **项目名** — 轮次 金额 | 投资方
```

### Step 5: 决定是否通知主人

**主动通知**（满足任一）：
- 发现与主人业务直接相关的信息
- AI Agent 领域的重大突破
- OpenClaw 有重要更新
- 特别有趣/有价值的发现

**静默记录**：
- 常规更新
- 与主人业务关联不大的内容
- 深夜时段（23:00-08:00）

## 与 smart-heartbeat 的集成

在 smart-heartbeat 的评估表中新增：

| 检查项 | 基础分 | 加分条件 |
|--------|--------|----------|
| 外部学习 | 1 | 距上次 >24h +2；主人提到相关话题 +2 |

## 周报汇总

每周审计时（self-audit），从 `memory/learning/` 中提炼本周最值得关注的 5-10 条信息，写入审计报告的"本周外部动态"章节。

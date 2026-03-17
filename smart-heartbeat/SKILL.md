---
name: smart-heartbeat
description: |
  智能心跳。基于最近对话上下文和当前状态，动态决定 heartbeat 时该关注什么。

  **当以下情况时使用此 Skill**：
  (1) heartbeat 触发时，用于决定该检查什么
  (2) 需要评估当前是否有值得主动通知的事项
  (3) 需要根据上下文调整关注优先级
---

# Smart Heartbeat

让 heartbeat 从机械轮询进化为上下文感知的智能调度。

## 核心思路

传统 heartbeat：固定检查列表，轮流执行。
智能 heartbeat：根据最近发生的事，动态决定该关注什么。

## 决策框架

每次 heartbeat 触发时，按以下流程决策：

### Step 1: 读取上下文

快速扫描以下信息源（不要全读，只看关键信息）：
1. `memory/YYYY-MM-DD.md`（今天的日志）— 今天发生了什么
2. 最近 2-3 条与主人的对话 — 主人最近在关注什么
3. `memory/heartbeat-state.json` — 上次检查了什么、什么时候
4. `memory/pending-tasks.md`（如果存在）— 有没有待处理的任务

### Step 2: 评估优先级

根据上下文，给以下检查项打分（1-5）：

| 检查项 | 基础分 | 加分条件 |
|--------|--------|----------|
| 飞书日历 | 2 | 主人提到会议 +2；距上次检查 >4h +1 |
| 飞书消息 | 2 | 主人在等回复 +3；有未读 @提及 +2 |
| OpenClaw 版本 | 1 | 距上次检查 >24h +2 |
| FreeTodo 服务 | 1 | 上次检查挂了 +3 |
| 配置同步 | 1 | 凌晨时段 +3；今天没同步 +2 |
| 待办任务提醒 | 2 | 有临近截止的任务 +3 |
| 记忆整理 | 1 | 距上次整理 >7天 +2；memory 文件 >50个 +2 |
| ClawHub | 0 | 距上次尝试 >24h +1（限流中保持 0） |

### Step 3: 执行

- 选择得分最高的 1-3 项执行
- 总执行时间控制在合理范围内
- 执行完更新 `memory/heartbeat-state.json`

### Step 4: 决定是否通知

**通知主人的条件**（满足任一）：
- 日历：2 小时内有会议
- 消息：有重要未读消息或 @提及
- 服务：FreeTodo 挂了且重启失败
- 版本：OpenClaw 有新版本
- 任务：有今天截止的任务

**静默的条件**：
- 深夜（23:00-08:00），除非紧急
- 所有检查项正常
- 主人明显在忙（最近消息密集）

## 状态文件格式

`memory/heartbeat-state.json` 扩展格式：

```json
{
  "lastChecks": {
    "calendar": "2026-03-17T20:00:00+08:00",
    "messages": "2026-03-17T18:00:00+08:00",
    "openclawVersion": "2026-03-17T02:00:00+08:00",
    "freetodo": "2026-03-17T20:00:00+08:00",
    "configSync": "2026-03-17T02:00:00+08:00",
    "memoryCompress": "2026-03-10T00:00:00+08:00",
    "clawhub": "2026-03-17T19:00:00+08:00"
  },
  "lastConfigSync": "2026-03-17",
  "lastSelfAudit": "2026-03-16",
  "recentContext": {
    "humanFocus": "自我进化",
    "pendingItems": [],
    "serviceIssues": []
  }
}
```

## 与 HEARTBEAT.md 的关系

HEARTBEAT.md 定义了**所有可能的检查项**。
本 skill 决定**每次 heartbeat 实际执行哪些**。

HEARTBEAT.md 是菜单，smart-heartbeat 是点菜的逻辑。

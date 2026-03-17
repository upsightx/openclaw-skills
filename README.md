# OpenClaw 自定义 Skills

为 [OpenClaw](https://github.com/openclaw/openclaw) 打造的个人 Skill 集合。

## 什么是 Skill？

Skill 是基于 Markdown 的指令集，告诉 AI Agent 在特定场景下该怎么做。不需要写代码，纯 Markdown 即可 — 包含决策逻辑、工具调用方式和执行流程。

```
skill-name/
├── SKILL.md          # 核心：场景识别 + 执行流程
└── references/       # 可选：参考资料、模板等
```

## Skill 列表

| Skill | 说明 |
|-------|------|
| **auto-extract-task** | 从对话中自动提取任务，创建到飞书任务 |
| **deploy-helper** | 项目部署助手 — Docker 优先测试、常见坑点速查、子 Agent 指令模板 |
| **memory-compress** | 记忆压缩 — 定期归档旧日志、提炼关键信息、保持记忆精简 |
| **self-audit** | 每周自我审计 — 回顾表现、提取教训、发现改进点 |
| **smart-heartbeat** | 智能心跳调度 — 根据最近上下文动态决定检查优先级 |

## 安装

将 skill 文件夹复制到 OpenClaw 的 skills 目录：

```bash
# 默认路径
~/.openclaw/skills/

# 或在 openclaw.json 中自定义路径
```

OpenClaw 启动时会自动发现并注册 skill。

## License

MIT

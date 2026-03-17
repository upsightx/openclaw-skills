# OpenClaw Custom Skills

Personal skills for [OpenClaw](https://github.com/openclaw/openclaw) — an AI agent framework.

## What are Skills?

Skills are Markdown-based instruction sets that tell the AI agent how to handle specific scenarios. No code required — just structured prompts with decision logic, tool usage patterns, and execution flows.

```
skill-name/
├── SKILL.md          # Core: scenario matching + execution flow
└── references/       # Optional: templates, examples
```

## Skills

| Skill | Description |
|-------|-------------|
| **auto-extract-task** | Automatically extract actionable tasks from conversations and create them in Feishu Tasks |
| **deploy-helper** | Safe project deployment SOP — Docker-first testing, common pitfall solutions, sub-agent templates |
| **memory-compress** | Periodic memory compression — archive old logs, distill key learnings, keep memory lean |
| **self-audit** | Weekly self-audit routine — review performance, extract lessons, identify improvements |
| **smart-heartbeat** | Context-aware heartbeat scheduling — dynamically prioritize checks based on recent activity |

## Installation

Copy any skill folder into your OpenClaw skills directory:

```bash
# Default location
~/.openclaw/skills/

# Or custom path configured in openclaw.json
```

OpenClaw auto-discovers skills on startup.

## License

MIT

# Claude-Codex 协作工作区

Claude Code 与 OpenAI Codex 双 AI 协作的实战工作区。**Claude Code** 负责架构设计、代码审查与需求沟通，**Codex CLI** 负责具体代码生成与执行，发挥各自优势高效完成开发任务。

## 协作模式

```
用户需求 → Claude Code（设计 + 审查） → Codex CLI（执行 + 生成） → Claude Code（审查 + 修复） → 交付
```

| 角色 | 工具 | 职责 |
|------|------|------|
| 🏗️ 架构师 | Claude Code | 方案设计、需求分析、代码审查、问题修复 |
| 🔨 执行者 | Codex CLI | 代码生成、文件操作、命令执行 |

## 项目结构

```
.
├── CLAUDE.md                   # Claude Code 工作指引
├── README.md                   # 项目说明
└── kol-recruitment-form.html   # KOL 招募静态表单页面
```

## 环境依赖

- **Claude Code** — AI 编程助手 CLI
- **OpenAI Codex** — AI 代码生成 CLI（`codex-cli 0.145.0-alpha.30`）
- 模型配置：`gpt-5.6-sol` / 自定义 API

## 快速开始

```bash
# 克隆仓库
git clone https://github.com/lzy-coder123/claude-codex.git
cd claude-codex

# Codex CLI 路径
export PATH="$HOME/AppData/Local/OpenAI/Codex/bin/e2d6a5ee2cac801c:$PATH"

# 用 Codex 执行任务（非交互式）
codex exec -C . -s workspace-write "<任务描述>"

# 直接浏览器打开表单页面
start kol-recruitment-form.html
```

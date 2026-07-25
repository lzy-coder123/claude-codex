# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Claude Code 与 OpenAI Codex 协作工作区。Claude Code 负责架构设计、代码审查、需求沟通；Codex CLI 负责具体代码生成和执行。

## 协作模式

- **Claude Code 角色**: 设计方案、审查代码、修复问题、与用户沟通
- **Codex CLI 角色**: 通过 `codex exec` 执行具体生成任务
- Codex 位于 `$HOME/AppData/Local/OpenAI/Codex/bin/e2d6a5ee2cac801c/codex.exe`
- 调用 Codex 时注意: 在非 git 仓库中需加 `--skip-git-repo-check`，如需保存会话记录不要加 `--ephemeral`

## Codex 常用命令

```bash
# 设置 PATH
export PATH="$HOME/AppData/Local/OpenAI/Codex/bin/e2d6a5ee2cac801c:$PATH"

# 非交互式执行（不保存会话）
codex exec -C <工作目录> -s workspace-write --skip-git-repo-check --ephemeral "<任务描述>"

# 非交互式执行（保存会话，Desktop 可见）
codex exec -C <工作目录> -s workspace-write --skip-git-repo-check "<任务描述>"

# 代码审查
codex review
```

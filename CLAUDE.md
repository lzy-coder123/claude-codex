# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Claude Code 与 OpenAI Codex 协作工作区。包含 KOL 招募表单、n8n 自动化工作流、飞书多维表格集成。

## 当前技术栈

- **前端**: 单文件 HTML 表单，GitHub Pages 部署
- **自动化**: n8n（本地 Docker 部署），Webhook 触发
- **数据存储**: 飞书多维表格（API 写入）
- **邮件**: QQ 邮箱 SMTP 自动回复
- **容器**: Docker Desktop + WSL 2

## n8n 工作流

```
Webhook → Code(修数组) → HTTP(Token飞书) → Code1(拼数据) → HTTP(写飞书) → Email(回复)
```

- 本地 n8n: `http://localhost:5678`
- 生产 Webhook: `http://localhost:5678/webhook/1c7325d4-61c3-4cc1-a311-67d6d40cb2e3`
- 工作流已发布，永久在线

## Docker 命令

```bash
docker start n8n     # 启动 n8n
docker stop n8n      # 停止 n8n
docker ps            # 查看状态
docker logs n8n      # 查看日志
```

容器启动参数：`N8N_BLOCK_ENV_ACCESS_IN_NODE=false`, `FEISHU_APP_ID`, `FEISHU_APP_SECRET`

## Git 注意

- `kol-recruitment-form.html` 在 .gitignore 中（含本地 webhook URL，不上传）
- .gitignore 已配置：node_modules, .env, auth.json, 临时文件等

## 相关文件

- `QUICKSTART.md` — 快速启动指南
- `KNOWLEDGE.md` — 完整知识梳理
- `README.md` — 项目说明
- `kol-recruitment-form.html` — 本地表单（不上传 GitHub）

## Codex 调用

```bash
export PATH="$HOME/AppData/Local/OpenAI/Codex/bin/e2d6a5ee2cac801c:$PATH"
codex exec -C <工作目录> -s workspace-write --skip-git-repo-check --ephemeral "<任务描述>"
```

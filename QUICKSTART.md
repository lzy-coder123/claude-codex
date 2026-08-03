# 快速启动指南

## 开机后三步

1. 双击桌面 **Docker Desktop** → 等鲸鱼图标不动
2. 打开终端 → `docker start n8n`
3. 浏览器 → `http://localhost:5678`

## 本地测试表单

```bash
docker start n8n
cd ~/Claude-codex
python -m http.server 8080
```

浏览器 → `http://localhost:8080/kol-recruitment-form.html`

## 常用命令

```bash
docker start n8n      # 启动 n8n
docker stop n8n       # 停止 n8n
docker ps             # 看有没有在跑
docker logs n8n       # 看日志
```

## Webhook 地址

本地生产：`http://localhost:5678/webhook/1c7325d4-61c3-4cc1-a311-67d6d40cb2e3`
本地测试：`http://localhost:5678/webhook-test/1c7325d4-61c3-4cc1-a311-67d6d40cb2e3`

## 环境变量

Docker 启动参数：
- `FEISHU_APP_ID=cli_aaeeca52c1f85be4`
- `FEISHU_APP_SECRET=<你的飞书App Secret>`
- `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`

## 工作流结构

```
Webhook → Code → HTTP(Token) → Code1 → HTTP(写表格) → Email
```

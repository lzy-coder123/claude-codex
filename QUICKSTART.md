# 快速启动指南

## 开机后

Docker Desktop 设了开机自启，n8n 和 cloudflared 设了自动恢复。开机等 2 分钟就行。

如需手动：

```bash
docker start n8n cloudflared
```

## 访问地址

| 用途 | 地址 |
|------|------|
| n8n 编辑器 | `http://localhost:5678` |
| 公网表单 | `https://lzy-coder123.github.io/claude-codex/` |
| 本地表单 | `http://localhost:8080/kol-recruitment-form.html` |

## 公网隧道

```bash
docker logs cloudflared | grep trycloudflare  # 查看当前公网地址
```

⚠️ 重启电脑后隧道 URL 会变，需要更新 `kol-recruitment-form.html` 中的 webhook 地址并重新部署。

## 常用命令

```bash
docker ps                        # 看容器状态
docker start n8n cloudflared     # 启动
docker stop n8n cloudflared      # 停止
docker logs n8n                  # n8n 日志
docker logs cloudflared          # 隧道日志
```

## 部署到 GitHub Pages

```bash
cd ~/Claude-codex
cp kol-recruitment-form.html index.html
git add index.html
git commit -m "更新表单"
git push origin main            # CI/CD 自动部署
```

## Webhook 地址

本地：`http://localhost:5678/webhook/1c7325d4-61c3-4cc1-a311-67d6d40cb2e3`
公网：`https://<隧道域名>/webhook/1c7325d4-61c3-4cc1-a311-67d6d40cb2e3`

## 环境变量

- `FEISHU_APP_ID=cli_aaeeca52c1f85be4`
- `FEISHU_APP_SECRET=<你的飞书App Secret>`
- `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`
- `N8N_TUNNEL=true`（可选，社区版可能不生效）

## 工作流

```
Webhook → Code → HTTP(Token) → Code1 → HTTP(写飞书) → Email
```

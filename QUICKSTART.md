# 快速启动指南

## 1. 启动本机服务

启动 Docker Desktop，等待 Docker 引擎可用，然后启动 n8n：

```powershell
docker start n8n
```

公网报名表依赖 ngrok 隧道。如果 ngrok 使用 Docker 且容器名为 `ngrok`：

```powershell
docker start ngrok
docker logs ngrok
```

如果实际容器名不同，以 `docker ps -a` 的结果为准。

## 2. 启动静态页面

在仓库根目录执行：

```powershell
py -m http.server 8080
```

访问地址：

| 用途 | 地址 |
|------|------|
| 本地报名页 | `http://localhost:8080/index.html` |
| 本地数据看板 | `http://localhost:8080/kol-dashboard.html` |
| n8n 编辑器 | `http://localhost:5678` |
| 公网报名页 | `https://lzy-coder123.github.io/claude-codex/` |

## 3. 检查 n8n 工作流

系统包含两条独立的数据链路。

报名写入工作流：

```text
Webhook
  -> Code（表单格式处理）
  -> HTTP Request（飞书 Token）
  -> Code（飞书字段映射）
  -> HTTP Request（写入飞书）
  -> Email（确认邮件）
```

看板统计工作流：

```text
Webhook GET /get-stats
  -> HTTP Request（飞书 Token）
  -> HTTP Request（读取飞书记录）
  -> Code（聚合看板数据）
  -> Respond to Webhook（返回 JSON）
```

看板使用正式地址：

```text
http://localhost:5678/webhook/get-stats
```

统计工作流必须保存并切换为 Active，之后不需要点击 Execute workflow。页面打开时请求一次，并每 60 秒自动刷新。

## 4. CORS 设置

本地看板运行在 8080，n8n 运行在 5678，属于不同 Origin。统计 Webhook 的响应应包含：

```text
Access-Control-Allow-Origin: http://localhost:8080
```

若修改静态服务器端口，需要同步修改该响应头。

## 5. 发布 GitHub Pages

提交并推送 `main`：

```powershell
git add index.html kol-dashboard.html README.md QUICKSTART.md KNOWLEDGE.md CLAUDE.md AGENTS.md
git commit -m "更新 KOL 运营看板"
git push origin main
```

`.github/workflows/deploy.yml` 会把仓库内容自动发布到 `gh-pages`。无需手动检出或提交 `gh-pages` 分支。

## 6. 常见问题

### Webhook 未注册

- `/webhook-test/` 仅在点击 Execute workflow 后接收一次测试请求。
- `/webhook/` 要求工作流已保存并处于 Active 状态。

### 看板显示同步失败

依次检查：

1. Docker Desktop 和 n8n 是否运行。
2. `get-stats` 工作流是否 Active。
3. 浏览器网络请求是否返回 200 和 JSON。
4. n8n 是否返回正确的 CORS 响应头。
5. 飞书 Token、应用权限和表格授权是否有效。

### 看板显示 `--`

对应字段没有被统计接口返回，或所有飞书记录均为“未填写”。优先检查飞书原始字段名与 n8n Code 节点的映射。

## 安全边界

本地看板包含达人姓名、账号、平台和报名时间。不要直接把统计 Webhook 暴露到公网；若确需远程访问，应先增加登录或 Token 鉴权、HTTPS 和严格 CORS。

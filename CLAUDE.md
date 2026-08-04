# CLAUDE.md

本文件为 Claude Code 在本仓库中的补充说明。项目约束以 `AGENTS.md` 为准。

## 项目概述

这是一个 KOL 招募与运营数据系统：

```text
GitHub Pages 报名页
  -> n8n 报名工作流
  -> 飞书多维表格 + 确认邮件

本地运营看板
  -> n8n 统计工作流
  -> 飞书聚合数据
```

## 技术栈

- 前端：单文件 HTML、CSS、JavaScript。
- 图表：Chart.js CDN。
- 自动化：本机 Docker 中的 n8n。
- 数据存储：飞书多维表格。
- 邮件：QQ 邮箱 SMTP。
- 公网入口：GitHub Pages + ngrok。

## 页面

- `index.html`：GitHub Pages 首页和生产报名表。
- `kol-dashboard.html`：本地运营看板，每 60 秒读取统计接口。
- `kol-recruitment-form.html`：被 Git 忽略的本地报名页副本，覆盖前先比较 webhook 差异。

本地启动：

```powershell
py -m http.server 8080
```

## n8n 工作流

报名写入：

```text
Webhook -> Code -> 飞书 Token -> 字段映射 -> 写飞书 -> Email
```

看板统计：

```text
GET /webhook/get-stats -> 飞书 Token -> 读取记录 -> 聚合统计 -> JSON 响应
```

统计接口需要返回：

- `totalSignups`
- `newLast3Days`
- `averageFollowers`
- `coreInfluencerRatio`
- `domainStats`
- `platformStats`
- `followerStats`
- `dailySignups`
- `recentSignups`
- `updatedAt`

生产 Webhook 只有在工作流保存并处于 Active 状态时才会注册。测试地址 `/webhook-test/` 只接收点击 Execute workflow 后的一次请求。

## Git 与部署

- `main` 是源分支。
- `.github/workflows/deploy.yml` 在 push 后自动发布到 `gh-pages`。
- 不要手动修改 `gh-pages`，也不要把本地表单或凭据加入 Git。
- 不执行 push、生产 Webhook 调用或容器启停，除非用户明确要求。

## 安全边界

看板会展示姓名、账号等报名信息。当前统计接口使用本机 `localhost`，不要直接替换为无鉴权的公网地址。远程看板需要先设计身份验证、HTTPS 和 CORS 白名单。

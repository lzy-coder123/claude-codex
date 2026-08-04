# KOL 招募与运营数据系统

一个由静态报名页、n8n 自动化、飞书多维表格和运营数据看板组成的轻量 KOL 招募系统。

## 系统能力

- 公网 KOL 招募表单，部署在 GitHub Pages。
- n8n Webhook 接收报名数据并执行字段转换。
- 自动写入飞书多维表格并发送确认邮件。
- 本地运营看板每 60 秒读取统计接口并更新 KPI、图表和最近报名记录。
- GitHub Actions 在 `main` 更新后自动发布到 `gh-pages`。

## 数据链路

```text
报名者
  -> GitHub Pages 表单
  -> ngrok 公网 Webhook
  -> 本机 Docker / n8n
  -> 飞书多维表格 + 邮件确认

运营人员
  -> 本地数据看板
  -> n8n GET /webhook/get-stats
  -> 飞书记录统计结果
```

## 项目结构

```text
.
|-- index.html                     # GitHub Pages 报名页
|-- kol-dashboard.html             # KOL 运营数据看板
|-- kol-recruitment-form.html      # 本地表单副本，Git 忽略
|-- .github/workflows/deploy.yml   # GitHub Pages 自动部署
|-- QUICKSTART.md                  # 启动与发布指南
|-- KNOWLEDGE.md                   # 技术原理和排错记录
|-- AGENTS.md                      # AI Agent 项目约束
`-- CLAUDE.md                      # Claude Code 工作说明
```

## 本地运行

先启动 Docker Desktop 和 n8n，再在仓库根目录运行：

```powershell
py -m http.server 8080
```

访问：

- 报名页：`http://localhost:8080/index.html`
- 数据看板：`http://localhost:8080/kol-dashboard.html`
- n8n：`http://localhost:5678`

看板请求正式统计接口 `http://localhost:5678/webhook/get-stats`。对应 n8n 工作流必须保存并处于 Active 状态。

## 看板数据契约

统计接口返回 JSON 对象，核心结构如下：

```json
{
  "totalSignups": 17,
  "newLast3Days": 15,
  "averageFollowers": 410588,
  "coreInfluencerRatio": 64.7,
  "domainStats": {},
  "platformStats": {},
  "followerStats": {},
  "dailySignups": [],
  "recentSignups": [],
  "updatedAt": "2026-08-04T09:08:41.248Z"
}
```

字段详情和 n8n 聚合方式见 [KNOWLEDGE.md](KNOWLEDGE.md)。

## GitHub Pages

公网报名页：

```text
https://lzy-coder123.github.io/claude-codex/
```

推送 `main` 后，GitHub Actions 会自动更新 `gh-pages`，无需手动切换分支。

`kol-dashboard.html` 也会作为静态文件发布，但当前接口指向本机 `localhost:5678`，只适合本机运营使用。不要在没有身份验证的情况下把包含姓名和账号的统计接口暴露到公网。

## 安全说明

- 不提交飞书 App Secret、邮箱授权码、Token、Cookie 或真实报名数据。
- `.env`、`auth.json`、`*.secret` 和本地表单副本保持 Git 忽略。
- 公网统计接口必须增加鉴权，并限制 CORS 来源。
- 表单 Webhook 与数据看板 Webhook 是两条独立工作流，不要混用。

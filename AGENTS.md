# AGENTS.md

本文件是 AI Agent 在本仓库中的项目入口。开始修改前，先阅读本文件以及与任务相关的源码和文档；不要套用 PPT、Node 服务或其他通用项目的目录与工具约定。

## 项目定位

这是一个 KOL 招募自动化项目，主要链路为：

```text
GitHub Pages 静态表单
  -> 公网隧道 Webhook
  -> 本机 Docker 中的 n8n
  -> 飞书多维表格
  -> QQ 邮箱确认邮件
```

仓库只包含静态前端、GitHub Pages 部署配置和运维知识文档。n8n 工作流、飞书表格、邮箱配置、Docker 容器及其运行状态属于仓库外部系统，不要假定它们可由本仓库直接修改或验证。

## 关键文件

- `index.html`：生产表单和 GitHub Pages 入口，是已跟踪的前端事实源。
- `kol-recruitment-form.html`：本地表单副本，已被 `.gitignore` 忽略，可能包含与生产不同的 webhook。修改或覆盖前必须先比较内容。
- `.github/workflows/deploy.yml`：`main` 分支 push 后部署到 `gh-pages`。
- `QUICKSTART.md`：本机 Docker、隧道和部署操作指南。
- `KNOWLEDGE.md`：表单提交、CORS、n8n、飞书、Docker 和隧道原理记录。
- `CLAUDE.md`：Claude Code 使用说明。
- `test-naked.html`、`test-simple.html`：历史 webhook 连通性测试页，不是生产入口。

## 实现约束

- 前端保持为无需构建的单文件 HTML/CSS/JavaScript，除非用户明确要求引入构建工具或框架。
- 保持中文界面、移动端可用性和现有表单字段语义。
- 表单字段名是 n8n 的数据契约。修改 `name`、选项值或编码方式前，必须说明对 n8n 和飞书字段映射的影响。
- 当前提交使用 `application/x-www-form-urlencoded`、`XMLHttpRequest` 和 `ngrok-skip-browser-warning` 请求头。调整提交方式时，要同时考虑 CORS 预检、ngrok 警告页和旧移动浏览器兼容性。
- 不要把“请求已发出”等同于“提交成功”。涉及提交反馈的修改应根据 HTTP 状态、超时和网络错误显示结果，并防止重复提交。
- `index.html` 是生产版本。若任务也要求更新本地副本，先检查 `kol-recruitment-form.html` 的 webhook 和本地差异，只同步明确需要的部分。
- 页面没有现成测试框架。不要为小改动擅自引入大型依赖；优先使用静态检查、浏览器手工验证和最小化脚本检查。

## 安全边界

- 不得提交 App Secret、邮箱授权码、访问令牌、Cookie、个人数据或其他凭据。
- `.env`、`auth.json`、`*.secret` 和 `*_secret*` 必须继续保持忽略。
- webhook URL 属于运行配置。除非用户明确要求，不要更换、公开新增或猜测新的地址。
- 不要在未获授权时调用生产 webhook、写入飞书表格或发送测试邮件；页面本地验证应避免产生真实报名数据。
- 修改日志和文档示例时使用占位符，不要复制真实 Secret 或 Token。

## 文档一致性

- `QUICKSTART.md`、`KNOWLEDGE.md`、`CLAUDE.md` 和实际实现中的隧道类型、容器名称、端口、部署分支及 webhook 使用方式应保持一致。
- 当前实现使用 ngrok 固定域名；若改用 Cloudflare Tunnel 或其他方案，要同步更新相关文档。
- Markdown 文档使用 UTF-8 和中文；命令、变量名及代码标识保留英文。

## 验证要求

根据改动范围执行以下检查：

1. 查看 `git diff --check`，确保没有空白或补丁格式问题。
2. 修改表单时，在桌面和移动端宽度检查布局、必填校验、错误清理、重置和提交状态。
3. 修改提交逻辑时，至少验证成功、非 2xx、网络错误、超时和重复点击场景；生产 webhook 测试需要用户明确授权。
4. 修改部署工作流时，检查 YAML 语法、权限、发布目录和 `main -> gh-pages` 行为。
5. 修改字段或选项时，核对 n8n 字段映射和飞书字段类型，并在无法访问外部系统时明确说明未验证部分。

## 常用命令

```powershell
# 本地启动静态页面
py -m http.server 8080

# 查看工作区状态和差异
git status --short --branch
git diff --check
git diff

# 查看本机自动化服务（Docker Desktop 需要已启动）
docker ps
docker logs n8n
```

本地页面入口为 `http://localhost:8080/index.html`。除非用户明确要求，不要执行 `git push`、触发生产部署、启动或停止容器。

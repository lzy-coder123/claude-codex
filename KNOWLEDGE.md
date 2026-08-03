# 知识梳理：从表单到自动化全栈

## 一、项目全景

```
报名者打开网页 → 填表提交 → n8n 收到数据 → 自动写飞书表格 + 发邮件
```

涉及的技术：

| 层面 | 技术 | 作用 |
|------|------|------|
| 前端 | HTML + CSS + JS | 表单页面，给用户填的 |
| 部署 | GitHub Pages | 把 HTML 变成公网可访问的网址 |
| 自动化 | n8n | 接收表单数据，串联后续动作 |
| 数据存储 | 飞书多维表格 | 报名数据汇总管理 |
| 邮件 | QQ邮箱 SMTP | 自动回复报名者 |
| 运行环境 | Docker + WSL 2 + Hyper-V | 让 n8n 在自己电脑上 24 小时跑 |

---

## 二、表单页面是怎么工作的

### HTML 表单的核心机制

```html
<form action="接收地址" method="POST" target="隐藏窗口">
  <input name="name" placeholder="姓名">
  <input name="phone" placeholder="手机号">
  <button type="submit">提交</button>
</form>
```

三个关键属性：

| 属性 | 作用 | 通俗解释 |
|------|------|------|
| `action="地址"` | 数据发到哪 | 快递单上的收货地址 |
| `method="POST"` | 怎么发 | POST = 邮寄包裹，GET = 寄明信片 |
| `target="隐藏窗口"` | 在哪收结果 | 数据在后台发，页面不跳走 |

### 为什么不用 AJAX/fetch

表单页面(lzy-coder123.github.io)和数据管道(lzycoder123.app.n8n.cloud)是不同的域名。浏览器会拦截跨域 fetch 请求（CORS 策略）。但**原生表单提交不受 CORS 限制**，所以用 hidden iframe + 原生 form submit 绕过去。

---

## 三、n8n 工作流详解

### 工作流结构

```
Webhook → Code → HTTP(Token) → Code1 → HTTP(写表格) → Email
```

### 每个节点做什么

| 节点 | 类型 | 输入 | 输出 | 为什么需要 |
|------|------|------|------|------|
| **Webhook** | 触发器 | POST 请求 | 表单原始数据 | 接收表单的"耳朵" |
| **Code** | 数据处理 | 表单数据 | content 字段变数组 | 飞书多选字段必须传数组 |
| **HTTP(Token)** | API 调用 | App ID + Secret | tenant_access_token | 飞书 API 需要临时通行证 |
| **Code1** | 数据处理 | 表单数据 | 飞书格式的 fields | 字段名中英文映射 + 日期转时间戳 |
| **HTTP(写表格)** | API 调用 | fields + Token | record_id | 新增一条记录到多维表格 |
| **Email** | 发送邮件 | 表单原始数据 | — | 发确认邮件给报名者 |

### 数据怎么在节点间流动

n8n 里每个节点处理完数据，把结果传给下一个节点。后面节点可以通过表达式引用前面节点的数据：

```
{{ $('Webhook').first().json.body.name }}
    │         │       │    │    │
    │         │       │    │    └── 具体字段（报名者的姓名）
    │         │       │    └── body 对象
    │         │       └── JSON 数据
    │         └── 第一个数据项
    └── 引用名为 "Webhook" 的节点
```

### 为什么需要两个 HTTP Request 节点

飞书的 API 调用分两步：

1. **拿 Token**：用 App ID + Secret 换一个 2 小时有效的临时通行证
2. **写数据**：带着 Token 去写表格

这是飞书的 OAuth 认证机制——不能直接把 Secret 传给数据接口。

### 为什么需要两个 Code 节点

- **Code**：修内容领域格式。飞书多选字段需要数组 `["美妆护肤"]`，但表单只选一个时传来的是字符串 `"美妆护肤"`
- **Code1**：字段映射 + 格式转换。表单用英文 key（name, phone），飞书用中文列名（姓名, 手机号）；日期要转毫秒时间戳

---

## 四、飞书多维表格集成

### 需要什么

| 配置 | 从哪里拿 |
|------|------|
| App ID | 飞书开放平台 → 创建应用 |
| App Secret | 同上 |
| BASE ID | 多维表格 URL 里 `base/` 后面那串 |
| TABLE ID | URL 里 `table=` 后面那串 |
| 权限 | 开放平台开通 + 表格里添加应用 |

### 为什么会出现权限错误

飞书的安全机制有两道锁：

1. **应用权限**（开放平台里开通）——应用有资格访问多维表格
2. **表格授权**（飞书客户端里添加应用）——应用被允许访问**这张**表格

两道都过了才能读写。

### 字段类型匹配

飞书表格里不同列类型对数据格式有要求：

| 列类型 | 需要的格式 |
|------|------|
| 文本 | 字符串 |
| 电话 | 字符串（存数字就行） |
| 单选 | 字符串，必须匹配预设选项 |
| 多选 | **数组** `["选项1", "选项2"]` |
| 日期 | **毫秒时间戳** 或 `yyyy-MM-dd` 格式 |
| 邮箱 | 字符串 |

---

## 五、容器与虚拟化

### 为什么要用 Docker

n8n 自部署需要 Linux 环境。在 Windows 上直接跑很麻烦（依赖冲突、数据库配置等）。

Docker 把 n8n 和它需要的所有依赖打包成一个"胶囊"（容器），一条命令拉下来就能跑，不需要手动配置环境。

### 为什么访问 localhost:5678 就能用

```
docker run ... -p 5678:5678 ...
                └───┬───┘
                端口映射
```

`-p 5678:5678` 的意思是：把你电脑的 5678 端口，映射到容器内部的 5678 端口。

```
你的浏览器              容器内部
   │                      │
   └── localhost:5678 ────┘ -p 5678:5678
          ↓                   ↓
      你的电脑端口           n8n 监听端口
```

就像快递柜——快递员放进 5678 号柜子，你用钥匙打开拿件。

### Hyper-V → WSL 2 → Docker → n8n 的四层关系

```
┌──────────────────────────────────────┐
│              Windows 11              │
│  ┌────────────────────────────────┐  │
│  │          Hyper-V               │  │  ← Windows 自带虚拟机引擎
│  │  ┌──────────────────────────┐  │  │
│  │  │        WSL 2             │  │  │  ← 微软官方轻量 Linux 子系统
│  │  │  ┌────────────────────┐  │  │  │
│  │  │  │   Docker 引擎      │  │  │  │  ← 管理容器的工具
│  │  │  │  ┌──────────────┐  │  │  │  │
│  │  │  │  │  n8n 容器     │  │  │  │  │  ← 跑 n8n 的隔离环境
│  │  │  │  │  (端口5678)   │  │  │  │  │
│  │  │  │  └──────────────┘  │  │  │  │
│  │  │  └────────────────────┘  │  │  │
│  │  └──────────────────────────┘  │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

| 层级 | 谁做的 | 你需不需要管 |
|------|------|------|
| **Hyper-V** | 微软，Windows 自带 | 不需要 |
| **WSL 2** | 微软 + Docker Desktop 自动装 | 基本不需要 |
| **Docker Desktop** | Docker 公司 | 开机后打开就行 |
| **n8n 容器** | 你 `docker run` 创建的 | 需要知道怎么启动/停止 |

### Docker Desktop vs docker run

| Docker Desktop | docker run |
|------|------|
| 是引擎，后台一直跑 | 是命令，敲一下就行 |
| 管所有容器、镜像、网络 | 创建和启动一个具体容器 |
| 关了所有容器都停 | 容器启动后就靠引擎维持 |

### n8n 云版 vs 本地 Docker 版

| | 云版 | Docker 本地版 |
|------|------|------|
| 在哪跑 | n8n 公司服务器 | 你的电脑 |
| Webhook 激活 | 付费才给 | 免费，随时激活 |
| 优点 | 开机就能用 | 完全自由 |
| 缺点 | 免费版功能受限 | 关机就停 |
| 适合场景 | 临时测试 | 长期使用 |

---

## 六、常用命令速查

### Docker

```bash
docker ps                    # 查看运行中的容器
docker stop n8n              # 停止 n8n
docker start n8n             # 启动 n8n（容器已存在）
docker restart n8n           # 重启 n8n
docker logs n8n              # 查看 n8n 日志
docker run -d --name n8n -p 5678:5678 n8nio/n8n  # 首次创建并启动
```

### Git（本项目）

```bash
cd ~/Claude-codex
git add -A
git commit -m "说明"
git push origin main
# 同步到 GitHub Pages
git checkout gh-pages
git checkout main -- kol-recruitment-form.html
cp kol-recruitment-form.html index.html
git add -A
git commit -m "同步"
git push origin gh-pages
git checkout main
```

### WSL

```bash
wsl --status          # 查看状态
wsl --list --verbose  # 列出所有发行版
wsl --update          # 更新 WSL 内核
```

---

## 八、内网穿透

### 为什么需要

n8n 跑在本地 Docker 里（`localhost:5678`），只有你的电脑能访问。想让公网（GitHub Pages、手机）能 POST 到你的 webhook，需要把 localhost 暴露出去。

### 原理

```
公网请求 → 隧道服务器 → 加密隧道 → 你电脑上的隧道客户端 → localhost:5678
```

隧道服务器在公网上，你的电脑主动连接它，建立一条加密通道。公网请求打到服务器上，通过隧道转发到你的电脑。

### 方案对比

| | Cloudflare Tunnel | ngrok 免费版 |
|------|------|------|
| 域名 | 每次重启随机变 | 固定域名，不变 |
| 浏览器请求 | 直接转发 | 免费版会插警告页 |
| 稳定性 | 好 | 好 |
| 适用 | 临时测试 | 长期使用 |

### ngrok 固定域名

免费账号可申请 1 个固定域名。Docker 启动方式：

```bash
docker run -d --name ngrok --restart=always --net host \
  ngrok/ngrok http 5678 \
  --url=你的固定域名.ngrok-free.dev \
  --authtoken=你的Token
```

### ngrok 警告页绕过

ngrok 免费版在浏览器请求时会先返回警告页，POST 数据丢失。

**解决**：不用原生 HTML 表单提交，改用 JavaScript XHR 发请求，加请求头 `ngrok-skip-browser-warning: 1`。

```javascript
var xhr = new XMLHttpRequest();
xhr.open('POST', webhookUrl);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.setRequestHeader('ngrok-skip-browser-warning', '1');  // 关键
xhr.send(body);
```

⚠️ 加自定义头会触发 CORS 预检（OPTIONS 请求），需要隧道服务支持 CORS。ngrok 免费版支持。

---

## 九、JavaScript 表单提交方式演进

本项目经历了三种表单提交方式：

| 阶段 | 方式 | 为什么换 |
|------|------|------|
| 1 | fetch API + JSON | CORS 跨域被浏览器拦截 |
| 2 | 原生 `<form>` + iframe | ngrok 警告页拦截 |
| 3 | XHR + 手动拼参数 + ngrok 头 | ✅ 最终方案 |

### 最终方案关键点

**不用 FormData，手动拼参数**：

```javascript
// FormData + URLSearchParams 在移动端 select/radio 取值可能不准
// 手动遍历更稳定
var parts = [];
var els = form.querySelectorAll('input, select, textarea');
for (var i = 0; i < els.length; i++) {
  var el = els[i];
  if (!el.name) continue;                                    // 没 name 跳过
  if (el.type === 'checkbox' && !el.checked) continue;       // 没勾跳过
  if (el.type === 'radio' && !el.checked) continue;          // 没选跳过
  parts.push(encodeURIComponent(el.name) + '=' + encodeURIComponent(el.value));
}
var body = parts.join('&');
```

**不用 fetch，用 XHR**：

两者都能发请求，XHR 在旧移动浏览器兼容性更好，且行为更可控。

---

## 十、GitHub Actions CI/CD 排错

### YAML 语法敏感

YAML 对缩进和空格极其敏感：

```yaml
steps:
  - uses: xxx    # ✅ steps 缩进2空格，- 缩进再2空格，uses 和 - 之间1空格
  - uses:xxx     # ❌ 缺空格
   - uses: xxx    # ❌ 缩进多了
```

### GitHub Push Protection

GitHub 会扫描提交历史中的密钥（API Key、Token、Secret）。一旦检测到，拒绝 push。

**解决**：
1. 替换敏感值为占位符
2. 用 `git reset --soft HEAD~N` 回滚，重新提交干净版本
3. 或用 `git rebase -i` 清理历史

```bash
git reset --soft HEAD~2    # 撤销最近2个 commit，改动保留
# 修改文件，去除密钥
git add .
git commit -m "干净版本"
git push origin main
```

| 概念 | 一句话解释 |
|------|------|
| **Webhook** | 一个 URL，收到 POST 请求就触发工作流 |
| **CORS** | 浏览器不让不同域名之间发 AJAX 请求 |
| **CORS 预检** | 加自定义头时浏览器先发 OPTIONS 问"能发吗" |
| **内网穿透** | 把 localhost 变成公网 URL |
| **Tunnel** | 加密隧道，公网请求走隧道转发到本地 |
| **API** | 程序的对外接口，发 HTTP 请求就能用 |
| **Token** | 临时通行证，代替密码，过期要重拿 |
| **SMTP** | 发邮件的标准协议 |
| **端口映射** | 把电脑端口和容器端口连起来 |
| **容器** | 打包好的运行环境，搬家即用 |
| **镜像** | 容器的模板，相当于安装包 |
| **WSL 2** | Windows 里跑的轻量 Linux |
| **Hyper-V** | Windows 自带的虚拟机引擎 |

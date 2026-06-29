# Hermes Agent 部署包

> 一键部署 Hermes Agent 配置/技能/插件，支持三种路由方案

## 📦 目录结构

```
hermes-pack/
├── config/
│   ├── config.yaml            ← 完整配置（密钥已剔除）
│   ├── SOUL.md                ← Agent 人格设定
│   ├── .env.template          ← 环境变量模板
│   └── auth.json.template     ← 凭证模板
├── skills/software-development/
│   ├── screenlingua/          ← 截图翻译项目技能
│   ├── python-testing/        ← Python 测试约定
│   └── windows-development/   ← Windows 开发排坑
├── memories/MEMORY.md         ← 跨会话记忆参考
├── setup.ps1                  ← Windows 一键部署脚本
├── setup.sh                   ← Linux/macOS 一键部署脚本
└── README.md                  ← 本文件
```

## 🚀 快速部署

```powershell
# Windows（管理员 PowerShell）
git clone git@github.com:DTALEX66/hermes.git
cd hermes
.\setup.ps1
```

```bash
# Linux / macOS
git clone git@github.com:DTALEX66/hermes.git
cd hermes
chmod +x setup.sh && ./setup.sh
```

脚本自动完成：安装 Hermes → 写入配置 → 安装 3 个本地技能 → 安装依赖 → 启用 5 个插件 + 3 个工具集。

---

## 🧭 三种路由方案

根据当前机器的网络环境，选择最适合的方案：

### 方案A：Codex++ 本地代理中转

> **适用场景**：本机有 Codex++ 运行时（端口 57322），所有请求统一经过本地代理

```
Hermes ──→ Codex++ 代理 (127.0.0.1:57322) ──→ DeepSeek / 其他后端
```

```bash
# 启用方案A
hermes config set model.provider custom
hermes config set model.base_url http://127.0.0.1:57322/v1
hermes config set model.default deepseek-v4-flash
# api_key 从 ~/.codex/auth.json 的 OPENAI_API_KEY 读取
python3 -c "import json; key=json.load(open(r'~/.codex/auth.json'))['OPENAI_API_KEY']; __import__('subprocess').run(['hermes','config','set','model.api_key',key])"
```

**优点**：统一管理所有 API 请求，Hermes 和 Codex CLI 共享一条链路  
**缺点**：依赖 Codex++ 进程（端口 57322），换机器需换方案

---

### 方案B：DeepSeek 直连（默认）

> **适用场景**：任何能访问 `api.deepseek.com` 的机器，零依赖最稳定

```
Hermes ──→ DeepSeek API (api.deepseek.com)
```

```bash
# 启用方案B（默认）
hermes config set model.provider deepseek
hermes config set model.base_url https://api.deepseek.com/v1
hermes config set model.default deepseek-v4-flash
hermes config set model.api_key ''
# .env 中设置 DEEPSEEK_API_KEY=***
```

**优点**：零依赖，最稳定，新机器首选  
**缺点**：仅限 DeepSeek 模型

---

### 方案C：OpenAI Codex OAuth（GPT）

> **适用场景**：网络无限制（能访问 `chatgpt.com`），有 ChatGPT 订阅

```
Hermes ──→ ChatGPT Codex API (OAuth) ──→ GPT-4o / 其他
```

```bash
# 1️⃣ OAuth 认证（首次只需一次）
hermes auth add openai-codex
# 浏览器打开 https://auth.openai.com/codex/device，输入验证码

# 2️⃣ 启用方案C
hermes config set model.provider openai-codex
hermes config set model.default gpt-4o
hermes config set model.base_url ''
hermes config set model.api_key ''
```

**优点**：用 ChatGPT 订阅直接调 GPT 模型，无需 API Key  
**缺点**：需要能访问 `chatgpt.com` 的网络环境

---

## 🔄 快速切换方案

`config.yaml` 中已预置三种方案的注释模板，直接编辑切换：

```yaml
# 切换方案A（Codex++ 代理）
model:
  default: deepseek-v4-flash
  provider: custom
  base_url: http://127.0.0.1:57322/v1
  api_key: sk-...

# 切换方案B（DeepSeek 直连）
model:
  default: deepseek-v4-flash
  provider: deepseek
  base_url: https://api.deepseek.com/v1
  api_key: ''

# 切换方案C（GPT OAuth）
model:
  default: gpt-4o
  provider: openai-codex
  base_url: ''
  api_key: ''
```

> ⚠️ 切换方案后需 `/reset` 或重启 Hermes 生效

---

## 🔑 部署后手动配置

### 1. API Key

编辑 `%LOCALAPPDATA%\hermes\.env`：

```env
# 方案B 需要
DEEPSEEK_API_KEY=你的DeepSeek密钥

# 方案C 不需要（OAuth 无密钥）
# 方案A 用到的是 Codex 已有的 Key，自动读取
```

### 2. 加载项目技能

```bash
hermes -s screenlingua
skill_view(name='screenlingua')
```

### 3. 插件生效

插件和工具集需新会话生效：`/reset` 或重启 Hermes。

## 🛠️ 常用命令

```bash
hermes config               # 查看完整配置
hermes config edit          # 编辑配置
hermes model                # 交互式切换模型
hermes skills list          # 查看已安装技能
hermes plugins list         # 查看已安装插件
hermes tools list           # 查看已启用工具集
hermes doctor               # 环境健康检查
hermes auth list            # 查看 OAuth 凭证
```

## ❌ 不包含的内容（需手动配置）

- API 密钥（`.env`）
- OAuth 令牌（`auth.json`）
- 会话历史（`state.db`）
- cron 运行时数据
- 缓存文件和日志

## 📝 备注

- 内置 62 个技能由 Hermes 自动安装
- 部署包仅包含 3 个本地自定义技能（screenlingua/python-testing/windows-dev）
- 建议部署后运行 `hermes doctor` 做全面检查
- 如果 Codex++ 被删除，切到方案B 即可

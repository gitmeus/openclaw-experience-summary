# CLIProxyAPI 完全教程：统一你的 AI CLI 工具

## 🎯 什么是 CLIProxyAPI？

**CLIProxyAPI** 是一个开源的代理服务器，它可以将各种 AI CLI 工具包装成 **OpenAI/Gemini/Claude/Codex 兼容的 API 服务**。

简单说：**一个 API，搞定所有 AI 命令行工具！** 🚀

---

## ✨ 核心亮点

| 功能 | 说明 |
|------|------|
| 🌐 **统一协议** | 使用 OpenAI 兼容接口访问所有模型 |
| 🔄 **多账号负载** | Gemini/OpenAI/Claude/Qwen 多账号轮询 |
| 📱 **OAuth 登录** | 无需 API Key，用你的订阅直接登录 |
| ⚡ **流式响应** | 支持流式和非流式输出 |
| 🔧 **工具调用** | 完整支持 function calling |
| 🖼️ **多模态** | 文字 + 图片输入 |
| 🎯 **智能路由** | 自动 fallback 到可用模型 |

---

## 🛠️ 支持的工具和模型

### 直接支持（通过 OAuth）

| 工具 | 模型示例 | 说明 |
|------|----------|------|
| **Gemini CLI** | Gemini 2.5 Pro, Flash | 免费版可用 |
| **ChatGPT Codex** | GPT-4.5, GPT-5 | 需有 Codex 订阅 |
| **Claude Code** | Claude Sonnet, Opus | 需有 Claude 订阅 |
| **Qwen Code** | Qwen-Plus, Max | 通义千问代码助手 |
| **iFlow** | iFlow 模型 | 第三方服务 |

### 兼容客户端

任何支持 OpenAI API 的客户端都能用：

- **Amp CLI** 及其 IDE 扩展
- **OpenCode**、**Cursor**、**Claude Code**
- 自定义脚本/应用

---

## 📦 安装与部署

### 方式一：Docker（推荐）

```bash
# 拉取镜像
docker pull routerforme/cliproxyapi:latest

# 运行
docker run -d \
  --name cliproxyapi \
  -p 8080:8080 \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/data:/app/data \
  routerforme/cliproxyapi:latest
```

### 方式二：本地二进制

```bash
# 下载最新版本
wget https://github.com/router-for-me/CLIProxyAPI/releases/latest/download/cliproxyapi-linux-amd64
chmod +x cliproxyapi-linux-amd64

# 运行
./cliproxyapi-linux-amd64
```

### 方式三：源码编译

```bash
git clone https://github.com/router-for-me/CLIProxyAPI.git
cd CLIProxyAPI
go build -o cliproxyapi .
./cliproxyapi
```

---

## ⚙️ 配置说明

CLIProxyAPI 使用 YAML 配置文件，默认路径：

- Linux: `~/.config/cliproxyapi/config.yaml`
- macOS: `~/Library/Application Support/cliproxyapi/config.yaml`

### 基础配置示例

```yaml
server:
  port: 8080
  host: "0.0.0.0"

providers:
  gemini:
    accounts:
      - email: "user1@gmail.com"
        # OAuth 会自动处理
      - email: "user2@gmail.com"
        # 多账号轮询

  openai:
    oauth:
      client_id: "your-client-id"
      client_secret: "your-secret"
    accounts:
      - email: "codex1@example.com"
      - email: "codex2@example.com"

  claude:
    accounts:
      - email: "claude-user@example.com"

  qwen:
    accounts:
      - email: "qwen-user@example.com"

logging:
  level: "info"
  file: "cliproxyapi.log"
```

### 进阶配置：模型路由

```yaml
routes:
  - pattern: "gpt-*"
    provider: "openai"
    fallback: ["gemini", "claude"]
  - pattern: "claude-*"
    provider: "claude"
    fallback: ["gemini"]
  - pattern: "gemini-*"
    provider: "gemini"
    priority: 1

load_balancing:
  strategy: "round_robin"  # 或 "random"
  health_check: true
```

---

## 🔑 OAuth 配置（关键）

### Gemini/OpenAI/Claude OAuth

1. 前往对应平台开发者控制台创建 OAuth 应用
2. 获取 `client_id` 和 `client_secret`
3. 配置回调 URL 为 `http://localhost:8080/oauth/callback`
4. 填入配置文件

首次运行时，CLIProxyAPI 会打开浏览器让你登录授权。

### 一次性认证流程

```bash
# 启动服务后访问
http://localhost:8080/auth/gemini
# 浏览器会打开 Gemini 登录页面，完成授权后自动回调
```

---

## 🌐 API 使用示例

### 标准 OpenAI 格式

```bash
# 基本对话
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-2.5-pro",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'

# 流式响应
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4",
    "messages": [{"role": "user", "content": "Write a poem"}],
    "stream": true
  }'

# 多模态（图片）
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "user",
        "content": [
          {"type": "text", "text": "Describe this image"},
          {"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}}
        ]
      }
    ]
  }'
```

### 在 Claude Code 中使用

设置环境变量：

```bash
export ANTHROPIC_API_BASE=http://localhost:8080
export ANTHROPIC_API_KEY=dummy-key  # 任意值，OAuth 会处理
claude "你好，帮我写个 Python 脚本"
```

### 在 Cursor/OpenCode 中使用

在设置中修改 API Endpoint：

```
Base URL: http://localhost:8080/v1
API Key: any-value
```

---

## 📊 多账号负载均衡

CLIProxyAPI 自动管理多个账号 quota，当某个账号超额时自动切换到下一个。

**配置示例**：

```yaml
providers:
  gemini:
    accounts:
      - email: "alice@gmail.com"
        priority: 1
      - email: "bob@gmail.com"
        priority: 2
      - email: "charlie@gmail.com"
        priority: 3
```

**使用策略**：
- `round_robin`：轮流使用
- `random`：随机选择
- `priority`：按优先级，低 quota 账号 down

---

## 🎯 实际使用场景

### 场景 1：免费使用 Gemini Pro

1. 你有 3 个 Google 账号
2. 配置到 `gemini.accounts`
3. CLIProxyAPI 开始轮询，每个账号都有配额
4. 通过 OpenAI 接口任意调用

### 场景 2：统一 Claude Code 和 GPT-4

开发时想用 Claude 写代码，但有时需要 GPT：
- 设置路由规则，优先 Claude，fallback GPT
- 客户端只需一个 API endpoint

### 场景 3：团队共享 AI 资源

- 部署 CLIProxyAPI 到服务器
- 团队成员通过内网访问
- 统一配额管理，避免个人账号超额

---

## 🐛 常见问题

### Q1: OAuth 授权失败？

**检查点**：
1. 确保回调 URL 正确：`http://localhost:8080/oauth/callback`
2. 防火墙/代理是否允许本地访问
3. 查看日志：`tail -f cliproxyapi.log`
4. 删除 `~/.config/cliproxyapi/` 下缓存重试

### Q2: 模型列表为空？

```bash
# 手动刷新
curl http://localhost:8080/admin/refresh

# 或重启服务
systemctl restart cliproxyapi
```

### Q3: 配额用尽怎么办？

- 添加更多账号
- 配置 `fallback` 到其他 provider
- 检查 `quota` 页面看哪个账号超额

### Q4: 如何监控使用情况？

访问管理 API（需配置管理密钥）：

```bash
curl -H "X-Admin-Key: your-secret" \
  http://localhost:8080/admin/stats
```

### Q5: 支持 Ollama 或本地模型吗？

目前不支持，但你可以配置 `openrouter` 作为上游 provider。

---

## 📚 相关项目生态

CLIProxyAPI 有很多衍生工具：

| 项目 | 用途 | 平台 |
|------|------|------|
| **vibeproxy** | macOS menu bar 管理 Claude/GPT 订阅 | macOS |
| **Subtitle Translator** | 字幕翻译（基于 Gemini） | Web |
| **CCS** (Claude Code Switch) | 快速切换 Claude 账号/模型 | CLI |
| **ProxyPal** | macOS GUI 管理 CLIProxyAPI | macOS |
| **Quotio** | 配额追踪 + 智能 failover | macOS |
| **CodMate** | 完整 AI 编程助手管理 | macOS |
| **ProxyPilot** | Windows TUI 管理工具 | Windows |
| **ZeroLimit** | Windows 配额监控面板 | Windows |
| **霖君** | 跨平台统一管理 | macOS/Win/Linux |
| **CPA-XXX Panel** | Web 管理后台 | Web |

---

## 🔒 安全建议

- ⚠️ 不要暴露 `8080` 端口到公网（除非配置了认证）
- ✅ 生产环境使用反向代理 + HTTPS + Basic Auth
- ✅ 定期更新到最新版本
- ✅ `admin` API 必须设置强密钥
- ✅ 配置文件不要提交到 Git

---

## 📈 维护与更新

```bash
# 查看状态
curl http://localhost:8080/health

# 强制刷新模型缓存
curl -X POST http://localhost:8080/admin/refresh

# 查看日志（Docker）
docker logs -f cliproxyapi

# 更新镜像
docker pull routerforme/cliproxyapi:latest
docker restart cliproxyapi
```

---

## 📝 快速检查清单

- [ ] 选择部署方式（Docker / 二进制 / 源码）
- [ ] 配置 `config.yaml`（至少一个 provider）
- [ ] 设置 OAuth 凭据
- [ ] 启动服务并验证 `http://localhost:8080/health`
- [ ] 测试 API 调用 `curl /v1/chat/completions`
- [ ] 配置客户端（Claude Code / Cursor 等）
- [ ] 生产环境加固（HTTPS、认证、监控）

---

## 📮 获取帮助

- 📖 官方文档：https://help.router-for.me/
- 💬 GitHub Issues：https://github.com/router-for-me/CLIProxyAPI/issues
- 🆘 中文文档：https://help.router-for.me/cn/
- ⭐ 如果觉得好用，来给个 Star 吧！

---

## 🎉 总结

CLIProxyAPI 是 **AI 编程助手多账号管理的终极解决方案**：

- ✅ 统一 API，兼容所有主流客户端
- ✅ 免费使用多个 Gemini/OpenAI/Claude 账号
- ✅ 自动负载均衡 + 智能 fallback
- ✅ 丰富的生态工具（macOS/Windows GUI）
- ✅ 开源免费，MIT 协议

**适合人群**：
- 使用 Claude Code、Cursor、OpenCode 等工具的开发者
- 拥有多个 AI 订阅，不想频繁切换
- 想要统一管理配额和账号的团队

---

<div align="center">

**🚀 开始你的统一 AI 之旅！**

[GitHub](https://github.com/router-for-me/CLIProxyAPI) | [帮助文档](https://help.router-for.me/) | [生态项目](#相关项目生态)

</div>

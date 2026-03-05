# 小米音箱接入 AI 教程：让小爱同学连接 DeepSeek/OpenAI/Gemini

## 🎯 什么是 MiGPT-Next？

**MiGPT-Next** 是基于 [MiGPT](https://github.com/idootop/mi-gpt) 的升级版本，让小米智能音箱（小爱同学）接入大语言模型（AI）。从此小爱同学不再是简单的语音助手，而是能够与您智能对话、回答问题、执行任务的 AI 伙伴。

---

## ✨ 核心功能

| 功能 | 说明 |
|------|------|
| **自定义 AI 回复** | 让小爱音箱说出 AI 的回答，而不仅是内置回答 |
| **多模型支持** | DeepSeek、OpenAI (GPT-4/GPT-3.5)、Gemini、Claude 等 |
| **关键词触发** | 配置触发词（如"小爱同学，AI"）才调用 AI |
| **自定义提示词** | 可以定义 AI 的角色和回复风格 |
| **多平台部署** | Docker、Node.js、软路由 (iStoreOS)、NAS 均可 |
| **开源免费** | MIT 协议，无使用成本 |

---

## 🚀 快速开始（5 分钟部署）

### 前置准备

1. ✅ 小米智能音箱（任何支持 MiOT 的小爱音箱）
2. ✅ 小米账号（知道 userId 和密码）
3. ✅ AI API 密钥（任选一个）：
   - DeepSeek：https://platform.deepseek.com/api_keys
   - OpenAI：https://platform.openai.com/api-keys
   - 其他兼容 OpenAI 格式的 API（如 OneAPI、中转服务）
4. ✅ 一台可运行 Docker 的设备（软路由、NAS、服务器、甚至电脑）

### 部署步骤

#### 步骤 1：拉取镜像并创建配置文件

在您的设备上执行：

```bash
# 1. 创建项目目录
mkdir -p /opt/migpt-next
cd /opt/migpt-next

# 2. 创建配置文件 config.js
nano config.js  # 或用 vim、vi 等任意编辑器
```

将以下内容粘贴到 `config.js`，并**修改红色部分**：

```javascript
export default {
  speaker: {
    userId: "你的小米ID（不是手机号/邮箱）",
    password: "你的小米账号密码",
    passToken: "从浏览器Cookie复制出来的passToken",
    did: "你米家里的音箱设备名（一字不差）或DID",
  },

  openai: {
    // 支持的模型：
    // DeepSeek: "deepseek-chat" 或 "deepseek-reasoner"
    // OpenAI: "gpt-4o-mini", "gpt-4o", "gpt-4-turbo", "gpt-3.5-turbo"
    model: "deepseek-chat",
    baseURL: "https://api.deepseek.com/v1",  // 或 https://api.openai.com/v1
    apiKey: "你的API密钥",
  },

  prompt: {
    system: "你是一个智能助手，请根据用户的问题给出简洁、准确、友好的回答。",
  },

  // 自定义消息处理（可选）
  async onMessage(engine, msg) {
    console.log("收到消息：", msg.text);
    // 只有包含"测试"才回复固定内容
    if (msg.text.includes("测试")) {
      return { text: "测试成功！AI 已连接～" };
    }
    // 可以在这里添加更多自定义逻辑
  },
};
```

**如何获取配置参数？**

| 参数 | 获取方法 |
|------|----------|
| `userId` | 登录小米账号 https://account.xiaomi.com → 点击个人信息页面 → URL 中的 `userId` |
| `password` | 小米账号密码 |
| `passToken` | 浏览器登录小米账号 → F12 → Application → Cookies → 找到 `passToken` 值 |
| `did` / 设备名 | 小米 home 米家 App → 智能家居 → 点击音箱设备 → 查看设备名称（必须一字不差） |
| `apiKey` | 对应 AI 服务商的控制台获取 |

#### 步骤 2：运行容器

```bash
# 拉取镜像
docker pull idootop/migpt-next:latest

# 前台运行（调试用，能看到实时日志）
docker run -it --rm \
  -v /opt/migpt-next/config.js:/app/config.js \
  idootop/migpt-next:latest

# 如果运行正常，按 Ctrl+C 停止，然后后台常驻
docker run -d --name migpt-next \
  --restart=unless-stopped \
  -v /opt/migpt-next/config.js:/app/config.js \
  idootop/migpt-next:latest
```

#### 步骤 3：测试

对小米音箱说：
> **"小爱同学，测试"**

如果配置正确，小爱会播放："测试成功！AI 已连接～"

---

## ⚙️ 详细配置指南

### 1. 获取小米账号信息

#### 方法一：通过米家 App（推荐）
- 打开米家 App → 我的 → 账户信息 → 查看用户 ID
- 设备名称：智能家居 → 选择音箱 → 设备详情页顶部显示名称

#### 方法二：通过浏览器（获取 passToken）
1. 访问 https://account.xiaomi.com 并登录
2. 按 **F12** 打开开发者工具
3. 切换到 **Application**（或 Storage）标签
4. 左侧 Cookies → `https://account.xiaomi.com`
5. 搜索 `passToken`，复制其值（一串长字符串）
6. 如果找不到，尝试在登录后刷新页面再查看

#### 注意
- `passToken` 可能过期，如果失效需要重新登录获取
- 部分新版小米账号可能使用其他验证方式，可参考项目 Issue 讨论

### 2. AI 服务选择

#### DeepSeek（推荐，性价比高）
- 注册：https://platform.deepseek.com
- API 密钥：控制台 → API Keys → Create API Key
- `baseURL`: `https://api.deepseek.com/v1`（或 `https://api.deepseek.com`）
- 模型：
  - `deepseek-chat`（V3，通用对话）
  - `deepseek-reasoner`（R1，推理能力更强）

#### OpenAI（最稳定）
- 注册：https://platform.openai.com
- API 密钥：API Keys → Create new secret key
- `baseURL`: `https://api.openai.com/v1`
- 模型：
  - `gpt-4o-mini`（性价比高）
  - `gpt-4o`（全能旗舰）
  - `gpt-4-turbo`
  - `gpt-3.5-turbo`

#### Gemini（Google）
- 注册：https://aistudio.google.com
- API 密钥：左侧 Get API Key
- `baseURL`: `https://generativelanguage.googleapis.com/v1beta`
- 模型：`gemini-2.0-flash` 等

### 3. 提示词系统

在 `config.js` 的 `prompt.system` 中设置 AI 的角色：

```javascript
prompt: {
  // 示例：设置 AI 为专业程序员
  system: "你是一个资深程序员，擅长解释技术问题。回答简洁明了，使用代码块。",
}
```

或者设置成其他角色：
- 翻译官："你是一个专业翻译，将用户输入精准翻译成英文..."
- 老师："你是一位耐心的老师，用通俗易懂的方式讲解概念..."
- 客服："你是一个客服助手，以友好、专业的方式回答问题..."

### 4. 高级配置选项

完整的配置参数可以参考项目主 README。额外常用参数：

```javascript
{
  // 调用 AI 的关键词（触发词）
  callAIKeywords: ["AI", "人工智能", "大模型"],
  
  // 是否连续对话（需要设备支持 TTS）
  continuousConversation: false,
  
  // 流式响应（默认开启，延迟更低）
  stream: true,
}
```

---

## 🎬 视频教程

👉 **观看完整演示**：

[![小米音箱接入 AI 教程](https://img.youtube.com/vi/yXbApmUg-K0/0.jpg)](https://www.youtube.com/watch?v=yXbApmUg-K0)

视频内容包括：
- 软路由 iStoreOS 部署 MiGPT-Next
- Docker 运行的完整流程
- 小米账号信息获取（passToken、userId）
- DeepSeek API 密钥申请
- 配置文件编写详解
- 测试与调试
- 常见问题解决

---

## 📦 部署平台对比

| 平台 | 优点 | 缺点 |
|------|------|------|
| **软路由 / iStoreOS** | 24小时运行，内网低延迟 | 需要特定固件支持 |
| **NAS (群晖/威联通)** | 稳定供电，随时可用 | 需要安装 Docker |
| **云服务器** | 公网可访问，永不关机 | 有持续费用 |
| **电脑/笔记本** | 快速测试，无额外成本 | 需要开机，不稳定 |
| **树莓派** | 低功耗，嵌入式 | 性能有限 |

**无论哪种平台，最终都是运行一个 Docker 容器**，所以配置文件是通用的。

---

## 🔧 常见问题

### Q1: 运行后一直提示"登录失败"？
**答**：可能是小米账号触发了安全验证。解决方案：
1. 在小米账号设置中开启"允许第三方应用登录"
2. 或参考项目 Issue #4：https://github.com/idootop/migpt-next/issues/4

### Q2: 小爱同学总是抢话，等 AI 回答时自己也在说话？
**答**：这是正常现象，新版本移除了连续对话/流式响应功能，因为延迟较大难以打断。建议：
- 在 `onMessage` 中手动调用 `engine.speaker.abortXiaoAI()` 打断
- 详见 FAQ 中的代码示例

### Q3: 控制台能看到 AI 回答，但音箱没声音？
**答**：TTS 可能失效。请修改 `onMessage` 函数，使用设备实际的 `ttsCommand` 参数：
```javascript
await engine.MiOT.doAction(5, 1, text); // 5,1 替换为你的设备值
```

### Q4: DeepSeek API 报 429 或时间段错误？
**答**：DeepSeek API 有速率限制，免费额度有限。可以：
- 升级到付费套餐
- 切换到 OpenAI API
- 使用中转服务（OneAPI 聚合）

### Q5: 让音箱只对特定关键词响应（避免误触发）？
**答**：配置 `callAIKeywords` 参数，例如：
```javascript
callAIKeywords: ["AI", "人工智能", "大模型", "智能助手"],
```
只有包含这些关键词才会调用 AI，否则走小爱原生逻辑。

---

## 🎓 进阶玩法

### 1. 多设备同步

如果有多个小爱音箱，可以：
- 复制容器，用不同 `config.js` 文件
- 或使用共享存储，通过 `did` 区分设备

### 2. 自定义回复规则

在 `onMessage` 中编写 JavaScript 代码，实现复杂逻辑：

```javascript
async onMessage(engine, msg) {
  const { text } = msg;
  
  // 查询天气
  if (text.includes("天气")) {
    const weather = await fetchWeather(text);
    return { text: `今天${weather.city}天气：${weather.condition}，${weather.temp}度` };
  }
  
  // 其他命令...
}
```

### 3. 接入本地模型

如果家里有运行本地大模型（Ollama、LM Studio），可以：
```javascript
openai: {
  baseURL: "http://localhost:11434/v1",  // Ollama 默认地址
  apiKey: "ollama",  // 不需要真实密钥
  model: "llama3.2:3b",  // 本地模型名
}
```

**优点**：完全免费、隐私安全、速度取决于本机性能

---

## 📊 功能对比表

| 特性 | MiGPT-Next | 原版小爱 | 其他第三方方案 |
|------|------------|----------|----------------|
| AI 对话 | ✅ 多模型支持 | ❌ 仅内置 | 部分支持 |
| 自定义提示词 | ✅ 完全自由 | ❌ 有限 | 有限 |
| 本地部署 | ✅ Docker 一键 | ❌ 云端 | 部分支持 |
| 免费使用 | ✅ 仅 API 费用 | ✅ 免费 | 可能有订阅费 |
| 隐私保护 | ✅ 数据不出本地 | ⚠️ 小米云端 | 取决于部署 |
| 多设备同步 | ✅ 支持 | ✅ | 视方案而定 |

---

## 🔗 相关资源

- [MiGPT-Next GitHub 仓库](https://github.com/idootop/migpt-next)
- [MiGPT 原版项目](https://github.com/idootop/mi-gpt)
- [演示视频](https://www.youtube.com/watch?v=yXbApmUg-K0)
- [小米账号获取教程](https://naiyous.com/10279.html)（外链）
- [DeepSeek 开放平台](https://platform.deepseek.com)
- [OpenAI API 文档](https://platform.openai.com/docs)

---

## ✅ 快速检查清单

- [ ] 准备小米账号（userId、password、passToken）
- [ ] 准备 AI API 密钥（DeepSeek/OpenAI/Gemini 三选一）
- [ ] 准备 Docker 运行环境（软路由/NAS/云服务器）
- [ ] 创建 `config.js` 配置文件并填写所有参数
- [ ] 运行 Docker 容器并查看日志
- [ ] 对小爱同学说测试命令，确认 AI 回复正常
- [ ] 调整提示词，让 AI 扮演您喜欢的角色
- [ ] （可选）配置开机自启和后台常驻
- [ ] （可选）设置自定义关键词避免误触发

---

**恭喜！您的小米音箱现在拥有真正的 AI 大脑！** 🎉

让小爱同学成为您的生活助手、学习伙伴或娱乐玩伴吧！

---

*教程创建日期*：2025-06-17  
*参考资源*：MiGPT-Next GitHub 仓库、奶油之家教程  
*适用版本*：MiGPT-Next latest

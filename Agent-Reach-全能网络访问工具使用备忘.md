# Agent Reach 使用备忘：解锁全网访问能力

## 📦 安装信息

- **项目地址**: https://github.com/Panniantong/agent-reach
- **安装方式**: `pip install https://github.com/Panniantong/agent-reach/archive/main.zip`
- **版本**: v1.3.0
- **Python 要求**: >= 3.10（使用 python3.11 安装）
- **配置命令**: `agent-reach install --env=auto`

---

## 🎯 核心能力

Agent Reach 是一个**网络访问工具集**，为 AI Agent 提供完整的互联网访问能力。它本身不是工具，而是一个**上游工具的安装器和管理器**。

安装后，您可以直接使用这些上游工具：
- `xreach` - Twitter/X 搜索和发布
- `yt-dlp` - YouTube、Bilibili 视频下载和信息提取
- `gh` CLI - GitHub 完整功能
- `mcporter` - MCP 服务器管理（小红书、抖音、LinkedIn 等）

---

## 📊 当前状态（5/13 可用）

### ✅ 开箱即用（无需配置）

| 渠道 | 功能 | 工具 |
|------|------|------|
| **GitHub** | 仓库读取、搜索、Fork、Issue、PR | `gh` CLI |
| **YouTube** | 视频信息、字幕提取 | `yt-dlp` |
| **Bilibili** | 视频信息、字幕提取 | `yt-dlp` |
| **RSS/Atom** | 订阅源读取 | `feedparser` |
| **任意网页** | 通过 Jina Reader 提取内容 | `curl + r.jina.ai` |

### ⬜ 需简单配置

| 渠道 | 所需配置 | 说明 |
|------|----------|------|
| **全网语义搜索** | `npm install -g mcporter`<br>`mcporter config add exa https://mcp.exa.ai/mcp` | Exa 搜索引擎 MCP |
| **Twitter/X** | `npm install -g xreach-cli` + Cookie | 搜索和发布推文 |
| **Reddit** | `agent-reach configure proxy URL` | 服务器需要住宅代理 |
| **小红书** | Docker + MCP + Cookie | 需要扫描二维码登录 |
| **抖音** | MCP 服务 + 可能硅基 API | 视频解析、无水印下载 |
| **LinkedIn** | Chromium + 浏览器登录 | 完整 Profile 搜索 |
| **Boss直聘** | Playwright + 扫码登录 | 职位搜索和打招呼 |
| **微信公众号** | 额外 pip 包安装 | 文章读取和搜索 |

---

## 🚀 快速命令参考

```bash
# 查看版本
agent-reach --version

# 健康检查（查看所有渠道状态）
agent-reach doctor

# 快速检查（用于定时任务）
agent-reach watch

# 检查更新
agent-reach check-update

# 配置（示例）
agent-reach configure twitter-cookies "你的Cookie字符串"
agent-reach configure proxy http://user:pass:ip:port
```

---

## 🔧 常见配置步骤

### 1. 全网语义搜索（Exa）

```bash
# 安装 mcporter
npm install -g mcporter

# 配置 Exa MCP（需要 Exa API Key）
mcporter config add exa https://mcp.exa.ai/mcp
# 按照提示输入 API Key
```

### 2. Twitter/X 访问

```bash
# 安装 xreach CLI
npm install -g xreach-cli

# 获取 Cookie（推荐使用 Cookie-Editor 插件）
# 1. 安装 Cookie-Editor Chrome 扩展
# 2. 登录 twitter.com
# 3. 点击插件 → Export → Header String
# 4. 复制字符串

# 配置 Cookie
agent-reach configure twitter-cookies "粘贴的字符串"
```

### 3. Reddit（服务器环境）

```bash
# 获取住宅代理（如 Webshare $1/月）
# 配置代理
agent-reach configure proxy http://user:pass@ip:port
```

### 4. 小红书（需要 Docker）

```bash
# 启动小红书 MCP 服务
docker run -d --name xiaohongshu-mcp -p 18060:18060 xpzouying/xiaohongshu-mcp

# 注册到 mcporter
mcporter config add xiaohongshu http://localhost:18060/mcp

# 登录方式（Cookie-Editor）：
# 1. 在小红书网页版登录
# 2. 用 Cookie-Editor 导出 Header String
# 3. Agent 会自动写入 MCP 的 cookie 文件
```

### 5. 抖音视频解析

```bash
# 安装 douyin-mcp-server
pip install douyin-mcp-server

# 启动 HTTP 服务（端口 18070）
# 方式一：使用 uv（推荐）
mkdir -p ~/.agent-reach/tools && cd ~/.agent-reach/tools
git clone https://github.com/yzfly/douyin-mcp-server.git && cd douyin-mcp-server
uv sync && uv run python run_http.py

# 方式二：直接用 Python
python -c "
from douyin_mcp_server.server import mcp
mcp.settings.host = '127.0.0.1'
mcp.settings.port = 18070
mcp.run(transport='streamable-http')
"

# 注册到 mcporter
mcporter config add douyin http://localhost:18070/mcp
```

### 6. LinkedIn（需要浏览器界面）

```bash
# 安装
pip install linkedin-scraper-mcp

# 本地电脑（有桌面）：直接扫码登录
linkedin-scraper-mcp --login --no-headless

# 服务器（无 UI）：通过 VNC
# 1. 安装 VNC 并启动
# apt install -y tigervnc-standalone-server
# vncserver :1 -geometry 1280x720
# 2. VNC 客户端连接服务器IP:5901
# 3. 在 VNC 终端运行：
export DISPLAY=:1
linkedin-scraper-mcp --login --no-headless

# 登录后启动 MCP 服务
linkedin-scraper-mcp --transport streamable-http --port 8001
mcporter config add linkedin http://localhost:8001/mcp
```

### 7. Boss直聘（需要扫码）

```bash
# 克隆仓库
mkdir -p ~/.agent-reach/tools && cd ~/.agent-reach/tools
git clone https://github.com/mucsbr/mcp-bosszp.git && cd mcp-bosszp
pip install -r requirements.txt
playwright install chromium

# 启动服务并扫码登录
python boss_zhipin_fastmcp_v2.py

# 通过 MCP 触发登录生成二维码
mcporter call 'bosszhipin.start_login()'
# 浏览器打开二维码图片完成扫码

# 注册到 mcporter
mcporter config add bosszhipin http://localhost:8000/mcp
```

---

## 🗺️ 工具映射表

安装完上游工具后，Agent 可以直接调用：

| 平台 | 上游工具 | 示例命令 |
|------|----------|----------|
| Twitter/X | `xreach` | `xreach search "query" --json` |
| YouTube | `yt-dlp` | `yt-dlp --dump-json URL` |
| Bilibili | `yt-dlp` | `yt-dlp --dump-json URL` |
| Reddit | `curl` | `curl -s "https://reddit.com/r/xxx.json"` |
| GitHub | `gh` | `gh search repos "query"` |
| 任意网页 | `curl + Jina` | `curl -s "https://r.jina.ai/URL"` |
| Exa 搜索 | `mcporter` | `mcporter call 'exa.web_search_exa(...)'` |
| 小红书 | `mcporter` | `mcporter call 'xiaohongshu.search_feeds(...)'` |
| 抖音 | `mcporter` | `mcporter call 'douyin.parse_douyin_video_info(...)'` |
| LinkedIn | `mcporter` | `mcporter call 'linkedin.get_person_profile(...)'` |
| Boss直聘 | `mcporter` | `mcporter call 'bosszhipin.search_jobs_tool(...)'` |
| RSS | `feedparser` | `python3 -c "import feedparser; ..."` |

---

## 🔐 安全建议

### Cookie 使用

- **优先使用次要账号**：对于需要 Cookie 的平台（Twitter、小红书等），不要用主号
- **原因**：
  1. 平台可能检测非浏览器 API 调用并封号
  2. Cookie 泄露的风险，次要号限制损失

### Cookie 导入方式（推荐）

使用 **Cookie-Editor** Chrome 插件导出：
1. 安装 [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
2. 在目标网站登录
3. 点击插件 → Export → Header String
4. 复制字符串给 Agent

这是最简单最可靠的方式。

---

## 🕵️‍♂️ 故障排除

### `agent-reach` 命令找不到

```bash
# 检查是否安装在 PATH 中
which agent-reach

# 如果找不到，可能需要添加用户 bin 目录
export PATH="$HOME/.local/bin:$PATH"
# 或永久添加到 ~/.zshrc / ~/.bashrc
```

### 某个渠道显示 ❌

运行 `agent-reach doctor` 查看具体原因，然后：
- 缺少依赖？按提示安装
- 需要配置？参考上方配置步骤
- 网络问题？检查代理设置

### yt-dlp 版本太旧

```bash
# 升级 yt-dlp
python3.11 -m pip install -U yt-dlp
```

### mcporter 无法连接 MCP 服务

```bash
# 确认服务在运行
curl http://localhost:18060/mcp  # 或其他端口

# 查看 mcporter 配置
mcporter config list
```

---

## 🕐 定时监控（OpenClaw 特有）

如果使用 OpenClaw，可以设置每日自动检查：

```bash
# 创建 cron 任务（每天一次）
agent-reach watch
# 如果输出包含"全部正常"，静默
# 如果包含问题（❌ ⚠️）或新版本（🆕），发送报告
```

设置命令：
```bash
cron add --expr "0 9 * * *" --session-target isolated \
  --payload '{"kind":"agentTurn","message":"运行 agent-reach watch 命令。如果输出包含\"全部正常\"，不需要通知用户，静默结束。如果输出包含问题（❌ ⚠️）或新版本（🆕），把完整报告发给用户，并建议修复方案。如果有新版本可用，问用户是否要升级（升级命令：pip install --upgrade https://github.com/Panniantong/agent-reach/archive/main.zip）。"}'
```

---

## 📚 参考资源

- **项目仓库**: https://github.com/Panniantong/agent-reach
- **安装文档**: https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/install.md
- **Exa MCP**: https://mcp.exa.ai/mcp
- **小红书 MCP**: https://github.com/xpzouying/xiaohongshu-mcp
- **抖音 MCP**: https://github.com/yzfly/douyin-mcp-server
- **LinkedIn MCP**: https://github.com/stickerdaniel/linkedin-mcp-server
- **Boss直聘 MCP**: https://github.com/mucsbr/mcp-bosszp

---

## ✅ 完成检查

- [x] Agent Reach v1.3.0 已安装
- [x] 基本渠道可用（GitHub、YouTube、Bilibili、RSS、网页）
- [ ] （可选）安装 mcporter 解锁 Exa 搜索
- [ ] （可选）配置 Twitter Cookie
- [ ] （可选）配置代理访问 Reddit
- [ ] （可选）按需配置小红书、抖音、LinkedIn、Boss直聘

---

**祝使用愉快！** 🌐

*创建日期*: 2025-06-17  
*Agent Reach 版本*: 1.3.0

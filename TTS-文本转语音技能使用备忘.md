# TTS 文本转语音技能使用备忘

## 📦 安装信息

- **技能来源**: `NoizAI/skills` 仓库
- **技能名称**: `tts`
- **安装位置**: `~/.openclaw/workspace/.agents/skills/tts`
- **支持代理**: Antigravity, OpenClaw, Gemini CLI, OpenCode, Amp 等 41 个代理

---

## 🎯 核心功能

- ✅ 文本转语音（简单模式）
- ✅ 语音克隆（参考音频）
- ✅ 时间线配音（SRT 精确对齐）
- ✅ 情绪控制（Noiz 后端）
- ✅ 多平台发送（Telegram、Discord、飞书）
- ✅ 支持 EPUB/PDF 转音频

---

## 🔄 两种后端

| 后端 | 特点 | 适用场景 |
|------|------|----------|
| **Kokoro** (默认) | 本地免费，无需 API | 简单朗读、批量转换 |
| **Noiz** | 云端高质量 | 语音克隆、情感控制、精确时长 |

---

## ⚡ 最常用命令

### 1. 简单朗读

```bash
# 直接播放声音（需要扬声器）
bash skills/tts/scripts/tts.sh speak -t "你好，世界"

# 保存为文件
bash skills/tts/scripts/tts.sh speak -t "Hello World" -o output.mp3

# 从文件读取文本
bash skills/tts/scripts/tts.sh speak -f article.txt -o out.mp3
```

### 2. 语音克隆

```bash
# 使用本地参考音频
bash skills/tts/scripts/tts.sh speak -t "这是克隆的声音" \
  --ref-audio ./reference.wav \
  -o clone.mp3

# 使用网络音频
bash skills/tts/scripts/tts.sh speak -t "Hello" \
  --ref-audio https://example.com/voice.wav \
  -o clone.wav
```

### 3. 发送到 Telegram

```bash
# 先设置环境变量
export TELEGRAM_BOT_TOKEN="your_bot_token"
export TELEGRAM_CHAT_ID="your_chat_id"

# 发送语音消息
bash skills/tts/scripts/tts.sh speak_and_send_telegram \
  -t "这条消息作为语音发送"
```

### 4. 文档转音频（EPUB/PDF/文本）

```bash
# EPUB 或 PDF（需要先转换为文本）
bash skills/tts/scripts/tts.sh speak -f book.txt -o audiobook.mp3
```

---

## 📝 时间线模式（SRT 配音）

适用于配音、字幕对齐、视频旁白。

### 步骤 1：生成 SRT 字幕

```bash
# 从文本生成 SRT（中文推荐 cps=4，英文 cps=15）
bash skills/tts/scripts/tts.sh to-srt -i article.txt -o article.srt
bash skills/tts/scripts/tts.sh to-srt -i article.txt -o article.srt --cps 15 --gap 500
```

选项：
- `--cps`: 每行字符数（默认 4，中文；英文约 15）
- `--gap`: 分段间隔（毫秒，默认 500）

### 步骤 2：创建语音映射（voice map）

创建 JSON 文件控制每段的语音设置。

**Kokoro 示例** (`voice_map.json`)：
```json
{
  "default": { "voice": "zf_xiaoni", "lang": "cmn" },
  "segments": {
    "1": { "voice": "zm_yunxi" },
    "5-8": { "voice": "af_sarah", "lang": "en-us", "speed": 0.9 }
  }
}
```

**Noiz 示例**（支持情感、参考音频）：
```json
{
  "default": { "voice_id": "voice_123", "target_lang": "zh" },
  "segments": {
    "1": { "voice_id": "voice_host", "emo": { "Joy": 0.6 } },
    "2-4": { "reference_audio": "./refs/guest.wav" }
  }
}
```

### 步骤 3：渲染音频

```bash
# Kokoro
bash skills/tts/scripts/tts.sh render \
  --srt article.srt \
  --voice-map voice_map.json \
  -o narration.wav

# Noiz（使用云端，情感控制）
bash skills/tts/scripts/tts.sh render \
  --srt article.srt \
  --voice-map voice_map.json \
  --backend noiz \
  --auto-emotion \
  -o narration.wav
```

---

## 🔧 环境配置

### 1. Telegram 发送支持

```bash
export TELEGRAM_BOT_TOKEN="123:ABC"
export TELEGRAM_CHAT_ID="123456789"
```

### 2. Discord 发送支持

```bash
export DISCORD_BOT_TOKEN="your_token"
export DISCORD_CHANNEL_ID="123456789"
```

### 3. 飞书发送支持

```bash
export FEISHU_APP_ID="your_app_id"
export FEISHU_APP_SECRET="your_app_secret"
export FEISHU_CHAT_ID="your_chat_id"
export FEISHU_TENANT_ACCESS_TOKEN="your_token"
```

### 4. Noiz API 配置（语音克隆需要）

```bash
# 获取 API Key: https://developers.noiz.ai/api-keys
bash skills/tts/scripts/tts.sh config --set-api-key YOUR_API_KEY
```

---

## 🎤 后端选择指南

| 需求 | 推荐后端 |
|------|----------|
| 简单朗读，零配置 | Kokoro（默认） |
| EPUB/PDF 有声书 | Kokoro |
| 语音混合（多声音融合） | Kokoro |
| **语音克隆**（模仿音色） | **Noiz** ✅ |
| **情感控制**（开心、悲伤等） | **Noiz** ✅ |
| **精确的段落时长控制** | **Noiz** ✅ |
| 三者都需要 | Noiz（唯一支持） |

---

## 📦 依赖要求

- **基础**: `bash`, `ffmpeg`（时间线模式需要）
- **Kokoro**: 已内置或自行安装本地 TTS 引擎
- **Noiz**: API 密钥 + 网络连接

检查 ffmpeg：
```bash
ffmpeg -version
```

---

## 🗂️ 文件结构

```
.agents/skills/tts/
├── SKILL.md              # 完整文档
├── scripts/
│   ├── tts.sh           # 主入口脚本
│   ├── noiz_tts.py      # Noiz 后端实现
│   ├── render_timeline.py # 时间线渲染
│   ├── text_to_srt.py   # 文本转 SRT
│   └── third_party_sender.sh # 平台发送
└── [examples/]          # 示例配置（如有）
```

---

## 💡 快速提示

1. **默认是 Kokoro 后端**，免费且无需配置
2. **语音克隆**和**情感控制**需要 Noiz API Key
3. **SRT 时间轴模式**需要 `ffmpeg` 在 PATH 中
4. 发送到消息平台前先测试基本朗读功能
5. 环境变量可以保存在 `~/.bashrc` 或 `~/.zshrc`

---

## 🚨 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| `ffmpeg: command not found` | 缺少 ffmpeg | `brew install ffmpeg` (macOS) 或包管理器安装 |
| 语音克隆无效 | 未配置 Noiz API Key | `bash tts.sh config --set-api-key YOUR_KEY` |
| 发送失败 | 环境变量未设置 | 检查 `TELEGRAM_BOT_TOKEN` 等 |
| SRT 渲染报错 | 时间格式不正确 | 确保 SRT 文件符合标准格式 |

---

## 📖 参考文档

- 完整技能文档：`~/.openclaw/workspace/.agents/skills/tts/SKILL.md`
- 在线文档：参考技能仓库的 reference.md
- 脚本帮助：`bash skills/tts/scripts/tts.sh --help`

---

## ✅ 完成检查

- [ ] 确认 `ffmpeg` 已安装
- [ ] 测试简单朗读：`speak -t "测试"`
- [ ] （可选）配置 Noiz API Key 启用高级功能
- [ ] （可选）配置消息平台环境变量
- [ ] 保存此备忘供日后查阅

---

**祝使用愉快！** 🎤

*创建日期*: 2025-06-17  
*技能版本*: NoizAI/skills latest

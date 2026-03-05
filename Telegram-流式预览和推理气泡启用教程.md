# Telegram 流式预览与推理气泡启用教程

## 📋 概述

本教程将指导您如何在 OpenClaw 的 Telegram 频道中启用以下功能：

- **流式预览（Live stream preview）**：在消息生成过程中实时显示草稿
- **推理气泡（Reasoning bubble）**：显示模型思考过程（推理链）

---

## 🎯 功能说明

### 1. 流式预览（Native Drafts + Message Edits）

OpenClaw 支持在 Telegram 中实时预览生成的消息：

| 场景 | 行为 |
|------|------|
| **私聊（DM）** | 使用 Telegram 原生草稿 `sendMessageDraft` 实时更新，无额外预览消息 |
| **群组/话题** | 创建预览消息，完成后原地编辑，不产生第二条消息 |
| **复杂回复（如图片）** | 回退到正常发送，自动清理预览消息 |

**配置选项**：`channels.telegram.streaming`
- `off` - 禁用流式预览
- `partial`（默认） - 启用草稿/编辑预览
- `block` - 块级流式预览（较旧模式）
- `progress` - 与 `partial` 相同（跨通道兼容）

### 2. 推理气泡（Reasoning Stream）

在消息生成过程中，模型推理的中间步骤可以实时推送到对话中：

- 使用命令：`/reasoning stream`
- 推理内容会显示在流式预览中
- 最终消息仅包含答案，不包含推理文本

---

## ⚙️ 配置步骤

### 步骤 1：检查当前配置

```bash
openclaw config get channels.telegram.streaming
openclaw config get channels.telegram.streamMode
```

预期输出：
- 如果未设置，返回空（使用默认值 `partial`）
- 如果存在 `streamMode`，它是 legacy 字段，会自动映射

### 步骤 2：启用流式预览（可选）

当前默认值已经是 `partial`，通常无需修改。如需显式设置：

```bash
# 设置 streaming 为 partial（推荐）
openclaw config set channels.telegram.streaming partial

# 或完全禁用
# openclaw config set channels.telegram.streaming off
```

**注意**：`streamMode` 和 `streaming` 会自动映射，建议使用 `streaming` 作为标准配置项。

### 步骤 3：重启 Gateway 使配置生效

```bash
openclaw gateway restart
```

### 步骤 4：在 Telegram 中启用推理流

在与 bot 的私聊或群组中，发送：

```
/reasoning stream
```

此命令仅对**当前会话**生效。如需持久化，需要在配置文件中设置。

**持久化推理流配置**（可选）：

```bash
# 在会话配置中启用推理流（高级用法，通常无需全局开启）
openclaw config set agents.defaults.reasoning.streaming true
```

---

## 🧪 验证功能

### 测试流式预览

1. 在 Telegram 中向 bot 发送一个需要较长生成时间的问题，例如：
   ```
   请写一个 1000 字的短篇故事，关于一只会说话的小猫。
   ```

2. **观察现象**：
   - ✅ DM：您会看到一个正在输入的草稿，内容逐渐填充
   - ✅ 群组：bot 发送一条预览消息，内容逐步更新，完成后原地编辑为最终版本
   - ❌ 不应出现两条消息（预览 + 最终）

3. **预期行为**：
   - 消息以打字机效果逐字出现
   - 无网络延迟卡顿感
   - 最终消息只有一次编辑

### 测试推理气泡

1. 在 Telegram 中启用推理流：
   ```
   /reasoning stream
   ```

2. 发送一个复杂问题：
   ```
   解释量子纠缠的原理，并用通俗的比喻说明。
   ```

3. **观察现象**：
   - ✅ 流式预览中会先显示模型思考过程（通常在 `<think>` 标签内或特殊格式）
   - ✅ 最终消息仅包含答案，不包含推理内容
   - ✅ 推理过程对您可见，但对对话中的其他人可见性取决于 Telegram 的编辑历史

4. 发送 `/reasoning stream` 可切换回普通模式。

---

## 🔧 故障排除

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 没有流式预览，一次性显示全部 | `streaming` 设置为 `off` | `openclaw config set channels.telegram.streaming partial` |
| 预览消息后出现第二条消息 | 使用 `block` 模式或复杂媒体内容 | 切换为 `partial`，纯文本测试 |
| 草稿不更新 | Telegram Bot API 9.5+ 才支持 `sendMessageDraft` | 确保 bot 运行在支持的 API 版本（2026年3月1日起已全面支持） |
| 推理内容不显示 | 未启用 `/reasoning stream` | 发送该命令或在配置中持久开启 |
| 流式预览卡顿 | 网络延迟或 `draftChunk` 配置过小 | 检查网络，调整 `draftChunk.minChars` / `maxChars` |

---

## 📊 相关配置项

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `channels.telegram.streaming` | `partial` | 流式预览模式 |
| `channels.telegram.draftChunk.minChars` | 200 | 草稿最小字符数 |
| `channels.telegram.draftChunk.maxChars` | 800 | 草稿最大字符数 |
| `channels.telegram.draftChunk.breakPreference` | `paragraph` | 分块偏好（段落边界） |
| `channels.telegram.blockStreamingCoalesce.minChars` | 100 | 块流合并最小字符 |
| `channels.telegram.blockStreamingCoalesce.maxChars` | 400 | 块流合并最大字符 |
| `channels.telegram.blockStreamingCoalesce.idleMs` | 300 | 块流空闲时间（毫秒） |

---

## 🎓 最佳实践

1. **默认配置**通常已经最优，无需修改
2. **推理流**建议在需要调试或学习时临时开启，避免影响日常使用体验
3. **群组环境**使用流式预览时，确保 `requireMention` 设置符合预期
4. **媒体消息**（图片、音频）不会使用草稿预览，会自动回退
5. **网络不稳定**时，流式效果可能打折扣，建议检查连接

---

## 📝 配置示例

完整的 Telegram 频道配置（包含流式预览）：

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_BOT_TOKEN",
      dmPolicy: "pairing",
      groupPolicy: "allowlist",
      streaming: "partial",           // 启用流式预览
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"
      },
      blockStreamingCoalesce: {
        minChars: 100,
        maxChars: 400,
        idleMs: 300
      }
    }
  }
}
```

---

## ✅ 完成检查清单

- [ ] 确认 `channels.telegram.streaming` 设置为 `partial`（或期望值）
- [ ] 重启 Gateway 使配置生效
- [ ] 在 Telegram 中发送长消息测试流式预览
- [ ] 发送 `/reasoning stream` 测试推理气泡
- [ ] 验证 DM 和群组行为均符合预期

---

## 🔗 相关文档

- [官方文档 - Telegram 流式预览](https://docs.openclaw.ai/channels/telegram#live-stream-preview-native-drafts-message-edits)
- [OpenClaw 配置参考](/gateway/configuration-reference#telegram)
- [Telegram Bot API 9.5 更新日志](https://core.telegram.org/bots/api-changelog#march-1-2026)

---

*教程创建日期*：2025-06-17  
*基于 OpenClaw 版本*：0.1.7+  
*适用 Telegram Bot API*：9.5+

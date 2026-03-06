# 🎧 小爱音箱刷入小智 AI 超详细教程 ✨

> 将普通小爱音箱变身全能 AI 助手！支持 GPT 对话、TTS 播报、蓝牙播放、智能家居控制

---

## 📦 核心亮点

| 功能 | 说明 |
|------|------|
| 🤖 **AI 对话** | 接入 OpenAI/Claude，支持上下文记忆 |
| 🔊 **蓝牙播放** | 高质量音频输出，秒变智能音箱 |
| 📻 **TTS 播报** | 新闻/天气/日程语音通知 |
| 🏠 **原厂保留** | 小爱同学功能完全正常 |
| 🎵 **音乐平台** | QQ 音乐、网易云音乐依旧可用 |

**一句话总结**：一台音箱 = AI 对话 + 音乐播放 + 智能家居中枢 🎯

---

## 📹 视频教程（建议先看👀）

<div align=center>

[![▶️ 点击观看完整刷机过程](https://img.youtube.com/vi/xxxxxxxx/hqdefault.jpg)](https://youtu.be/xxxxxxxx)

**👆 15分钟手把手教学，小白也能轻松跟上！**

</div>

---

> 💡 **提示**：视频中演示的型号是 **小爱音箱 Pro**，其他型号步骤基本一致

---

## 🔧 准备工作（必看！）

### 支持的设备列表

| 设备 | 建议程度 | 备注 |
|------|----------|------|
| 小爱音箱 Pro | ⭐⭐⭐⭐⭐ | 完美支持，推荐首选 |
| 小爱音箱 Play | ⭐⭐⭐⭐ | 功能完整，音质优秀 |
| 小爱音箱 Art | ⭐⭐⭐⭐ | 带屏版，需额外配置 |
| 小米 Sound | ⭐⭐⭐ | 音质最佳，刷机难度略高 |
| 小爱智能闹钟 | ⭐⭐ | 屏幕小，功能受限 |

⚠️ **不支持**：Redmi 小爱音箱 Play 增强版（芯片加密）

---

### 软件准备

| 工具 | 下载地址 | 用途 |
|------|----------|------|
| adb 工具包 | https://developer.android.com/studio/releases/platform-tools | 连接并控制设备 |
| Openxiaoai 固件 | https://github.com/OpenXiaoAI-Community/Openxiaoai/releases | 刷机镜像 |
| USB Driver (Win) | 见下方 | Windows 驱动 |

**固件版本选择**：
- **最新稳定版**：v1.3.5（推荐新手）
- **开发版**：v1.4.0-dev（有风险，功能多）

---

### 预检查清单

- [ ] 设备电量充足（>50%）或连接电源
- [ ] 电脑已安装 adb 并加入 PATH
- [ ] 下载好对应型号的固件 zip 文件
- [ ] **备份重要数据**（刷机会清空）

---

## 🔍 第一步：启用开发者选项

1. **打开小爱音箱 App**
2. 进入 **设备设置** → **关于**
3. 连续点击 **版本号** 7 次
4. 看到提示"您已处于开发者模式"
5. 返回设置页面，找到 **开发者选项**
6. 开启：
   - ✅ **ADB 调试**
   - ✅ **网络调试**（可选，用于无线连接）

> 💡 如果**没有开发者选项**，说明设备被厂商锁定，需要先刷入第三方 Recovery（见附录 A）

---

## 🔌 第二步：连接 ADB

### Windows 用户（首次需驱动）

1. 插入 USB 线（音箱需处于 **开机状态**）
2. 设备管理器找到新硬件，手动选择驱动 `android_winusb.inf`
3. 打开 CMD，执行：
```cmd
adb devices
```
4. 看到类似 `192.168.50.106:5555   device` 表示成功

### Mac/Linux 用户

```bash
# 无线连接（需先在同局域网）
adb connect 192.168.50.106:5555
adb devices
# 看到 "device" 字样即连接成功
```

> 💡 可通过 `adb shell` 测试连接，看到 `xiaozhi:~$` 提示符即表示成功

---

## 📦 第三步：刷入 Openxiaoai 固件

### 3.1 进入 Recovery 模式

```bash
adb reboot recovery
```

设备会自动重启并进入蓝色 Recovery 界面（TWRP/Custom Recovery）

---

### 3.2 清除数据（⚠️ 会清空所有数据）

在 Recovery 界面操作：
- 点击 **Wipe** → **Format Data**
- 输入 `yes` 确认
- 等待完成

> ⚠️ **此步骤务必执行**，否则新固件可能无法启动！

---

### 3.3 安装固件

**方法 A：通过 ADB 推送（推荐）**

```bash
# 1. 推送固件到设备
adb push Openxiaoai_v1.3.5.zip /sdcard/

# 2. 在 Recovery 点击 Install
# 选择 /sdcard/Openxiaoai_v1.3.5.zip
# 滑动确认安装

# 3. 等待完成（约 3-5 分钟）
```

**方法 B：U 盘安装**

1. 将固件复制到 U 盘（FAT32 格式）
2. 插入音箱 USB 口
3. Recovery 中选择 `Install from USB`

---

### 3.4 重启

```bash
adb reboot
```

⏳ **首次启动较慢（5-10分钟）**，请耐心等待。看到小智 AI 欢迎界面即表示成功！

---

## ⚙️ 第四步：初始配置

### 4.1 网络设置

1. 音箱会自动进入配网模式（音响响"等待配网"）
2. 打开小智 AI App → 添加设备
3. 选择音箱型号 → 连接 Wi-Fi
4. 等待设备上线

---

### 4.2 登录与 AI 配置

1. **登录小智账号**（或注册）
2. **绑定 AI 服务**：
   - 点击 **设置** → **AI 配置**
   - 选择提供商：
     - 🟢 **OpenAI**（需 API Key）
     - 🔵 ** Claude**（需 API Key）
     - 🟡 **国内中转**（可选，网络更快）
3. 输入 API Key，测试连接
4. 调整参数：
   - **模型**: `gpt-4o-mini`（性价比高）或 `gpt-4o`
   - **Temperature**: `0.7`（平衡创造性）
   - **Max Tokens**: `500`（控制回复长度）

---

### 4.3 语音与 TTS 设置

- **唤醒词**: 可自定义（默认"小智小智"）
- **语音音色**: 小燕、许久、亲和女声等
- **语速**: 推荐 0.9-1.1（自然）
- **音量**: 根据设备调整

---

## 🎯 第五步：功能测试

### ✅ 语音唤醒测试

试着说出："小智小智，今天天气怎么样？"

**预期回应**：
1. 音箱亮灯（或播放提示音）
2. 播放 TTS 语音："今天是晴天，最高气温 25°C..."

---

### ✅ 蓝牙连接测试

将音箱作为普通蓝牙音箱使用：

```bash
# Mac
blueutil --pair "Xiaoai-Pro"
blueutil --connect "Xiaoai-Pro"

# 或系统设置中直接配对
```

播放音乐测试：
- Apple Music / Spotify / 本地音乐
- 音质明显提升（小爱 Pro 版 360° 环绕声）

---

### ✅ 智能家居控制测试

如果您已配置 Home Assistant：

```bash
# 通过 OpenClaw 发送指令
say "打开客厅空调"
# 小智 AI 应响应并控制设备
```

---

## 🆘 常见问题解决

### ❌ adb devices 找不到设备？

**症状**：`adb devices` 输出为空或 `unauthorized`

**解决**：
1. 确认 USB 调试已开启
2. 电脑上是否弹出 **USB 授权弹窗**？点击 **允许**
3. 删除 `~/.android/adbkey*` 重新生成密钥
4. 换 USB 线或接口（有些线只充电不传数据）

---

### ❌ 刷机后卡在 Logo 界面？

**原因**：固件不兼容或刷机中断

**解决**：
1. 进入 Recovery 重新刷机
2. **关键**：在 Recovery 里 **格式化 /system 分区**
3. 重新安装 Openxiaoai 固件
4. 如果还不行，尝试 **旧版本固件**（v1.3.0）

---

### ❌ 蓝牙配对后无法连接？

**症状**：显示"已配对，未连接"

**解决**：
```bash
# 完全移除并重新配对
blueutil --unpair "Xiaoai-Pro"
blueutil --remove "Xiaoai-Pro"

# 重置音箱蓝牙
在音箱上长按蓝牙键 10 秒（听到提示音）

# 重新配对
blueutil --pair "Xiaoai-Pro"
blueutil --connect "Xiaoai-Pro"
```

---

### ❌ 语音唤醒不灵敏？

**优化**：
1. 检查麦克风是否被遮挡
2. 调整唤醒词（短一点更容易识别）
3. 在设置中开启 **Always Listen**（持续监听）
4. 减少背景噪音

---

## 💡 高级配置

### 接入 OpenClaw 新闻播报

在您的 `cron` 任务中添加：

```bash
# 每日 09:00 生成新闻并播报
0 9 * * * cd /Users/ttnk/.openclaw/workspace && \
  python3 scripts/tech_news_digest.py && \
  say -v "Ting-Ting" -f Tech_News_Digest_Latest.md | \
  bluealsa_aplay "XX:XX:XX:XX:XX:XX"
```

> 替换 `XX:XX:XX:XX:XX:XX` 为音箱的蓝牙 MAC 地址

---

### 集成 Home Assistant

```yaml
# configuration.yaml
device_tracker:
  - platform: asuswrt
    host: 192.168.50.106
    username: admin
    password: your_password

# 语音通知
tts:
  - platform: google_translate
    service_name: google_say
    language: zh-CN
```

---

## 📚 附录

### A. 如果设备没有开发者选项

某些批次的小爱音箱被厂商限制了 ADB，需要先刷入第三方 Recovery：

1. 使用 **Mi Flash** 工具解锁（需小米账号）
2. 刷入 **TWRP Recovery**
3. 再执行本文第三步（刷 Openxiaoai）

详细解锁教程：https://www.xiaomi.com/解锁

---

### B. 恢复原厂固件

如果想回到小爱同学原厂系统：

1. 下载官方固件（从小米社区）
2. Recovery → Install 选择官方 zip
3. 重启即可

---

### C. 资源链接

| 项目 | 链接 |
|------|------|
| Openxiaoai 固件 | https://github.com/OpenXiaoAI-Community/Openxiaoai/releases |
| 官方文档 | https://openxiaoai.readthedocs.io/ |
| 社区论坛 | https://bbs.xiaoai.tech/ |
| 问题反馈 | https://github.com/OpenXiaoAI-Community/Openxiaoai-issues |
| Telegram 群 | https://t.me/xiaoai_chat |

---

## 🎉 完成！

🎯 恭喜！你的小爱音箱现在是一个**全能 AI 助手**了

✨ **新功能一览**：
- ✅ 和 GPT/Claude 自由对话
- ✅ 高品质蓝牙音箱
- ✅ 智能播报新闻天气
- ✅ 保持所有原厂智能家居功能
- ✅ 持续 OTA 升级

---

## 💬 交流与支持

遇到问题？加入社区：

- 📱 **Telegram 群**: @xiaoai_chat
- 💬 **Discord**: https://discord.gg/xiaoai
- 🐛 **Bug 反馈**: GitHub Issues

**觉得教程有用？**  
👉 **点赞 + 收藏**，关注我获取更多智能家居教程！🔥

---

**最后更新**: 2025-06-20  
**教程版本**: v1.0  
**适用固件**: Openxiaoai v1.3.5+

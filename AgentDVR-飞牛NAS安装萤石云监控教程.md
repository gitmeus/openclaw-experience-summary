# 🎯 AgentDVR 在飞牛 NAS 上接入萤石云摄像头完整教程

## 📋 环境信息

| 项目 | 配置 |
|------|------|
| **NAS** | 飞牛私有云 (Armbian 版，x86_64) |
| **Docker** | ✅ 已安装 |
| **摄像头** | 萤石云 CS-C6H31WFR (4mm) |
| **IP 地址** | 192.168.50.108 |
| **验证码** | AG8NXG |
| **序列号** | 732060177 |
| **Web 端口** | 8090 |
| **管理员** | admin / zzaaqq11@@ZZXXCC |

---

## 🚀 安装步骤

### 1️⃣ 前期准备

#### 1.1 确认 Docker 运行状态

```bash
# 检查 Docker 是否运行
sudo systemctl status docker

# 如果未运行，启动
sudo systemctl start docker
sudo systemctl enable docker

# 查看 Docker 信息
docker info
```

#### 1.2 创建存储目录

飞牛 NAS 路径假设为 `/vol1/1000/docker/AgentDVR/`，请根据实际情况调整：

```bash
# 创建配置目录
sudo mkdir -p /vol1/1000/docker/AgentDVR/config/
sudo mkdir -p /vol1/1000/docker/AgentDVR/media/
sudo mkdir -p /vol1/1000/docker/AgentDVR/commands/

# 设置权限（确保 PUID=1000 有读写权限）
sudo chown -R 1000:1000 /vol1/1000/docker/AgentDVR/
```

---

### 2️⃣ 安装 AgentDVR

#### 2.1 基础安装（不推荐）

❌ 默认方式（无硬件访问）：
```bash
docker run -d \
  --name=AgentDVR \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Asia/Shanghai \
  -p 8090:8090 \
  mekayelanik/ispyagentdvr:latest
```

**问题**：缺少硬件访问，摄像头无法添加。

---

#### 2.2 推荐安装（Host 网络模式）

✅ **使用 host 网络模式**，直接访问本地硬件：

```bash
sudo docker run -d \
  --name=AgentDVR \
  --network host \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Asia/Shanghai \
  -v /vol1/1000/docker/AgentDVR/config/:/AgentDVR/Media/XML/ \
  -v /vol1/1000/docker/AgentDVR/media/:/AgentDVR/Media/WebServerRoot/Media/ \
  -v /vol1/1000/docker/AgentDVR/commands:/AgentDVR/Commands/ \
  --restart unless-stopped \
  mekayelanik/ispyagentdvr:latest
```

**关键参数说明**：
- `--network host`：使用宿主机网络，Docker 容器可直接访问局域网设备
- `-v ...`：挂载三个目录，持久化配置、录像文件和命令
- `--restart unless-stopped`：开机自启

---

#### 2.3 验证安装

```bash
# 查看容器状态
sudo docker ps | grep AgentDVR

# 查看日志
sudo docker logs -f AgentDVR

# 检查端口监听
sudo ss -tlnp | grep 8090
```

---

### 3️⃣ 初始配置

#### 3.1 访问 Web 界面

浏览器打开：`http://飞牛NASIP:8090`

首次访问：
1. 设置管理员账号（建议使用您提供的 `admin`）
2. 配置时区：Asia/Shanghai
3. 初始化完成

#### 3.2 添加萤石云摄像头

萤石云摄像头有两种接入方式：

##### 方式 A：通过 ONVIF 协议（如果支持）

1. 进入 **Devices** → **Add Device**
2. 选择 **ONVIF Camera**
3. 填写：
   - **IP Address**: `192.168.50.108`
   - **Port**: `80` 或 `8080`（萤石云默认）
   - **Username**: 您的萤石云账号（通常是手机号或邮箱）
   - **Password**: 萤石云密码
4. 点击 **Detect**，自动发现摄像头
5. 选择视频流和音频流（如有）
6. 保存

⚠️ **注意**：部分萤石云摄像头 ONVIF 功能被限制，可能无法连接。

---

##### 方式 B：RTSP 流地址（通用）

如果 ONVIF 不可用，使用 RTSP 直接连接。

**萤石云 RTSP 通用格式**：
```
rtsp://<用户名>:<密码>@<IP地址>:<端口>/Streaming/Channels/<通道号>
```

**具体参数**：
- 用户名：您的萤石云登录账号
- 密码：萤石云密码
- IP：`192.168.50.108`
- 端口：`80` 或 `554`（RTSP 默认）
- 通道号：通常 `101`（主码流）或 `102`（子码流）

**示例**：
```
rtsp://admin:zzaaqq11@@ZZXXCC@192.168.50.108:80/Streaming/Channels/101
```

**在 AgentDVR 中添加**：
1. **Add Device** → **Generic Camera**
2. **Model**: Select → **RTSP**
3. **Source**: 粘贴 RTSP URL
4. **Name**: 自定义（如 "萤石云-客厅"）
5. 保存

---

#### 3.3 验证视频流

添加成功后：
- 点击摄像头名称，查看实时画面
- 检查是否流畅，无延迟
- 如有问题，查看 AgentDVR 日志

---

### 4️⃣ 高级配置

#### 4.1 录像设置

1. 进入 **Recording** → **Schedule**
2. 设置录像模式：
   - **Continuous**：24小时连续录像
   - **Schedule**：指定时间段
   - **Motion Detection**：移动侦测录像（推荐）
3. 配置存储路径（已挂载 `/AgentDVR/Media/WebServerRoot/Media/`）

#### 4.2 移动侦测

进入摄像头设置：
- **Motion Detection**：启用
- 调整灵敏度（默认 50%）
- 绘制检测区域（点击画面设置）
- 设置报警动作：录像、通知、抓拍

#### 4.3 远程访问（可选）

如果需要在外部网络访问：

1. **端口映射**（路由器）：
   - 外部端口 → 飞牛NASIP:8090
   - 建议改非标准端口增强安全

2. **HTTPS 配置**（AgentDVR Pro 功能）
3. **DDNS 域名**：绑定动态域名访问

⚠️ **安全建议**：不要暴露到公网，除非配置强密码 + HTTPS

---

### 5️⃣ 常见问题

#### Q1: 容器启动后立即退出？

**检查**：
```bash
docker logs AgentDVR
```

**可能原因**：
- 目录权限不足：`chown -R 1000:1000 /vol1/1000/docker/AgentDVR/`
- 端口冲突：8090 已被占用，修改映射或停止其他服务
- PUID/PGID 错误：确保用户存在 `id 1000`

---

#### Q2: 摄像头添加失败？

**排查步骤**：

1. **网络连通性**：
```bash
ping 192.168.50.108
```

2. **RTSP 地址验证**（使用 VLC）：
```bash
vlc "rtsp://admin:zzaaqq11@@ZZXXCC@192.168.50.108:80/Streaming/Channels/101"
```

3. **萤石云账号密码**：确认是否启用了 RTSP 权限（在萤石云 App 设置中）
4. **查看 AgentDVR 日志**：`docker logs AgentDVR`

---

#### Q3: 视频流卡顿/延迟高？

- 降低码流：在萤石云 App 设置子码流（102 通道）
- 使用硬件加速（GPU 直通，需要 NAS 支持）
- 检查网络带宽：局域网至少 10Mbps
- 调整 AgentDVR 的缓冲区设置

---

#### Q4: 录像文件存储位置？

已挂载目录：
- 配置文件：`/vol1/1000/docker/AgentDVR/config/`
- 录像文件：`/vol1/1000/docker/AgentDVR/media/`

查看录像：
```bash
ls -lh /vol1/1000/docker/AgentDVR/media/
```

---

#### Q5: 如何备份配置？

定期备份整个目录：
```bash
tar -czf agentdvr_backup_$(date +%Y%m%d).tar.gz /vol1/1000/docker/AgentDVR/
```

恢复时，停止容器，恢复文件，重新启动。

---

### 6️⃣ 安全建议

| 措施 | 说明 |
|------|------|
| **修改默认密码** | admin 密码改为强密码 |
| **防火墙** | 仅允许局域网访问 8090 端口 |
| **更新** | 定期 `docker pull mekayelanik/ispyagentdvr:latest` |
| **日志监控** | 查看访问日志，异常 IP 封禁 |
| **定期备份** | 录像文件和配置分开存储 |

---

### 7️⃣ 删除/重建容器

如需重新安装：

```bash
# 停止并删除
sudo docker stop AgentDVR
sudo docker rm AgentDVR

# 删除数据（谨慎！会丢失录像）
# sudo rm -rf /vol1/1000/docker/AgentDVR/

# 重新拉取镜像
sudo docker pull mekayelanik/ispyagentdvr:latest

# 重新运行（见 2.2 节）
```

---

## 📊 配置总览

```yaml
NAS: 飞牛私有云 (Armbian)
Docker: latest
镜像: mekayelanik/ispyagentdvr:latest
容器名: AgentDVR
网络: host
端口: 8090 (Web 界面)
数据卷:
  - /vol1/1000/docker/AgentDVR/config/ → /AgentDVR/Media/XML/
  - /vol1/1000/docker/AgentDVR/media/ → /AgentDVR/Media/WebServerRoot/Media/
  - /vol1/1000/docker/AgentDVR/commands → /AgentDVR/Commands/
环境变量:
  - PUID=1000
  - PGID=1000
  - TZ=Asia/Shanghai
摄像头: 萤石云 CS-C6H31WFR
RTSP: rtsp://admin:zzaaqq11@@ZZXXCC@192.168.50.108:80/Streaming/Channels/101
```

---

## 🎯 验证清单

- [ ] Docker 运行正常
- [ ] AgentDVR 容器启动成功
- [ ] 能访问 `http://NASIP:8090`
- [ ] 管理员登录成功
- [ ] 添加 RTSP 摄像头成功
- [ ] 实时画面正常显示
- [ ] 移动侦测触发录像
- [ ] 录像文件保存到指定目录
- [ ] 移动端可访问（内网）

---

## 📚 相关资源

- **AgentDVR 官网**: https://www.ispyconnect.com/
- **Docker 镜像**: https://hub.docker.com/r/mekayelanik/ispyagentdvr
- **萤石云 RTSP 指南**: 在萤石云 App → 设置 → 高级设置 → RTSP
- **飞牛 NAS Docker 文档**: https://doc.fe猛nas.com/

---

## 💡 提示

1. **ONVIF vs RTSP**：优先 RTSP，更通用
2. **网络模式**：必须用 `host` 才能发现局域网摄像头
3. **权限**：确保 PUID=1000 对挂载目录有读写权限
4. **验证码/序列号**：这些信息在萤石云设备详情页可找到，RTSP 不需要
5. **多摄像头**：重复添加步骤，注意通道号（101, 102, 201...）

---

## ❓ 获取帮助

- AgentDVR 论坛：https://forums.ispyconnect.com/
- GitHub Issues：https://github.com/mekayelanik/ispyagentdvr/issues
- 飞牛 NAS 社区

---

**祝您监控系统搭建顺利！** 🎉

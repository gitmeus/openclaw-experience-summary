# MoonTVPlus 教程：基于 MoonTV 二次开发的增强版影视聚合播放器

## 🎬 什么是 MoonTVPlus？

**MoonTVPlus** 是基于 [MoonTV v100](https://github.com/MoonTechLab/LunaTV) 二次开发的增强版影视聚合播放器。在原版基础上新增了**外部播放器支持、视频超分、弹幕系统、评论抓取**等实用功能，提供更强大的观影体验。

---

## 🚀 核心优势

| 功能 | 说明 |
|------|------|
| **多源聚合搜索** | 一次搜索立刻返回全源结果 |
| **外部播放器** | 支持 PotPlayer、VLC、MPV、MX Player、nPlayer、IINA 等 |
| **视频超分 (Anime4K)** | WebGPU 实时画质增强，支持 1.5x/2x/3x/4x |
| **弹幕系统** | 完整弹幕搜索、匹配、加载，支持屏蔽和持久化 |
| **豆瓣评论抓取** | 自动抓取豆瓣电影短评，支持分页 |
| **观影室** | 多人同步观影 + 实时聊天 + 语音通话（实验性） |
| **离线下载** | 服务器端下载视频，支持断点续传 |
| **私人影库** | 接入 OpenList 或 Emby，打造专属影库 |

---

## 📱 用户体验

- **响应式布局**：桌面侧边栏 + 移动底部导航，自适应所有屏幕
- **PWA 支持**：离线缓存、可安装到桌面/主屏
- **收藏 + 继续观看**：多端同步进度（Kvrocks/Redis/Upstash）
- **智能去广告**：自动跳过视频广告，支持自定义去广告代码

---

## 🛠️ 技术栈

| 类别 | 技术 |
|------|------|
| 前端框架 | Next.js 14 (App Router) |
| 语言 | TypeScript 4 |
| UI & 样式 | Tailwind CSS 3 |
| 播放器 | ArtPlayer + HLS.js |
| 代码质量 | ESLint · Prettier · Jest |
| 部署 | Docker / Vercel / Cloudflare Workers |

---

## 🚀 快速部署

MoonTVPlus 支持多种部署方式，选择最适合你的：

### 方式一：Docker（推荐）

#### 使用 Kvrocks 存储（推荐）

创建 `docker-compose.yml`：

```yaml
services:
  moontv-core:
    image: ghcr.io/mtvpls/moontvplus:latest
    container_name: moontv-core
    restart: on-failure
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=your_secure_password
      - NEXT_PUBLIC_STORAGE_TYPE=kvrocks
      - KVROCKS_URL=redis://moontv-kvrocks:6666
    networks:
      - moontv-network
    depends_on:
      - moontv-kvrocks

  moontv-kvrocks:
    image: apache/kvrocks
    container_name: moontv-kvrocks
    restart: unless-stopped
    volumes:
      - kvrocks-data:/var/lib/kvrocks/data
    networks:
      - moontv-network

networks:
  moontv-network:
    driver: bridge

volumes:
  kvrocks-data:
```

启动：
```bash
docker-compose up -d
```

访问：`http://localhost:3000`

#### 使用 Redis 存储

```yaml
services:
  moontv-core:
    image: ghcr.io/mtvpls/moontvplus:latest
    container_name: moontv-core
    restart: on-failure
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=admin_password
      - NEXT_PUBLIC_STORAGE_TYPE=redis
      - REDIS_URL=redis://moontv-redis:6379
    networks:
      - moontv-network
    depends_on:
      - moontv-redis
    volumes:
      - ./data:/data  # 开启持久化

  moontv-redis:
    image: redis:alpine
    container_name: moontv-redis
    restart: unless-stopped
    networks:
      - moontv-network
```

#### 使用 Upstash 存储（无服务器）

```yaml
services:
  moontv-core:
    image: ghcr.io/mtvpls/moontvplus:latest
    container_name: moontv-core
    restart: on-failure
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=admin_password
      - NEXT_PUBLIC_STORAGE_TYPE=upstash
      - UPSTASH_URL=https://xxx.upstash.io
      - UPSTASH_TOKEN=your_upstash_token
```

---

### 方式二：Vercel 一键部署（推荐使用 Upstash 存储）

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/mtvpls/MoonTVPlus)

点击上方按钮，选择 Fork 到您的账户，配置环境变量即可自动部署。

**注意**：Vercel 部署不支持观影室（WebSocket）功能。

#### 配置 Upstash 存储（推荐）

1. **访问 Upstash 控制台**：https://console.upstash.com/redis
   - 可以使用 **GitHub 账号自动登录**（一键授权）

2. **创建 Redis 数据库**：
   - 点击 "Create Database"
   - 选择区域（建议选择离您用户最近的区域）
   - 免费套餐足够个人使用

3. **获取连接信息**：
   - 在数据库详情页面，找到 **"Connection"** 或 **"Connect"** 选项卡
   - 复制 **"HTTPS Endpoint"** → 这就是 `UPSTASH_URL`
   - 复制 **"TOKEN"** → 这就是 `UPSTASH_TOKEN`

4. **在 Vercel 配置环境变量**：
   - `USERNAME` = 您选择的管理员用户名（如 `admin`）
   - `PASSWORD` = 您设置的强密码
   - `NEXT_PUBLIC_STORAGE_TYPE` = `upstash`
   - `UPSTASH_URL` = 从 Upstash 复制的 HTTPS Endpoint
   - `UPSTASH_TOKEN` = 从 Upstash 复制的 TOKEN

5. **完成部署**：
   - 配置完成后，Vercel 自动部署
   - 访问生成的 URL（如 `https://your-project.vercel.app`）

---

### 方式三：Cloudflare Workers（免费边缘计算）

Cloudflare Workers 提供免费的边缘计算服务，通过 GitHub Actions 可实现自动化部署。

#### 前置要求

- Cloudflare 账号
- Fork 本项目到 GitHub
- Upstash Redis 实例（推荐）

#### 配置步骤

**1. 获取 Cloudflare API Token 和 Account ID**

访问 [Cloudflare Dashboard](https://dash.cloudflare.com/)
- 点击右上角头像 → My Profile → API Tokens
- 点击 "Create Token"
- 选择 "Edit Cloudflare Workers" 模板
- 或使用自定义 Token，需要权限：
  - Account - Cloudflare Workers Scripts - Edit
  - Account - D1 - Edit（仅在使用 D1 数据库时需要）
- 复制生成的 API Token
- 在 Dashboard 首页右侧获取 Account ID

**2. 配置 GitHub Secrets**

进入 Fork 的仓库 → Settings → Secrets and variables → Actions → New repository secret

| Secret 名称 | 说明 | 示例值 |
|------------|------|--------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API Token | your_api_token_here |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare Account ID | abc123def456 |
| `USERNAME` | 站长账号 | admin |
| `PASSWORD` | 站长密码 | your_secure_password |
| `NEXT_PUBLIC_STORAGE_TYPE` | 存储类型 | upstash |
| `UPSTASH_URL` | Upstash Redis URL | https://xxx.upstash.io |
| `UPSTASH_TOKEN` | Upstash Redis Token | your_upstash_token |

**3. 触发部署**

- **方式一（手动）**：进入仓库的 Actions 页面 → 选择 "Deploy to Cloudflare" workflow → Run workflow
- **方式二（自动）**：推送代码到 main 分支会自动触发（需修改 `.github/workflows/cloudflare-deploy.yml`）

**4. 查看部署状态**

- Actions 页面查看进度
- 部署成功后访问：`https://your-project-name.your-account.workers.dev`
- 或在 Cloudflare Dashboard 的 Workers & Pages 中查看

**5. 绑定自定义域名（可选）**

Cloudflare Dashboard → Worker → Settings → Triggers → Custom Domains → 添加域名

**6. 使用 D1 数据库（可选）**

如需使用 Cloudflare D1 替代 Upstash：
- 在 Cloudflare Dashboard 创建 D1 数据库
- 复制数据库 ID
- 设置 `NEXT_PUBLIC_STORAGE_TYPE=d1`
- 添加 `D1_DATABASE_ID` Secret
- 无需配置 UPSTASH 相关 Secret

---

## ⚙️ 配置指南

### 1. 初始配置

部署完成后，应用为空壳，**无内置播放源**，需要自行配置。

访问 `https://your-site.com/admin`（或 `/configure`），使用设置的 `USERNAME` 和 `PASSWORD` 登录管理后台。

### 2. 配置文件格式

示例：

```json
{
  "cache_time": 7200,
  "api_site": {
    "dyttzy": {
      "api": "http://xxx.com/api.php/provide/vod",
      "name": "示例资源",
      "detail": "http://xxx.com"
    }
    // ... 更多站点
  },
  "custom_category": [
    {
      "name": "华语",
      "type": "movie",
      "query": "华语"
    }
  ]
}
```

**字段说明**：

- `cache_time`：接口缓存时间（秒）
- `api_site`：资源站配置
  - `key`：唯一标识（小写字母/数字）
  - `api`：资源站 JSON API 地址
  - `name`：显示名称
  - `detail`：详情页根 URL（部分站点需要）
- `custom_category`：自定义分类，基于豆瓣搜索

### 3. 环境变量说明

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `USERNAME` | 站长账号（必填） | - |
| `PASSWORD` | 站长密码（必填） | - |
| `CRON_PASSWORD` | 定时任务 API 密码 | mtvpls |
| `NEXT_PUBLIC_STORAGE_TYPE` | 存储类型：redis/kvrocks/upstash/d1 | - |
| `KVROCKS_URL` | Kvrocks 连接 URL | - |
| `REDIS_URL` | Redis 连接 URL | - |
| `UPSTASH_URL` | Upstash HTTPS ENDPOINT | - |
| `UPSTASH_TOKEN` | Upstash TOKEN | - |
| `NEXT_PUBLIC_SITE_NAME` | 站点名称 | MoonTV |
| `ANNOUNCEMENT` | 站点公告 | 法律声明 |
| `SITE_BASE` | 站点 URL（用于生成绝对链接） | - |

详见完整文档中的"环境变量"章节。

---

## 🎬 视频教程

👉 **观看完整演示**：

[![MoonTVPlus 部署使用教程](https://img.youtube.com/vi/RQ9uJyUqpM8/0.jpg)](https://www.youtube.com/watch?v=RQ9uJyUqpM8)

视频内容包括：
- 项目介绍与功能展示
- Docker 部署完整流程
- Vercel 和 Cloudflare Workers 部署
- 配置文件编写
- 播放源添加与管理
- 常见问题解决

---

## ✨ 高级功能

### 1. 外部播放器跳转

支持多种外部播放器：
- PotPlayer
- VLC
- MPV
- MX Player
- nPlayer
- IINA

在播放页面点击外部播放器按钮，即可复制 m3u8 链接或直接调用本地播放器。

### 2. 视频超分 (Anime4K)

基于 WebGPU 技术实现实时视频画质增强：
- 支持 1.5x、2x、3x、4x 超分
- 需要浏览器支持 WebGPU
- 避免在 HTTP 环境使用（需 HTTPS）

### 3. 弹幕系统

- 完整弹幕搜索、匹配、加载
- 弹幕设置持久化
- 弹幕屏蔽功能
- 需要额外部署 [danmu_api](https://github.com/huangxd-/danmu_api) 后端

### 4. 豆瓣评论抓取

- 自动抓取豆瓣电影短评
- 支持分页加载
- 可配置豆瓣数据源代理

### 5. 观影室（多人同步）

- 多人同步观影
- 实时文字聊天
- 语音通话（WebRTC）
- **注意**：Vercel 部署不支持，需外部服务器

配置：
```bash
WATCH_ROOM_ENABLED=true
WATCH_ROOM_SERVER_TYPE=internal  # 或 external
```

### 6. TVBOX 订阅

生成 TVBOX 格式订阅链接：
```bash
ENABLE_TVBOX_SUBSCRIBE=true
TVBOX_SUBSCRIBE_TOKEN=your_secure_token
```
登录后点击用户菜单"订阅"按钮获取链接。

---

## 📊 存储选择对比

| 存储方案 | 优点 | 缺点 |
|---------|------|------|
| **Kvrocks** (Docker) | 数据持久化，性能好 | 需要维护 Redis 服务 |
| **Redis** (Docker) | 通用，易用 | 有丢失数据风险（需开启持久化） |
| **Upstash** (Serverless) | 免费，无运维 | 仅适合轻量使用 |
| **D1** (Cloudflare) | 与 Workers 集成好 | 需要 Cloudflare 付费套餐 |

---

## 🔐 安全与隐私

### 重要提醒

- **请设置强密码**并**关闭公网注册**
- 仅供个人使用，勿公开分享实例链接
- 遵守当地法律法规
- 项目不在中国大陆地区提供服务
- 使用产生的法律风险由用户自行承担

### 建议的安全配置

```bash
# 管理后台设置
USERNAME=strong_username
PASSWORD=very_secure_password_here

# 关闭公开注册（通过管理后台设置）
# 定期备份数据
```

---

## 🔧 常见问题

### Q1: 部署后为什么没有播放源？
**答**：项目为空壳，需要自行配置 `api_site` 资源站。参考"配置文件"章节。

### Q2: 如何寻找资源站 API？
**答**：搜索"蘋果CMS"、"马克思CMS"等影视采集站的 API 接口，注意版权法律风险。

### Q3: 弹幕功能无法使用？
**答**：需要额外部署 [danmu_api](https://github.com/huangxd-/danmu_api) 后端，并配置 `DANMAKU_API_BASE` 和 `DANMAKU_API_TOKEN`。

### Q4: 超分功能提示浏览器不支持？
**答**: WebGPU 需要 Chrome 113+ 或 Edge 113+，且需要 HTTPS 环境（localhost 除外）。

### Q5: Vercel 部署后观影室不可用？
**答**：Vercel 不支持 WebSocket 长连接，观影室需要外部服务器或使用 Cloudflare Workers/Docker。

### Q6: 如何备份数据？
**答**：
- Kvrocks/Redis：备份数据卷或 RDB 文件
- Upstash：通过控制台导出
- D1：通过 Cloudflare 控制台导出

---

## 📈 版本历史与致谢

**核心贡献者**：mtvpls 及社区贡献者

**源自项目**：
- [MoonTV](https://github.com/MoonTechLab/LunaTV)
- [LibreTV](https://github.com/LibreSpark/LibreTV)
- [ts-nextjs-tailwind-starter](https://github.com/theodorusclarence/ts-nextjs-tailwind-starter)

**开源组件**：
- ArtPlayer、HLS.js
- Tailwind CSS、Next.js
- 以及所有 Cucumber 友好社区项目

---

## 🔗 相关资源

- [GitHub 仓库](https://github.com/mtvpls/MoonTVPlus)
- [演示视频](https://www.youtube.com/watch?v=RQ9uJyUqpM8)
- [原版 MoonTV](https://github.com/MoonTechLab/LunaTV)
- [弹幕后端 danmu_api](https://github.com/huangxd-/danmu_api)
- [外部观影室服务器 watch-room-server](https://github.com/tgs9915/watch-room-server)

---

## ✅ 快速检查清单

- [ ] 选择部署方式（Docker/Vercel/Cloudflare）
- [ ] 完成部署并访问应用
- [ ] 登录管理后台配置播放源
- [ ] 设置站点名称和公告
- [ ] 测试搜索和播放功能
- [ ] 如需弹幕，部署 danmu_api
- [ ] 如需观影室，配置外部服务器（Vercel 除外）
- [ ] 设置定期任务更新播放记录
- [ ] 开启 HTTPS 并配置访问控制
- [ ] 定期备份数据

---

**享受您的私人影院体验！** 🍿

---

*教程创建日期*：2025-06-17  
*参考资源*：MoonTVPlus GitHub 仓库、官方文档  
*适用版本*：MoonTVPlus latest

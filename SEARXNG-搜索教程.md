# SEARXNG 搜索技能使用教程

## 📚 什么是 SearXNG？

SearXNG 是一个**隐私优先的元搜索引擎**，它聚合多个搜索引擎（Google、DuckDuckGo、Brave、Bing 等）的结果，不记录用户数据，不设置追踪 Cookie。

您的实例地址：**https://s.136222.xyz**

---

## ⚙️ 系统集成方式

### 1️⃣ 技能位置
```
/Users/ttnk/.openclaw/workspace/skills/searxng-search/
```

### 2️⃣ 配置记录
在 `TOOLS.md` 中保留了实例地址：
```markdown
### SearXNG
- Instance: https://s.136222.xyz
- 特性: 多引擎聚合（Brave, DuckDuckGo, Google 等），隐私友好
- 使用技能: `searxng-search`
```

### 3️⃣ 工作原理
技能通过调用 SearXNG 的 JSON API：
```
https://s.136222.xyz/search?q=查询词&format=json
```
获取结构化结果，然后自动去重、按相关性排序、提取关键信息。

---

## 🎯 如何使用

### 基本调用方式
您可以直接对我说：
> "搜索一下 OpenClaw 的相关信息"

我会自动：
1. 构造搜索 URL（使用配置的实例地址）
2. 调用 `web_fetch` 获取 JSON 结果
3. 解析并提取前 3-5 条结果
4. 以清晰的中文格式返回给您

### 示例对话
```
您: 搜索 Python asyncio 教程
我: 为您找到以下结果：
1. [Python asyncio 官方文档](https://docs.python.org/3/library/asyncio.html)
   摘要: asyncio 是用于编写并发代码的库，使用 async/await 语法...

2. [Real Python:AsyncIO 入门](https://realpython.com/async-io-python/)
   摘要: 本教程将介绍 asyncio 模块的核心概念，包括事件循环...
```

---

## 🛠️ 高级功能

SearXNG 支持多种搜索参数：

| 参数 | 说明 | 示例 |
|------|------|------|
| `categories` | 限制类别 | `general`（综合）, `news`（新闻）, `images`（图片） |
| `language` | 结果语言 | `zh-CN`（简体中文）, `en`（英文） |
| `safesearch` | 安全搜索 | `0`（关闭）, `1`（中等）, `2`（严格） |
| `time_range` | 时间范围 | `day`（一天）, `week`, `month`, `year` |
| `pageno` | 页码 | `1`, `2`, `3` |

**示例命令：**
- 搜索过去一周的中文技术新闻：
  ```
  https://s.136222.xyz/search?q=人工智能&categories=news&language=zh-CN&time_range=week&format=json
  ```

---

## 📋 结果结构说明

返回的 JSON 包含以下关键字段：

```json
{
  "query": "搜索词",
  "number_of_results": 总结果数,
  "results": [
    {
      "title": "标题",
      "url": "链接地址",
      "content": "摘要内容",
      "engine": "搜索引擎名（google/duckduckgo/brave等）",
      "score": "相关性评分（越高越好）"
    }
  ],
  "suggestions": ["相关搜索词1", "相关搜索词2"]
}
```

系统会自动：
- ✅ 去重（相同 URL 只保留评分最高的）
- ✅ 按 `score` 降序排序
- ✅ 提取最相关的前几条结果

---

## ⚠️ 注意事项

1. **实例可用性**：确保 `https://s.136222.xyz` 可访问且未限流
2. **跨域问题**：如果遇到 CORS 错误，需要在 SearXNG 配置中允许跨域
3. **结果差异**：不同实例配置的引擎不同，结果可能有差异
4. **缓存建议**：频繁查询可考虑缓存结果以减少服务器负载

---

## 🔧 自定义配置

如果您想更换实例，只需修改 `TOOLS.md` 中的 `Instance` 地址：

```markdown
### SearXNG
- Instance: https://你的新实例地址/  # 修改这里
```

然后重启 OpenClaw 即可生效。

---

## 💡 使用技巧

- **精确搜索**：使用引号包裹关键词，如 `"OpenClaw 教程"`
- **排除词**：使用减号，如 `OpenClaw -bug`
- **站内搜索**：`site:example.com 关键词`
- **文件类型**：`filetype:pdf 机器学习`

---

## 🎓 快速检查清单

✅ 实例地址已配置：`https://s.136222.xyz`  
✅ 技能已安装到系统  
✅ 可自动调用并解析结果  
✅ 支持多语言、多类别搜索  
✅ 结果已优化（去重、排序）

---

## 📞 需要帮助？

如果您遇到：
- 搜索结果异常 → 检查实例是否可访问
- 返回格式错误 → 确认实例支持 JSON 格式（`?format=json`）
- 想要新的功能 → 我可以修改技能文件添加

---

*创建日期：2025-06-17*  
*作者：伴伴 (Bàn Bàn)*

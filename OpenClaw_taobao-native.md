# OpenClaw taobao-native 完整配置指南

> **更新日期**: 2026年4月13日  
> **适用版本**: 淘宝桌面客户端 2.5.1+  
> **文档版本**: v1.0

下面是一份 **专门针对「淘宝桌面客户端 2.5.1」版本** 的 **taobao-native 官方规范使用指南（从 0 到可用）**。  
我会严格按 **淘宝 2.5.x 实际行为 + 官方 MCP / SKILL 文档要求** 来讲，保证你照着做 **一定能跑起来**。

---

## 📋 目录

- [文档说明](#文档说明)
- [方案对比](#方案对比)
- [一、淘宝桌面客户端 2.5.1 与 taobao-native 的关系](#一淘宝桌面客户端-251-与-taobao-native-的关系)
- [二、在淘宝桌面客户端 2.5.1 中开启 taobao-native](#二在淘宝桌面客户端-251-中开启-taobao-native)
- [三、OpenClaw 中使用 taobao-native](#三openclaw-中使用-taobao-native)
- [四、taobao-native 能力边界](#四taobao-native-能力边界)
- [五、官方推荐使用方法](#五官方推荐使用方法)
- [六、验证配置](#六验证配置)
- [七、常见问题](#七常见问题)
- [八、故障排查](#八故障排查)
- [九、实战示例](#九实战示例)
- [十、最佳实践](#十最佳实践)
- [参考资源](#参考资源)
- [更新日志](#更新日志)
- [附录](#附录)

---

## 📖 文档说明

### 适用人群
- ✅ 使用 OpenClaw 的 AI 助手用户
- ✅ 需要淘宝购物自动化的开发者
- ✅ 熟悉 MCP 协议的技术人员

### 前置要求
- 淘宝桌面客户端 2.5.1 或更高版本
- OpenClaw 已安装并配置
- Node.js v22.16+ 或 v24
- 基本的命令行操作能力

### 与浏览器自动化方案的区别

本文档介绍的是 **官方 MCP 方案**，与之前的浏览器自动化方案有本质区别：

| 对比项 | taobao-native (官方MCP) | 浏览器自动化 (Playwright) |
|--------|------------------------|--------------------------|
| **授权方式** | 淘宝客户端授权 | Cookie 管理 |
| **实现机制** | 官方 API 调用 | 网页元素操作 |
| **稳定性** | ⭐⭐⭐⭐⭐ 官方维护 | ⭐⭐⭐ 受页面变化影响 |
| **反爬虫** | ✅ 无问题 | ⚠️ 需要反检测 |
| **配置难度** | 简单(仅需开启) | 复杂(需要多个模块) |
| **功能限制** | 有官方限制 | 理论上无限制 |
| **账号安全** | ✅ 官方支持 | ⚠️ 可能被封 |
| **推荐场景** | 日常购物助手 | 数据采集/测试 |

---

## 🔄 方案对比

### 何时使用 taobao-native (推荐)

✅ **个人日常购物**  
✅ **合规的 AI 购物助手**  
✅ **长期稳定使用**  
✅ **官方功能已满足需求**

### 何时使用浏览器自动化

⚠️ **需要评价内容抓取**  
⚠️ **批量数据采集**  
⚠️ **自定义复杂流程**  
⚠️ **测试/研究目的**

---

# 一、淘宝桌面客户端 2.5.1 与 taobao-native 的关系

在 **2.5.0 开始（含 2.5.1）**，淘宝桌面客户端正式内置了：

> ✅ **原生 MCP（Model Context Protocol）服务**
>
> ✅ 对外暴露的 MCP 名称：`taobao-native`

这意味着：

*   `taobao-native` **不是你安装的插件**
*   也不是网页脚本 / 爬虫
*   而是 👉 **淘宝桌面客户端自己启动的本地服务**

默认 MCP 地址（2.5.1 未改变）：

```text
http://localhost:3654/mcp
```

---

# 二、在淘宝桌面客户端 2.5.1 中开启 taobao-native

> ⚠️ taobao-native 没有 OAuth 或弹窗授权  
> **“开启 MCP” 本身就是授权**

### ✅ 操作步骤（必须在 2.5.1 客户端里做）

1.  打开 **淘宝桌面客户端 2.5.1**
2.  右上角 → **设置**
3.  进入 **AI 设置**
4.  打开（勾选） ✅ **启用 MCP 服务**

完成后：

*   淘宝会在本机启动 MCP
*   你本人当前的淘宝登录态 = taobao-native 的工作身份

---

# 三、OpenClaw 中使用 taobao-native

> 以下是 **淘宝 2.5.1 + OpenClaw 的官方推荐接入方式**

## Step 1：安装 MCP 中间件（必须）

```bash
npm install -g mcporter
```

---

## Step 2：配置 MCP 地址

编辑（或创建）：

```bash
~/.openclaw/workspace/config/mcporter.json
```

内容：

```json
{
  "mcpServers": {
    "taobao-native": {
      "type": "streamableHttp",
      "url": "http://localhost:3654/mcp"
    }
  }
}
```

说明：

*   `3654` 是淘宝 2.5.1 默认端口
*   不要随便改
*   如果你在淘宝设置页看到不同端口，以设置页为准

---

## Step 3：安装 taobao-native 官方 Skill

### 3.1 安装 mcporter Skill（必需）

```bash
mkdir -p ~/.openclaw/workspace/skills/mcporter
cd ~/.openclaw/workspace/skills/mcporter

curl -O https://tblifecdn.taobao.com/taobaopc/skills/mcporter/SKILL.md
```

### 3.2 安装 taobao-native Skill（核心）

```bash
mkdir -p ~/.openclaw/workspace/skills/taobao-native
cd ~/.openclaw/workspace/skills/taobao-native

curl -O https://tblifecdn.taobao.com/taobaopc/skills/taobao-native/SKILL.md
```

---

## Step 4：重启 OpenClaw

```bash
openclaw gateway restart
```

确认三点：

*   ✅ OpenClaw 正常启动
*   ✅ Skills 中出现 `taobao-native`
*   ✅ 状态为绿色 / 可用

---

# 四、taobao-native 能力边界

### ✅ 官方允许的能力

*   搜索商品
*   查看商品详情（价格 / 规格 / 店铺）
*   多商品比价
*   加入购物车
*   提交订单（⚠️ 支付需人工确认）
*   查看订单 / 催发货 / 开发票
*   收藏 / 浏览历史

### ❌ 官方明确限制

*   ❌ 商品评价内容
*   ❌ 自动支付
*   ❌ 高频无限制调用

---

# 五、官方推荐使用方法

淘宝官方 **强烈建议按 MCP 工作流**，不要一步到位：

```text
1️⃣ navigate（导航）
2️⃣ read_page_content（读取页面）
3️⃣ scan_page_elements（扫描可操作项）
4️⃣ click_element / input_text（操作）
5️⃣ 重复 2–4
```

---

# 六、验证配置

在 OpenClaw 聊天里直接说：

```text
帮我在淘宝搜索「机械键盘」，对比前三个价格
```

如果出现：

*   淘宝桌面客户端自动跳转
*   OpenClaw 返回结构化商品信息

✅ 说明 **淘宝 2.5.1 + taobao-native 已正确启用**

---

# 七、常见问题

### Q1：2.5.1 和 2.5.0 用法有区别吗？

✅ **没有区别**  
2.5.1 是稳定性修复版本，MCP 与 taobao-native 行为一致

---

### Q2：能不用 OpenClaw，直接用 taobao-native 吗？

✅ 可以（CLI）  
但官方主要面向 **AI Agent（OpenClaw / QoderWork）** 集成

---

# 八、故障排查

## 8.1 MCP 服务无法启动

**症状**：
```
Error: Cannot connect to taobao-native MCP
```

**排查步骤**：

1. **检查淘宝客户端版本**
```bash
# 查看版本号(在淘宝客户端"关于"中)
# 必须 >= 2.5.0
```

2. **确认 MCP 已启用**
- 淘宝客户端 → 设置 → AI 设置
- 确保 "启用 MCP 服务" 已勾选

3. **检查端口占用**
```bash
# Windows
netstat -ano | findstr 3654

# macOS/Linux
lsof -i :3654
```

4. **重启淘宝客户端**
- 完全退出淘宝(包括托盘图标)
- 重新打开并等待 10 秒

---

## 8.2 OpenClaw 找不到 taobao-native

**症状**：
```
Skill 'taobao-native' not found
```

**解决方案**：

1. **检查 Skill 文件是否存在**
```bash
ls ~/.openclaw/workspace/skills/taobao-native/SKILL.md
ls ~/.openclaw/workspace/skills/mcporter/SKILL.md
```

2. **手动下载 Skill**
```bash
# mcporter Skill
mkdir -p ~/.openclaw/workspace/skills/mcporter
cd ~/.openclaw/workspace/skills/mcporter
curl -O https://tblifecdn.taobao.com/taobaopc/skills/mcporter/SKILL.md

# taobao-native Skill
mkdir -p ~/.openclaw/workspace/skills/taobao-native
cd ~/.openclaw/workspace/skills/taobao-native
curl -O https://tblifecdn.taobao.com/taobaopc/skills/taobao-native/SKILL.md
```

3. **重启 OpenClaw Gateway**
```bash
openclaw gateway restart
openclaw doctor
```

---

## 8.3 工具调用失败

**症状**：
```
Tool execution failed: navigate
```

**常见原因**：

1. **淘宝客户端未登录**
   - 确保淘宝客户端已登录
   - MCP 使用当前登录用户身份

2. **mcporter 配置错误**
```json
// 检查 ~/.openclaw/workspace/config/mcporter.json
{
  "mcpServers": {
    "taobao-native": {
      "type": "streamableHttp",  // 必须是 streamableHttp
      "url": "http://localhost:3654/mcp"  // 检查端口
    }
  }
}
```

3. **网络问题**
```bash
# 测试 MCP 连接
curl http://localhost:3654/mcp
```

---

## 8.4 操作超时

**症状**：
```
Timeout waiting for element
```

**优化方案**：

遵循官方推荐的工作流：
```
navigate → read_page_content → scan_page_elements → click_element
```

不要跳步骤，让 AI 按顺序执行。

---

# 九、实战示例

## 9.1 基础搜索比价

**对话示例**：
```
用户: 在淘宝帮我搜索"机械键盘"，找出300元以下性价比最高的3个

OpenClaw 执行流程:
1. navigate(淘宝主页)
2. input_text(搜索框, "机械键盘")
3. click_element(搜索按钮)
4. read_page_content(商品列表)
5. 筛选价格 <= 300
6. 按销量/评价综合排序
7. 返回前3个推荐
```

---

## 9.2 加购物车并查看

**对话示例**：
```
用户: 把第2个商品加入购物车，然后显示我的购物车

OpenClaw 执行流程:
1. click_element(第2个商品的加购按钮)
2. navigate(购物车页面)
3. read_page_content(购物车列表)
4. 返回商品列表和总价
```

---

## 9.3 订单处理

**对话示例**：
```
用户: 帮我结算购物车，收货地址选默认的

OpenClaw 执行流程:
1. click_element(结算按钮)
2. read_page_content(确认订单页)
3. 验证地址/商品信息
4. 提交订单(⚠️ 支付需人工确认)
5. 返回订单号
```

---

## 9.4 订单查询

**对话示例**：
```
用户: 查看我最近3个订单的物流状态

OpenClaw 执行流程:
1. navigate(我的淘宝-订单)
2. read_page_content(订单列表)
3. 提取最近3个订单
4. 返回物流信息
```

---

# 十、最佳实践

## 10.1 性能优化

### ✅ 推荐做法
- 让 AI 按照官方工作流循序渐进
- 每次操作后等待页面加载完成
- 使用明确的商品描述词

### ❌ 避免做法
- 不要跳过 `read_page_content` 直接操作
- 不要频繁切换页面
- 不要使用模糊指令

---

## 10.2 安全建议

1. **账号安全**
   - 不要在公共设备上启用 MCP
   - 定期检查淘宝授权状态
   - 重要操作保持人工确认

2. **隐私保护**
   - OpenClaw 不会存储你的支付信息
   - 所有操作都在本地完成
   - MCP 仅在淘宝客户端运行时工作

3. **合规使用**
   - 仅用于个人购物
   - 不要用于商业爬取
   - 尊重淘宝服务条款

---

## 10.3 对话技巧

### 高效指令示例

✅ **明确具体**
```
"搜索华为P60手机，价格4000-5000，按销量排序，显示前5个"
```

❌ **模糊不清**
```
"帮我找个手机"
```

✅ **分步执行**
```
"先搜索商品，然后比较价格，最后加入购物车"
```

❌ **一步到位**
```
"直接给我买最好的"
```

---

## 10.4 与浏览器自动化共存

如果你同时使用两种方案：

```json
// OpenClaw 配置建议
{
  "browser": {
    "enabled": true,
    "profiles": {
      "taobao-scraping": {  // 用于数据采集
        "userDataDir": "~/.openclaw/browser-data/scraping"
      }
    }
  }
}
```

**使用场景分离**：
- `taobao-native` → 日常购物、订单管理
- `browser` → 评价抓取、价格监控

---

# 参考资源

## 官方文档
- [淘宝桌面客户端官网](https://tbbbk.com/)
- [MCP 协议规范](https://modelcontextprotocol.io/)
- [OpenClaw 官方文档](https://docs.openclaw.ai/)

## 社区资源
- [淘宝 MCP 社区讨论](https://jianghu.taobao.com/)
- [OpenClaw Discord](https://discord.gg/clawd)

## 相关文章
- [淘宝 PC 端 MCP 接入指南](https://tbbbk.com/taobao-pc-mcp-openclaw-guide-2026/)
- [小众软件：淘宝 PC AI MCP](https://www.appinn.com/taobao-pc-ai-mcp/)

---

# 更新日志

## v1.0 (2026-04-13)
- ✅ 初始版本发布
- ✅ 完整配置指南
- ✅ 方案对比章节
- ✅ 故障排查指南
- ✅ 实战示例
- ✅ 最佳实践
- ✅ 参考资源

---

# 附录

## A. 完整配置文件示例

### mcporter.json
```json
{
  "mcpServers": {
    "taobao-native": {
      "type": "streamableHttp",
      "url": "http://localhost:3654/mcp",
      "timeout": 30000
    }
  }
}
```

### openclaw.json 相关配置
```json
{
  "agent": {
    "model": "openai/gpt-4",
    "workspace": "~/.openclaw/workspace"
  },
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "allowlist": ["bash", "mcporter", "taobao-native"]
      }
    }
  }
}
```

---

## B. MCP 工具完整列表

根据官方 SKILL.md，taobao-native 提供的工具：

| 工具名称 | 功能 | 参数 |
|---------|------|------|
| `navigate` | 页面导航 | `url: string` |
| `read_page_content` | 读取页面内容 | - |
| `scan_page_elements` | 扫描可操作元素 | `selector?: string` |
| `click_element` | 点击元素 | `element_id: string` |
| `input_text` | 输入文本 | `element_id: string, text: string` |
| `scroll_page` | 滚动页面 | `direction: 'up'|'down'` |
| `get_cart` | 获取购物车 | - |
| `add_to_cart` | 加入购物车 | `product_id: string` |
| `submit_order` | 提交订单 | `order_data: object` |
| `get_orders` | 获取订单列表 | `status?: string` |

---

## C. 调试命令

```bash
# 查看 OpenClaw 日志
openclaw logs --tail 100 | grep taobao

# 测试 MCP 连接
curl -X POST http://localhost:3654/mcp \
  -H "Content-Type: application/json" \
  -d '{"method":"tools/list"}'

# 查看 Skill 状态
openclaw skills list | grep taobao

# 重新加载配置
openclaw gateway restart
openclaw doctor
```

---

## D. 一句话总结

> **淘宝桌面客户端 2.5.1 中：**
>
> ✅ taobao-native = 官方 MCP 服务  
> ✅ "启用 MCP" = 官方授权  
> ✅ AI 只是帮你操作「你本人的淘宝客户端」

---

**文档结束**  
如需帮助，请参考[官方文档](https://docs.openclaw.ai/)或加入[社区讨论](https://discord.gg/clawd)

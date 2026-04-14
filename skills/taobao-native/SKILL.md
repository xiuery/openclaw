---
name: taobao-native
description: 通过淘宝桌面客户端完成购物相关操作。Use when 用户需要搜索商品、查看详情、加入购物车、下单购买、查看订单、催发货、开发票等淘宝/天猫购物操作。
version: 1.0.0
metadata:
  openclaw:
    emoji: "🛒"
---

# 淘宝桌面客户端购物助手

## 适用场景

当用户的任务涉及以下购物相关操作时，**必须通过 mcporter 调用 taobao-native MCP 工具**：

- 搜索商品、比价
- 查看商品详情（价格、店铺、图片）
- 加入购物车
- 查看/管理订单（催发货、开发票等）
- 查看浏览历史、收藏
- 通过旺旺与商家聊天
- 任何涉及淘宝、天猫的页面操作

## 优势

- 直接操作用户已登录的淘宝桌面客户端，无需处理登录、验证码或反爬
- 使用 Chromium 原生输入事件，比浏览器自动化更可靠
- 在用户本地桌面应用中执行，不会打开额外浏览器窗口

## 如何调用
确保mcporter在系统PATH中，如果不在通过npm install -g mcporter来安装下

```bash
# 列出所有可用工具
mcporter list taobao-native --schema

# 调用工具示例
mcporter call taobao-native.search_products --args '{"keyword":"连衣裙"}' --output json
mcporter call taobao-native.navigate --args '{"page":"cart"}' --output json
mcporter call taobao-native.navigate --args '{"page":"cart","searchKey":"面膜"}' --output json
mcporter call taobao-native.navigate --args '{"page":"order","searchKey":"耳机"}' --output json
mcporter call taobao-native.read_page_content --args '{"scope":"main"}' --output json
mcporter call taobao-native.scan_page_elements --args '{"filter":"购物车"}' --output json
mcporter call taobao-native.click_element --args '{"index":5}' --output json
mcporter call taobao-native.add_to_cart --args '{"itemId":"601542707852","sku":["黑色","XL"]}' --output json
mcporter call taobao-native.open_chat --args '{"productName":"连衣裙","message":"请问什么时候发货？", "source": "order"}' --output json
```

始终加 `--output json` 以获取结构化结果。

## 工作流程

1. `navigate` — 导航到目标页面（购物车/订单页可传 searchKey 自动筛选）
2. `read_page_content` — 读取页面可见文本
3. `scan_page_elements` — 扫描可交互元素
4. `click_element` 或 `input_text` — 执行交互
5. `close_page` — 任务完成后关闭页面

## 工具速查

### 导航

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| navigate | 导航到淘宝预设页面 | page: home/cart/order/message/tmall/..., searchKey?: 搜索关键词(仅 cart/order 页面支持，导航后自动筛选) |
| navigate_to_url | 打开任意 URL | url |
| close_page | 关闭当前任务页面（首页除外） | - |

### 页面读取

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| get_current_tab | 获取当前标签页 URL 和标题 | - |
| read_page_content | 提取页面可见文本 | scope?, maxLength?, offset? |
| scroll_page | 滚动页面 | direction: up/down/top/bottom, selector?, amount? |
| inspect_page | 诊断页面 DOM 状态 | - |

### 页面交互

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| scan_page_elements | 扫描可交互元素，返回带序号的列表 | filter?, scope? |
| click_element | 点击元素 | index 或 text |
| input_text | 输入文本 | text, index?, placeholder?, scope?, submit? |

### 搜索 & 商品

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| search_products | 搜索商品并返回结果列表 | keyword |
| add_to_cart | 加入购物车（自动处理 SKU 和弹窗） | itemId?, sku? |
| get_browse_history | 获取浏览历史 | type: product/search/shop |

### 旺旺聊天

| 工具 | 用途 | 关键参数 |
|------|------|----------|
| open_chat | 【发消息必选】打开旺旺聊天并发送消息，复合工具自动完成全流程 | source(场景), message(消息内容) |
| send_chat_message | 在已打开的旺旺页面中继续发消息 | message(消息内容), shopName?(切换店铺) |

**open_chat 场景说明：**
- `source: cart` - 从购物车找商品发消息，需传 `productName`(商品关键词)
- `source: order` - 从订单列表找商品发消息，需传 `productName`(商品关键词)
- `source: search` - 搜索商品后给商家发消息，需传 `query`(搜索词)

**触发关键词：** 当用户提到"问问商家"、"旺旺"、"聊天"、"咨询"、"发消息"时，必须使用 `open_chat` 工具

## 注意事项

- 任务完成后**必须**关闭页面（首页除外）
- `search_products`、`add_to_cart`、`open_chat` 等工具已内置完整流程，调用成功后请勿再手动操作
- 导航后页面需加载时间，请等待后再读取内容
- `read_page_content` 默认最多返回 5000 字符，用 scope 缩小范围；返回值包含 `remainingLength` 字段，若 `truncated: true` 且 `remainingLength > 0`，可传入 `offset` 分段读取后续内容（如 `offset: 5000`）
- `input_text` 的 submit=true 可自动回车提交（搜索框场景）
- 旺旺聊天必须使用 `open_chat` 工具，不要手动导航到聊天页面

## 场景示例

### 商详获取主图图片
- **目标**：提取商品详情页的主图图片链接。
- **mcp调用链路**：
  1. `navigate_to_url`(url)：打开商详页
  2. `read_page_content`()：读取页面内容，返回值中 `[商品主图]` 和 `[商品图]` 标签已自动提取图片 URL，无需指定 scope
- **实测返回示例**：
  ```
  [商品图] https://img.alicdn.com/imgextra/.../O1CN01yyy.jpg
  ```

### 评价查看与总结
- **目标**：查看并总结某个商品的用户评价。
- **mcp调用链路**：
  1. `navigate_to_url`(url)：进入商详
  2. `click_element`(text: "用户评价")：点击评价 tab，`pageChanges.added` 会直接返回评价内容
  3. `read_page_content`()：读取完整页面，评价内容在 `用户评价·N` 之后；商详页默认只展示 2 条预览评价
  4. （可选）`click_element`(text: "查看全部评价")：如需查看更多评价，点击后会打开左侧评价抽屉面板（非 URL 跳转）
  5. （可选）`read_page_content`(scope: "[class*=Drawer]")：读取抽屉中的全部评价内容（约 20 条）；可用 `scroll_page`(direction: "down") 在抽屉内加载更多

### 购物车删除商品
- **目标**：在购物车中删除指定商品。
- **mcp调用链路**：
  1. `navigate`(page: "cart")：进入购物车
  2. `input_text`(text: "商品关键词", placeholder: "搜索购物车内商品", submit: true)：在购物车搜索框中搜索目标商品，筛选后只显示匹配的商品
  3. `scan_page_elements`()：**不传 filter**，扫描完整 DOM 树；搜索已大幅减少商品数量，完整树中商品名 `<a>商品标题</a>` 和删除按钮 `<div>删除</div>` 按顺序交替排列，可准确对应
  4. 根据 DOM 树中商品标题和规格信息（如 `颜色分类：冲牙器-绿色`），找到目标商品**紧跟其后**的 `<div>删除</div>` 索引
  5. `click_element`(index: 对应的删除按钮索引)：点击该商品的删除按钮（**无需先勾选复选框**）
  6. `click_element`(text: "删除")或`click_element`(index: N)：在确认弹窗中点击确认删除

### 商详旺旺聊天中发送商品链接
- **目标**：在商品详情页打开旺旺聊天，将商品（或右侧推荐商品）发送给商家。
- **mcp调用链路**：
  1. `navigate_to_url`(url)：打开商详页
  2. `click_element`(text: "联系客服")：点击商详页底部的「联系客服」按钮，打开旺旺聊天页面
  3. `scan_page_elements`(filter: "发送")：扫描右侧栏，返回：
     - `发送宝贝链接` — 发送当前正在咨询的商品
     - `发送商品`（多个） — 发送右侧「历史逛过」或「本店推荐」中的商品
     - 商品名 `<div>商品标题</div>` 紧跟在对应 `<div>发送商品</div>` **之前**
  4. `click_element`(index: 对应的「发送宝贝链接」或「发送商品」索引)：点击后商品链接自动发送到聊天中
- **实测 DOM 结构**：
  ```
  [146] <div>发送宝贝链接</div>          ← 当前咨询的商品
  [152] <div>商品A标题</div>             ← 历史逛过 / 本店推荐
  [153] <div>发送商品</div>              ← 点击发送商品A
  [156] <div>商品B标题</div>
  [157] <div>发送商品</div>              ← 点击发送商品B
  ```
- **关键说明**：
  - 右侧栏分为「正在咨询的宝贝」「本店订单」「历史逛过」「本店推荐」四个区域
  - 如需发送特定推荐商品，先从 DOM 中匹配商品标题，再点击其紧跟的 `发送商品` 按钮

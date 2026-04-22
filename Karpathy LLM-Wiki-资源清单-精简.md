# Karpathy LLM-Wiki 资源清单

> **编写日期**：2026年4月8日
> 
> **文档性质**：资源清单

---

## 方案一：LLM-Wiki Skill

### 部署模式

**🏠 个人本地运行** - 每个用户在自己的电脑上独立运行
- ✅ 无需服务器和数据库
- ✅ 数据存储在个人电脑
- ✅ 多人使用互不影响（各自独立）

### 核心资源需求

**AI编程助手（三选一）**：

**选项1：Claude Code**
- **所需订阅**：Claude Pro 或 Claude API（按使用计费）
- **访问方式**：Web界面 / VS Code扩展
- **支持平台**：Windows / Mac / Linux
- **官方网站**：https://claude.ai/

**选项2：OpenAI Codex**
- **所需订阅**：OpenAI API（按使用计费）
- **访问方式**：API调用 / VS Code扩展
- **模型**：GPT-4 / GPT-3.5-turbo
- **支持平台**：Windows / Mac / Linux
- **官方网站**：https://platform.openai.com/

**选项3：OpenClaw**
- **所需订阅**：兼容OpenAI API的模型（可使用本地或第三方服务）
- **特点**：开源工具，灵活配置
- **支持平台**：Windows / Mac / Linux

---

## 方案二：LLM-Wiki 开源实现

### 核心资源需求

#### 1. 模型资源

| 用途 | 模型 |
|------|------|
| **MCP集成** | Claude API |
| **OCR处理** | Mistral API |

#### 2. 硬件要求

| 组件 | 配置 | 说明 |
|------|------|------|
| **Web服务器** | 8核16GB × 1-2台 | 运行Next.js前端 |
| **API服务器** | 8核32GB × 1-2台 | 运行FastAPI后端 |
| **数据库** | 根据规模选择 | PostgreSQL单机/集群 |
| **对象存储** | 1TB+ | S3或兼容服务 |

---

**文档结束**

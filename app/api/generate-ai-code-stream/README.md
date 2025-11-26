# Generate AI Code Stream API

## 概述

这是一个 Next.js API 路由，用于通过 AI 模型流式生成 React 代码。它支持多个 AI 提供商（Groq、Anthropic、OpenAI、Google），并实现了智能的代码编辑和生成功能。

## 核心功能

### 1. 多 AI 提供商支持

支持以下 AI 提供商：
- **Groq**: 默认提供商，支持 Llama 等模型
- **Anthropic**: Claude 系列模型
- **OpenAI**: GPT 系列模型（包括 GPT-5 推理模型）
- **Google**: Gemini 系列模型

支持通过 Vercel AI Gateway 统一管理 API 调用。

### 2. 智能编辑模式

#### 两种工作模式

**初始生成模式** (`isEdit = false`)
- 从零开始创建完整的 React 应用
- 生成所有必需的组件（Header、Hero、Footer 等）
- 使用标准 Tailwind CSS 类
- 确保代码完整性和可运行性

**编辑模式** (`isEdit = true`)
- 对现有代码进行精确修改
- 使用智能文件选择算法
- 支持两种编辑策略：
  - **Agentic Search Workflow**: AI 驱动的搜索和定位
  - **Keyword-based Fallback**: 关键词匹配备用方案

#### 编辑模式工作流程

```
用户请求 → AI 意图分析 → 搜索计划生成 → 执行搜索 → 选择目标文件 → 生成精确修改
```

**步骤详解：**

1. **意图分析** (`/api/analyze-edit-intent`)
   - 分析用户请求的编辑类型
   - 生成搜索关键词
   - 确定编辑范围

2. **搜索执行** (`executeSearchPlan`)
   - 在文件内容中搜索相关代码
   - 返回匹配结果和位置信息

3. **目标选择** (`selectTargetFile`)
   - 从搜索结果中选择最佳目标文件
   - 确定精确的修改位置

4. **代码生成**
   - 生成最小化的代码修改
   - 保留所有未修改的代码

### 3. 会话上下文管理

#### 会话状态结构

```typescript
ConversationState {
  conversationId: string
  startedAt: number
  lastUpdated: number
  context: {
    messages: ConversationMessage[]      // 对话历史
    edits: ConversationEdit[]            // 编辑记录
    projectEvolution: {                  // 项目演进
      majorChanges: MajorChange[]
    }
    userPreferences: {}                  // 用户偏好
  }
}
```

#### 上下文优化策略

- **消息限制**: 保留最近 15 条消息
- **编辑记录**: 保留最近 8 次编辑
- **主要变更**: 保留最近 2 次重大变更
- **总长度限制**: 上下文不超过 2000 字符

### 4. 文件缓存系统

#### 缓存结构

```typescript
global.sandboxState = {
  fileCache: {
    files: {
      [path: string]: {
        content: string
        lastModified: number
      }
    }
    manifest: FileManifest
    lastSync: number
    sandboxId: string
  }
}
```

#### 缓存策略

- 优先使用后端缓存
- 缓存未命中时从沙箱获取
- 支持增量更新

### 5. 流式响应处理

#### 流式架构

```
AI 模型 → TextStream → 实时解析 → 进度更新 → 客户端
```

#### 支持的进度事件

- `status`: 状态更新（"正在分析..."）
- `stream`: 原始文本流
- `conversation`: 对话式文本
- `component`: 组件生成进度
- `package`: 检测到的 npm 包
- `complete`: 生成完成
- `error`: 错误信息
- `warning`: 警告信息

### 6. 包管理

#### 自动包检测

从以下来源检测 npm 包：
1. `<package>` XML 标签
2. `<packages>` 批量标签
3. ES6 import 语句解析

#### 检测规则

- 跳过相对路径导入 (`./`, `../`)
- 跳过内置模块 (`react`, `react-dom`)
- 跳过路径别名 (`@/`)
- 支持作用域包 (`@heroicons/react`)

### 7. 代码完整性保障

#### 截断检测

自动检测以下问题：
- 未闭合的 `<file>` 标签
- 不匹配的大括号（差异 > 3）
- 不完整的 HTML 标签
- 过短的文件内容

#### 自动修复

启用 `enableTruncationRecovery` 时：
1. 检测截断的文件
2. 为每个文件生成补全请求
3. 替换原始内容
4. 验证完整性

### 8. Morph Fast Apply 模式

当启用 Morph API 时，使用 `<edit>` 块格式：

```xml
<edit target_file="src/components/Header.jsx">
  <instructions>修改背景颜色为蓝色</instructions>
  <update>className="bg-blue-500"</update>
</edit>
```

优势：
- 更快的应用速度
- 更小的数据传输
- 精确的修改定位

## 系统提示词架构

### 核心规则层级

1. **基础规则**: 代码风格、文件结构
2. **编辑规则**: 精确修改、保留现有代码
3. **完整性规则**: 无截断、完整文件
4. **包管理规则**: 何时使用外部包
5. **UI/UX 规则**: Tailwind 使用、响应式设计

### 关键约束

- ✅ 使用标准 Tailwind 类（`bg-white`, `text-gray-900`）
- ❌ 禁止自定义类（`bg-background`, `text-foreground`）
- ✅ 完整文件输出，无省略号
- ❌ 禁止截断代码
- ✅ 最小化修改范围
- ❌ 禁止重写整个组件

## API 接口

### 请求格式

```typescript
POST /api/generate-ai-code-stream

{
  prompt: string                    // 用户请求
  model?: string                    // AI 模型（默认: openai/gpt-oss-20b）
  isEdit?: boolean                  // 是否为编辑模式
  context?: {
    sandboxId?: string              // 沙箱 ID
    currentFiles?: Record<string, string>  // 当前文件
    structure?: string              // 文件结构
    conversationContext?: {
      scrapedWebsites?: Array<{
        url: string
        timestamp: number
        content: any
      }>
      currentProject?: string
    }
  }
}
```

### 响应格式

Server-Sent Events (SSE) 流：

```typescript
// 进度事件
data: {"type":"status","message":"正在分析..."}

// 流式文本
data: {"type":"stream","text":"const ","raw":true}

// 组件完成
data: {"type":"component","name":"Hero","path":"src/components/Hero.jsx"}

// 完成事件
data: {"type":"complete","generatedCode":"...","files":5,"components":3}
```

## 错误处理

### 重试机制

- 最多重试 2 次
- 指数退避（2s, 4s）
- 服务不可用时自动切换模型

### 错误类型

1. **流式错误**: AI 服务不可用
2. **解析错误**: 代码格式问题
3. **截断错误**: 内容不完整
4. **缓存错误**: 文件获取失败

## 性能优化

### 1. 上下文管理

- 限制消息历史长度
- 清理旧编辑记录
- 压缩会话数据

### 2. 流式传输

- 实时进度更新
- 分块传输
- 禁用缓冲

### 3. 缓存策略

- 文件内容缓存
- Manifest 缓存
- 增量同步

## 配置选项

### 环境变量

```bash
# AI 提供商密钥
GROQ_API_KEY=
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
GEMINI_API_KEY=

# AI Gateway（可选）
AI_GATEWAY_API_KEY=

# Anthropic 自定义端点（可选）
ANTHROPIC_BASE_URL=

# OpenAI 自定义端点（可选）
OPENAI_BASE_URL=

# Morph Fast Apply（可选）
MORPH_API_KEY=

# 应用 URL
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 应用配置

```typescript
// config/app.config.ts
{
  codeApplication: {
    enableTruncationRecovery: true  // 启用截断恢复
  },
  ai: {
    defaultTemperature: 0.7         // 默认温度
  }
}
```

## 使用示例

### 初始生成

```typescript
const response = await fetch('/api/generate-ai-code-stream', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    prompt: '创建一个现代化的落地页，包含 Hero、Features 和 Footer',
    model: 'openai/gpt-4',
    isEdit: false
  })
});

const reader = response.body.getReader();
// 处理流式响应...
```

### 编辑现有代码

```typescript
const response = await fetch('/api/generate-ai-code-stream', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    prompt: '将 Header 的背景改为黑色',
    model: 'anthropic/claude-3-5-sonnet-20241022',
    isEdit: true,
    context: {
      sandboxId: 'abc123',
      currentFiles: { /* ... */ }
    }
  })
});
```

## 最佳实践

### 1. 编辑请求

- 明确指定要修改的组件
- 使用精确的描述（"改变颜色" 而非 "更新样式"）
- 一次只修改一个功能点

### 2. 初始生成

- 提供详细的需求描述
- 指定所需的组件和功能
- 说明设计风格偏好

### 3. 性能优化

- 使用编辑模式而非重新生成
- 保持会话上下文简洁
- 定期清理缓存

## 故障排查

### 常见问题

**问题**: 生成的代码被截断
- **原因**: Token 限制或流式中断
- **解决**: 启用 `enableTruncationRecovery`

**问题**: 编辑模式修改了过多文件
- **原因**: 意图分析不准确
- **解决**: 使用更精确的提示词

**问题**: 包未自动安装
- **原因**: 包检测失败
- **解决**: 使用 `<package>` 标签显式声明

## 技术栈

- **框架**: Next.js 15
- **AI SDK**: Vercel AI SDK
- **流式**: Server-Sent Events (SSE)
- **类型**: TypeScript
- **AI 模型**: Groq, Anthropic, OpenAI, Google

## 相关文件

- `/lib/context-selector.ts` - 文件选择逻辑
- `/lib/file-search-executor.ts` - 搜索执行器
- `/api/analyze-edit-intent/route.ts` - 意图分析 API
- `/types/conversation.ts` - 会话类型定义
- `/types/sandbox.ts` - 沙箱类型定义

## 更新日志

查看 Git 提交历史了解详细的更新记录。

## 许可证

参见项目根目录的 LICENSE 文件。

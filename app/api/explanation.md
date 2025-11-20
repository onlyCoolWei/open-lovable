`app/api` 文件夹包含 Next.js API 路由，用于处理后端功能。主要功能如下：

## 核心功能模块

### 1. AI 代码生成与应用
- `generate-ai-code-stream/route.ts` - 流式生成代码（支持 Groq、OpenAI、Anthropic、Google）
- `apply-ai-code-stream/route.ts` - 流式应用生成的代码到沙箱
- `apply-ai-code/route.ts` - 非流式版本
- `analyze-edit-intent/route.ts` - 分析用户编辑意图，生成搜索计划

### 2. 沙箱管理
- `create-ai-sandbox/route.ts` - 创建 AI 沙箱环境（旧版）
- `create-ai-sandbox-v2/route.ts` - 创建沙箱（新版，使用工厂模式）
- `get-sandbox-files/route.ts` - 获取沙箱文件列表和结构
- `sandbox-status/route.ts` - 查询沙箱状态
- `sandbox-logs/route.ts` - 获取沙箱日志
- `kill-sandbox/route.ts` - 终止沙箱

### 3. 包管理
- `install-packages/route.ts` - 安装 npm 包（旧版）
- `install-packages-v2/route.ts` - 安装包（新版）
- `detect-and-install-packages/route.ts` - 自动检测并安装依赖

### 4. 网站抓取
- `scrape-website/route.ts` - 使用 Firecrawl 抓取网站内容
- `scrape-url-enhanced/route.ts` - 增强版 URL 抓取
- `scrape-screenshot/route.ts` - 抓取网站截图

### 5. 对话状态管理
- `conversation-state/route.ts` - 管理对话历史、编辑记录、项目演进

### 6. Vite 开发服务器管理
- `monitor-vite-logs/route.ts` - 监控 Vite 日志
- `check-vite-errors/route.ts` - 检查 Vite 错误
- `report-vite-error/route.ts` - 报告 Vite 错误
- `restart-vite/route.ts` - 重启 Vite
- `clear-vite-errors-cache/route.ts` - 清除错误缓存

### 7. 其他工具
- `run-command/route.ts` - 在沙箱中执行命令（旧版）
- `run-command-v2/route.ts` - 执行命令（新版）
- `create-zip/route.ts` - 创建项目压缩包
- `search/route.ts` - 搜索功能

## 工作流程

1. 用户输入 → `generate-ai-code-stream` 生成代码
2. 代码应用 → `apply-ai-code-stream` 写入沙箱
3. 包安装 → `install-packages-v2` 安装依赖
4. 文件管理 → `get-sandbox-files` 获取文件结构
5. 状态跟踪 → `conversation-state` 记录对话历史

## 技术特点

- 流式处理：使用 Server-Sent Events (SSE) 实时返回进度
- 多模型支持：Groq、OpenAI、Anthropic、Google
- 智能编辑：通过 AI 分析编辑意图，精确定位需要修改的文件
- 沙箱隔离：每个项目运行在独立的沙箱环境中

这是一个 AI 驱动的代码生成与编辑平台的后端 API 系统。
`generationProgress` 用于跟踪和管理 AI 代码生成的实时进度和状态。

## 主要作用

### 1. 状态管理
跟踪生成过程的各个阶段：
- `isGenerating`: 是否正在生成代码
- `isStreaming`: 是否正在流式传输代码
- `isThinking`: AI 是否在思考
- `isEdit`: 是否为编辑模式

### 2. 进度追踪
记录生成的具体内容：
- `streamedCode`: 实时流式传输的代码内容
- `files`: 已生成/解析的文件列表（包含路径、内容、类型）
- `currentFile`: 当前正在生成的文件
- `components`: 已生成的组件列表
- `status`: 当前状态文本（如 "Generating code..."）

### 3. 思考过程显示
显示 AI 的思考过程：
- `thinkingText`: AI 思考的文本内容
- `thinkingDuration`: 思考持续时间

### 4. UI 渲染
用于渲染多个 UI 组件：

```117:141:app/generation/page.tsx
  const [generationProgress, setGenerationProgress] = useState<{
    isGenerating: boolean;
    status: string;
    components: Array<{ name: string; path: string; completed: boolean }>;
    currentComponent: number;
    streamedCode: string;
    isStreaming: boolean;
    isThinking: boolean;
    thinkingText?: string;
    thinkingDuration?: number;
    currentFile?: { path: string; content: string; type: string };
    files: Array<{ path: string; content: string; type: string; completed: boolean; edited?: boolean }>;
    lastProcessedPosition: number;
    isEdit?: boolean;
  }>({
    isGenerating: false,
    status: '',
    components: [],
    currentComponent: 0,
    streamedCode: '',
    isStreaming: false,
    isThinking: false,
    files: [],
    lastProcessedPosition: 0
  });
```

- 代码生成标签页：显示实时生成的代码和文件列表
- 文件浏览器：显示已生成的文件树
- 进度条：显示组件生成进度
- 聊天消息：显示生成的文件列表
- 状态指示器：显示 "Live generation" 或 "Editing code"

### 5. 数据流处理
在 `sendChatMessage` 和 `startGeneration` 中，从 SSE 流中解析并更新状态，实时显示：
- 新生成的文件
- 正在编辑的文件
- 流式传输的代码内容
- AI 思考过程

总结：`generationProgress` 是代码生成功能的核心状态管理对象，负责跟踪生成进度、管理文件列表，并驱动相关 UI 的实时更新。
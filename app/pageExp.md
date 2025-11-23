## 页面整体作用

这是 Open Lovable 的主页组件，提供两种输入方式：
1. URL 直接抓取：输入网站 URL，直接进入生成流程
2. 关键词搜索：搜索关键词，从搜索结果中选择网站进行克隆

## 核心功能模块

### 1. 状态管理 (第 46-58 行)

```46:58:app/page.tsx
  const [url, setUrl] = useState<string>("");
  const [selectedStyle, setSelectedStyle] = useState<string>("1");
  const [selectedModel, setSelectedModel] = useState<string>(appConfig.ai.defaultModel);
  const [isValidUrl, setIsValidUrl] = useState<boolean>(false);
  const [showSearchTiles, setShowSearchTiles] = useState<boolean>(false);
  const [searchResults, setSearchResults] = useState<SearchResult[]>([]);
  const [isSearching, setIsSearching] = useState<boolean>(false);
  const [hasSearched, setHasSearched] = useState<boolean>(false);
  const [isFadingOut, setIsFadingOut] = useState<boolean>(false);
  const [showSelectMessage, setShowSelectMessage] = useState<boolean>(false);
  const [showInstructionsForIndex, setShowInstructionsForIndex] = useState<number | null>(null);
  const [additionalInstructions, setAdditionalInstructions] = useState<string>('');
```

### 2. URL 验证 (第 61-72 行)

- `validateUrl()`: 基础 URL 格式验证
- `isURL()`: 判断输入是否为 URL（根据是否包含域名特征）

### 3. 样式选择器 (第 74-83 行)

提供 8 种设计风格：
- Glassmorphism、Neumorphism、Brutalism、Minimalist、Dark Mode、Gradient Rich、3D Depth、Retro Wave

### 4. 核心提交逻辑 (第 90-175 行)

`handleSubmit()` 处理三种场景：
- 选择搜索结果：保存选中结果并跳转到生成页
- 直接输入 URL：立即跳转到生成页
- 搜索关键词：调用搜索 API 并展示结果

### 5. 搜索功能 (第 178-204 行)

`performSearch()`:
- 调用 `/api/search`
- 处理加载和错误状态
- 更新搜索结果

## UI 组件结构

### 1. 顶部导航栏 (第 212-239 行)
- 粘性定位
- 品牌 Logo
- GitHub 链接

### 2. Hero 区域 (第 242-486 行)
- 背景效果组件 (Pixi、Flame、ASCII 特效)
- 标题和描述
- 主输入框（支持 URL/搜索切换）

### 3. 动态输入框 (第 285-412 行)

根据状态显示不同 UI：
- 搜索模式：显示搜索图标和输入框
- URL 模式：显示抓取图标和输入框
- 选择模式：显示选择提示和“重新搜索”按钮

### 4. 选项面板 (第 415-476 行)

仅在输入有效 URL 时展开：
- 样式选择器（8 个按钮）
- AI 模型下拉选择
- 附加说明输入框

### 5. 搜索结果轮播 (第 489-759 行)

三种状态：
- 加载中：骨架屏无限滚动
- 有结果：卡片轮播展示（每张卡片支持悬停交互）
- 无结果：空状态提示

### 6. 搜索结果卡片交互 (第 543-740 行)

每个卡片提供两种克隆方式：
- 即时克隆：直接跳转
- 添加说明：展开输入框，可添加自定义指令

## 数据流转

```
用户输入 → URL验证/搜索判断 → 
├─ URL: 直接跳转到 /generation
└─ 搜索词: 调用 /api/search → 展示结果卡片 → 
   用户选择 → 保存到 sessionStorage → 跳转到 /generation
```

## sessionStorage 数据传递

跳转前保存的数据：
- `targetUrl`: 目标网站 URL
- `selectedStyle`: 选中的样式 ID
- `selectedModel`: 选中的 AI 模型
- `autoStart`: 自动开始标识
- `siteMarkdown`: 网站内容（如果有）
- `additionalInstructions`: 额外说明

## 动画效果

1. 无限滚动轮播：30 秒循环，悬停暂停
2. 骨架屏闪烁：加载状态动画
3. 淡入淡出：搜索结果切换过渡
4. 平滑滚动：自动滚动到结果区域

## 技术特点

- 客户端组件 (`"use client"`)
- 响应式设计（移动端/桌面端适配）
- 开发环境集成 Locator UI（用于调试）
- 使用 Sonner 进行消息提示
- 模块化组件结构

该页面是应用的入口，负责收集用户意图并传递到生成页面进行网站克隆。


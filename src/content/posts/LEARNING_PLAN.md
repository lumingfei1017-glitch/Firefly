# Pi Agent Harness 学习计划

> 项目地址: `D:\code\lc_worksapce\pi` | `https://github.com/earendil-works/pi-mono`
>
> Pi 是一个类 Claude Code 的 **编码智能体框架 (Coding Agent Harness)**，支持命令行交互、远程会话、多 LLM 提供商、终端 UI 渲染等能力。

---

## 一、项目概览

### 1.1 这是什么？

Pi 是一个**模块化、可扩展的 AI 编码助手**，核心功能是：
- 在终端中与 LLM 进行交互式对话
- 自动调用工具（读文件、写文件、搜索、执行命令等）
- 支持多轮 Agent 循环（思考→调用工具→观察结果→继续思考）
- 支持多种 LLM 提供商（Anthropic、OpenAI、Google、Mistral 等 30+）
- 支持远程会话（client/server 架构）
- 拥有自己的终端 UI 渲染引擎

### 1.2 技术栈

| 技术 | 用途 |
|------|------|
| TypeScript | 全部源代码，使用 erasable 语法 |
| Node.js >= 22 | 运行时 |
| Bun | 可选的二进制打包 |
| esbuild | 构建打包 |
| Biome | 格式化和 lint |
| vitest | 测试框架 |
| CBOR | 二进制协议序列化 |
| SQLite | 会话持久化 |

### 1.3 Monorepo 结构一览

```
pi/
├── packages/
│   ├── chord/          # ★ 应用组合运行时 (基础设施层)
│   ├── ai/             # ★ 统一 LLM API (多提供商适配)
│   ├── agent/          # ★ 智能体核心 (Agent Loop + 工具)
│   ├── tui/            # ★ 终端 UI 渲染引擎
│   ├── coding-agent/   # ★ 主入口 CLI 应用
│   ├── protocol/       # 远程会话协议 (CBOR)
│   ├── client/         # 远程客户端
│   ├── server/         # 远程服务端
│   ├── telemetry/      # 遥测合约
│   ├── session-backends/sqlite-node/  # SQLite 会话存储
│   └── evals/          # 评估框架
└── scripts/            # 构建/发布脚本
```

---

## 二、学习路线

按依赖关系从底层到顶层，共 6 个阶段。

---

### 阶段 1：基础设施 — `chord` 包

**难度：★★★☆☆** | **前置：无**

#### 2.1 这是什么

`chord` 是 Pi 的**应用组合运行时**，类似一个轻量级 IoC 容器。它管理服务的生命周期、依赖注入、RPC 通信和插件系统。

#### 2.2 关键文件

| 文件 | 内容 |
|------|------|
| `packages/chord/src/api.ts` | 核心 API 定义 |
| `packages/chord/src/context/` | 服务上下文 (Context) 实现 |
| `packages/chord/src/services/` | 服务生命周期管理 |
| `packages/chord/src/delta/` | 增量更新机制 |
| `packages/chord/src/facets/` | 切面 (Facets) 扩展点 |
| `packages/chord/src/node.ts` | Node.js 适配 |

#### 2.3 关键概念

- **Service** — 可注册/可解析的具名服务实例
- **Context** — 服务间的共享上下文，携带请求级数据
- **Facet** — 服务的切面（类似 AOP），可以拦截、增强服务行为
- **Delta** — 增量更新机制，用于高效的状态同步
- **Bundler** — 将多个服务打包为可发布的模块

#### 2.4 学习要点

- [ ] 理解 `Service` 的注册和解析流程
- [ ] 理解 `Context` 如何传递和使用
- [ ] 阅读 `chord/src/api.ts` — 这是最核心的接口定义
- [ ] 理解 `Facet` 的切面机制（这是 Pi 扩展性的基础）
- [ ] 理解 `Delta` 的增量更新模式

#### 2.5 为什么重要

chord 是 Pi 的骨架，**所有上层包都依赖它**。理解 chord 就理解了 Pi 如何组织代码。

---

### 阶段 2：AI 抽象层 — `ai` 包

**难度：★★★★☆** | **前置：阶段 1**

#### 3.1 这是什么

`ai` 包是 Pi 的**统一 LLM API**，为 30+ 不同 LLM 提供商提供统一的调用接口。

#### 3.2 关键文件

| 文件 | 内容 |
|------|------|
| `packages/ai/src/types.ts` | 核心类型定义 (Message, Model, Transport 等) |
| `packages/ai/src/index.ts` | 包入口，导出所有公共 API |
| `packages/ai/src/api/` | 各提供商的 API 实现 |
| `packages/ai/src/providers/` | 各提供商的模型注册和配置 |
| `packages/ai/src/models.ts` | 模型类型定义 |
| `packages/ai/src/models-store.ts` | 模型存储和查询 |
| `packages/ai/src/model-catalog.ts` | 模型目录管理 |
| `packages/ai/src/compat/` | 兼容层 (旧版 API 适配) |
| `packages/ai/src/auth/` | 认证系统 (OAuth, credential store) |
| `packages/ai/src/providers/anthropic.ts` | Anthropic 示例实现 |
| `packages/ai/src/providers/openai.ts` | OpenAI 示例实现 |

#### 3.3 关键概念

- **Transport** — 与 LLM 通信的传输层抽象
- **Model** — 模型元数据（名称、上下文窗口、价格等）
- **Provider** — 提供商适配器（将统一 API 转为各提供商 API）
- **Message** — 统一消息格式（支持 text, image, tool_use, tool_result 等）
- **StreamFn** — 流式调用函数签名
- **Credential Store** — 多提供商认证凭据管理

#### 3.4 学习要点

- [ ] 理解 `Message` 类型体系 — 这是所有 LLM 通信的基础
- [ ] 理解 `Transport` 接口 — 如何抽象不同 LLM 的 API 差异
- [ ] 理解 `Provider` 的注册机制 — 如何添加新提供商
- [ ] 阅读一个具体的 Provider 实现（如 `anthropic.ts`），理解适配模式
- [ ] 理解 `Model` 的发现和选择流程
- [ ] 理解 OAuth 认证流程（`auth/` 目录）
- [ ] 理解模型数据生成（`scripts/generate-models.ts` → `models.generated.ts`）

#### 3.5 难点

- 提供商的 **API 差异巨大**（流式 vs 非流式、tool use 格式、thinking 支持等），需要统一抽象
- 模型元数据自动发现和更新机制
- 认证系统支持多种模式（API key, OAuth, 环境变量）

---

### 阶段 3：Agent 核心 — `agent` 包

**难度：★★★★★** | **前置：阶段 1, 2**

#### 4.1 这是什么

`agent` 包是 Pi 的**智能体运行时**，实现了 Agent Loop（思考→调用工具→观察→继续思考）的核心循环。

#### 4.2 关键文件

| 文件 | 内容 |
|------|------|
| `packages/agent/src/agent.ts` | Agent 核心逻辑，包含 `createAgent` API |
| `packages/agent/src/agent-loop.ts` | Agent Loop 实现（主循环逻辑） |
| `packages/agent/src/types.ts` | 核心类型定义 |
| `packages/agent/src/stream-fn.ts` | 流式函数封装 |
| `packages/agent/src/harness/` | Agent Harness — 完整的 agent 运行环境 |
| `packages/agent/src/harness/agent-harness.ts` | Harness 主入口 |
| `packages/agent/src/harness/context.ts` | 执行上下文 |
| `packages/agent/src/harness/runtime/` | 运行时（执行、reducer 等） |
| `packages/agent/src/harness/session/` | 会话管理 |
| `packages/agent/src/harness/tools/` | 工具系统 |
| `packages/agent/src/harness/env/` | 环境适配（Node.js 等） |
| `packages/agent/src/search/` | 搜索功能实现 |

#### 4.3 关键概念

- **Agent Loop** — 核心循环: `LLM 响应 → 解析工具调用 → 执行工具 → 返回结果 → 继续`
- **AgentState** — 智能体状态（消息历史、工具列表、模型配置等）
- **AgentTool** — 工具定义（名称、参数 schema、执行函数）
- **StreamFn** — 流式调用 LLM 的函数
- **Harness** — 完整的 Agent 运行环境（包含工具注册、会话管理、上下文等）
- **Tool Execution Mode** — 工具执行模式（单步、全部、直到完成等）
- **Queue Mode** — 消息队列模式

#### 4.4 Agent Loop 流程

```
用户输入 → prepareNextTurn → 调用 LLM
  → LLM 返回文本/工具调用
  → beforeToolCall (钩子)
  → 执行工具
  → afterToolCall (钩子)
  → shouldStopAfterTurn? (停止判断)
  → 继续或返回结果
```

#### 4.5 学习要点

- [ ] 理解 Agent Loop 的完整流程（`agent-loop.ts`）
- [ ] 理解 `createAgent` 的配置项和作用
- [ ] 理解 `AgentState` 的状态管理
- [ ] 理解工具定义和执行机制（`AgentTool` 接口）
- [ ] 理解 Hook 系统（`beforeToolCall`, `afterToolCall`, `prepareNextTurn` 等）
- [ ] 理解 Harness 的完整架构（`harness/` 目录）
- [ ] 理解会话（Session）的生命周期管理
- [ ] 理解搜索功能的实现（`search/` 目录）

#### 4.6 难点

- **Agent Loop 的异步流控制** — 流式 LLM 响应、工具执行、状态更新的交织
- **状态管理** — 消息历史的增量更新、并发控制
- **Hook 系统** — 提供了极大的扩展性，但也增加了理解难度
- **Harness 架构** — 包含多个子系统（session, runtime, tools, env），需要全局理解

---

### 阶段 4：终端 UI — `tui` 包

**难度：★★★★☆** | **前置：无（独立）**

#### 5.1 这是什么

`tui` 包是 Pi 的**终端 UI 渲染引擎**，支持差分渲染、布局系统、组件化等。

#### 5.2 关键文件

| 文件 | 内容 |
|------|------|
| `packages/tui/src/tui.ts` | TUI 引擎核心 |
| `packages/tui/src/tui-alt-screen.ts` | 备用屏幕模式 |
| `packages/tui/src/tui-main-screen.ts` | 主屏幕模式 |
| `packages/tui/src/terminal.ts` | 终端抽象层 |
| `packages/tui/src/layout.ts` | 布局系统 |
| `packages/tui/src/layout-node.ts` | 布局节点 |
| `packages/tui/src/components/` | 组件库（Markdown, Editor, Text, Box 等） |
| `packages/tui/src/keybindings.ts` | 按键绑定系统 |
| `packages/tui/src/keys.ts` | 按键处理 |
| `packages/tui/src/editor-component.ts` | 编辑器组件 |

#### 5.3 关键概念

- **差分渲染** — 只更新变化的部分，提高性能
- **备用屏幕** — 使用终端 alternate screen buffer
- **布局系统** — VStack, HStack, ScrollView 等布局组件
- **组件树** — 组件组合成树，递归渲染
- **按键绑定** — 可配置的快捷键系统
- **编辑器** — 内嵌的终端编辑器

#### 5.4 学习要点

- [ ] 理解 TUI 引擎的渲染流程
- [ ] 理解差分渲染的实现原理
- [ ] 理解布局系统（`layout.ts` + `layout-node.ts`）
- [ ] 理解组件生命周期和渲染
- [ ] 阅读几个核心组件（Markdown, Editor, Text, Box）
- [ ] 理解按键处理和绑定系统
- [ ] 理解备用屏幕和主屏幕的区别（`tui-plan.md` 有详细说明）
- [ ] 理解 Native 模块（Windows / macOS 终端适配）

#### 5.5 难点

- **差分渲染算法** — 如何高效计算终端内容的差异
- **布局系统** — 约束布局的递归计算
- **终端兼容性** — 不同终端（Windows Terminal, iTerm2, xterm 等）的差异处理
- **Native 模块** — 通过 Node.js Native Addon 实现底层终端操作

---

### 阶段 5：主应用 — `coding-agent` 包

**难度：★★★★★** | **前置：阶段 1, 2, 3, 4**

#### 6.1 这是什么

`coding-agent` 是 Pi 的**主 CLI 应用**，整合了所有下层包，提供完整的编码助手体验。

#### 6.2 关键文件

| 文件                                                                | 内容                                          |
| ----------------------------------------------------------------- | ------------------------------------------- |
| `packages/coding-agent/src/main.ts`                               | **应用入口点** — CLI 参数解析、模式选择、初始化               |
| `packages/coding-agent/src/cli.ts`                                | CLI 入口（bin 指向）                              |
| `packages/coding-agent/src/config.ts`                             | 全局配置                                        |
| `packages/coding-agent/src/core/agent-session.ts`                 | Agent 会话实现                                  |
| `packages/coding-agent/src/core/agent-session-services.ts`        | 会话服务创建                                      |
| `packages/coding-agent/src/core/agent-session-runtime.ts`         | 会话运行时                                       |
| `packages/coding-agent/src/core/sdk.ts`                           | **SDK 入口** — 对外暴露的 API                      |
| `packages/coding-agent/src/core/tools/`                           | 工具实现（bash, read, write, edit, grep, find 等） |
| `packages/coding-agent/src/core/system-prompt.ts`                 | 系统提示词                                       |
| `packages/coding-agent/src/core/settings-manager.ts`              | 设置管理                                        |
| `packages/coding-agent/src/core/session-manager.ts`               | 会话管理                                        |
| `packages/coding-agent/src/core/model-runtime.ts`                 | 模型运行时                                       |
| `packages/coding-agent/src/core/model-resolver.ts`                | 模型解析                                        |
| `packages/coding-agent/src/core/extensions/`                      | 扩展系统                                        |
| `packages/coding-agent/src/modes/`                                | 运行模式（交互式、打印模式、RPC 等）                        |
| `packages/coding-agent/src/modes/interactive/interactive-mode.ts` | 交互模式主入口                                     |
| `packages/coding-agent/src/modes/interactive/tui-renderer.ts`     | TUI 渲染器                                     |
| `packages/coding-agent/src/modes/print-mode.ts`                   | 非交互打印模式                                     |
| `packages/coding-agent/src/extensions/`                           | 内置扩展                                        |

#### 6.3 运行模式

| 模式 | 说明 |
|------|------|
| **Interactive Mode** | 全屏 TUI 交互式对话（默认模式） |
| **Print Mode** | 一次性问答模式（`-p "提问"`） |
| **RPC Mode** | JSON-RPC 协议模式（用于程序化调用） |
| **Client Mode** | 连接远程 Pi 服务端 |
| **Server Mode** | 启动远程 Pi 服务端 |

#### 6.4 核心工具集

| 工具 | 说明 |
|------|------|
| `bash` | 执行 Shell 命令 |
| `read` | 读取文件内容 |
| `write` | 写入文件 |
| `edit` | 编辑文件（精确替换） |
| `grep` | 搜索文件内容 |
| `find` | 查找文件 |
| `ls` | 列出目录 |

#### 6.5 学习要点

- [ ] 从 `main.ts` 入手，理解应用启动流程
- [ ] 理解 CLI 参数解析和模式选择
- [ ] 理解 `createAgentSession` 的完整流程 — 这是核心 API
- [ ] 理解会话的生命周期管理（`SessionManager`）
- [ ] 理解设置系统（`SettingsManager`）
- [ ] 理解模型解析和选择流程（`model-resolver.ts`）
- [ ] 理解工具注册和执行机制（`core/tools/` 目录）
- [ ] 理解交互模式的 UI 渲染（`tui-renderer.ts`）
- [ ] 理解扩展系统（`extensions/` 目录）
- [ ] 理解系统提示词构建（`system-prompt.ts`）
- [ ] 理解会话保存和恢复机制
- [ ] 理解项目信任机制（`project-trust.ts`, `trust-manager.ts`）

#### 6.6 难点

- **应用启动流程复杂** — 涉及大量的初始化、配置加载、认证检查
- **多模式切换** — 同一核心逻辑在不同模式下有不同的表现
- **工具实现** — 每个工具的实现需要考虑多种边界情况
- **会话管理** — 持久化、恢复、历史管理
- **扩展系统** — 插件的加载、生命周期管理

---

### 阶段 6：远程协议 — `protocol` + `client` + `server` 包

**难度：★★★☆☆** | **前置：阶段 1**

#### 7.1 这是什么

Pi 支持远程会话，允许客户端连接远程服务器。这三个包实现了这一能力。

#### 7.2 关键文件

| 包 | 文件 | 内容 |
|----|------|------|
| protocol | `cbor/codec.ts` | CBOR 编解码 |
| protocol | `framing.ts` | 消息帧协议 |
| protocol | `protocol.ts` | 协议定义 |
| client | `client.ts` | 客户端实现 |
| client | `connection.ts` | 连接管理 |
| client | `transport.ts` | 传输层抽象 |
| server | `server.ts` | 服务端实现 |
| server | `listener.ts` | 监听器 |
| server | `session-router.ts` | 会话路由 |

#### 7.3 学习要点

- [ ] 理解 CBOR 协议和消息帧结构
- [ ] 理解客户端-服务端通信流程
- [ ] 理解传输层抽象（支持 Unix Socket 等）

---

## 三、学习路径图

```
阶段 1: chord (应用组合)
    ↓
阶段 2: ai (统一 LLM API) ──→ 阶段 4: tui (终端 UI) [独立]
    ↓                              ↓
阶段 3: agent (智能体核心) ──→ 阶段 5: coding-agent (主应用)
                                    ↓
                             阶段 6: protocol/client/server (远程)
```

**建议并行学习：** 阶段 4 (tui) 与阶段 2-3 没有依赖关系，可以并行学习。

---

## 四、关键阅读顺序

### 快速入门（1 小时内）

```
1. README.md                    → 项目概览
2. AGENTS.md                    → 开发规则（理解代码风格和约定）
3. CONTRIBUTING.md              → 贡献指南
4. packages/chord/PLANNING.md   → chord 设计理念
5. packages/coding-agent/src/main.ts → 应用入口，理解整体流程
```

### 深入核心（1-2 天）

```
1. packages/ai/src/types.ts         → AI 核心类型
2. packages/ai/src/api/anthropic.ts  → 具体 LLM 适配示例
3. packages/agent/src/types.ts       → Agent 核心类型
4. packages/agent/src/agent-loop.ts  → Agent Loop 核心逻辑
5. packages/agent/src/agent.ts       → Agent 创建和管理
6. packages/tui/src/tui.ts           → TUI 引擎
7. packages/tui/src/components/markdown.ts → 核心组件
8. packages/coding-agent/src/core/sdk.ts → SDK 入口
```

### 全面掌握（1 周）

```
1. packages/agent/src/harness/       → 完整的 Harness 架构
2. packages/coding-agent/src/core/   → 所有核心模块
3. packages/coding-agent/src/modes/  → 所有运行模式
4. packages/coding-agent/src/core/tools/ → 工具实现
5. packages/chord/src/               → 完整 chord 理解
6. packages/protocol/src/            → 远程协议
7. tui-plan.md                       → TUI 布局系统设计文档
```

---

## 五、难点地图

### 🔴 核心难点（理解即可，无需精通）

| 难点 | 位置 | 说明 |
|------|------|------|
| Agent Loop 异步流控制 | `agent/src/agent-loop.ts` | 流式 LLM、工具执行、状态更新的交织 |
| 多提供商适配 | `ai/src/providers/` | 30+ 提供商的 API 差异统一 |
| TUI 差分渲染 | `tui/src/tui.ts` | 高效计算终端内容差异的算法 |
| 会话管理 | `coding-agent/src/core/session-manager.ts` | 持久化、恢复、并发控制 |
| 布局系统 | `tui/src/layout.ts` | 约束布局的递归计算 |

### 🟡 扩展难点（按需学习）

| 难点 | 位置 | 说明 |
|------|------|------|
| 插件/扩展系统 | `coding-agent/src/core/extensions/` | 插件的加载和生命周期 |
| OAuth 认证 | `ai/src/auth/` | 多提供商认证流程 |
| CBOR 协议 | `protocol/src/cbor/` | 二进制序列化协议 |
| 模型数据生成 | `ai/scripts/generate-models.ts` | 自动发现和更新模型元数据 |
| 项目信任机制 | `coding-agent/src/core/trust-manager.ts` | 安全模型 |

---

## 六、调试与测试

### 运行测试

```bash
# 从根目录运行所有测试
./test.sh

# 运行特定包的测试
cd packages/agent && node ../node_modules/.bin/vitest --run

# TUI 测试 (使用 node:test)
cd packages/tui && node --test test/specific.test.ts
```

### 本地运行

```bash
# 从源码运行
./pi-test.sh

# 构建后运行
npm run build && ./pi-test.sh
```

### 开发流程

```bash
npm run check    # 检查代码风格、类型等（提交前必须执行）
npm run build    # 构建所有包
```

---

## 七、推荐学习资源

1. **项目文档**: [pi.dev/docs](https://pi.dev/docs/latest)
2. **设计文档**: `tui-plan.md` — TUI 布局系统设计
3. **RFCs**: [rfc.earendil.com/keyword/pi/](https://rfc.earendil.com/keyword/pi/) — 长期规划和设计讨论
4. **核心代码注释**: 项目代码注释质量很高，特别是 `agent-loop.ts`, `agent.ts`, `types.ts`
5. **AGENTS.md**: 包含开发规则和代码质量要求，帮助理解代码风格

---

## 八、总结

### 核心学习路径

```
了解项目 → 理解 chord → 理解 AI 抽象层 → 理解 Agent 核心 → 理解 TUI → 理解主应用
```

### 关键收获

完成学习后，你将理解：

- 如何构建一个**多提供商 LLM 统一 API**
- 如何实现 **Agent Loop（智能体循环）**
- 如何设计一个**终端 UI 渲染引擎**
- 如何构建一个**可扩展的编码助手框架**
- 如何实现**远程会话协议**
- 如何设计**插件/扩展系统**

### 一句话总结

> Pi 是一个**模块化、可扩展的 AI 编码智能体框架**，核心是 Agent Loop 驱动 LLM + 工具调用，通过 `chord` 组合运行时、`ai` 统一 LLM 接口、`agent` 实现智能体逻辑、`tui` 提供终端渲染、`coding-agent` 整合为完整 CLI 应用。
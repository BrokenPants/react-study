---
name: React 进阶：跨越边界与编译革命
overview: 在掌握 React 核心调度与并发机制的基础上，进一步探索 React 的全栈架构（SSR/RSC）以及底层边缘系统（事件系统、编译器优化）。
todos:
  - id: ssr-hydration-deep-dive
    content: 阶段八：SSR 与 Hydration 深度原理 (注水与 mismatch 的根源)
    status: completed
  - id: rsc-exploration
    content: 阶段九：React Server Components (RSC) (Flight 协议与流式序列化)
    status: completed
  - id: synthetic-events-priority
    content: 阶段十：合成事件系统 (事件委派与优先级绑定)
    status: completed
  - id: react-compiler-analysis
    content: 阶段十一：React Compiler (Forget) (编译时依赖分析与自动 memo 化)
    status: completed
isProject: false
---

### 核心目标

深入理解 React 如何从一个客户端 UI 库扩展到服务器端，并研究其在编译时和运行时边缘系统的深度优化。

### 阶段规划

#### 阶段八：SSR 与 Hydration 的“身份对齐”

- **重点：** 研究 `react-dom/server` 如何将 Fiber 树转换为 HTML 字符串。
- **难点：** 理解 **Hydration（注水）** 过程。为什么 React 会报 “Hydration mismatch” 错误？它是如何在客户端重新构建 Fiber 树并与现有 DOM 绑定的？
- **源码入口：** `packages/react-dom-bindings/src/client/ReactFiberConfigDOM.js` 中的注水相关逻辑。

#### 阶段九：React Server Components (RSC) 与 Flight 协议

- **重点：** 拆解 **Flight 协议**。RSC 返回的不是 HTML，而是一种特殊的序列化流，它是如何描述组件树的？
- **难点：** 服务器组件与客户端组件的“交织”渲染。如何在 Client Fiber 树中挂载 Server Fiber 的占位符？
- **源码入口：** `packages/react-server/src/ReactFlightServer.js`。

#### 阶段十：合成事件系统 (Synthetic Events)

- **重点：** 为什么 React 17+ 将事件挂载在 `root` 而非 `document`？
- **难点：** 事件优先级与调度系统的集成。一个 `onClick` 事件是如何变成一个 `SyncLane` 任务的？
- **源码入口：** `packages/react-dom-bindings/src/events/DOMPluginEventSystem.js`。

#### 阶段十一：React Compiler (Forget)

- **重点：** 理解“编译时优化”的边界。
- **难点：** 编译器如何通过分析依赖图，自动注入 `useMemo` 和 `useCallback` 的逻辑，从而消除手动优化的负担。
- **源码入口：** `compiler/` 目录。

### 学习方法

- **对比实验：** 观察 SSR 场景下，React 18 的 `renderToPipeableStream` 如何实现渐进式注水。
- **协议抓包：** 拦截 RSC 的网络请求，手动解析其序列化格式。
- **编译预览：** 利用 React Compiler Playground 观察代码转换前后的差异。

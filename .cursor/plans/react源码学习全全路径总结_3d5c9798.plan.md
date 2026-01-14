---
name: React 源码学习-全阶段：从底层到全栈并发架构
overview: 完整拆解 React 18+ 及 19+ 核心架构，涵盖 Fiber 结构、并发特性、SSR/RSC、Actions 架构、use API 以及 Activity 模式。
todos:
  - id: fiber-definition
    content: 阶段一：Fiber 节点定义与双缓存机制 (虚拟调用栈、alternate 身份复用)
    status: completed
  - id: work-loop-exploration
    content: 阶段二：Render 阶段核心循环 (DFS 遍历、可中断 of workLoopConcurrent)
    status: completed
  - id: commit-phase-exploration
    content: 阶段三：Commit 阶段副作用提交与 DOM 交付 (Mutation 与 Layout 阶段区分)
    status: completed
  - id: hooks-deep-dive
    content: 阶段四：Hooks 链表存储与闭包状态隔离 (环形更新队列 queue.pending)
    status: completed
  - id: diff-algorithm-mastery
    content: 阶段五：Diff 算法详解 (lastPlacedIndex 移动量尺策略)
    status: completed
  - id: scheduler-lanes-exploration
    content: 阶段六：Lanes 优先级模型与 Scheduler 调度 (5ms 时间切片、x & -x 位运算)
    status: completed
  - id: suspense-concurrency
    content: 阶段七：并发特性 (从“能暂停”到“并发”的身份一致性保护)
    status: completed
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
  - id: react19-actions-deep-dive
    content: 阶段十二：Actions 架构与 useActionState 原理 (异步 Transition 串行队列)
    status: completed
  - id: react19-use-api-exploration
    content: 阶段十三：use API 的内部逻辑与 Promise 解包机制 (打破 Hook 规则的 Thenable 跟踪)
    status: completed
  - id: react19-optimistic-updates
    content: 阶段十四：useOptimistic 的平行宇宙管理逻辑 (基于 passthrough 的 Rebase 策略)
    status: completed
  - id: react19-metadata-resources
    content: 阶段十五：元数据提升与资源加载优化 (ReactDOMFloat 与 Suspense-on-Commit)
    status: completed
  - id: react19-activity-api-exploration
    content: 阶段十六：Activity (Offscreen) API 与组件“冬眠”机制 (低优先级 Lane 的后台静默更新)
    status: completed
isProject: false
---

# React 源码学习全全路径总结 (大师版)

恭喜！你已经完成了从 React 18 到 React 19 核心架构的巅峰探索。这是一份涵盖了现代 UI 框架所有底层黑科技的完整技术版图。

## 1. 核心架构层级

- **底层 (Infrastructure)：** Fiber 链表、双缓存、位掩码 Flags、DOM 节点“刺青” (internalInstanceKey)。
- **渲染流程 (Process)：** 递/归遍历、同步 Commit、客户端 Hydration (认领模式)。
- **并发与调度 (Concurrency)：** Lane 模型、5ms 时间切片、事件驱动的优先级提升 (Priority Boosting)。
- **全栈协议 (Protocol)：** Flight 协议流式传输、RSC 服务器序列化边界、Action 串行任务队列。
- **高级交互 (Interaction)：** 乐观更新 (Rebase 策略)、`use` API 的 Promise 解包与挂起重试。
- **资源与环境 (Environment)：** 自动元数据提升 (Hoisting)、样式挂起交付、Activity (Offscreen) 组件冬眠机制。

## 2. 关键知识复盘 (React 19 进阶)

- **Actions 的原子性：** `useActionState` 通过串行队列解决了异步竞态问题，让状态像接力棒一样在 `previousState` 中安全传递。
- **`use` 的规则破坏力：** 它通过抛出 Promise 并在完成时触发重绘，实现了在循环/条件语句中解包数据的能力，底层依赖 `ThenableState` 进行复用。
- **乐观更新的“滤镜”模型：** `useOptimistic` 始终以外部 `passthrough` 为真理基准，通过在基准上叠加临时“补丁”来实现瞬间回退，无需手动清理。
- **资源加载的原子化：** React 19 能够感知 CSS 加载状态，在 `Commit` 阶段如果关键样式未就绪会暂停交付，彻底消灭了样式闪烁 (FOUC)。
- **Activity 的后台运行：** 通过 `OffscreenLane` (极低优先级)，React 实现了隐藏组件的“静默更新”，在不占用主线程资源的前提下保证了切换时的“秒回现场”。

## 3. 架构师视角的终极思考

React 的终极形态是一个**“跨时空的资源调度器”**：

1.  **管理时间**：通过 Concurrent Mode 将 CPU 计算切片。
2.  **管理空间**：通过 RSC 将代码执行在最合适的物理位置（服务器/浏览器）。
3.  **管理网络**：通过 Flight 和 Streaming 将数据流与渲染流合二为一。
4.  **管理交互**：通过 Actions 和 Optimistic UI 让用户感知不到网络的延迟。

## 4. 登峰造极：你的专家身份

你现在不仅理解代码如何运行，更理解了 React 是如何通过 **“挂起、中断、恢复、重放、提升”** 这一系列精密操作，在极度动态的 Web 环境中维持确定性的。

---
**React 18 & 19 源码全路径学习圆满结束。代码的灵魂，已随你而动。**

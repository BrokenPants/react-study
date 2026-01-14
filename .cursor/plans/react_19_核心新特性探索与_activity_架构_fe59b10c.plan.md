---
name: React 19 核心新特性探索与 Activity 架构
overview: 在 React 19 核心特性的基础上，增加对实验性 Activity (Offscreen) API 的深度探索，理解 React 如何处理“隐藏但存活”的组件树。
todos:
  - id: react19-actions-deep-dive
    content: 阶段十二：Actions 架构与 useActionState 原理
    status: completed
  - id: react19-use-api-exploration
    content: 阶段十三：use API 的内部逻辑与 Promise 解解机制
    status: completed
  - id: react19-optimistic-updates
    content: 阶段十四：useOptimistic 的平行宇宙管理逻辑
    status: completed
  - id: react19-metadata-resources
    content: 阶段十五：元数据提升与资源加载优化 (ReactDOMFloat)
    status: completed
  - id: react19-activity-api-exploration
    content: 阶段十六：Activity (Offscreen) API 与组件“冻结”机制
    status: completed
isProject: false
---

### 核心目标

全面拆解 React 19 的新特性，并深入研究 Activity (Offscreen) 这一影响未来 React 导航与性能优化的底层 API。

### 阶段规划

#### 阶段十二：Actions 架构与 useActionState 原理

- **重点：** 研究 `useActionState` 和 `useFormStatus`。
- **难点：** 理解异步 Action 如何与 TransitionLane 协同，实现自动的 Pending 状态管理。
- **源码入口：** `packages/react-reconciler/src/ReactFiberHooks.js` 中关于 Action 的新逻辑。

#### 阶段十三：`use` API 的“解构”

- **重点：** `use(Promise)` 和 `use(Context)` 的不同处理。
- **难点：** 研究 React 如何在组件内部暂停并重新触发渲染而不破坏 Hook 链表。
- **源码入口：** `packages/react-reconciler/src/ReactFiberThenable.js`。

#### 阶段十四：`useOptimistic` 与状态回滚机制

- **重点：** 乐观更新的内存模型。
- **难点：** 在并发模式下，React 如何保证乐观状态与真实服务器返回状态的最终一致性。
- **源码入口：** `packages/react-reconciler/src/ReactFiberHooks.js`。

#### 阶段十五：React 19 资源加载与元数据

- **重点：** 自动提升（Hoisting）与预加载。
- **难点：** 研究渲染器如何识别元数据标签并在 Commit 阶段将其调度到 `<head>`。
- **源码入口：** `packages/react-dom-bindings/src/shared/ReactDOMFloat.js`。

#### 阶段十六：Activity (Offscreen) API 与隐藏组件状态

- **重点：** 理解 `Activity` 组件如何“冻结”子树。
- **难点：** 研究在 `mode="hidden"` 时，Fiber 树如何保留 State 但跳过 DOM 更新与副作用执行。
- **源码入口：** `packages/react-reconciler/src/ReactFiberActivityComponent.js` (或搜索 `OffscreenComponent` 标签)。

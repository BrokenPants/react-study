# React 19 Action 架构与并发调度深度解析

## 1. useActionState 核心流程
`useActionState` 是 React 19 处理 Action 状态的核心 Hook，它通过内部队列和 Transition 机制，实现了异步状态的自动追踪与顺序执行。

### 核心生命周期
```mermaid
flowchart TD
    Start([dispatch 触发]) --> CreateNode[1. 创建 ActionNode]
    subgraph TransitionPhase [Transition 阶段]
        CreateNode --> IsTransition{处于 Transition?}
        IsTransition -->|Yes| SetPending[2. 乐观更新 isPending=true]
        IsTransition -->|No| UpdateState[2. 正常状态更新]
    end
    UpdateState --> Queue[3. 进入串行队列 actionQueue]
    Queue --> Exec[4. 执行 Action 函数]
    Exec --> Await[5. 等待 Promise / 获取结果]
    Await --> Finalize[6. 更新内部状态 & 触发重渲染]
    Finalize --> ReRender[7. 组件渲染 & 自动解包 Thenable]
    ReRender --> End([返回 [state, dispatch, isPending]])
```

## 2. startTransition 原理与“纠缠”机制
`startTransition` 的本质是通过全局标记降级更新优先级。React 19 通过“纠缠”解决了异步丢失上下文的问题。

### 关键源码片段
- **上下文切换**：通过全局变量 `ReactSharedInternals.T` 标记范围。
```javascript
// packages/react/src/ReactStartTransition.js
export function startTransition(scope) {
  const prevTransition = ReactSharedInternals.T;
  ReactSharedInternals.T = currentTransition; // 开启保护伞
  try {
    const returnValue = scope();
    // ...
  } finally {
    ReactSharedInternals.T = prevTransition; // 恢复
  }
}
```

- **异步纠缠 (Entanglement)**：当 Action 返回 Promise 时，锁定全局 Lane。
```javascript
// packages/react-reconciler/src/ReactFiberAsyncAction.js
export function entangleAsyncAction(transition, thenable) {
  if (currentEntangledListeners === null) {
    currentEntangledLane = requestTransitionLane(transition); // 锁定车道
  }
  currentEntangledPendingCount++;
  thenable.then(pingEngtangledActionScope, pingEngtangledActionScope);
}
```

## 3. React 优先级系统 (Lanes)
React 使用 31 位二进制位管理优先级，确保高优先级任务（救护车）能中断低优先级任务（保护伞）。

### 优先级群组
| 优先级 | 车道 (Lane) | 场景 | 特性 |
| :--- | :--- | :--- | :--- |
| **同步** | `SyncLane` | 用户点击、输入 | 阻塞式，不可中断 |
| **连续** | `InputContinuousLane` | 滚动、拖拽 | 快速响应，可中断 |
| **默认** | `DefaultLane` | API 请求返回, Timer | 正常调度 |
| **过渡** | `TransitionLanes` (1-14) | 切换页面、搜索过滤 | **并发、可中断、多车道隔离** |
| **空闲** | `IdleLane` | 埋点、预渲染 | 最低优先级 |

### 为什么有 14 条 TransitionLane？
1. **独立并发**：不同区域的 Transition（如左侧树和底部日志）互不干扰，挂起一个不影响另一个。
2. **防竞态 (Race Condition)**：通过轮询车道，React 能够识别出哪个搜索结果是“最新”的，从而丢弃过时渲染。

## 4. 解决的核心问题
- **异步上下文丢失**：解决了 `await` 之后 `setState` 变成同步更新导致的掉帧。
- **Action 顺序性**：确保并发触发的多个 Action 按顺序执行，且 `prevState` 始终正确。
- **并发细粒度控制**：通过多车道模型实现了不同 UI 任务的并行与隔离。

## 5. 待办事项
- [ ] 深入研究 `useOptimistic` 如何与 `useActionState` 的纠缠机制配合。
- [ ] 探索 Lane 模型的“过期机制”：当低优先级任务被插队太久，如何强制转为同步。
- [ ] 实验多个 `useActionState` 同时运行时，底层 `actionQueue` 的合并策略。

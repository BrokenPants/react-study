# React 19 `useOptimistic` 源码架构深度解析

## 1. 核心技术点总结

本次对话深入剖析了 React 19 新特性 `useOptimistic` 的底层实现机制，重点探讨了它如何与 React 的 **Lane 模型** 和 **Action 架构** 协同工作，实现“无感知”的状态回滚。

### 关键发现
- **`hook.baseState` 的特殊性**：与 `useState` 不同，`useOptimistic` 在每次更新时都会将 `baseState` 强制重置为外部传入的 `passthrough` (props)。它不维护持久的“真理”，而是以父组件的 props 为基准。
- **信号回滚 (Revert Lane)**：乐观更新被赋予了一个特殊的 `revertLane`（通常与当前的 Transition 绑定）。
- **位运算触发跳过**：在 `updateReducerImpl` 中，通过 `isSubsetOfLanes(renderLanes, revertLane)` 判断当前渲染是否为 Transition 的“收尾渲染”。如果是，则直接跳过该更新，实现自动回滚。

---

## 2. 核心流程图 (Transition Cleanup)

```mermaid
flowchart TD
    A[用户触发 Action] --> B[startTransition 开启]
    B --> C[addOptimisticMessage 派发补丁]
    C --> D{updateReducerImpl 计算}
    D -- "revertLane ∉ renderLanes" --> E[应用补丁: 看到 '发送中...']
    E --> F[异步 Action 执行中]
    F --> G[Action 结束 (成功或失败)]
    G --> H[React 发起收尾渲染]
    H --> I{isSubsetOfLanes 检查}
    I -- "revertLane ⊆ renderLanes" --> J[跳过补丁: continue]
    J --> K[渲染最新 Props 数据]
```

---

## 3. 关键源码片段分析

### A. 派发函数的绑定 (ReactFiberHooks.js)
`dispatch` 并不是每次渲染新生成的，而是在 `mount` 阶段通过 `bind` 牢牢锁定了它所属的 `Fiber` 和 `UpdateQueue`。

```javascript
// packages/react-reconciler/src/ReactFiberHooks.js
const dispatch: Dispatch<A> = (queue.dispatch = (dispatchReducerAction.bind(
  null,
  currentlyRenderingFiber,
  queue,
): any));
```

### B. 自动回滚逻辑 (updateReducerImpl)
这是 `useOptimistic` 魔法发生的地点。

```javascript
// packages/react-reconciler/src/ReactFiberHooks.js:1469
if (isSubsetOfLanes(renderLanes, revertLane)) {
  // 信号命中：Transition 已结束，跳过这个乐观更新
  update = update.next;
  continue; 
} else {
  // 信号未命中：继续保留补丁，并进行 Rebase (留级) 处理
  const clone = { /* ... */ };
  // ... 将更新加入 baseQueue ...
}
```

---

## 4. 推荐实践模式：Action + 乐观更新

为了在 Action 失败时依然能优雅地处理 UI，建议结合 `useActionState` 使用：

```javascript
function Thread({ initialMessages }) {
  // 1. 管理正式状态与错误结果
  const [messages, formAction] = useActionState(async (prev, formData) => {
    try {
      const res = await deliverMessage(formData.get("message"));
      return [res, ...prev]; // 成功：更新正式状态
    } catch (e) {
      return [{ text: "失败", error: true }, ...prev]; // 失败：返回带错误的正式状态
    }
  }, initialMessages);

  // 2. 贴上乐观补丁
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages, 
    (state, msg) => [{ text: msg, sending: true }, ...state]
  );

  return (
    <form action={(fd) => { addOptimistic(fd.get("message")); formAction(fd); }}>
      {/* 渲染 optimisticMessages */}
    </form>
  );
}
```

---

## 5. 待办事项 (Next Steps)
- [ ] **深入探索 `useActionState` 的排队机制**：研究多个并发 Action 是如何在 Fiber 级别实现串行执行的。
- [ ] **验证 `Entanglement` (纠缠) 细节**：在 `updateReducerImpl` 中，`didReadFromEntangledAsyncAction` 是如何精确控制渲染挂起的。
- [ ] **性能分析**：大规模并发乐观更新下，`updateReducerImpl` 的循环遍历对长列表性能的影响。

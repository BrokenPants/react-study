# React 调度与任务过期机制总结

本次对话深入探讨了 React Scheduler 如何处理低优先级任务过期的问题，以及这一机制对性能和编码实践的影响。

## 1. 核心技术点

### 1.1 任务饥饿与过期时间 (expirationTime)
为了防止低优先级任务被高优先级任务无限期“插队”（饥饿现象），Scheduler 为每个任务计算一个绝对的过期时间：
- \`expirationTime = startTime + timeout\`
- 不同优先级的 \`timeout\` 不同（如 Normal 为 5000ms，Immediate 为 -1ms）。

### 1.2 强制同步执行机制
当一个任务在 \`taskQueue\` 中等待时间超过了其 \`expirationTime\`，它将失去“可中断性”：
- **正常任务**：在每执行一个 Fiber 后会检查 \`shouldYield()\`，若时间片用完则让出主线程。
- **过期任务**：\`currentTask.expirationTime <= currentTime\` 为真，Scheduler 会跳过时间片检查，强制同步执行该任务直到完成。

### 1.3 Scheduler 任务队列结构
Scheduler 内部维护两个最小堆（Min-Heap）：
- **taskQueue**: 存放已就绪的任务，按 \`expirationTime\` 排序。
- **timerQueue**: 存放延迟任务，按 \`startTime\` 排序。

## 2. 关键代码片段

### Scheduler 核心工作循环 (伪代码)
\`\`\`javascript
// packages/scheduler/src/forks/Scheduler.js
function workLoop(initialTime) {
  let currentTime = initialTime;
  currentTask = peek(taskQueue); 

  while (currentTask !== null) {
    // 如果任务未过期且时间片用完，则中断
    if (currentTask.expirationTime > currentTime && shouldYieldToHost()) {
      break;
    }

    const callback = currentTask.callback;
    const didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
    
    // 执行任务，didUserCallbackTimeout 告知任务是否已过期
    const continuationCallback = callback(didUserCallbackTimeout);
    
    currentTime = getCurrentTime();
    // ...处理后续逻辑
  }
}
\`\`\`

### React 与 Scheduler 的衔接
\`\`\`javascript
// packages/react-reconciler/src/ReactFiberWorkLoop.js
function workLoopConcurrent() {
  // 可中断模式
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress); 
  }
}

function workLoopSync() {
  // 同步模式（用于过期任务或离散事件）
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
\`\`\`

## 3. 流程图说明

\`\`\`mermaid
flowchart TD
    start[Scheduler 开始执行任务] --> checkExpired{任务是否过期?}
    checkExpired -- "是 (expirationTime <= now)" --> syncExec[同步执行: 不允许中断/不检查时间片]
    checkExpired -- "否" --> checkTime{当前帧是否有剩余时间?}
    checkTime -- "有" --> execUnit[执行工作单元]
    checkTime -- "无" --> yieldHost[让出主线程: 等待下一帧]
    syncExec --> finish[任务完成]
    execUnit --> finish
    finish --> next[检查下一个任务]
    next --> start
\`\`\`

## 4. 解决的问题与启示

- **解决的问题**：通过 \`expirationTime\` 保证了低优先级任务最终一定会被执行，避免了 UI 更新的无限期停滞。
- **启示 1**：并发模式并非性能银弹。若单个组件计算逻辑过重，React 无法在组件内部中断，依然会导致卡顿。
- **启示 2**：组件拆分的颗粒度决定了响应度。Fiber 架构的中断发生在节点之间，更细的拆分有助于 React 更灵活地让出主线程。

## 5. 待办事项
- [in_progress] 深入理解 Lanes (赛道) 机制如何与 Scheduler Task 映射。
- [pending] 探索 \`ensureRootIsScheduled\` 的具体实现。
- [pending] 验证多版本 \`setState\` 合并时的优先级抢占逻辑。

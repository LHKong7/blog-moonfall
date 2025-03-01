# React中的常量


### Scheduler中

### React 上下文：

#### 执行上下文 Context

- executionContext： 
  - 表示React当前所处的**执行上下文**
  - 作用：
    - 管理React内部的状态，确保不同阶段的操作（渲染、提交）能够正确处理流程
    - 区分执行阶段：不同阶段可能需要不同的逻辑和优先级
    - 避免不必要的重复工作：根据上下文标志，可以判断某些工作是否已经完成或可以跳过。
  - 常见的上下文：
    - NoContext
    - RenderContext
    - CommitContext
    - BatchContext
    - LegacyUnbatchedContext
  - 相关操作：
    - 进入某个上下文: `executionContext |= RenderContext;`
    - 退出某个上下文: `executionContext &= ~RenderContext;`
    - 检查当前上下文: `executionContext & RenderContext`
  - 实战代码：
  ```javascript
    if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
        // React 内部运行，可以直接调用某些方法或逻辑
        return now();
    } else {
        // React 外部运行，可能在浏览器事件中，需要额外处理
        return cachedTime;
    }

    if (executionContext & RenderContext) {
        console.log('Currently rendering components');
    } else if (executionContext & CommitContext) {
        console.log('Applying updates to the DOM');
    }
  ```

类型包括：
```javascript
var NoContext = 0;
var BatchedContext = 1;
var RenderContext = 2;
var CommitContext = 4;
```

#### Lane 车道模型
Lane类型的为二进制变量通过位掩码的特性在频繁运算时占用内存小，计算速度快。

Lane是对于expirationTime的重构, 以前使用expirationTime表示的字段, 都改为了lane：
```
renderExpirationtime -> renderLanes
update.expirationTime -> update.lane
fiber.expirationTime -> fiber.lanes
fiber.childExpirationTime -> fiber.childLanes
root.firstPendingTime and root.lastPendingTime -> fiber.pendingLanes
```

占有低位比特位的Lane变量对应的优先级越高:
- 最高优先级为SyncLanePriority对应的车道为SyncLane = 0b0000000000000000000000000000001.
- 最低优先级为OffscreenLanePriority对应的车道为OffscreenLane = 0b1000000000000000000000000000000.


车道模型：
- SyncLane: 同步更新，最高优先级
- TransitionLane： 过渡任务
- NoLane：没有分配优先级


#### Dispatcher
在 React 源码中 `react-reconciler/src/ReactFiberHooks.js`， Dispatcher（调度器）是一个核心机制，用于管理 Hooks 的实现和分发。它的核心作用是在 React 的不同阶段（如渲染、更新、并发模式）动态切换 Hooks 的具体实现，从而保证 Hooks 的正确性和性能优化。

Dispatcher 是 React Hooks 系统的底层调度器，主要负责以下任务：
1. 动态绑定 Hooks 的实现 在 React 的渲染流程中，不同的阶段（如挂载、更新、服务端渲染）需要使用不同的 Hooks 实现。Dispatcher 通过动态切换 currentDispatcher 来确保正确的 Hooks 函数被调用。
2. 处理并发模式下的 Hooks 行为：在并发模式（Concurrent Mode）下，Hooks 的行为需要适应可中断的渲染任务。Dispatcher 负责协调并发模式下 Hooks 的状态管理。
3. 隔离环境差异： 在客户端渲染、服务端渲染（SSR）、测试环境等不同场景中，Hooks 的实现可能不同。Dispatcher 通过环境隔离确保 Hooks 的一致性。

```javascript
// ReactFiberHooks.js
let currentDispatcher = null;

// 获取当前 Dispatcher
export function resolveDispatcher() {
  if (currentDispatcher === null) {
    throw new Error('Hooks can only be called inside the body of a function component.');
  }
  return currentDispatcher;
}
```
在 React 的渲染流程中，通过 renderWithHooks 函数动态设置 currentDispatcher：

```javascript
// ReactFiberHooks.js
export function renderWithHooks(current, workInProgress, Component, props) {
  // 根据当前渲染阶段选择不同的 Dispatcher
  if (current !== null && current.memoizedState !== null) {
    // 更新阶段：使用更新阶段的 Hooks 实现
    ReactCurrentDispatcher.current = HooksDispatcherOnUpdate;
  } else {
    // 挂载阶段：使用挂载阶段的 Hooks 实现
    ReactCurrentDispatcher.current = HooksDispatcherOnMount;
  }

  // 执行函数组件，此时组件内调用的 Hooks 会从 currentDispatcher 中获取
  const children = Component(props);
  return children;
}
```

不同阶段的Hooks实现：
- 挂载阶段（Mount）：Hooks 需要初始化状态（如 useState 的初始值）并创建副作用（如 useEffect）。
- 更新阶段（Update）：Hooks 需要复用之前的状态，并根据依赖变化触发更新。
```javascript
// ReactFiberHooks.js
const HooksDispatcherOnMount = {
  useState: mountState,
  useEffect: mountEffect,
  // 其他 Hooks...
};

const HooksDispatcherOnUpdate = {
  useState: updateState,
  useEffect: updateEffect,
  // 其他 Hooks...
};
```
Dispatcher 的工作原理
当你在函数组件中调用 useState、useEffect 等 Hook 时，实际上是通过 currentDispatcher 调用当前阶段的实现：
```javascript
// 例如 useState 的实现
function useState(initialState) {
  const dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
```

具体流程示例：
1. 初次渲染（Mount）：
   - renderWithHooks 设置 currentDispatcher = HooksDispatcherOnMount。
   - 组件内调用 useState 实际执行 mountState，初始化状态。
2. 更新渲染（Update）：
   - renderWithHooks 设置 currentDispatcher = HooksDispatcherOnUpdate。
   - 组件内调用 useState 实际执行 updateState，复用已有状态。

Dispatcher 的设计意义
1. 性能优化：挂载和更新阶段的行为不同，分开实现可以避免冗余判断（如 if (isMount)。
2. 逻辑解耦：将 Hooks 的实现细节封装在 Dispatcher 中，对外提供统一的 API。
3. 环境适配：在服务端渲染（SSR）或测试环境中，可以注入自定义的 Dispatcher：

场景	Dispatcher 的作用
挂载阶段	初始化 Hooks 状态和副作用。
更新阶段	复用已有状态，处理更新逻辑。
并发模式	协调可中断渲染中的 Hooks 行为。
服务端渲染	提供无副作用的 Hooks 实现。


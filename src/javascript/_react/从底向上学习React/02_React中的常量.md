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



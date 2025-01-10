# React 初始化过程总览



```

import React, { useEffect } from "react";
import { createRoot } from 'react-dom/client';

const App = () => {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  useEffect(() => {
    setCount(10);
  }, []);

  return (
    <div>
      <h1>Counter: {count}</h1>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
};

const reactRoot = document.querySelector('#app');

const root = createRoot(reactRoot);
root.render(<App />);

```



### createRoot 初始化ReactDOM





### Render 全流程

1. 通过 `ReactDOM.prototype.render()` 来触发第一次应用级别更新：

   内部调用 `updateContainer(children, root, null, null);` :

   - children: App 组件
   - root: FiberRootNode

2. 调用 `updateContainer` 

   - 入参分别为：
     - `element` : 
       - App component
     - `container` : 
       - FiberRootNode 
     - `parentComponent` : null
     - `callback` :  null
   - 流程：
     1. 拿到 `FiberRootNode` 的 `current`  -> FiberNode (FiberRootNode的current) Root of a host tree
     1. `getContextForSubtree` 
     1. `createUpdate(eventTime, lane);` 创建更新对象
     
        - Update:
          - eventTime:
          - Payload: 对应着的是 App 组件
     1. `enqueueUpdate` ： 将更新推出待更新队列中
     
        - 函数定义： `function enqueueUpdate(fiber, update, lane)` 
     
        - 简单来说： 该函数会 帮助参数中的 Fiber 预定state 或 props 更新
     
        - 流程：
     
          1. 从入参 fiber 中拿到更新队列：
     
             ```
             var updateQueue = fiber.updateQueue;
             
             if (updateQueue === null) {
               // If the fiber has been unmounted, the update is ignored.
               return null;
             }
             ```
     
             
     
          2. 拿到共享队列（share queue）SharedQueue 是存储该组件的挂起更新的位置。这允许更新在渲染周期中持续存在。
     
          3. 区分 **unsafe render phase updates** (which need immediate attention) 和 **safe updates** (which can be batched for later processing).
     1. `scheduleUpdateOnFiber` ： 负责在特定 Fiber 节点上调度工作（更新），确保在 React 的并发渲染模型中对更新进行优先级排序并正确处理。该函数协调更新过程，处理正常更新、渲染阶段更新等。
     
        - markRootUpdated(root, lane, eventTime); This marks the `root` Fiber node as having a pending update in the given `lane` (priority bucket). This is essential for React to know that the tree needs to be processed.
     
        - 更新调度：
     
          ```
          if (root === workInProgressRoot) {
            if ((executionContext & RenderContext) === NoContext) {
              workInProgressRootInterleavedUpdatedLanes = mergeLanes(workInProgressRootInterleavedUpdatedLanes, lane);
            }
          
            if (workInProgressRootExitStatus === RootSuspendedWithDelay) {
              markRootSuspended$1(root, workInProgressRootRenderLanes);
            }
          }
          ```
     
          - If the root Fiber is already being worked on (`workInProgressRoot`), React marks the update as interleaved with the current rendering process.
     
          - If the current rendering is already suspended (e.g., due to a `Suspense` boundary), React marks it as suspended and prioritizes processing the new update.
     
        - ensureRootIsScheduled(root, eventTime); 这是每个并发任务的入口点, 即任何通过调度程序。
     1. `entangleTransitions`  : 负责管理“转换通道”并确保与转换相关的更新相互纠缠在一起。这有助于 React 正确处理具有复杂依赖关系的更新，并确保转换行为可预测，即使在并发模式下也是如此。
     
        - 做了什么：
     
          1. Adding the new lane to the `sharedQueue` of the Fiber.
     
          2. Merging it with other pending lanes related to transitions.
     
          3. Marking the root Fiber with the updated set of entangled lanes.



### 初次渲染的流程

通过 `ReactDOM.prototype.render()` 来触发第一次应用级别更新：



内部调用 `updateContainer(children, root, null, null);` :

- 负责什么：

- 函数解释：

  - 入参

    - children
    - root

  - 函数流程解析：

    ```javascript
    function updateContainer(element, container, parentComponent, callback) {
      console.log("updateContainer called")
      {
        onScheduleRoot(container, element);
      }
    
      var current$1 = container.current;
      var eventTime = requestEventTime();
      var lane = requestUpdateLane(current$1);
    
      {
        markRenderScheduled(lane);
      }
    
      var context = getContextForSubtree(parentComponent);
    
      if (container.context === null) {
        container.context = context;
      } else {
        container.pendingContext = context;
      }
    
      {
        if (isRendering && current !== null && !didWarnAboutNestedUpdates) {
          didWarnAboutNestedUpdates = true;
    
          error('Render methods should be a pure function of props and state; ' + 'triggering nested component updates from render is not allowed. ' + 'If necessary, trigger nested updates in componentDidUpdate.\n\n' + 'Check the render method of %s.', getComponentNameFromFiber(current) || 'Unknown');
        }
      }
    
      var update = createUpdate(eventTime, lane); // Caution: React DevTools currently depends on this property
      // being called "element".
    
      update.payload = {
        element: element
      };
      callback = callback === undefined ? null : callback;
    
      if (callback !== null) {
        {
          if (typeof callback !== 'function') {
            error('render(...): Expected the last optional `callback` argument to be a ' + 'function. Instead received: %s.', callback);
          }
        }
    
        update.callback = callback;
      }
    
      var root = enqueueUpdate(current$1, update, lane);
    
      if (root !== null) {
        scheduleUpdateOnFiber(root, current$1, lane, eventTime);
        entangleTransitions(root, current$1, lane);
      }
    
      return lane;
    }
    ```





### scheduleUpdateOnFiber

- 负责什么： 是其更新调度系统的核心部分（schedule）。负责在特点Fiber节点上调度更新（update对象）。并且确保在React的并发渲染模型中对更新进行优先级排序及正确处理。
- 函数执行流程

```javascript
function scheduleUpdateOnFiber(root, fiber, lane, eventTime) {
  checkForNestedUpdates(); // 1. Check for Nested Updates:
  console.log("root: ", root, "fiber: ", fiber, " in scheduleUpdateOnFiber")

  
  /// 2. Log and Debugging Warnings (Development Mode):
  {
    if (isRunningInsertionEffect) {
      error('useInsertionEffect must not schedule updates.');
    }
  }

  {
    if (isFlushingPassiveEffects) {
      didScheduleUpdateDuringPassiveEffects = true;
    }
  } // Mark that the root has a pending update.
	// END OF Log and Debugging Warnings (Development Mode):

  // 3. Mark the Root as Updated:
  markRootUpdated(root, lane, eventTime);

  // 4. Handle Render-Phase Updates:
  if ((executionContext & RenderContext) !== NoLanes && root === workInProgressRoot) {
    // This update was dispatched during the render phase. This is a mistake
    // if the update originates from user space (with the exception of local
    // hook updates, which are handled differently and don't reach this
    // function), but there are some internal React features that use this as
    // an implementation detail, like selective hydration.
    warnAboutRenderPhaseUpdatesInDEV(fiber); // Track lanes that were updated during the render phase
  } else {
    // This is a normal update, scheduled from outside the render phase. For
    // example, during an input event.
    {
      if (isDevToolsPresent) {
        addFiberToLanesMap(root, fiber, lane);
      }
    }

    warnIfUpdatesNotWrappedWithActDEV(fiber);

    // 5. Normal Update Scheduling:
    if (root === workInProgressRoot) {
      // Received an update to a tree that's in the middle of rendering. Mark
      // that there was an interleaved update work on this root. Unless the
      // `deferRenderPhaseUpdateToNextBatch` flag is off and this is a render
      // phase update. In that case, we don't treat render phase updates as if
      // they were interleaved, for backwards compat reasons.
      if ( (executionContext & RenderContext) === NoContext) {
        workInProgressRootInterleavedUpdatedLanes = mergeLanes(workInProgressRootInterleavedUpdatedLanes, lane);
      }

      if (workInProgressRootExitStatus === RootSuspendedWithDelay) {
        // The root already suspended with a delay, which means this render
        // definitely won't finish. Since we have a new update, let's mark it as
        // suspended now, right before marking the incoming update. This has the
        // effect of interrupting the current render and switching to the update.
        // TODO: Make sure this doesn't override pings that happen while we've
        // already started rendering.
        markRootSuspended$1(root, workInProgressRootRenderLanes);
      }
    }

    // 6. Ensure the Root is Scheduled:
    ensureRootIsScheduled(root, eventTime);

    // 7. Legacy Synchronous Updates:
    if (lane === SyncLane && executionContext === NoContext && (fiber.mode & ConcurrentMode) === NoMode && // Treat `act` as if it's inside `batchedUpdates`, even in legacy mode.
    !( ReactCurrentActQueue$1.isBatchingLegacy)) {
      // Flush the synchronous work now, unless we're already working or inside
      // a batch. This is intentionally inside scheduleUpdateOnFiber instead of
      // scheduleCallbackForFiber to preserve the ability to schedule a callback
      // without immediately flushing it. We only do this for user-initiated
      // updates, to preserve historical behavior of legacy mode.
      resetRenderTimer();
      flushSyncCallbacksOnlyInLegacyMode();
    }
  }
}
```



- executionContext： 追踪当前React在哪个阶段：
  - **RenderContext:** React is currently rendering the tree.
  - **NoContext:** React is idle, and updates are being scheduled normally.
- 函数都做了什么：
  1. **Schedules an Update:** It ensures that the `fiber` and its `root` are marked for reconciliation and processing in React's rendering pipeline.
  2. **Handles Priority:** It assigns a priority (`lane`) to the update and integrates it into React's scheduling system.
  3. **Handles Special Cases:** It warns about improper updates (e.g., render-phase updates) and handles edge cases like updates to a suspended root or updates during passive effects.
  4. **Triggers Rendering:** If necessary, it ensures the root Fiber is scheduled for rendering.





### ensureRootIsScheduled
- 作用/定义：这个函数处理参数 root 上的调度工作，管理回调的优先级确保React的render不会因为高优先级任务造成饥饿现象。
- 流程：

```javascript

```

























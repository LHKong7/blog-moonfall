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
初始化客户端的React DOM对象。

`createRoot(container, options?) ` 一共接受两个参数：
- container：真实DOM元素
- options：
  - onCaughtError: 当 React 在错误边界中捕获错误时调用回调。使用错误边界捕获的错误以及包含 componentStack 的 errorInfo 对象进行调用。
  - onUncaughtError: 当抛出错误且未被错误边界捕获时调用的回调。使用引发的错误和包含 componentStack 的 errorInfo 对象进行调用。
  - onRecoverableError: React 自动从错误中恢复时调用的回调。调用时会抛出一个 React 错误，以及一个包含 componentStack 的 errorInfo 对象。某些可恢复的错误可能包含原始错误原因作为 error.cause。
  - identifierPrefix: React 使用 useId 生成的 ID 的字符串前缀。有助于避免在同一页面上使用多个根时发生冲突。

`createRoot` 作为初始函数，用作初始化React Fiber根节点和合成事件

```javascript
// 
var root = createContainer(container, ConcurrentRoot, null, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError);
markContainerAsRoot(root.current, container);
var rootContainerElement = container.nodeType === COMMENT_NODE ? container.parentNode : container;
listenToAllSupportedEvents(rootContainerElement);
return new ReactDOMRoot(root);
```

`createContainer`: 
- 调用 `createFiberRoot` 创建
  - `createFiberRoot` ：
    - 创建 `new FiberRootNode()`
    - 调用 `createHostRootFiber` ： 创建mode为3的Fiber节点
    - 修改指向：
    ```javascript
    var uninitializedFiber = createHostRootFiber(tag, isStrictMode);
    root.current = uninitializedFiber;
    uninitializedFiber.stateNode = root;
    ```
    - 初始化更新队列 `initializeUpdateQueue(uninitializedFiber);`
    - 返回 `FiberRootNode` 节点


`markContainerAsRoot`:

```javascript
function markContainerAsRoot(hostRoot, node) {
  node[internalContainerInstanceKey] = hostRoot;
}
```

`listenToAllSupportedEvents`:
所有原生DOM事件用来监听在根节点容器上。这个函数做了一件事：通过事件代理的方式，在 `root` DOM元素上注册大部分原生事件。换言之，所有React写法的事件都监听在 `root` 元素上。
通过这种方式, 所有通过React注册的事件都由一个事件中心处理器来处理，每个React事件（例如 `onClick`）作为独立个体保存在Fiber数据结构内。通过这种方式做到了合并更新（batch updates），减少了内存使用，在不同浏览器中做兼容。

Q:
- React如何触发和生成合成事件合成事件？
- 整体流程是什么样的
- 在组件中，原生事件和合成事件一起使用会发生什么？


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
     2. `getContextForSubtree` 
     3. `createUpdate(eventTime, lane);` 创建更新对象
     
        - Update:
          - eventTime:
          - Payload: 对应着的是 App 组件
     4. `enqueueUpdate` ： 将更新放入`container.current`待更新队列(shared)中
     
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
     5. `scheduleUpdateOnFiber` ： 负责在特定 Fiber 节点上调度工作（更新），确保在 React 的并发渲染模型中对更新进行优先级排序并正确处理。该函数协调更新过程，处理正常更新、渲染阶段更新等。
     
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
  - 将带更新的内容放到根节点的更新队列中

- 函数解释：
  - 这个函数主要做了4件事：
    1. 初始化工作：
        - 拿到 `FiberRootNode` 的 `current`  -> FiberNode (FiberRootNode的current) Root of a host tree
        - `createUpdate(eventTime, lane);` 创建更新对象
          - Update:
             - eventTime:
             - Payload: 在初次渲染的时候 对应着的是 App 组件
    2. 调用 `enqueueUpdate` 将更新放入`container.current`待更新队列(shared)中
    3. 调用 `scheduleUpdateOnFiber` ： 负责在特定 Fiber 节点上调度工作（更新），确保在 React 的并发渲染模型中对更新进行优先级排序并正确处理。该函数协调更新过程，处理正常更新、渲染阶段更新等。
    4. `entangleTransitions`  : 负责管理“转换通道”并确保与转换相关的更新相互纠缠在一起。这有助于 React 正确处理具有复杂依赖关系的更新，并确保转换行为可预测，即使在并发模式下也是如此。
  - 入参
    - children
    - root

  - 函数流程解析：

    ```javascript
    function updateContainer(element, container, parentComponent, callback) {
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
- 作用/定义：这个函数处理参数 root 上的调度工作，管理回调的优先级确保React的render不会因为高优先级任务造成饥饿现象。所有的调度核心任务也会由这个函数触发。当没有更新任务需要进行时，这个函数会直接返回。主要作用有：
  - 查是否有任务需要调度：如果没有任务，直接返回。如果有任务，根据任务的优先级选择合适的调度方式（同步、并发或批量更新）。
  - 调度任务：如果是同步任务，直接调用 performSyncWorkOnRoot。如果是并发任务，调用 scheduleCallbackForRoot 将任务加入调度队列。
  - 处理任务优先级：如果当前任务被更高优先级的任务打断，取消当前任务的调度，并重新调度更高优先级的任务。
- 流程：

```javascript
function ensureRootIsScheduled(root, currentTime) {
  console.log("==ensureRootIsScheduled==")
  var existingCallbackNode = root.callbackNode; // Check if any lanes are being starved by other work. If so, mark them as
  // expired so we know to work on those next.

  markStarvedLanesAsExpired(root, currentTime); // Determine the next lanes to work on, and their priority.

  var nextLanes = getNextLanes(root, root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes);

  // 当没有更新任务需要进行时，这个函数会直接返回
  if (nextLanes === NoLanes) {
    // Special case: There's nothing to work on.
    if (existingCallbackNode !== null) {
      cancelCallback$1(existingCallbackNode);
    }

    root.callbackNode = null;
    root.callbackPriority = NoLane;
    return;
  } // We use the highest priority lane to represent the priority of the callback.


  var newCallbackPriority = getHighestPriorityLane(nextLanes); // Check if there's an existing task. We may be able to reuse it.

  var existingCallbackPriority = root.callbackPriority;

  if (existingCallbackPriority === newCallbackPriority && // Special case related to `act`. If the currently scheduled task is a
  // Scheduler task, rather than an `act` task, cancel it and re-scheduled
  // on the `act` queue.
  !( ReactCurrentActQueue$1.current !== null && existingCallbackNode !== fakeActCallbackNode)) {
    {
      // If we're going to re-use an existing task, it needs to exist.
      // Assume that discrete update microtasks are non-cancellable and null.
      // TODO: Temporary until we confirm this warning is not fired.
      if (existingCallbackNode == null && existingCallbackPriority !== SyncLane) {
        error('Expected scheduled callback to exist. This error is likely caused by a bug in React. Please file an issue.');
      }
    } // The priority hasn't changed. We can reuse the existing task. Exit.


    return;
  }

  if (existingCallbackNode != null) {
    // Cancel the existing callback. We'll schedule a new one below.
    cancelCallback$1(existingCallbackNode);
  } // Schedule a new callback.


  var newCallbackNode;

  if (newCallbackPriority === SyncLane) {
    // Special case: Sync React callbacks are scheduled on a special
    // internal queue
    if (root.tag === LegacyRoot) {
      if ( ReactCurrentActQueue$1.isBatchingLegacy !== null) {
        ReactCurrentActQueue$1.didScheduleLegacyUpdate = true;
      }

      scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
    } else {
      scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
    }

    {
      // Flush the queue in a microtask.
      if ( ReactCurrentActQueue$1.current !== null) {
        // Inside `act`, use our internal `act` queue so that these get flushed
        // at the end of the current scope even when using the sync version
        // of `act`.
        ReactCurrentActQueue$1.current.push(flushSyncCallbacks);
      } else {
        scheduleMicrotask(function () {
          // In Safari, appending an iframe forces microtasks to run.
          // https://github.com/facebook/react/issues/22459
          // We don't support running callbacks in the middle of render
          // or commit so we need to check against that.
          if ((executionContext & (RenderContext | CommitContext)) === NoContext) {
            // Note that this would still prematurely flush the callbacks
            // if this happens outside render or commit phase (e.g. in an event).
            flushSyncCallbacks();
          }
        });
      }
    }

    newCallbackNode = null;
  } else {
    var schedulerPriorityLevel;

    switch (lanesToEventPriority(nextLanes)) {
      case DiscreteEventPriority:
        schedulerPriorityLevel = ImmediatePriority;
        break;

      case ContinuousEventPriority:
        schedulerPriorityLevel = UserBlockingPriority;
        break;

      case DefaultEventPriority:
        schedulerPriorityLevel = NormalPriority;
        break;

      case IdleEventPriority:
        schedulerPriorityLevel = IdlePriority;
        break;

      default:
        schedulerPriorityLevel = NormalPriority;
        break;
    }

    newCallbackNode = scheduleCallback$1(schedulerPriorityLevel, performConcurrentWorkOnRoot.bind(null, root));
  }

  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
} // This is the entry point for every concurrent task, i.e. anything that
// goes through Scheduler.
```


### performConcurrentWorkOnRoot
- 负责什么:
- 现有的问题和解释：
  - 为什么在最后还需要调用一次 `ensureRootIsScheduled(root, now());`:
    - 确保所有任务都被正确处理，具体原因如下：
      1. 处理未完成的任务：在并发模式下，任务可能被中断（如时间切片用完或更高优先级任务插入），此时 Fiber 树的渲染可能尚未完成。
      2. 处理新任务: 在渲染过程中，可能会触发新的更新（如状态更新或副作用回调），这些更新需要被调度。ensureRootIsScheduled 会检查是否有新的任务需要处理，并将其加入调度队列。
      3. 确保调度的一致性: React 的调度机制是动态的，任务优先级可能会变化（如用户交互触发了更高优先级的更新）。调用 ensureRootIsScheduled 可以确保调度器始终以最新的优先级处理任务。
- 函数解释：
  - 流程: 
    1. 如果有被动副作用，先清除副作用
    2. 根据不同的状态，调用 `renderRootConcurrent(root, lanes)` 或 `renderRootSync(root, lanes);`
    3. 最后调用检查函数 `ensureRootIsScheduled(root, now());` 
  - 入参： `(root, didTimeout)`

```javascript
function performConcurrentWorkOnRoot(root, didTimeout) {
  console.log("=performConcurrentWorkOnRoot==")
  {
    resetNestedUpdateFlag();
  } // Since we know we're in a React event, we can clear the current
  // event time. The next update will compute a new event time.


  currentEventTime = NoTimestamp;
  currentEventTransitionLane = NoLanes;

  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    throw new Error('Should not already be working.');
  } // Flush any pending passive effects before deciding which lanes to work on,
  // in case they schedule additional work.


  var originalCallbackNode = root.callbackNode;
  var didFlushPassiveEffects = flushPassiveEffects();

  if (didFlushPassiveEffects) {
    // Something in the passive effect phase may have canceled the current task.
    // Check if the task node for this root was changed.
    if (root.callbackNode !== originalCallbackNode) {
      // The current task was canceled. Exit. We don't need to call
      // `ensureRootIsScheduled` because the check above implies either that
      // there's a new task, or that there's no remaining work on this root.
      return null;
    }
  } // Determine the next lanes to work on, using the fields stored
  // on the root.


  var lanes = getNextLanes(root, root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes);

  if (lanes === NoLanes) {
    // Defensive coding. This is never expected to happen.
    return null;
  } // We disable time-slicing in some cases: if the work has been CPU-bound
  // for too long ("expired" work, to prevent starvation), or we're in
  // sync-updates-by-default mode.
  // TODO: We only check `didTimeout` defensively, to account for a Scheduler
  // bug we're still investigating. Once the bug in Scheduler is fixed,
  // we can remove this, since we track expiration ourselves.


  var shouldTimeSlice = !includesBlockingLane(root, lanes) && !includesExpiredLane(root, lanes) && ( !didTimeout);
  var exitStatus = shouldTimeSlice ? renderRootConcurrent(root, lanes) : renderRootSync(root, lanes);

  console.log("end of render RootSync")

  if (exitStatus !== RootInProgress) {
    if (exitStatus === RootErrored) {
      // If something threw an error, try rendering one more time. We'll
      // render synchronously to block concurrent data mutations, and we'll
      // includes all pending updates are included. If it still fails after
      // the second attempt, we'll give up and commit the resulting tree.
      var errorRetryLanes = getLanesToRetrySynchronouslyOnError(root);

      if (errorRetryLanes !== NoLanes) {
        lanes = errorRetryLanes;
        exitStatus = recoverFromConcurrentError(root, errorRetryLanes);
      }
    }

    if (exitStatus === RootFatalErrored) {
      var fatalError = workInProgressRootFatalError;
      prepareFreshStack(root, NoLanes);
      markRootSuspended$1(root, lanes);
      ensureRootIsScheduled(root, now());
      throw fatalError;
    }

    if (exitStatus === RootDidNotComplete) {
      // The render unwound without completing the tree. This happens in special
      // cases where need to exit the current render without producing a
      // consistent tree or committing.
      //
      // This should only happen during a concurrent render, not a discrete or
      // synchronous update. We should have already checked for this when we
      // unwound the stack.
      markRootSuspended$1(root, lanes);
    } else {
      // The render completed.
      // Check if this render may have yielded to a concurrent event, and if so,
      // confirm that any newly rendered stores are consistent.
      // TODO: It's possible that even a concurrent render may never have yielded
      // to the main thread, if it was fast enough, or if it expired. We could
      // skip the consistency check in that case, too.
      var renderWasConcurrent = !includesBlockingLane(root, lanes);
      var finishedWork = root.current.alternate;

      if (renderWasConcurrent && !isRenderConsistentWithExternalStores(finishedWork)) {
        // A store was mutated in an interleaved event. Render again,
        // synchronously, to block further mutations.
        exitStatus = renderRootSync(root, lanes); // We need to check again if something threw

        if (exitStatus === RootErrored) {
          var _errorRetryLanes = getLanesToRetrySynchronouslyOnError(root);

          if (_errorRetryLanes !== NoLanes) {
            lanes = _errorRetryLanes;
            exitStatus = recoverFromConcurrentError(root, _errorRetryLanes); // We assume the tree is now consistent because we didn't yield to any
            // concurrent events.
          }
        }

        if (exitStatus === RootFatalErrored) {
          var _fatalError = workInProgressRootFatalError;
          prepareFreshStack(root, NoLanes);
          markRootSuspended$1(root, lanes);
          ensureRootIsScheduled(root, now());
          throw _fatalError;
        }
      } // We now have a consistent tree. The next step is either to commit it,
      // or, if something suspended, wait to commit it after a timeout.


      root.finishedWork = finishedWork;
      root.finishedLanes = lanes;
      finishConcurrentRender(root, exitStatus, lanes);
    }
  }

  ensureRootIsScheduled(root, now());

  if (root.callbackNode === originalCallbackNode) {
    // The task node scheduled for this root is the same one that's
    // currently executed. Need to return a continuation.
    return performConcurrentWorkOnRoot.bind(null, root);
  }

  return null;
}
```

### renderRootConcurrent

### renderRootSync
- 函数职责
- 函数流程:
  - 标志着进入到 `renderContext`: `executionContext |= RenderContext;` 
  - 准备新的渲染环境 `prepareFreshStack(root, lanes);`
  - 调用 `workLoopSync();` 
- 函数入参
- 其他细节:
  - 是否涉及设计模式
  - 调用堆栈
- 已知问题：

```javascript
function renderRootSync(root, lanes) {
  console.log("==renderRootSync==")
  var prevExecutionContext = executionContext;
  executionContext |= RenderContext;
  var prevDispatcher = pushDispatcher(); // If the root or lanes have changed, throw out the existing stack
  // and prepare a fresh one. Otherwise we'll continue where we left off.
  console.log("first render workInProgressRoot: ", workInProgressRoot)

  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    {
      if (isDevToolsPresent) {
        var memoizedUpdaters = root.memoizedUpdaters;

        if (memoizedUpdaters.size > 0) {
          restorePendingUpdaters(root, workInProgressRootRenderLanes);
          memoizedUpdaters.clear();
        } // At this point, move Fibers that scheduled the upcoming work from the Map to the Set.
        // If we bailout on this work, we'll move them back (like above).
        // It's important to move them now in case the work spawns more work at the same priority with different updaters.
        // That way we can keep the current update and future updates separate.


        movePendingFibersToMemoized(root, lanes);
      }
    }

    workInProgressTransitions = getTransitionsForLanes();
    prepareFreshStack(root, lanes);
  }

  {
    markRenderStarted(lanes);
  }

  do {
    try {
      workLoopSync();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);

  resetContextDependencies();
  executionContext = prevExecutionContext;
  popDispatcher(prevDispatcher);

  if (workInProgress !== null) {
    // This is a sync render, so we should have finished the whole tree.
    throw new Error('Cannot commit an incomplete root. This error is likely caused by a ' + 'bug in React. Please file an issue.');
  }

  {
    markRenderStopped();
  } // Set this to null to indicate there's no in-progress render.


  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;
  return workInProgressRootExitStatus;
} // The work loop is an extremely hot path. Tell Closure not to inline it.

```


### prepareFreshStack
- 函数职责： 为新的渲染任务准备一个干净的 Fiber 树和任务栈。它的核心作用是确保每次渲染任务都在一个独立的环境中执行，避免任务之间的状态污染。
  1. 重置 Fiber 树的上下文：为新的渲染任务初始化 WorkInProgress 树。
  2. 清理任务栈：确保任务栈中没有残留的状态或副作用。
  3. 设置任务优先级：根据当前任务的优先级，初始化调度相关的状态。
- 函数流程
- 函数入参
- 其他细节:
  - 是否涉及设计模式
  - 调用堆栈
- 已知问题：

```javascript
function prepareFreshStack(root, lanes) {
  root.finishedWork = null;
  root.finishedLanes = NoLanes;
  var timeoutHandle = root.timeoutHandle;

  if (timeoutHandle !== noTimeout) {
    // The root previous suspended and scheduled a timeout to commit a fallback
    // state. Now that we have additional work, cancel the timeout.
    root.timeoutHandle = noTimeout; // $FlowFixMe Complains noTimeout is not a TimeoutID, despite the check above

    cancelTimeout(timeoutHandle);
  }

  if (workInProgress !== null) {
    var interruptedWork = workInProgress.return;

    while (interruptedWork !== null) {
      var current = interruptedWork.alternate;
      unwindInterruptedWork(current, interruptedWork);
      interruptedWork = interruptedWork.return;
    }
  }

  workInProgressRoot = root;
  var rootWorkInProgress = createWorkInProgress(root.current, null); // 创建WorkInProgress树
  workInProgress = rootWorkInProgress;
  workInProgressRootRenderLanes = subtreeRenderLanes = workInProgressRootIncludedLanes = lanes;
  workInProgressRootExitStatus = RootInProgress;
  workInProgressRootFatalError = null;
  workInProgressRootSkippedLanes = NoLanes;
  workInProgressRootInterleavedUpdatedLanes = NoLanes;
  workInProgressRootPingedLanes = NoLanes;
  workInProgressRootConcurrentErrors = null;
  workInProgressRootRecoverableErrors = null;
  finishQueueingConcurrentUpdates();

  {
    ReactStrictModeWarnings.discardPendingWarnings();
  }

  return rootWorkInProgress;
}
```

### workLoopSync()

```javascript
function workLoopSync() {
  console.log("workLoopSync==")
  // Already timed out, so perform work without checking if we need to yield.
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```

### performUnitOfWork(unitOfWork)
- 函数职责
- 函数流程:
  - 标志着进入到 `renderContext`: `executionContext |= RenderContext;` 
  - 准备新的渲染环境 `prepareFreshStack(root, lanes);`
  - 调用 `workLoopSync();` 
- 函数入参
- 其他细节:
  - 是否涉及设计模式
  - 调用堆栈
- 已知问题：

```javascript
function performUnitOfWork(unitOfWork) {
  // The current, flushed, state of this fiber is the alternate. Ideally
  // nothing should rely on this, but relying on it here means that we don't
  // need an additional field on the work in progress.
  var current = unitOfWork.alternate;
  setCurrentFiber(unitOfWork);
  var next;

  if ( (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
  }

  resetCurrentFiber();
  unitOfWork.memoizedProps = unitOfWork.pendingProps;

  if (next === null) {
    // If this doesn't spawn new work, complete the current work.
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner$2.current = null;
}
```

### beginwork
- 函数职责
- 函数流程:
  - 标志着进入到 `renderContext`: `executionContext |= RenderContext;` 
  - 准备新的渲染环境 `prepareFreshStack(root, lanes);`
  - 调用 `workLoopSync();` 
- 函数入参
- 其他细节:
  - 是否涉及设计模式
  - 调用堆栈
- 已知问题：

```javascript
function beginWork(current, workInProgress, renderLanes) {
  {
    if (workInProgress._debugNeedsRemount && current !== null) {
      // This will restart the begin phase with a new fiber.
      return remountFiber(current, workInProgress, createFiberFromTypeAndProps(workInProgress.type, workInProgress.key, workInProgress.pendingProps, workInProgress._debugOwner || null, workInProgress.mode, workInProgress.lanes));
    }
  }

  if (current !== null) {
    var oldProps = current.memoizedProps;
    var newProps = workInProgress.pendingProps;

    if (oldProps !== newProps || hasContextChanged() || ( // Force a re-render if the implementation changed due to hot reload:
     workInProgress.type !== current.type )) {
      // If props or context changed, mark the fiber as having performed work.
      // This may be unset if the props are determined to be equal later (memo).
      didReceiveUpdate = true;
    } else {
      // Neither props nor legacy context changes. Check if there's a pending
      // update or context change.
      var hasScheduledUpdateOrContext = checkScheduledUpdateOrContext(current, renderLanes);

      if (!hasScheduledUpdateOrContext && // If this is the second pass of an error or suspense boundary, there
      // may not be work scheduled on `current`, so we check for this flag.
      (workInProgress.flags & DidCapture) === NoFlags) {
        // No pending updates or context. Bail out now.
        didReceiveUpdate = false;
        return attemptEarlyBailoutIfNoScheduledUpdate(current, workInProgress, renderLanes);
      }

      if ((current.flags & ForceUpdateForLegacySuspense) !== NoFlags) {
        // This is a special case that only exists for legacy mode.
        // See https://github.com/facebook/react/pull/19216.
        didReceiveUpdate = true;
      } else {
        // An update was scheduled on this fiber, but there are no new props
        // nor legacy context. Set this to false. If an update queue or context
        // consumer produces a changed value, it will set this to true. Otherwise,
        // the component will assume the children have not changed and bail out.
        didReceiveUpdate = false;
      }
    }
  } else {
    console.log("current is null")
    didReceiveUpdate = false;

    if (getIsHydrating() && isForkedChild(workInProgress)) {
      // Check if this child belongs to a list of muliple children in
      // its parent.
      //
      // In a true multi-threaded implementation, we would render children on
      // parallel threads. This would represent the beginning of a new render
      // thread for this subtree.
      //
      // We only use this for id generation during hydration, which is why the
      // logic is located in this special branch.
      var slotIndex = workInProgress.index;
      var numberOfForks = getForksAtLevel();
      pushTreeId(workInProgress, numberOfForks, slotIndex);
    }
  } // Before entering the begin phase, clear pending update priority.
  // TODO: This assumes that we're about to evaluate the component and process
  // the update queue. However, there's an exception: SimpleMemoComponent
  // sometimes bails out later in the begin phase. This indicates that we should
  // move this assignment out of the common path and into each branch.


  workInProgress.lanes = NoLanes;
  console.log("workInProgress: ", workInProgress)
  switch (workInProgress.tag) {
    case IndeterminateComponent:
      {
        return mountIndeterminateComponent(current, workInProgress, workInProgress.type, renderLanes);
      }

    case LazyComponent:
      {
        var elementType = workInProgress.elementType;
        return mountLazyComponent(current, workInProgress, elementType, renderLanes);
      }

    case FunctionComponent:
      {
        var Component = workInProgress.type;
        var unresolvedProps = workInProgress.pendingProps;
        var resolvedProps = workInProgress.elementType === Component ? unresolvedProps : resolveDefaultProps(Component, unresolvedProps);
        return updateFunctionComponent(current, workInProgress, Component, resolvedProps, renderLanes);
      }

    case ClassComponent:
      {
        var _Component = workInProgress.type;
        var _unresolvedProps = workInProgress.pendingProps;

        var _resolvedProps = workInProgress.elementType === _Component ? _unresolvedProps : resolveDefaultProps(_Component, _unresolvedProps);

        return updateClassComponent(current, workInProgress, _Component, _resolvedProps, renderLanes);
      }

    case HostRoot:
      return updateHostRoot(current, workInProgress, renderLanes);

    case HostComponent:
      return updateHostComponent(current, workInProgress, renderLanes);

    case HostText:
      return updateHostText(current, workInProgress);

    case SuspenseComponent:
      return updateSuspenseComponent(current, workInProgress, renderLanes);

    case HostPortal:
      return updatePortalComponent(current, workInProgress, renderLanes);

    case ForwardRef:
      {
        var type = workInProgress.type;
        var _unresolvedProps2 = workInProgress.pendingProps;

        var _resolvedProps2 = workInProgress.elementType === type ? _unresolvedProps2 : resolveDefaultProps(type, _unresolvedProps2);

        return updateForwardRef(current, workInProgress, type, _resolvedProps2, renderLanes);
      }

    case Fragment:
      return updateFragment(current, workInProgress, renderLanes);

    case Mode:
      return updateMode(current, workInProgress, renderLanes);

    case Profiler:
      return updateProfiler(current, workInProgress, renderLanes);

    case ContextProvider:
      return updateContextProvider(current, workInProgress, renderLanes);

    case ContextConsumer:
      return updateContextConsumer(current, workInProgress, renderLanes);

    case MemoComponent:
      {
        var _type2 = workInProgress.type;
        var _unresolvedProps3 = workInProgress.pendingProps; // Resolve outer props first, then resolve inner props.

        var _resolvedProps3 = resolveDefaultProps(_type2, _unresolvedProps3);

        {
          if (workInProgress.type !== workInProgress.elementType) {
            var outerPropTypes = _type2.propTypes;

            if (outerPropTypes) {
              checkPropTypes(outerPropTypes, _resolvedProps3, // Resolved for outer only
              'prop', getComponentNameFromType(_type2));
            }
          }
        }

        _resolvedProps3 = resolveDefaultProps(_type2.type, _resolvedProps3);
        return updateMemoComponent(current, workInProgress, _type2, _resolvedProps3, renderLanes);
      }

    case SimpleMemoComponent:
      {
        return updateSimpleMemoComponent(current, workInProgress, workInProgress.type, workInProgress.pendingProps, renderLanes);
      }

    case IncompleteClassComponent:
      {
        var _Component2 = workInProgress.type;
        var _unresolvedProps4 = workInProgress.pendingProps;

        var _resolvedProps4 = workInProgress.elementType === _Component2 ? _unresolvedProps4 : resolveDefaultProps(_Component2, _unresolvedProps4);

        return mountIncompleteClassComponent(current, workInProgress, _Component2, _resolvedProps4, renderLanes);
      }

    case SuspenseListComponent:
      {
        return updateSuspenseListComponent(current, workInProgress, renderLanes);
      }

    case ScopeComponent:
      {

        break;
      }

    case OffscreenComponent:
      {
        return updateOffscreenComponent(current, workInProgress, renderLanes);
      }
  }

  throw new Error("Unknown unit of work tag (" + workInProgress.tag + "). This error is likely caused by a bug in " + 'React. Please file an issue.');
}
```



# React 中的工具函数:

### requestEventTime:
- 判断React当前所处上下文，并根据上下文返回时间
- 

```javascript
function requestEventTime() {
  // 如果当前处于 React 的渲染或提交阶段， 表示React控制了执行流，直接调用 `now()` 获取当前时间
  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    return now();
  }

  // 如果已经有一个缓存的事件时间，则直接返回该时间，而不调用 now() 重新获取。 （React 在浏览器事件处理过程中可能会多次触发更新，重复调用 now() 会导致不必要的性能开销。）
  if (currentEventTime !== NoTimestamp) {
    // Use the same start time for all updates until we enter React again.
    return currentEventTime;
  } // This is the first update since React yielded. Compute a new start time.


  // 如果 currentEventTime 没有被设置（即 NoTimestamp），说明这是自 React 上次让出控制权以来的第一个更新。
  currentEventTime = now();
  return currentEventTime;
}
```


### requestUpdateLane
- 确定更新任务优先级的方法，根据当前任务的上下文，运行模式，任务的来源（比如事件或过渡）等条件 分配适当的更新优先级；确保任务能够按顺序执行

- 应用场景：
  1.	同步任务：如直接调用 setState，在同步模式下立即更新。
	2.	过渡任务：在 startTransition 中执行的任务，为低优先级。
	3.	事件处理：根据事件类型（如点击、输入）动态分配优先级。
	4.	渲染阶段更新：警告开发者避免在渲染阶段触发状态更新。

```javascript
function requestUpdateLane(fiber) {
  // Special cases
  var mode = fiber.mode;

  if ((mode & ConcurrentMode) === NoMode) {
    return SyncLane;
  } else if ( (executionContext & RenderContext) !== NoContext && workInProgressRootRenderLanes !== NoLanes) {
    // This is a render phase update. These are not officially supported. The
    // old behavior is to give this the same "thread" (lanes) as
    // whatever is currently rendering. So if you call `setState` on a component
    // that happens later in the same render, it will flush. Ideally, we want to
    // remove the special case and treat them as if they came from an
    // interleaved event. Regardless, this pattern is not officially supported.
    // This behavior is only a fallback. The flag only exists until we can roll
    // out the setState warning, since existing code might accidentally rely on
    // the current behavior.
    return pickArbitraryLane(workInProgressRootRenderLanes);
  }

  var isTransition = requestCurrentTransition() !== NoTransition;

  if (isTransition) {
    if ( ReactCurrentBatchConfig$3.transition !== null) {
      var transition = ReactCurrentBatchConfig$3.transition;

      if (!transition._updatedFibers) {
        transition._updatedFibers = new Set();
      }

      transition._updatedFibers.add(fiber);
    } // The algorithm for assigning an update to a lane should be stable for all
    // updates at the same priority within the same event. To do this, the
    // inputs to the algorithm must be the same.
    //
    // The trick we use is to cache the first of each of these inputs within an
    // event. Then reset the cached values once we can be sure the event is
    // over. Our heuristic for that is whenever we enter a concurrent work loop.


    if (currentEventTransitionLane === NoLane) {
      // All transitions within the same event are assigned the same lane.
      currentEventTransitionLane = claimNextTransitionLane();
    }

    return currentEventTransitionLane;
  } // Updates originating inside certain React methods, like flushSync, have
  // their priority set by tracking it with a context variable.
  //
  // The opaque type returned by the host config is internally a lane, so we can
  // use that directly.
  // TODO: Move this type conversion to the event priority module.


  var updateLane = getCurrentUpdatePriority();

  if (updateLane !== NoLane) {
    return updateLane;
  } // This update originated outside React. Ask the host environment for an
  // appropriate priority, based on the type of event.
  //
  // The opaque type returned by the host config is internally a lane, so we can
  // use that directly.
  // TODO: Move this type conversion to the event priority module.


  var eventLane = getCurrentEventPriority();
  return eventLane;
}
```


### createUpdate:
- 创建一个更新对象

```javascript
function createUpdate(eventTime, lane) {
  var update = {
    eventTime: eventTime,
    lane: lane,
    tag: UpdateState,
    payload: null,
    callback: null,
    next: null
  };
  return update;
}
```



### getContextForSubtree






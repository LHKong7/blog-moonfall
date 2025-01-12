# React 中的工具函数:

### requestEventTime

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



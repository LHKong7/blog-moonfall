# React 中的对象

### ReactElement:

#### 定义：

```typescript
export type ReactElement = {
    $$typeof: any, // 辨别ReactElement对象
    type: any, // ReactElmenet种类
    key: any, // 默认值为null
    ref: any,
    props: any,
    // ReactFiber 对应的Fiber节点
    _owner: any,

    // __DEV__
    _store: {validated: boolean, ...},
    _self: React$Element<any>,
    _shadowChildren: any,
    _source: Source,
};
```

#### 一句话要点： 在jsx中写的函数式组件，会在编译阶段阶段转变成ReactElement，并在React内部流程中使用。

#### 转变过程：

` jsx` -> `@babel/preset-react`  -> `React.createElement`  -> `React.Element` 

TODO:  补全细节

TODO：[React.createElement 与 JSX runtime](https://github.com/reactjs/rfcs/blob/createlement-rfc/text/0000-create-element-changes.md#motivation)

#### 核心要点：

- type决定了节点的种类，
  - type的值可以是DOM原声节点，也可能是React内部的节点类型 （我们可以在 `packages/shared/ReactSymbols.js` 中找到类型的定义。

- `props.children` : 该属性表示嵌套在React元素的开始和结束标记之间的内容。

关于函数式组件的ReactElement：

- 如果在函数中使用了 Hooks：则会处理 Hooks的逻辑例如，创建
- 如果没有使用：在协调的过程中会直接直接跳过hooks的处理，直接拿到子节点的 `ReactElement` 

在协调过程中， 会通过 `ReactElemnt` 的类型生成对应的 `fiber` 节点。（并不是一一对应的关系）

### Fiber 对象

#### 定义：

```typescript
// 一个Fiber对象代表一个即将渲染或者已经渲染的组件(ReactElement), 一个组件可能对应两个fiber(current和WorkInProgress)
// 单个属性的解释在后文(在注释中无法添加超链接)
export type Fiber = {

  tag: WorkTag,
  key: null | string,
  elementType: any,
  type: any,
  stateNode: any,
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,
  index: number,
  ref:
    | null
    | (((handle: mixed) => void) & { _stringRef: ?string, ... })
    | RefObject,
  pendingProps: any,
  memoizedProps: any,
  updateQueue: mixed,
  memoizedState: any,
  dependencies: Dependencies | null,
  mode: TypeOfMode,
  flags: Flags, 
  subtreeFlags: Flags,
  deletions: Array<Fiber> | null, 

  nextEffect: Fiber | null, 
  firstEffect: Fiber | null, 
  lastEffect: Fiber | null,

  lanes: Lanes,
  childLanes: Lanes,
  alternate: Fiber | null,

  // 性能统计相关(开启enableProfilerTimer后才会统计)
  // react-dev-tool会根据这些时间统计来评估性能
  actualDuration?: number, // 本次更新过程, 本节点以及子树所消耗的总时间
  actualStartTime?: number, // 标记本fiber节点开始构建的时间
  selfBaseDuration?: number, // 用于最近一次生成本fiber节点所消耗的时间
  treeBaseDuration?: number, // 生成子树所消耗的时间的总和
};
```



##### 一句话要点：Fiber树的基础组成部分， 它是将渲染工作分割成小的增量单元。使更新可中断并提高响应能力。



#### 核心细节：

- 关键属性介绍：我们可以将React Fiber的定义分为八种类型：
  - 结构类型：定义了fiber树的结构
    - `retrun` : 指向树中的父Fiber，形成链上的父子连接。
    - `child` :  指向该 Fiber 的第一个子节点。如果有多个孩子，它们通过兄弟指针链接。
    - `sibling` : 指向下一个同级Fiber。它与 child 一起形成兄弟姐妹的链表。
    - `index` : 指示该 Fiber 在其兄弟中的位置。在协调过程中很有用，可以匹配新老孩子
  - 身份类型：
    - `tag` : 用来表示当前Fiber节点的类型，比如： host component (div), function component 等
    - `key` : 用来标识元素在兄弟节点中的的唯一性，会来决定节点的操作为： 插入、删除或者移动
    - `elementType` : JSX 中元素的原始类型（例如，函数或类组件，或者像“div”这样的字符串）。
    - `type` : 经过各种转换（例如高阶组件、延迟加载、记忆化）后的解析类型。一般来讲和`fiber.elementType`一致
    - `stateNode` : 指向宿主环境的实例或组件实例：
      - 类组件， 类的实例
      - host component： 真实DOM节点
      - 函数式组件/Fragments ： 通常为空
  - props，state和数据依赖相关属性：这些字段存储组件的数据并帮助确定需要重新渲染的内容。
    - `pendingProps`: 用于即将进行的渲染的props。 从最新的React Element 中提取并与 memoizedProps 进行比较以检测更改。
    - `memoizedProps`: 上次完成的渲染期间使用的props。React 使用它来查看 props 是否已更改，从而影响是否需要更新。
    - `updateQueue`: 已为此组件安排的更新队列（例如，setState 调用）。对于类组件，状态更新在处理之前会累积在这里。
    - `memoizedState`: 最后渲染的state。协调和渲染后，它保存当前的“已提交”状态，该状态应与屏幕上显示的内容一致。
    - `dependencies`:该 Fiber 具有的任何外部数据依赖性，例如订阅的上下文。如果它们发生变化，Fiber可能需要重新渲染
  - ref管理: ref允许直接访问渲染的元素或组件实例，绕过正常的数据流。
    - `ref`: 如果ReactElement附加了一个ref， React 将在组件安装或更新后更新此引用，以便开发人员可以直接访问 DOM 节点或组件实例。
  - 模式和配置： 定义该 Fiber 及其子树的运行模式。
    - `mode`: 描述组件运行模式的位字段（例如 ConcurrentMode、StrictMode 或 NoMode）。此模式影响 React 计划和处理更新的方式。
  - 副作用管理： 在协调期间，React 会跟踪哪些节点需要在提交阶段执行插入、更新、删除或回调调用等操作。
    - `flags`: 一个描述需要应用哪些副作用的位字段（例如，放置、更新、删除）。
    - `subtreeFlags`: 该 Fiber 子树中的所有副作用，使 React 能够快速确定是否有任何后代需要处理。
    - `deletions`: 当一个子节点被移除时，代表该子节点的 Fiber 会存储在这里。在提交阶段，React 将完成删除
    - `nextEffect`: 链表形态的副作用，在新版本的react中， 这个属性可能过时了。
  - 调度与优先级： React 使用基于优先级的模型（通道）来安排更新，以允许并发功能和响应式 UI。
    - `lanes`: 代表该 Fiber 中待处理工作的优先级。Lanes就像优先级的“桶”，让 React 管理首先处理哪些更新。
    - `childLanes`:  其子级的优先级。如果子级有待处理的更新，父级知道它不能跳过它们，并且必须继续沿树向下工作。
    - `alternate`: 连接current与workInProgress的属性。
      - 每个展示的元素有两个与之相关的Fiber：
        - current：目前展示的 Fiber
        - workInProgress：为下一次渲染准备的Fiber
  - 表现分析属性：以下属性为开发阶段启用 `enableProfilerTimer` 为true时才会使用到的属性。
    - `actualDuration`: 在当前更新中渲染此 Fiber 及其子树所花费的总时间。
    - `actualStartTime`: Fiber 在当前周期开始渲染时的时间戳。
    - `actualStartTime`: 在上一次提交中仅渲染此 Fiber（不包括其子项）所花费的估计时间。
    - `treeBaseDuration`:  在上一次提交中渲染此 Fiber 的子树所花费的估计时间。

#### 不同类型的 FiberNode

##### FiberRootNode

- 定义：包含了APP所需要的全部meta信息。它的 `current` 指向 真实Fiber树，每次一个新树的创建，它的`current` 都会指向新的 `HostRoot` 
- 功能：
  - 指向当前真实Fiber树， 每次React流程结束后，会生成一个 `WIP Fiber` 树， 而这时，`FiberRootNode` 的 `current` 会指向这个 `WIP Fiber` 树
  -  存储了调度信息，包含哪些通道有待更新。
  - 可以跟踪影响整个应用程序的待处理更新。它可以存储事件时间（触发更新时）、挂起的转换或调度程序用来决定首先处理哪些更新的信息。

```typescript
function FiberRootNode(containerInfo, tag, hydrate, identifierPrefix, onRecoverableError) {
  this.tag = tag;
  this.containerInfo = containerInfo;
  this.pendingChildren = null;
  this.current = null;
  this.pingCache = null;
  this.finishedWork = null;
  this.timeoutHandle = noTimeout;
  this.context = null;
  this.pendingContext = null;
  this.callbackNode = null;
  this.callbackPriority = NoLane;
  this.eventTimes = createLaneMap(NoLanes);
  this.expirationTimes = createLaneMap(NoTimestamp);
  this.pendingLanes = NoLanes;
  this.suspendedLanes = NoLanes;
  this.pingedLanes = NoLanes;
  this.expiredLanes = NoLanes;
  this.mutableReadLanes = NoLanes;
  this.finishedLanes = NoLanes;
  this.entangledLanes = NoLanes;
  this.entanglements = createLaneMap(NoLanes);
  this.identifierPrefix = identifierPrefix;
  this.onRecoverableError = onRecoverableError;

  {
    this.pooledCache = null;
    this.pooledCacheLanes = NoLanes;
  }

  {
    this.mutableSourceEagerHydrationData = null;
  }

  {
    this.effectDuration = 0;
    this.passiveEffectDuration = 0;
  }

  {
    this.memoizedUpdaters = new Set();
    var pendingUpdatersLaneMap = this.pendingUpdatersLaneMap = [];

    for (var _i = 0; _i < TotalLanes; _i++) {
      pendingUpdatersLaneMap.push(new Set());
    }
  }

  {
    switch (tag) {
      case ConcurrentRoot:
        this._debugRootType = hydrate ? 'hydrateRoot()' : 'createRoot()';
        break;

      case LegacyRoot:
        this._debugRootType = hydrate ? 'hydrate()' : 'render()';
        break;
    }
  }
}
```







TODO: 是否需要画一个图

### Update & UpdateQueue 对象

UpdateQueue 是优先更新的链表。和Fiber一样，更新队列有两个： 

- 当前队列：当前屏幕展示的队列
- workInProrgess队列： 正在更改且支持异步更新的队列

Update队列的本质就是利用带有双指针的持久化的单链表来确保状态更新不会丢失，并支持可中断和恢复的状态切换。

- 持久化数据结构：这里的“持久”意味着一旦创建了一个节点，它在内存中仍然可用，并且列表的不同状态或版本可以引用它而不会丢失旧部分。这种方法可以避免在开始新的正在进行的渲染时“丢弃”过去的更新。

- 双指针（current 与 workInProgress）：系统维护两个引用相同底层更新链表的逻辑队列：

  - Current Queue：表示组件树最后提交到屏幕时的状态。
  - Work-in-Progress Queue：表示组件树在协调和准备下一次提交的过程中的状态。

  两者都指向相同的单链接更新列表，但可能位于不同的位置。 “当前”指针滞后，显示哪些更新已被完全处理和提交。 “正在进行的工作”指针可能会进一步前进，因为它当前正在处理当前状态尚未赶上的新更新。

- 将更新附加到两个队列：当新的更新到达时，它会附加到当前队列和正在进行的队列的末尾。将其添加到两者中：

  - 如果“当前”队列稍后被用作启动新的“正在进行的工作”的基础（例如，如果我们从当前状态重新启动正在进行的工作渲染），则新添加的更新仍将是那里。
  - 如果“work-in-progress”队列提交并与当前队列交换，这些更新仍然不会丢失，因为它们也事先附加到了当前队列。

  这确保了一致性：无论我们将来使用哪个队列作为起点，更新始终存在于持久列表中。不存在由于进程中重置或提交而跳过或丢失更新的风险。

- 避免重复更新：React 确保不会应用两次更新。一旦更新通过提交从正在进行的工作转变为当前工作，它将成为新的基线。两个队列中的指针都向前移动，确保系统始终“知道”哪些更新已处理，哪些更新正在等待处理。

#### update 对象

```
export type Update<State> = {
  // transition -> event time on the root.
  eventTime: number,
  lane: Lane,

  tag: 0 | 1 | 2 | 3,
  payload: any,
  callback: (() => mixed) | null,

  next: Update<State> | null,
};
```

用于表示组件状态或 props 的待处理更改。当 React即将更新组件的状态时，它会创建一个 Update 对象并添加到更新队列中，以便在后续的渲染阶段来处理这些更新。

- `eventTime`：何时计划更新的时间戳。React 使用 eventTime 与其调度算法结合来确定更新等待的时间并应用某些启发式方法，例如优先考虑较旧的更新。 eventTime 帮助 React 更好地协调多个更新并确保更灵敏的渲染。

- `lane`:  Lane字段决定了更新的优先级。

- `tag` : 该标签表示要执行的状态更新的“类型”。 React 的 Fiber 协调器使用这些数字代码以不同的方式处理更新

  - 0: Replace the state entirely with the new value. 常用more direct replacement
  - 1: Used internally for error boundaries to capture and handle errors. 
  - 2: Forces the component to re-render, ignoring `shouldComponentUpdate`.
  - 3: Partially update the state by merging in the provided object or applying the provided updater function. 常用  common `setState` scenario

- `payload` ： 有效负载是描述如何更新组件状态的数据或函数。根据标签的不同，它可以是：

  - 要合并到当前状态的新状态字段的对象。
  - 函数 (prevState, props) => newState 在给定先前状态的情况下返回新状态。
  - For error handling or forced updates, it might store additional information relevant to that operation.

  **该有效负载在渲染阶段被消耗，以计算组件的下一个 memoizedState。**核心

- `callback` ：

  - 如果不为空，则该字段包含一个回调函数，该函数应在组件状态更新且组件重新渲染后运行。它对应于类组件中setState(updater,callback)的第二个参数。提交更改后，React 将调用这些回调，以便开发人员可以运行需要访问更新的 UI 的副作用。

- `next` ： 更新存储为单链表的一部分。下一个字段指向队列中的下一个更新。在处理单个Fiber之前，可以累积多个更新。在协调期间，React 会遍历这些由 next 链接在一起的更新。这种链表结构有助于 React 在并发渲染期间有效地处理更新的批处理及其部分处理。





#### updateQueue对象

```
export type UpdateQueue<State> = {
  baseState: State,
  firstBaseUpdate: Update<State> | null,
  lastBaseUpdate: Update<State> | null,
  shared: SharedQueue<State>,
  effects: Array<Update<State>> | null,
};
```

每个**组件实例**都维护一个 UpdateQueue 来跟踪和处理其状态随时间的变化。 UpdateQueue 不会立即更新组件的状态；相反，它会保留待处理的更新，直到下一个渲染阶段。

- `baseState` : This is the state value upon which the next set of updates will be applied. Think of it as the “starting point” before the queued updates run. After processing all pending updates during a render, the resulting state becomes the new `baseState`.

  - For class components, after every commit, `baseState` generally reflects the component’s current, committed state.
  - `baseState` is crucial because, in concurrent mode, React may need to restart rendering with the last fully confirmed version of the state if an interruption or higher-priority update occurs.

- `firstBaseUpdate` : This points to the first update in the “base” list of updates that need to be processed. These are the updates that have not yet been integrated into `baseState`. Essentially, the updates between `firstBaseUpdate` and `lastBaseUpdate` represent the subset of the update queue that must still be processed to bring `baseState` up-to-date. If `firstBaseUpdate` is `null`, it means there are currently no unprocessed base updates for this fiber.

- `lastBaseUpdate`: This points to the last update in the “base” list. Along with `firstBaseUpdate`, it defines a linked list of updates that must be processed. After processing them during render, `lastBaseUpdate` may move forward, shrink, or become `null` if all updates are fully applied.

- `shared`: The `shared` property refers to a structure that holds pending updates that are shared between the current fiber and its work-in-progress counterpart. The `SharedQueue` typically looks like this:

  ```
  type SharedQueue<State> = {
    pending: Update<State> | null
  };
  ```

  The `pending` field in `shared` stores newly arrived updates in a circular linked list. When React transitions into a new render phase (creating a work-in-progress fiber), it transfers pending updates from `shared.pending` into the fiber’s base update list. By maintaining updates in `shared`, React can ensure no updates are lost during interruptions, restarts, or when switching between current and work-in-progress fibers. This dual-queue approach ensures that updates are always processed eventually, regardless of how many times work is paused or resumed.

- `effects` : This is an array of updates that have associated callbacks. When the updates are processed and committed, these callbacks (if provided via `setState(updater, callback)`) need to be executed. The `effects` array gathers those updates whose callbacks must be run after the component’s state is finalized and the DOM has been updated.

  - After the commit phase, React will iterate through this array, calling each callback.
  - Typically, `effects` is `null` if no updates have callbacks.



- **Base State & Updates:**
  React starts with `baseState`, which reflects the last known, stable state of the component. On top of `baseState`, React needs to apply the updates from `firstBaseUpdate` through `lastBaseUpdate`. These updates are processed during the render phase to generate a new `memoizedState`, which, after commit, becomes the next `baseState`.
- **Shared Queue & Pending Updates:**
  The `shared` queue accumulates incoming updates even if a render is in progress. When React begins a new render phase, it incorporates these pending updates into the base update list. This ensures no update is ever dropped, even if React discards a partial render and starts over.
- **Effects (Callbacks):**
  Some updates have callbacks that should run after the commit. The `effects` array holds onto these until the component’s changes have been flushed to the screen, at which point the callbacks are invoked.

### Why is This Complexity Needed?

React’s concurrent architecture must handle updates that can arrive at any time, even while a render is in progress. It should be able to pause, resume, and reorder work without losing track of any updates. The `UpdateQueue` and its associated structures (`baseState`, `shared`, `effects`) give React the flexibility to manage updates intelligently:

- **Concurrent Mode:**
  With concurrent mode, multiple versions of the UI can be in flight. `UpdateQueue` ensures that updates are never lost and can be reapplied if a render is restarted.
- **No Lost Updates:**
  By maintaining updates in both the `shared` and `base` lists, React can always rebuild the latest intended state, even if it temporarily discards some intermediate work.
- **Callbacks After Commit:**
  The `effects` array ensures that any callbacks associated with state updates are not lost and will be run once the UI is consistent.

**In short, `UpdateQueue` is central to how React coordinates asynchronous, interruptible updates, preserving correctness and ensuring that state changes and their associated callbacks are consistently and predictably applied.**







### Hook 对象

在函数式组件的Fiber中，`fiber.memoizedState` 指向一个hooks的对象。在调和的过程中，用来表示和管理组件内的React hook的状态。

```typescript
export type Hook = {
  memoizedState: any,
  baseState: any,
  baseQueue: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null,
  next: Hook | null,
};
```

- `memoizedState` : 存储 Hook 当前渲染的、完全计算的状态。 React 完成渲染函数组件后，该字段保存 Hook 在该渲染周期中确定的最终值。
- `baseState` :  
  - 表示了： baseState 是应用挂起更新的状态的快照。在并发和可中断的渲染场景中，React 可能需要重新启动渲染过程，并且拥有稳定的 baseState 可确保 React 在应用任何排队更新之前知道最后确认的状态。
  - 为什么需要？： 如果某些更新的优先级较低，React 可能会暂时跳过它们并在恢复工作时恢复到 baseState，以确保不会丢失或错误应用更新。
- `baseQueue`: 
  - 表示：
    - baseQueue 是尚未完全处理或提交到状态的更新对象的链表。这些是组件重新渲染时需要应用的“待处理更改”。
  - 将baseQueue 视为必须在baseState 之上应用才能达到memoizedState 的一组更新。如果发生任何中断（例如 React 决定屈服于用户交互或从新状态重新渲染），baseQueue 通过重播这些更新来帮助 React 重新导出最终状态。
- `queue` :
  - 表示：它保存待处理的更新，并且一旦处理，就会对齐baseQueue和baseState以生成memoizedState。
  - 包含了：
    - A reference to `baseState` (the initial state from which updates are calculated),
    - The first and last base updates not yet committed,
    - a `shared` structure that tracks newly scheduled updates.
  - 功能：
    - 当在函数组件中（从 Hook）调用 setState 之类的东西时，React 会创建一个 Update 并将其添加到此队列中。当React重新渲染组件时，它会查看此队列，处理更新以派生新的memoizedState，然后相应地更新baseState和baseQueue。
- `next` : 
  - 表示：链表中的next指针；创建与单个函数组件实例关联的 Hook 链表。每个功能组件可以有多个 Hook（例如 useState、useEffect、useReducer 等），React 将这些 Hook 节点按照渲染过程中遇到的顺序链接在一起。
  - 如何工作的？：当 React 调用你的函数组件时，它会按顺序运行每个 Hook 调用。在内部，它浏览这个 Hook 对象的单链接列表。 next 允许 React 在渲染之间维护 Hook 的正确顺序，确保第一个 useState 调用始终引用同一个 Hook 节点，第二个 useEffect 调用引用第二个 Hook 节点，依此类推。

思考的问题：

1. Why do we need this structure?
2. Concurrent Mode and Restarting Renders:
3. Stable Ordering:



### Summary

The `Hook` type is an internal mechanism that allows React to manage the state and updates of Hooks in function components. It keeps track of:

- The current (`memoizedState`) and baseline (`baseState`) states,
- Any pending updates through queues (`queue` and `baseQueue`),
- And the chain of Hooks (`next`) within a component.

Together, these fields give React the information it needs to efficiently schedule updates, handle concurrent rendering, and ensure that each Hook’s state corresponds exactly to the position and number of Hook calls in the component’s function body.



```typescript
type Update<S, A> = {
  lane: Lane,
  action: A,
  eagerReducer: ((S, A) => S) | null,
  eagerState: S | null,
  next: Update<S, A>,
  priority?: ReactPriorityLevel,
};
```





```typescript
type UpdateQueue<S, A> = {
  pending: Update<S, A> | null,
  dispatch: ((A) => mixed) | null,
  lastRenderedReducer: ((S, A) => S) | null,
  lastRenderedState: S | null,
};
```







### Task 对象



### WorkTags 常量定义

```
export type WorkTag =
  | 0
  | 1
  | 2
  | 3
  | 4
  | 5
  | 6
  | 7
  | 8
  | 9
  | 10
  | 11
  | 12
  | 13
  | 14
  | 15
  | 16
  | 17
  | 18
  | 19
  | 20
  | 21
  | 22
  | 23
  | 24
  | 25;
 
export const FunctionComponent = 0;
export const ClassComponent = 1;
export const IndeterminateComponent = 2; // Before we know whether it is function or class
export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5;
export const HostText = 6;
export const Fragment = 7;
export const Mode = 8;
export const ContextConsumer = 9;
export const ContextProvider = 10;
export const ForwardRef = 11;
export const Profiler = 12;
export const SuspenseComponent = 13;
export const MemoComponent = 14;
export const SimpleMemoComponent = 15;
export const LazyComponent = 16;
export const IncompleteClassComponent = 17;
export const DehydratedFragment = 18;
export const SuspenseListComponent = 19;
export const ScopeComponent = 21;
export const OffscreenComponent = 22;
export const LegacyHiddenComponent = 23;
export const CacheComponent = 24;
export const TracingMarkerComponent = 25;
```




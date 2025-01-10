# 阅读React源码 #1

### 初识React：


### React四大阶段：
- 触发阶段（Trigger）： 不论是初次挂载阶段（mount）还是重新渲染（re-render），在React运行时，我们会告知React应用的哪个部分会被更新，并通过这个阶段调用接下来的 `scheduleUpdateOnFiber()` 我们可以把这个阶段考虑为创建任务， 并最后把这个任务传递给 `scheduleCallback()`。 
  - useState 或 useReducer hook 当中的 `setState()` 
  - ReactDOM中的 `render()` 
- 调度阶段（Schedule）: 在这个阶段， React通过优先队列来管理任务的优先级 `scheduleCallback` 会在这个阶段执行，`workLoop`函数则是 Scheduler 在触发阶段创建的任务执行的函数。
  - 
- 渲染阶段（Render）：渲染阶段则会计算新的 Fiber 树结构并找出会有哪些更新到 host DOM上。在 `concurrent` 模式下，render阶段可以被中断并恢复。
- 提交阶段（Commit）：当一个新的 Fiber 树被创建后（带有新的更新），则可以进行更新 host DOM（DOM节点更新）， 在这个阶段中 所有的副作用（Effects）会被执行。与这个阶段相关的函数有：`commitMutationEffects(), flushPassiveEffects(), commitLayoutEffects()` 




### React Mount时都发生了什么

#### 初始化创建与触发：

创建的数据结构：
- `FiberRootNode` : 包含整个React应用所需的meta信息，它的 `current` 指向 真实Fiber树，每次一个新的Fiber树创建后，它的 `current` 会重新指向新的 `HostRoot` 
- `FiberNode` : Fiber树的基础组成单位，比较重要的属性包括：
  - tag：Fibernode的类型，例如：FunctionComponent, HostRoot, ContextConsumer, MemoComponent
  - stateNode：指向真实数据
  - child：指向子节点
  - sibling：指向兄弟节点
  - return: 指向父节点
  - elementType:                     
  - flags:
  - lanes: 表示了更新的优先级
  - childLanes：表示了子树的优先级
  - memoizedState: 对于函数式组件，它表示hooks
```javascript
// 一个Fiber对象代表一个即将渲染或者已经渲染的组件(ReactElement), 一个组件可能对应两个fiber(current和WorkInProgress)
// 单个属性的解释在后文(在注释中无法添加超链接)
export type Fiber = {|
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
  pendingProps: any, // 从`ReactElement`对象传入的 props. 用于和`fiber.memoizedProps`比较可以得出属性是否变动
  memoizedProps: any, // 上一次生成子节点时用到的属性, 生成子节点之后保持在内存中
  updateQueue: mixed, // 存储state更新的队列, 当前节点的state改动之后, 都会创建一个update对象添加到这个队列中.
  memoizedState: any, // 用于输出的state, 最终渲染所使用的state
  dependencies: Dependencies | null, // 该fiber节点所依赖的(contexts, events)等
  mode: TypeOfMode, // 二进制位Bitfield,继承至父节点,影响本fiber节点及其子树中所有节点. 与react应用的运行模式有关(有ConcurrentMode, BlockingMode, NoMode等选项).

  // Effect 副作用相关
  flags: Flags, // 标志位
  subtreeFlags: Flags, //替代16.x版本中的 firstEffect, nextEffect. 当设置了 enableNewReconciler=true才会启用
  deletions: Array<Fiber> | null, // 存储将要被删除的子节点. 当设置了 enableNewReconciler=true才会启用

  nextEffect: Fiber | null, // 单向链表, 指向下一个有副作用的fiber节点
  firstEffect: Fiber | null, // 指向副作用链表中的第一个fiber节点
  lastEffect: Fiber | null, // 指向副作用链表中的最后一个fiber节点

  // 优先级相关
  lanes: Lanes, // 本fiber节点的优先级
  childLanes: Lanes, // 子节点的优先级
  alternate: Fiber | null, // 指向内存中的另一个fiber, 每个被更新过fiber节点在内存中都是成对出现(current和workInProgress)

  // 性能统计相关(开启enableProfilerTimer后才会统计)
  // react-dev-tool会根据这些时间统计来评估性能
  actualDuration?: number, // 本次更新过程, 本节点以及子树所消耗的总时间
  actualStartTime?: number, // 标记本fiber节点开始构建的时间
  selfBaseDuration?: number, // 用于最近一次生成本fiber节点所消耗的时间
  treeBaseDuration?: number, // 生成子树所消耗的时间的总和
|};

```


在拥有 `ReactDOM` 实例后，我们会调用 `render` 函数来触发第一个渲染。

#### 初始化渲染：

1. `performConcurrentWorkOnRoot()` : 
   - 做了什么：
   - 注意的点：
     - 在初始化时时 sync 同步模式 
2. 初始化时，`performConcurrentWorkOnRoot()` 中调用 `renderRootSync()` : 
    - 做了什么：执行 `workLoopSync()` 
      - 
    - 注意的点：
      - `current` : 表示当前UI上的Fiber树
      - `workInProgress` : 正在构建的 Fiber树
3. `workLoopSync()` : 当正在构建的树不为空时，调用 `performUnitOfWork()`
4. `performUnitOfWork()`
   - 做了什么：这是 React 在单个 Fiber Node 上工作以查看是否有任何事情要做的地方。调用 `beginWork` 
   - 调用过程：
   - 注意的点：
5. `beginWork()` :
   - 做了什么：渲染过程发生的函数。
     - 如果是 mount 阶段， 则直接调用 `updateHostRoot`
   - 注意的点：
6. `prepareFreshStack()` : 每次开始新的渲染时，都会从当前 HostRoot 创建一个新的 workInProgress。它作为Fiber树的根。(在renderRootSync中调用的)
7. `updateHostRoot()` 由于是初始化阶段， 则在 beginwork走到 updateHostRoot 分支
   - `processUpdateQueue()` : 主要用于 `hostroot` 的更新与 `classComponent` 的更新： 它处理更新队列以计算组件的新状态。
   - `reconcileChildren()` : Here current and workInprogress both don't have child And nextChildren is <App/>
   - return workInProgress.child; After reconciling, new child is created for workInProgress Returning here means that workLoopSync() will handle it next
8. `reconcileChildren()` : diff 函数， 对比新的子与老的子，将正确的子节点设置到 正在构建的树上。
   - As mentioned above, FiberRootNode always has current, so we go to the 2nd branch - reconcileChildFibers. But since this is the initial mount, its child current.child is null. Also notice that we are setting child on workInProgress, since workInProgress is being constructed and it doesn’t have child yet.
   - 如上所述，FiberRootNode 始终有 current，因此我们转到第二个分支 - reconcileChildFibers。但由于这是初始挂载，因此其子 current.child 为 null。另请注意，我们正在 workInProgress 上设置 child，因为 workInProgress 正在构造中，而且还没有 child。
9. reconcileChildFibers() vs mountChildFibers()
   - The goal of reconcile is to try to reuse stuff we already have, we can treat mount as a special primitive version of reconcile, in which we always refresh everything.
   - mountChildFibers（只将根节点insert到子节点中） is actually a internal improvement to make things more explicit.
10. reconcileSingleElement()： 
    - 在 mount 阶段中： the newly created Fiber Node will be child on workInProgress.
    - One thing to notice is that when Fiber Node is created from custom component, its tag is IndeterminateComponent, not FunctionComponent yet.
11. placeSingleChild()： 
    - Notice that this is done on child, meaning in initial mount, the child of HostRoot is marked with Placement. In our demo code, it is <App/>.
    - 
12. mountIndeterminateComponent(): 我们将看到 beginWork() 中的下一个分支是 InminatedComponent。由于 <App/> 位于 HostRoot 下，并且正如已经提到的，自定义组件最初被标记为 InminatedComponent，因此当第一次 <App/> 被协调时(reconciled)，我们将来到这里。
13. updateHostComponent()
14. updateHostText()
15. completeWork(): 
    - is called on a fiber before its sibling is tried with beginWork().
    - Fiber Node has one important property - stateNode, which for intrinsic HTML tags, it refers to the actual DOM node. And the actual creation of DOM node is done in completeWork().
    - 
16. Initial mount in Commit phase
17. commitMutationEffects(): handles the DOM mutation.
    - recursivelyTraverseMutationEffects: This recursive call makes sure subtree is handled first
    - commitReconciliationEffects: ReconciliationEffects means Insertion .etc.
    - 
18. commitReconciliationEffects(): handles the Insertion, reordering .etc.
19. commitPlacement()

### Re-render的过程

#### Trigger 阶段
1. `lanes` 与 `childLanes` :
   - 是什么：
     - `lanes`: 对于本身未完成的工作
     - `childLanes` : 其子树的待处理工作
1.1. 当我们点击按钮时， setCount或者说 setState会被调用， 之后发生了：
- 从根到目标 Fiber 的路径将用 Lane 和 ChildLanes 标记，以指示在下次渲染中需要检查的位置。
- 更新对象会被 `scheduleUpdateOnFiber()` 调度，最终调用 `ensureRootIsScheduled()` 并在 `Scheduler` 中调度 `performConcurrentWorkOnRoot()` 。这个流程和初始化的过程有很多相似之处 
  - 更新的优先级是由更新的事件所决定的， 比如点击事件的优先级为 `DiscreteEventPriority` 它映射到SyncLane - 高优先级。


#### Re-render 阶段：
2. 基本渲染逻辑与初始挂载相同：React只会便利Fiber tree并更新需要更新的 Fiber
3. React 在创建新节点之前重用冗余的 Fiber 节点, 在re-render阶段调用 `prepareFreshStack` -> `createWorkInProgress` 会尝试（createWorkInProgress）复用Fiber树的节点，
4. `beginWork` 的更新逻辑
   - 
5. 



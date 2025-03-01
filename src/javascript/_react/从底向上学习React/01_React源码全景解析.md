# React 源码全景解析

大家好， 我是研发K，详细阅读React源码一直是我心里的一个梦想和想做的事情，之前由于自己的内阻力问题，一直拖延到现在。现在我想好好阅读下并总结成一个系列文章，我将采用从顶向下的顺序，全局预览React源码到每个细节分析，来分享我对于React源码的理解，希望能对每一个React开发工程师有帮助。那么闲话少说，让我们进入到第一篇的学习 React源码全景分析。

在本篇分享中，我将通过对React源码18.2.0的分析来解析：
- 源码目录结构分析
- 核心架构一览（Scheduler, Reconciler, Renderer）
- 核心流程分析：
  - 初始化：
  - 更新
这三点，希望阅读后，大家能够掌握其基本原理，并能在进行React编码过程中，在脑海中构造一个VDOM树。


## 源码仓库文件夹 React 源码 packages文件夹下的子文件夹分析

当我们开始阅读React源码时，我们通常会进入到`packages` 文件夹下， 里面共有36个子文件夹（基于react 18.2.0）（通过 `ls -1 | wc -l` ）。我们可以将这36个文件夹分类为4大类：
- 核心流程包：
  1. `react` : 核心React库， 提供了诸如`React.createElemnt、 React Hooks和组件` 等API。
  2. `react-dom` ：提供了React与浏览器DOM交互的功能，包含了 `ReactDOM.render` 等API。
  3. `react-native-renderer` ： 用于原生环境的React渲染器，用于适配与原生元素交互的协调过程。
  4. `react-reconciler` ： React核心引擎， 提供了Diff算法的过程，通常与 `react-dom 或 react-native-renderer` 一起使用。
  5.  `scheduler` : 一个独立的包，用于管理 React 更新的调度。它是一个协作调度程序，可以帮助 React 协调渲染，而不会不必要地阻塞主线程。 React 协调器在底层使用它来确定更新的优先级。
- 开发与测试包：
  1. `react-devtools, react-devtools-core, react-devtools-extensions, react-devtools-inline, react-devtools-shared, react-devtools-shell, react-devtools-timeline`: 这些包构成了 React DevTools 生态系统。它们提供浏览器扩展、共享组件和 DevTools 功能的其他层，帮助开发人员检查 React 组件树、挂钩和性能特征。
  2. `react-debug-tools` ： 提供 debug API，使您可以更轻松地检查 React 组件的内部状态和树以进行调试和测试。
  3. `react-suspense-test-utils`: 用于测试与 Suspense 相关的功能的实用程序，允许您模拟和验证加载状态和异步渲染行为。
  4. `dom-event-testing-library, jest-mock-scheduler, jest-react`: 帮助测试 React 组件和功能的实用程序。例如，dom-event-testing-library有助于在React环境中模拟和测试DOM事件，而jest-mock-scheduler则由React自己的测试套件在内部使用。
  5. `react-test-renderer`: 允许您将 React 组件渲染为 JSON 表示形式而不是浏览器 DOM，从而更轻松地创建可预测的测试和快照测试。
  6. `react-client` : 支持 React 的服务器组件并改进了服务器渲染功能。
  7. `react-noop-renderer` : 用于测试React协调过程与调度的库。
- 试验包:
  1. `react-art` : 将 React 与矢量图形的 ART 库集成。
  2. `react-cache` ： 用于探索缓存机制以支持 Suspense 获取数据。
  3. `react-fetch, react-fs, react-pg`： 实验包探索以 React 友好的方式与网络请求、文件系统读取或其他特定于平台的数据获取的集成。
  4. `react-interactions` ： 实验性事件/交互系统包，探索高级事件处理和用户交互模式。
  5. `react-server, react-server-dom-relay, react-server-dom-webpack, react-server-native-relay`: 这些包是 React Server Components 实验工作的一部分，直接在服务器上集成 React 渲染并使用不同的数据获取和捆绑策略（例如，使用 Relay 或 Webpack）。
- 工具包:
  1. `eslint-plugin-react-hooks` : ESLint 插件强制执行 Hooks 规则，通过分析代码模式确保正确使用 Hooks。
  2. `react-is` : 提供一组实用程序来识别和检查 React 元素、片段类型、门户等。
  3.  `use-subscription, use-sync-external-store` :  旨在将外部数据源或存储与 React 渲染周期集成的钩子。 use-sync-external-store 被用作 React 推荐模式的一部分，用于在并发模式下安全地读取外部存储。
  4.  `shared` : 多个 React 包使用的常见内部实用程序、常量和辅助函数。这段代码通常不适合公众使用，但有助于在 React 的不同部分保持通用逻辑 DRY（不要重复自己）。
  5.  `react-refresh` : 优化版本的 HMR 

## 核心架构一览（Scheduler, Reconciler, Renderer）
在React中，一个从更新到展示一共分为4个阶段，分别是：
- 触发阶段 Trigger： 在这个阶段中的主要任务为 ”通知部分应用需要渲染“，在这个阶段，更像是创建一个任务，并在之后将任务交给调度器处理。
- 调度阶段 Schedule：在这个阶段，React会通过优先队列与任务优先级来处理”待处理的任务“
- 渲染阶段 Render：通过 Diff算法 来计算出新的Fiber（WIP）树与当前Fiber树的区别，并将需要处理的更新应用到DOM上
- 提交阶段 Commit：处理需要被更新的内容到真实DOM中，并处理所有的Effects副作用。

在这几个阶段中，一共三个比较重要的部分发挥着作用，分别是调度器（Scheduler）、协调器（Reconciler）、渲染器（Renderer），React通过将UI抽象化，并将功能的解耦，实现了逻辑的复用、性能优化和平台无关性。

- Scheduler 调度器：
  - 职责：
    - 任务优先级管理：根据定义好的任务紧急程度，动态调整执行顺序。
    - 时间切片：将一个长任务分割为多个微任务，避免主线程堵塞。
    - 任务的中断与恢复：通过浏览器空闲时间执行低优先级任务

- Reconciler 协调器：
  - 职责：
    - 双缓存技术：维护当前树（Current Tree）和工作树（WorkInProgress Tree），实现无卡顿的异步渲染。
    - Diff算法：通过 Fiber 架构的链表结构，实现可中断的虚拟 DOM 差异计算。
    - Effect标记：生成副作用链表（如插入、更新、删除），描述具体需要渲染的变更。

- Renderer 渲染器：
  - 职责：
    - 平台渲染：将 Reconciler 计算的变更应用到具体环境（如 DOM API 或 Native 组件）。
    - 渲染优化：批量更新（Batching）、差异提交（Commit Phase）避免中间状态闪烁。

## 核心流程一览：

我们通常在React项目中都会通过这样的语法来初始化React应用：
```javascript
import { createRoot } from 'react-dom/client';
import React from 'react';

import App from './App.jsx';

const reactRoot = document.querySelector('#app');

const root = createRoot(reactRoot);
root.render(<App key={'app'} />);
```
这段代码涉及到两个部分：
- `createRoot` 初始化ReactDOM对象
- `render(<App key={'app'} />);` : mount React应用

### 初始化：
![初始化](./assets/01/init.png " React初始化")

初始化一共做了这两件事：
- 初始化`FiberRootNode`节点:
  - 用来管理 `Fiber` 树（current和WorkInProgress）
    ```
      FiberRootNode
    ├── current (指向当前 UI 的 Fiber 树)
    ├── workInProgress (指向正在渲染的新 Fiber 树)
    ```
  - 存储React更新队列
  - 协调调度，管理更新流程
- 进行React事件代理：
  - 所有事件监听器都会挂载在 `id=root` 的容器上
  - 事件冒泡到 节点 时，React 统一处理，然后找到事件真正发生的组件，调用相应的回调。
  - 在我们使用 `onClick` 事件时，React并不会真正的把事件绑定到对应的DOM元素上，而是通过这种代理的方式在React内部找到对应的事件回调进行执行与触发。

### 更新

在代码调用 `root.render(<App key={'app'} />);`
在这段代码中会触发React的第一次渲染流程。


### 车道模型中相关的函数定义:

```javascript
export function requestUpdateLane(fiber) {
  // Special cases
  const mode = fiber.mode;
  if ((mode & ConcurrentMode) === NoMode) {
    return (SyncLane: Lane);
  } else if (
    !deferRenderPhaseUpdateToNextBatch &&
    (executionContext & RenderContext) !== NoContext &&
    workInProgressRootRenderLanes !== NoLanes
  ) {
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

  const isTransition = requestCurrentTransition() !== NoTransition;
  if (isTransition) {
    if (__DEV__ && ReactCurrentBatchConfig.transition !== null) {
      const transition = ReactCurrentBatchConfig.transition;
      if (!transition._updatedFibers) {
        transition._updatedFibers = new Set();
      }

      transition._updatedFibers.add(fiber);
    }
    // The algorithm for assigning an update to a lane should be stable for all
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
  }

  // Updates originating inside certain React methods, like flushSync, have
  // their priority set by tracking it with a context variable.
  //
  // The opaque type returned by the host config is internally a lane, so we can
  // use that directly.
  // TODO: Move this type conversion to the event priority module.
  const updateLane: Lane = (getCurrentUpdatePriority(): any);
  if (updateLane !== NoLane) {
    return updateLane;
  }

  // This update originated outside React. Ask the host environment for an
  // appropriate priority, based on the type of event.
  //
  // The opaque type returned by the host config is internally a lane, so we can
  // use that directly.
  // TODO: Move this type conversion to the event priority module.
  const eventLane: Lane = (getCurrentEventPriority(): any);
  return eventLane;
}

```


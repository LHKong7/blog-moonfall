# React核心

## 核心流程

- 触发阶段 Trigger：在这个阶段的主要任务为 “通知React部分app需要渲染”，这个阶段更像是创建一个任务，并将任务交给调度器处理。
- 调度阶段 Schedule：简单来说，这个阶段就是通过优先队列通过任务优先级来处理任务。
- 渲染阶段 Render：通过diff算法来计算出新的Fiber树与当前树的区别，并将需要处理的更新应用到DOM上。
- 提交阶段 Commit：处理需要被更新的内容到真实DOM中，并处理所有的Effects副作用。


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





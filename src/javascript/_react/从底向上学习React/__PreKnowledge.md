# 前置知识：

- `Object.prototype.hasOwnProperty()`: 静态函数， 返回一个布尔值，表示属性是否为对象的自有属性（不是继承来的属性）

## React中的位运算
在JS中，所有的位运算都会按照 **有符号32位整型** 进行计算：
- 所以当操作数是浮点型时首先会被转换成整型, 再进行位运算
- 当操作数过大, 超过了Int32范围, 超过的部分会被截取

位掩码：通过位移定义的一组枚举常量, 可以利用位掩码的特性, 快速操作这些枚举产量(增加, 删除, 比较).

1. 属性增加 `|`： ABC = A | B | C
2. 属性删除 `&~` ： AB = ABC & ~C
3. 属性比较:
   1. AB当中包含B ：AB 当中包含 B: AB & B === B
   2. AB当中不包含C： AB 当中不包含 C: AB & C === 0
   3. A和B相等： A === B


## React 中的常量：

### Fiber的类型
```javascript
/**
 * Copyright (c) Facebook, Inc. and its affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

export type WorkTag =
  | 0 // FunctionComponent
  | 1 // ClassComponent
  | 2 // IndeterminateComponent
  | 3 // HostRoot
  | 4 // HostPortal
  | 5 // HostComponent
  | 6 // HostText
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

### HostRoot的类型

```javascript
export type RootTag = 0 | 1;

export const LegacyRoot = 0;
export const ConcurrentRoot = 1;
```

### React HostRoot Mode

```javascript
export type TypeOfMode = number;

export const NoMode = /*                         */ 0b000000;
// TODO: Remove ConcurrentMode by reading from the root tag instead
export const ConcurrentMode = /*                 */ 0b000001;
export const ProfileMode = /*                    */ 0b000010;
export const DebugTracingMode = /*               */ 0b000100;
export const StrictLegacyMode = /*               */ 0b001000;
export const StrictEffectsMode = /*              */ 0b010000;
export const ConcurrentUpdatesByDefaultMode = /* */ 0b100000;
```

### 更新队列

```javascript
var queue = {
   baseState: fiber.memoizedState, // 初始状态，通常是 fiber.memoizedState
   firstBaseUpdate: null, // 基础更新队列的第一个 update
   lastBaseUpdate: null, // 基础更新队列的最后一个 update
   shared: {
      pending: null, // 共享的更新队列，存放待处理的 update
      interleaved: null, // 并行模式下的交错更新
      lanes: NoLanes // 记录该 fiber 节点关联的更新 lanes（调度优先级）
   },
   effects: null // 存储副作用（effect）更新队列
};
fiber.updateQueue = queue;
```
`initializeUpdateQueue` 函数为 Fiber 树中的某个 Fiber 节点初始化更新队列，这个队列主要用于：
	1.	存储 setState 产生的更新，包括增量更新（partial state）。
	2.	维护不同优先级的更新（lanes），以支持并发模式（Concurrent Mode）。
	3.	管理任务批处理，合并更新，减少不必要的渲染。
	4.	存储副作用（effect），确保 useEffect、生命周期钩子等能够正确执行。

```javascript
const HostRootFiber = {
  tag: HostRoot,  // 根节点类型
  memoizedState: { count: 0 }, // 之前的状态
};

initializeUpdateQueue(HostRootFiber);

console.log(HostRootFiber.updateQueue);
/*
{
  baseState: { count: 0 },  // 继承 HostRoot 的 memoizedState
  firstBaseUpdate: null,
  lastBaseUpdate: null,
  shared: {
    pending: null,
    interleaved: null,
    lanes: NoLanes
  },
  effects: null
}
*/
```

#####


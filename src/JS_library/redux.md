## Redux

Redux 是一个可以被预测的JS状态管理器, Redux维护了一个中心`store` 来存储状态（state），根据相关的操作对 `store` 中的数据进行更新。

### Sub-Packages

#### Redux core
- Redux核心库
- 下载：
  - ```npm install redux```

#### Redux Toolkit
- Redux官网上推荐创建redux app的第三方库。封装了 Redux Core，包括了基础包与函数来生成redux应用，其中包括了：store setup，创建reducers和写 immutable更新逻辑，也包括了更高级的用法，比如`createSimpleSlice`, 可以在一次性创建整个slice
  - ```npm install @reduxjs/toolkit```

#### Redux DevTools Extension
- 开发者工具，展示了state的历史修改记录

### Redux的概念名词
- state management: 
  - state : 状态，数据源
  - view：declarative description of UI based on the current state
  - actions：the events that occur in the app based on user input, and trigger updates in the state
  - 通过 单向数据流 （one-way data flow） 来维护数据更新

- Immutability:
  - JS中的对象和数组都是引用值， 我们可以直接修改
  - 为了不可变地更新值，您的代码必须复制现有对象/数组，然后修改副本。

### 术语
- Actions： 一个 `action` 是一个JS对象包含了：
  -  `type`属性：来描述当前操作（"domain/eventName"）
  -  `payload`属性：包含了数据信息。

- Action creators：一个函数创建和返回 `action` 对象
    -  ```js
        const addTodo = text => {
            return {
                type: 'todos/todoAdded',
                payload: text
            }
        }
        ```
- Reducers: 用来接受当前状态和 `action` 对象的函数，用来决定如何更新 `state` 后返回一个新的 `state`:
  - `(state, action) => newState`
  - Reducers 需要保证以下规则：
    - 需要通过 `state` 和 `action` 来计算下一个 `state`值
    - 每次更新需要保证不改变当前 `state` 而是通过拷贝当前 `state` 并且修改拷贝值的内容后再更新
    - 需要是 pure function（纯函数）

- Store： 当前应用的全局 `state` 会保存在叫做 `store`的对象中, `store`是通过传递 `reducer` 来创建的，之后可以使用 `getState`来获取当前 state。
- Dispatch：The only way to update the state is to call `store.dispatch() and pass in an action object` .  
- Selectors: 用来拿到 `store` 中某一部分的state的函数，在相对较大的应用中，`selectors`可以避免

### Redux中数据流向

初始化：




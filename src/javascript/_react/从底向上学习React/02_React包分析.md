# React 

## 包含的知识：
- factory Design pattern: 创建一个React Element （ReactElement）:
  - Factory Method pattern: 将构造函数换成一个方法函数，通过在方法函数中进行new操作。通过Factory方法，我们在子类中重写工厂方法，从而改变其创建对象（object/product）的类型。在跨平台UI组件时，同时避免客户代码与具体UI类的耦合。



### React Ref 对象

- createRef: 创建一个内部current指针可被编辑或修改的对象。
```javascript
const refObject = {
    current: null,
};
```
- 

### React Context 对象
- createContext：创建一个可以被不同层级组件共享的JS对象；Context对象有两个比较重要的属性
  - Provider: 提供一个可以被任何后代使用的值。
  - Consumer: 允许后代订阅当前context


```javascript
const context: ReactContext<T> = {
    $$typeof: REACT_CONTEXT_TYPE,
    // As a workaround to support multiple concurrent renderers, we categorize
    // some renderers as primary and others as secondary. We only expect
    // there to be two concurrent renderers at most: React Native (primary) and
    // Fabric (secondary); React DOM (primary) and React ART (secondary).
    // Secondary renderers store their context values on separate fields.
    _currentValue: defaultValue,
    _currentValue2: defaultValue,
    // Used to track how many concurrent renderers this context currently
    // supports within in a single renderer. Such as parallel server rendering.
    _threadCount: 0,
    // These are circular
    Provider: (null: any),
    Consumer: (null: any),

    // Add these to use same hidden class in VM as ServerContext
    _defaultValue: (null: any),
    _globalName: (null: any),
};

context.Provider = {
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context,
};

context.Consumer = context;
```

通过上面的源码可发现， Provider和Consumer最终都会指向context 对象，保证了 `Provider` 和 `Consumer` 都指向了同一个对象，在更新Context时会保证内容的同步性，并减少了内存使用。


### forward ref

### lazy

### memo

### hooks

ReactCurrentDispatcher: React内部对象用来保存当前调度, 调度程序是一个提供 useState、useEffect 等钩子的实现的对象。

```javascript
const ReactCurrentDispatcher = {
  current: null,
};
```
`current`: 此属性指向当前渲染阶段（挂载或更新）的适当调度程序。

- React如何设置 `dispatcher` (当前调度)：
  - Mount
  - Update



- useCallback
- useContext
- useEffect


### element 相关:

#### createElement

### ReactElement: 用来创建React元素的工厂函数

- 入参：
  - `type`:
  - `key`:
  - `ref`:
  - `self`:
  - `source`:
  - `owner`:
  - `props`:

Element对象：
- `$$typeof`:
- `type`:
- `key`:
- `ref`:
- `props`: 
- `_owner`:


#### cloneElement

#### isValidElement

```javascript
export function isValidElement(object) {
  return (
    typeof object === 'object' &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
}
```

### Conurrent Mode:

#### transition

- useTransition: 和其他hooks一样
- startTransition
- useDeferredValue


### 需要考虑的内容：
- `props.children` : React通过 `mapChildren` 函数来帮助我们处理了。
- 

# popperjs 源码阅读

### recap
如果已经读了关于dom-align源码分析，那么对如何生成下来元素有了一定的了解，那么popperjs的源码的核心代码其实大同小异。

### more than functions
对比dom-align，popper提供了一个插件系统，其设计原理很像 `Strategy pattern`. Popperjs 的核心则是实用 `modifiers` （JS函数）来调整待定位元素的位置，不同的modifier提供了：碰撞检测，偏移量设置，翻转检测等特性。

## 源码分析
接下来让我们一起看其内部的源码实现，本次的源码基于 `"@popperjs/core": "2.11.8"`。我们从 `createPopper` 开始：


### 关键utils函数：

**setOptions**: 允许用户在运行时重新配置 Popper 实例。
```javascript
setOptions(setOptionsAction) {
    const options =
        typeof setOptionsAction === 'function'
        ? setOptionsAction(state.options)
        : setOptionsAction;

    // 从先前设置的修改器中清除任何现有的效果清理功能。
    cleanupModifierEffects();

    state.options = {
        ...defaultOptions,
        ...state.options,
        ...options,
    };

    // 该函数收集引用元素和弹出元素的可滚动祖先。这些用于了解哪些滚动事件可能会影响定位。
    state.scrollParents = {
        reference: isElement(reference)
        ? listScrollParents(reference)
        : reference.contextElement
        ? listScrollParents(reference.contextElement)
        : [],
        popper: listScrollParents(popper),
    };

    // 按依赖性和阶段对修饰符进行排序（例如，“beforeRead”、“read”、“afterRead”等）。
    const orderedModifiers = orderModifiers(
        mergeByName([...defaultModifiers, ...state.options.modifiers])
    );

    state.orderedModifiers = orderedModifiers.filter((m) => m.enabled);

    // 调用每个修饰符的effect()函数（如果提供），捕获任何返回的清理函数。
    runModifierEffects();

    return instance.update();
}
```




**createPopper**:
`const createPopper = popperGenerator({ defaultModifiers });`
指向 `popperGenerator` 函数，并添加一些默认的 `modifiers`, 包含：
- eventListeners,
- popperOffsets,
- computeStyles,
- applyStyles,
- offset,
- flip,
- preventOverflow,
- arrow,
- hide,

**popperGenerator**: 返回 `createPopper(reference，popper, options)` 函数


**createPopper(reference，popper, options)**:


```javascript
function createPopper(reference, popper, options = defaultOptions): Instance {
    let state: $Shape<State> = {
        placement: 'bottom',
        orderedModifiers: [],
        options: { ...DEFAULT_OPTIONS, ...defaultOptions },
        modifiersData: {},
        elements: {
        reference,
        popper,
        },
        attributes: {},
        styles: {},
    };

    let effectCleanupFns: Array<() => void> = [];
    let isDestroyed = false;

    const instance = {
        state,
        setOptions(setOptionsAction) {
        const options =
            typeof setOptionsAction === 'function'
            ? setOptionsAction(state.options)
            : setOptionsAction;

        cleanupModifierEffects();

        state.options = {
            // $FlowFixMe[exponential-spread]
            ...defaultOptions,
            ...state.options,
            ...options,
        };

        state.scrollParents = {
            reference: isElement(reference)
            ? listScrollParents(reference)
            : reference.contextElement
            ? listScrollParents(reference.contextElement)
            : [],
            popper: listScrollParents(popper),
        };

        // Orders the modifiers based on their dependencies and `phase`
        // properties
        const orderedModifiers = orderModifiers(
            mergeByName([...defaultModifiers, ...state.options.modifiers])
        );

        // Strip out disabled modifiers
        state.orderedModifiers = orderedModifiers.filter((m) => m.enabled);

        runModifierEffects();

        return instance.update();
        },

        // Sync update – it will always be executed, even if not necessary. This
        // is useful for low frequency updates where sync behavior simplifies the
        // logic.
        // For high frequency updates (e.g. `resize` and `scroll` events), always
        // prefer the async Popper#update method
        forceUpdate() {
        if (isDestroyed) {
            return;
        }

        const { reference, popper } = state.elements;

        // Don't proceed if `reference` or `popper` are not valid elements
        // anymore
        if (!areValidElements(reference, popper)) {
            return;
        }

        // Store the reference and popper rects to be read by modifiers
        state.rects = {
            reference: getCompositeRect(
            reference,
            getOffsetParent(popper),
            state.options.strategy === 'fixed'
            ),
            popper: getLayoutRect(popper),
        };

        // Modifiers have the ability to reset the current update cycle. The
        // most common use case for this is the `flip` modifier changing the
        // placement, which then needs to re-run all the modifiers, because the
        // logic was previously ran for the previous placement and is therefore
        // stale/incorrect
        state.reset = false;

        state.placement = state.options.placement;

        // On each update cycle, the `modifiersData` property for each modifier
        // is filled with the initial data specified by the modifier. This means
        // it doesn't persist and is fresh on each update.
        // To ensure persistent data, use `${name}#persistent`
        state.orderedModifiers.forEach(
            (modifier) =>
            (state.modifiersData[modifier.name] = {
                ...modifier.data,
            })
        );

        for (let index = 0; index < state.orderedModifiers.length; index++) {
            if (state.reset === true) {
            state.reset = false;
            index = -1;
            continue;
            }

            const { fn, options = {}, name } = state.orderedModifiers[index];

            if (typeof fn === 'function') {
            state = fn({ state, options, name, instance }) || state;
            }
        }
        },

        // Async and optimistically optimized update – it will not be executed if
        // not necessary (debounced to run at most once-per-tick)
        update: debounce<$Shape<State>>(
        () =>
            new Promise<$Shape<State>>((resolve) => {
            instance.forceUpdate();
            resolve(state);
            })
        ),

        destroy() {
        cleanupModifierEffects();
        isDestroyed = true;
        },
    };

    if (!areValidElements(reference, popper)) {
        return instance;
    }

    instance.setOptions(options).then((state) => {
        if (!isDestroyed && options.onFirstUpdate) {
        options.onFirstUpdate(state);
        }
    });

    // Modifiers have the ability to execute arbitrary code before the first
    // update cycle runs. They will be executed in the same order as the update
    // cycle. This is useful when a modifier adds some persistent data that
    // other modifiers need to use, but the modifier is run after the dependent
    // one.
    function runModifierEffects() {
        state.orderedModifiers.forEach(({ name, options = {}, effect }) => {
        if (typeof effect === 'function') {
            const cleanupFn = effect({ state, name, instance, options });
            const noopFn = () => {};
            effectCleanupFns.push(cleanupFn || noopFn);
        }
        });
    }

    function cleanupModifierEffects() {
        effectCleanupFns.forEach((fn) => fn());
        effectCleanupFns = [];
    }

    return instance;
};

```


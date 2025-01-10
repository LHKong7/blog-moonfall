# dom-align 源码分析

## 如何将一个DOM元素与另一个DOM元素对齐
在我们开始 `dom-align` 的源码前，先来思考如何将tooltip、dropdown或者弹窗进行定位和展示，通常需要我们相对于目标元素定位来overlay元素。

1. CSS：
对于简单的对齐，我们可以直接通过CSS来实现：
- 绝对定位: 将参考元素的父元素设置为 `position: relative`, 将目标元素设置为 `position: aboslute` 之后通过 `top, left, right, bottom` 和 margin负值来调整元素的对齐方式。
- 通过 `tranform` : 假设父元素为 `relative` 我们可以通过 `transform: translate`  属性来计算与目标元素的偏移量并进行对齐展示。

Pros：方法比较直接，不需要任何依赖
Cons：宽高并不是自适应，没办法随着窗口大小调整而改变（需要通过JS来监听事件与重新计算）。

2. JS对齐
当我们需要一个动态的 overlay 元素时，我们可以通过JS计算来进行对齐展示。以下几个步骤可以帮助我们定义一个简单的tooltip对齐：
   1. 计算目标元素的位置信息：我们可以通过 `getBoundingClientRect()` 来拿到 `top, left, width, height` 的信息。
   2. 计算overlay元素的展示的坐标信息。
   3. 检测是否发生碰撞。

```javascript
const refRect = referenceEl.getBoundingClientRect();
const popoverEl = document.getElementById('popover');

// 放在底部：
popoverEl.style.position = 'absolute';
popoverEl.style.top = `${window.scrollY + refRect.bottom}px`;  
popoverEl.style.left = `${window.scrollX + refRect.left + (refRect.width / 2) - (popoverEl.offsetWidth / 2)}px`;
```

优点: 完全代码控制，可以根据自己的需求完善代码。
缺点: 需要自己定义反转、碰撞检测和偏移逻辑。 

dom-align其本质则是通过JS来实现元素对齐，并添加了不同浏览器兼容，元素碰撞检测，翻转逻辑等。接下来让我们一起看看 `dom-align` 是如何实现元素对齐。

## dom-align 的实现：
源码中会涉及一些浏览器元素属性，在这里就不过多赘述了，如果想要了解的小伙伴也可以之前翻看我之前的文章：

#### 源码分析：
本篇文章基于 `dom-align@1.9.0`


在开始研究源码前，我们可以先来看一些其内部使用的工具函数：

**getClientPosition**: 返回元素相对于可见浏览器视口（即viewport）的位置（左侧和顶部）（如果html或body元素存在border则减去这个距离）
```javascript
function getClientPosition(elem) {
  let box;
  let x;
  let y;
  const doc = elem.ownerDocument;
  const body = doc.body;
  const docElem = doc && doc.documentElement; // 得到元素所在文档的 HTML 节点

  box = elem.getBoundingClientRect(); // 获取相对于viewport的坐标信息
  x = box.left;
  y = box.top;
  x -= docElem.clientLeft || body.clientLeft || 0;
  y -= docElem.clientTop || body.clientTop || 0;

  return {
    left: x,
    top: y,
  };
}
```

**getOffset**: 元素 (el) 相对于整个文档左上角的绝对位置
浏览器中的 `getBoundingClientRect()` 方法并不会
```javascript
function getOffset(el) {
  const pos = getClientPosition(el);
  const doc = el.ownerDocument;
  const w = doc.defaultView || doc.parentWindow;
  pos.left += getScrollLeft(w); // 包含X方向的滚动距离
  pos.top += getScrollTop(w); // 包含Y方向的滚动距离
  return pos; // 返回将滚动
}
```

**getRegion**: 它返回一个表示 DOM 中给定节点的边界“区域”的对象。对象中包括左侧、顶部、宽度和高度属性。
```javascript
function getRegion(node) {
  let offset;
  let w;
  let h;
  if (!utils.isWindow(node) && node.nodeType !== 9) {
    offset = utils.offset(node); // 调用 getClientPosition
    w = utils.outerWidth(node);
    h = utils.outerHeight(node);
  } else {
    const win = utils.getWindow(node);
    offset = {
      left: utils.getWindowScrollLeft(win), // 获取文档水平方向滚动的像素值
      top: utils.getWindowScrollTop(win), // 获取文档垂直方向滚动的像素值
    };
    w = utils.viewportWidth(win);
    h = utils.viewportHeight(win);
  }
  offset.width = w;
  offset.height = h;
  return offset;
}
```


**getVisibleRectForElement(target)**: 计算元素所在空间的可视区域,在计算过程中考虑到所有可滚动的祖先元素，窗口视口和固定定位。
```javascript
/**
 * 确保弹出窗口、下拉菜单或工具提示在可见边界内呈现。
 */
function getVisibleRectForElement(element) {
  const visibleRect = {
    left: 0,
    right: Infinity,
    top: 0,
    bottom: Infinity,
  };
  let el = getOffsetParent(element);
  const doc = utils.getDocument(element);
  const win = doc.defaultView || doc.parentWindow;
  const body = doc.body;
  const documentElement = doc.documentElement;

  // 1. 找到元素的最近祖先元素， 并更新可见区域的值
  while (el) {
    if (
      (navigator.userAgent.indexOf('MSIE') === -1 || el.clientWidth !== 0) &&
        (el !== body &&
         el !== documentElement &&
         utils.css(el, 'overflow') !== 'visible')
    ) {
      const pos = utils.offset(el);
      // add border
      pos.left += el.clientLeft;
      pos.top += el.clientTop;
      /**
       * top：其当前值与容器顶部边缘（包括边框）之间的最大值
       * right：其当前值与容器右边缘之间的最小值 (pos.left + el.clientWidth)。
       * left：其当前值与容器底部边缘之间的最小值 (pos.top + el.clientHeight)。
       *  bottom：成为其当前值和容器左边缘之间的最大值。
       */
      visibleRect.top = Math.max(visibleRect.top, pos.top);
      visibleRect.right = Math.min(visibleRect.right,
        // consider area without scrollBar
        pos.left + el.clientWidth);
      visibleRect.bottom = Math.min(visibleRect.bottom,
        pos.top + el.clientHeight);
      visibleRect.left = Math.max(visibleRect.left, pos.left);
    } else if (el === body || el === documentElement) {
      break;
    }
    el = getOffsetParent(el);
  }

  // 2. 如果有需要短暂设置元素的position属性为fixed
  let originalPosition = null;
  if (!utils.isWindow(element) && element.nodeType !== 9) {
    originalPosition = element.style.position;
    const position = utils.css(element, 'position');
    if (position === 'absolute') {
      element.style.position = 'fixed';
    }
  }

  // 3. 获取 viewport和document的空间信息
  // 滚动偏移量
  const scrollX = utils.getWindowScrollLeft(win);
  const scrollY = utils.getWindowScrollTop(win);
  // 浏览器窗口的可见宽度和高度
  const viewportWidth = utils.viewportWidth(win);
  const viewportHeight = utils.viewportHeight(win);
  // 文档的总可滚动宽度和高度。
  let documentWidth = documentElement.scrollWidth;
  let documentHeight = documentElement.scrollHeight;

  // 对于overflow属性调整文档宽度或高度
  const bodyStyle = window.getComputedStyle(body);
  if (bodyStyle.overflowX === 'hidden') {
    documentWidth = win.innerWidth;
  }
  if (bodyStyle.overflowY === 'hidden') {
    documentHeight = win.innerHeight;
  }

  // 4. 恢复元素属性设置
  if (element.style) {
    element.style.position = originalPosition;
  }

  // 5. 根据祖先的位置position属性进行不同的属性更新
  if (isAncestorFixed(element)) {
    // Clip by viewport's size.
    visibleRect.left = Math.max(visibleRect.left, scrollX);
    visibleRect.top = Math.max(visibleRect.top, scrollY);
    visibleRect.right = Math.min(visibleRect.right, scrollX + viewportWidth);
    visibleRect.bottom = Math.min(visibleRect.bottom, scrollY + viewportHeight);
  } else {
    // Clip by document's size.
    const maxVisibleWidth = Math.max(documentWidth, scrollX + viewportWidth);
    visibleRect.right = Math.min(visibleRect.right, maxVisibleWidth);

    const maxVisibleHeight = Math.max(documentHeight, scrollY + viewportHeight);
    visibleRect.bottom = Math.min(visibleRect.bottom, maxVisibleHeight);
  }

  // 6.检查可见区域是否合法
  return (
    visibleRect.top >= 0 &&
      visibleRect.left >= 0 &&
      visibleRect.bottom > visibleRect.top &&
      visibleRect.right > visibleRect.left
  ) ? visibleRect : null;
}
```


**isOutOfVisibleRect(target)**: 它检查给定元素（目标）是否完全位于可见矩形之外 - 可能是视口或可滚动容器的可见区域。
```javascript
function isOutOfVisibleRect(target) {
  const visibleRect = getVisibleRectForElement(target);
  const targetRegion = getRegion(target);

  return !visibleRect ||
    (targetRegion.left + targetRegion.width) <= visibleRect.left ||
    (targetRegion.top + targetRegion.height) <= visibleRect.top ||
    targetRegion.left >= visibleRect.right ||
    targetRegion.top >= visibleRect.bottom;
}
```

**getAlignOffset(region, align)**: 根据对齐字符串计算给定矩形区域内的点（左、上）。

**flipOffset(offset, index)**: 当原始一侧没有足够的空间，翻转偏移量。
```javascript
function flipOffset(offset, index) {
  offset[index] = -offset[index];
  return offset;
}
```

### doAlign 核心函数分析：
doAlign是dom-align的核心函数, 它计算 DOM 元素 (el) 相对于目标区域 (tgtRegion) 的放置（对齐）位置，考虑偏移、碰撞检测和潜在的“翻转”以将元素保持在可见区域内。

##### alignElement
`alignElement` 和 `alignPoint` 为两个 `dom-align` 暴露给用户层面的两个API， 其核心原理类型，本篇文章主要以 `alignElemnt` API为主。

简单概述下`alignElemnt` 与 `alignPoint` 的不同：
- alignElement是相对于元素来进行overlay元素的定位以及展示，而alignPoint则是根据传入的`(x, y)`的点坐标来进行定位。
- 实践场景：
  - 跟随鼠标的 `tooltip` 
  - 右键点击展示菜单

```javascript
function alignElement(el, refNode, align) {
  const target = align.target || refNode; // refNode
  const refNodeRegion = getRegion(target);

  const isTargetNotOutOfVisible = !isOutOfVisibleRect(target);

  return doAlign(el, refNodeRegion, align, isTargetNotOutOfVisible);
}
```

入参：
- el：待定位的元素(源节点)
- refNode：待定位元素的参考元素（目标元素）
- align: 配置信息：
  - points: 源节点
  - offset
  - targetOffset
  - overflow
  - useCssRight
  - useCssBottom
  - useCssTransform

流程分析：
1. 首先调用 `getRegion` 函数获取目标元素的区域信息
2. 判断相对于viewport 目标元素是否在可视区域内
3. 调用 `doAlign`


`doAlign(el, refNodeRegion, align, isTargetNotOutOfVisible);` :
- 入参分别为：
  - overlay元素
  - reference元素的左上角的点
  - align配置信息
  - 元素是否在可视区域内的布尔值


```javascript
function doAlign(el, tgtRegion, align, isTgtRegionVisible) {
  let points = align.points;
  let offset = align.offset || [0, 0];
  let targetOffset = align.targetOffset || [0, 0];
  let overflow = align.overflow;
  const source = align.source || el;
  offset = [].concat(offset);
  targetOffset = [].concat(targetOffset);
  overflow = overflow || {};
  const newOverflowCfg = {};
  let fail = 0;
  // 1. 计算待定位元素的可以放置的位置
  const visibleRect = getVisibleRectForElement(source);
  // 2. 获取待定位元素的盒子信息 { top, left, width, height }
  const elRegion = getRegion(source);

  // 3.标准化偏移量（支持百分比转换
  normalizeOffset(offset, elRegion);
  normalizeOffset(targetOffset, tgtRegion);

  // 4.初始化待定位元素出现的位置
  let elFuturePos = getElFuturePos(elRegion, tgtRegion, points, offset, targetOffset);
  let newElRegion = utils.merge(elRegion, elFuturePos);

  // 5.碰撞检测和翻转逻辑（如果元素必须保持可见）
  if (visibleRect && (overflow.adjustX || overflow.adjustY) && isTgtRegionVisible) {
    if (overflow.adjustX) {
      // 5a. 水平碰撞检测和以及可能的潜在翻转
      if (isFailX(elFuturePos, elRegion, visibleRect)) {
        const newPoints = flip(points, /[lr]/ig, {
          l: 'r',
          r: 'l',
        });
        const newOffset = flipOffset(offset, 0);
        const newTargetOffset = flipOffset(targetOffset, 0);
        const newElFuturePos = getElFuturePos(
          elRegion,
          tgtRegion,
          newPoints,
          newOffset,
          newTargetOffset
        );

        if (!isCompleteFailX(newElFuturePos, elRegion, visibleRect)) {
          fail = 1;
          points = newPoints;
          offset = newOffset;
          targetOffset = newTargetOffset;
        }
      }
    }

    if (overflow.adjustY) {
      // 5.b 垂直碰撞检测和潜在翻转
      if (isFailY(elFuturePos, elRegion, visibleRect)) {
        // 翻转对齐位置
        const newPoints = flip(points, /[tb]/ig, {
          t: 'b',
          b: 't',
        });
        // 偏移量翻转
        const newOffset = flipOffset(offset, 1);
        const newTargetOffset = flipOffset(targetOffset, 1);
        const newElFuturePos = getElFuturePos(
          elRegion,
          tgtRegion,
          newPoints,
          newOffset,
          newTargetOffset
        );

        if (!isCompleteFailY(newElFuturePos, elRegion, visibleRect)) {
          fail = 1;
          points = newPoints;
          offset = newOffset;
          targetOffset = newTargetOffset;
        }
      }
    }

    // 5c. 如果发生翻转，则使用新点/偏移重新计算位置
    if (fail) {
      elFuturePos = getElFuturePos(elRegion, tgtRegion, points, offset, targetOffset);
      utils.mix(newElRegion, elFuturePos);
    }

    // 5d. 测试翻转后是否在可视区域内
    const isStillFailX = isFailX(elFuturePos, elRegion, visibleRect);
    const isStillFailY = isFailY(elFuturePos, elRegion, visibleRect);

    // 检查翻转后水平或垂直是否仍然失败， 如果情况为true复原修改过的定位参数
    if (isStillFailX || isStillFailY) {
      points = align.points;
      offset = align.offset || [0, 0];
      targetOffset = align.targetOffset || [0, 0];
    }
    // 5e. 标记哪个轴需要调整
    newOverflowCfg.adjustX = overflow.adjustX && isStillFailX;
    newOverflowCfg.adjustY = overflow.adjustY && isStillFailY;

    // 5f. 如果需要，调整元素大小，使其适合visibleRect
    if (newOverflowCfg.adjustX || newOverflowCfg.adjustY) {
      newElRegion = adjustForViewport(elFuturePos, elRegion, visibleRect, newOverflowCfg);
    }
  }

  // 6.如果新区域的宽度/高度与原始区域不同，则更新 CSS 宽度/高度
  if (newElRegion.width !== elRegion.width) {
    utils.css(source, 'width', utils.width(source) + newElRegion.width - elRegion.width);
  }

  if (newElRegion.height !== elRegion.height) {
    utils.css(source, 'height', utils.height(source) + newElRegion.height - elRegion.height);
  }

  // 7. 最终定位：设置待初始化元素的left/top
  utils.offset(source, {
    left: newElRegion.left,
    top: newElRegion.top,
  }, {
    useCssRight: align.useCssRight,
    useCssBottom: align.useCssBottom,
    useCssTransform: align.useCssTransform,
    ignoreShake: align.ignoreShake,
  });

  return {
    points,
    offset,
    targetOffset,
    overflow: newOverflowCfg,
  };
}
```

##### alignElement 流程图


#### 总结
DomAlign帮我们处理了浏览器兼容，元素定位，碰撞检测以及翻转元素位置等问题。它也是antd下拉类型组件的基础依赖。如果本篇文章对您有帮助，也麻烦点赞和收藏，如果您对阅读源码感兴趣，也麻烦点下关注。之后也会带来更多关于源码阅读的文章。欢迎任何建议与意见～

# 一文了解清楚浏览器的位置属性

我们在开发过程中，会面临一些需要对DOM元素进行测量的场景：
- 放置tooltip
- 判断一个元素是否在浏览器页面可视区域内
- 元素重叠检测
- 滚动懒加载
- 拖拽功能
- 虚拟列表
本篇就让我们从顶向下掌握这类元素属性：

### window/document 相关：
- `window.screenTop/screenY`: 返回从用户浏览器上边界到物理屏幕最顶端的像素值。
- `window.screenLeft/screenX`: 返回用户浏览器左边框到物理屏幕左边边缘的像素值。
- `window.scrollX / pageXOffset`: 返回文档（document）水平方向滚动的像素值。
- `window.scrollY / pageYOffset`: 返回文档（docuemnt）垂直方向滚动的像素值。
- `document.documentElement.clientWidth`: 获取浏览器窗口宽度的像素值。
- `document.documentElement.clientHeight`: 获取浏览器窗口高度的像素值。
  - `window.innerWidth/innerHeight` vs. `document.documentElement.clientWidth/clientHeight`: 如果页面存在滚动条，并且滚动条占用像素，`clientWidth/clientHeight` 不包含滚动条的像素值， 而 `innerWidth/innerHeight` 则包含了滚动条的像素值。
- 如何找到包含滚动条的文档的宽高：我们通常需要在 `scroll, offset, client` 中取最大值：
```javascript
let scrollHeight = Math.max(
  document.body.scrollHeight, document.documentElement.scrollHeight,
  document.body.offsetHeight, document.documentElement.offsetHeight,
  document.body.clientHeight, document.documentElement.clientHeight
);
```

### 元素相关属性：


#### offset相关:
- `offsetParent`: 元素最近的被定位(除了`position: static`)的父级元素（如果没有则是 body）。
- `offsetTop`: 元素的上边框与其offsetParent元素上边框的偏移量。
- `offsetLeft`: 元素的左边框与其offsetParent元素左边框的偏移量。
- `offsetWidth`: 内容宽度 + padding宽度 + border宽度 + scrollbar宽度
- `offsetHeight`: 内容高度 + padding宽度 + border高度 + scrollbar高度

#### client相关：
- `clientTop`: 元素上边距的宽度。
- `clientLeft`: 元素左边距的宽度。
- `clientWidth`: 元素包含padding，不包括border，margin，**垂直**滚动条的宽度。`clientWidth = 元素width + 元素padding - 垂直滚动条width` 
- `clientHeight`: 元素包含padding，不包括border，margin，**水平**滚动条的宽度。 `clientHeight = 元素height + 元素padding - 水平滚动条height`

### scroll相关:
- `scrollWidth`: 元素内容宽度的一种度量，包括由于溢出而在屏幕上不可见的内容。（元素真实的宽度）
- `scrollHeight`: 元素内容高度的一种度量，包括由于溢出而在屏幕上不可见的内容。（元素真实的高度）
- `scrollTop`: 垂直方向滚动了多少像素值
- `scrollLeft`: 水平方向滚动了多少像素值


### 其他：
- `getBoundingClientRect()`: 返回一个对象，相对于视口的其元素的顶部、左侧、宽度和高度（而不是偏移父级）。
- `getComputedStyle` : 当我们需要获取元素几何信息时，我们不应该使用 `getComputedStyle` 来获取元素的CSS 高度和宽度，有两个考虑： 1.真实宽高基于`box-sizing`, 而获取CSS width和height属性时，可能获取到的并不是真实的宽高。2.当CSS中使用 auto属性时，CSS的width/height并反馈真实高度和宽度。

### 容易出错的地方：
- `offsetParent` : 返回的是 `position` 不为 `static` 的父级元素。
- `box-sizing: border-box` 不会改变 `offsetWidth` 和 `clientWidth` 的返回。
- 如果需要考虑到 `margin` 属性，应该使用 `getComputedStyle` 来进行测量。

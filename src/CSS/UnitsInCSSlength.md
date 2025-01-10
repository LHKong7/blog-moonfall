## CSS中的单位

### pixel简介
在开始了解CSS中的单位前，我们先来看一下像素（pixel）是什么意思，像素通常是指在照片或展示中的最小单位，每个像素可以是一个小方块或原点，当我们放大图像时，每个像素会变的更清晰，每个像素点包含了颜色（例如，RGB）和色度信息。而CSS中的px单位与我们所指的像素是两件事情。

### CSS单位简介：
在了解每个CSS单位前，我们可以根据使用输出媒介不同将单位分为绝对单位和相对单位，举例来说，当我们想让我们的页面对打印更友好时，我们应该考虑使用 `cm, mm` 等单位。

#### 绝对单位
- 定义：绝对单位通常指的是固定的物理测量单位，并不会随着展示工具或 `viewport` 的变动而变动。通常我们在打印或需要固定的大小时使用的样式。
- CSS中的绝对单位：
  - in（Inches）英寸，长度为2.54cm
  - cm（Centimeters）厘米
  - mm（Millimeters）毫米
  - pt（Points）点，通常用于印刷领域, `1pt = 1/72in`
  - pc（Picas）活字，通常用于印刷领域，`1pc = 12pt`

绝对单位中的转换关系: `1in = 2.54cm = 25.4mm = 72pt = 6pc`
在日常开发中，我们在大部分应该避免使用绝对单位，因为它的大小为实际物理大小，比如我们定义字体为 `12pt`， 无论在任何屏幕尺寸或分辨率的大小都为 `12pt`。这会导致字体在小屏幕预览时展现与预期不符。

#### 相对单位
- 定义：
- CSS中的相对单位：
  - 相对于字体大小的单位：
    - `em`（Em）：相对父元素字体大小的相对尺寸，通常可以在字体大小中使用，随着父元素的大小变化，设置为 `em` 的子元素大小也会变化。
    - `rem`（Rem）：相对根元素字体大小的相对尺寸
    - `ch`（Character Unit），数字0的宽度，需要注意的是不同的字体对`ch`的具体大小有着影响。在固定大小的字体中，`1ch = 一个字的宽` 在字体为不固定大小的情况时（例如，`Georgia`, 设置字体容器为 `20ch`时，仍会造成 `overflow`的情况发生。
    - `ic`: 相当于“水”这个字的宽度，目前还在实验阶段和`ch` 一样，具体的大小会收到不同字体的影响
  - 相对于父级元素的单位：
    -  `%` 百分比，相对于父元素的大小，比如父元素的宽为 `width: 400px` 子元素设置为 `width: 75%`, 那么子元素的宽为 `400*0.75 = 300px`。 
  - 相对于viewport的单位：
    - `vw` 相对于viewport的宽，`1vw` 为1% `viewport` 的宽
    - `vh` 相对于viewport的高 `1vh` 为1% `viewport` 的高
    - `vmin` 相对于viewport中宽高中更小的尺寸, `1vmin`为1% `viewport`中较小的值
    - `vmax` 相对于viewport中宽高中更大的尺寸, `1vmax`为1% `viewport`中较大的值

#### pixel
`px`（Pixels）: CSS中的px通常指的是相对于设备的大小，不同设备的**分辨率(resolution)**和**像素密度(pixel density)**有着不同的物理大小。通常大小为`1/96 inch ≈  0.0104167 inches` 。在打印机和高分辨率屏幕中，一个CSS像素会占多个设备像素。我们很难通过计算公式将`px`的具体物理尺寸在不同的设备中得出，更多内容可以[参考](https://stackoverflow.com/questions/21680629/getting-the-physical-screen-dimensions-dpi-pixel-density-in-chrome-on-androi)



References：
- https://www.w3.org/Style/Examples/007/units.en.html
- https://meyerweb.com/eric/thoughts/2018/06/28/what-is-the-css-ch-unit/
- https://web.dev/learn/css/sizing
- https://github.com/cognitom/paper-css

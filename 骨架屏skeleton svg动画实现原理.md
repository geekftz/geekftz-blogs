**1. 前言**

- 最近在弄骨架屏的需求，所以在研究业内主流的几个方案。
- 本篇文章基于[react-content-loader](https://github.com/danilowoz/react-content-loader)分析骨架屏 svg 动画的实现。

**2. 效果**

![QQ录屏20220506103828.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48f0d1a13b6a43efb4e89d315eb0c9b0~tplv-k3u1fbpfcp-watermark.image?)

**3. 代码**

```html
<!-- aria-labelledby属性标识了一个（或多个）元素，该元素标记了它所应用到的元素。这里指向内部title标签的id。 -->

<!-- viewBox属性的值是一个包含4个参数的列表 min-x, min-y, width and height， 以空格或者逗号分隔开， 在用户空间中指定一个矩形区域映射到给定的元素,查看属性preserveAspectRatio。 -->
<!-- min-x 视口横轴偏移量，min-y 视口纵轴偏移量 -->
<!-- width 视口宽度，height 视口高度 -->
<svg aria-labelledby="ml3udb-aria" viewBox="0 0 380 70">
  <!-- title 描述性字符串，该描述只能是纯文本，hover时做提示作用。 -->
  <title id="ml3udb-aria">Loading...</title>
  <!-- x、y相对定位偏移量，width、height宽高，这里rect为外层 -->

  <!-- clip-path CSS 属性使用裁剪方式创建元素的可显示区域。区域内的部分显示，区域外的隐藏。 -->
  <!-- 用 <url> 表示剪切元素的路径 -->

  <!-- style元素元素样式表直接在SVG内容中间嵌入。 -->
  <!-- 这里fill: url指向了id一致的linearGradient渐变标签的id -->
  <rect
    x="0"
    y="0"
    width="100%"
    height="100%"
    clip-path="url(#ml3udb-diff)"
    style="fill: url('#ml3udb-animated-diff')"
  ></rect>
  <!-- 把所有需要再次使用的引用元素定义在defs元素里面。 -->
  <!-- 这里主要是针对clip-path和style的引用 -->
  <defs>
    <clipPath id="ml3udb-diff">
      <!-- 3个小块，1正方形+2长条 -->
      <rect x="0" y="0" rx="5" ry="5" width="70" height="70"></rect>
      <rect x="80" y="17" rx="4" ry="4" width="300" height="20"></rect>
      <rect x="80" y="50" rx="3" ry="3" width="150" height="20"></rect>
    </clipPath>

    <!-- linearGradient元素用来定义线性渐变，用于图形元素的填充或描边。 -->
    <linearGradient id="ml3udb-animated-diff">
      <!-- 一个渐变上的颜色坡度，是用stop元素定义的。 -->

      <!-- offset属性用于表示该颜色位于渐变的什么位置上。这里指的是offset为整个外层元素横轴0%的地方的基点颜色 -->
      <!-- stop-color属性指示在渐变停止时使用的颜色。 -->
      <!-- stop-opacity属性定义给定颜色渐变停止的不透明度。 -->
      <stop offset="0%" stop-color="#f5f6f7" stop-opacity="1">
        <!-- animate动画元素放在形状元素的内部，用来定义一个元素的某个属性如何踩着时点改变。 -->

        <!-- 重点！这里values和keytimes是重点部分，先略过，第4点分析处会单独说明-->
        <!-- dur，即duration，动画时长 -->
        <!-- repeatCount属性表示动画将发生的次数。 -->
        <animate
          attributeName="offset"
          values="-2; -2; 1"
          keyTimes="0; 0.25; 1"
          dur="4s"
          repeatCount="indefinite"
        ></animate>
      </stop>
      <!-- 这里指的是offset为整个外层元素横轴50%的地方的基点颜色 -->
      <stop offset="50%" stop-color="red" stop-opacity="1">
        <animate
          attributeName="offset"
          values="-1; -1; 2"
          keyTimes="0; 0.25; 1"
          dur="4s"
          repeatCount="indefinite"
        ></animate>
      </stop>
      <!-- 这里指的是offset为整个外层元素横轴100%的地方（即终点）的基点颜色 -->
      <stop offset="100%" stop-color="#f5f6f7" stop-opacity="1">
        <animate
          attributeName="offset"
          values="0; 0; 3"
          keyTimes="0; 0.25; 1"
          dur="4s"
          repeatCount="indefinite"
        ></animate>
      </stop>
    </linearGradient>
  </defs>
</svg>
```

**4. 分析**  
这里，我们大致看下来，这个 svg 动画效果，由这几部分组成：svg 元素+视口盒子、title 声明/提示、rect 父级元素（包含了 defs 定义的 clip-path 下的 3 个子级元素）、linearGradient 渐变元素（包含 3 个 stop 坡度元素和内部的 animate 动画元素）。那么它是怎么实现动画效果的呢？

这里我们`从基点角度出发，从时间的维度去对比`：

- values 内部值的单位是 100%的外层元素宽度，即父级元素宽度。后续统一称为**基点偏移相对值**。
- keytimes 内部值的单位是整个动画时长的百分比例，满值为 1。后续统一称为**基点时间相对值**。
- values 和 keytimes 彼此对应，比如第一个 stop 基点元素，在 0s（时间系数 ratio \* 动画总时长 duration）时，value 为-2，指的是相对于 svg viewBox 起止点（左上角 x = 0, y = 0）横轴向左偏移了-200%的父级宽度。

| 基点元素 | values    | keytimes   |
| -------- | --------- | ---------- |
| stop1    | -2; -2; 1 | 0; 0.25; 1 |
| stop2    | -1; -1; 2 | 0; 0.25; 1 |
| stop3    | 0; 0; 3   | 0; 0.25; 1 |

- 基点时间相对值为 0 的情况下，stop1 到 stop2 的距离是|-2 - 0| = 2，基点时间相对值为 0.25 的情况下，stop1 到 stop2 的距离是|-2 - 0| = 2，基点时间占比为 1 的情况下，stop1 到 stop2 的距离是|1 - 3| = 2。这就说明整个 linearGradient 渐变元素的宽度是 2 个父级元素的宽度。
- 基点时间相对值从 0 到 0.25 的过程，这个 linearGradient 渐变元素没有动，短暂的停止了，如果动画总时长是 4s，那这个暂停时长就是 1s，如果动画总时长是 1s，那这个暂停时间就是 0.25s。
- 基点时间相对值从 0.25 到 1 的过程中，linearGradient 渐变元素从父级元素最左边向右边开始移动（你也可以理解它是当前的节点自己在模拟变化，具体的本篇文章不做扩展），最终的情况是 stop3 作为 offset 为 100%，距离 svg 父级元素的起点向右偏移了 3 \* 100%的距离。

我画个图来形容一下这块的整个过程：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5857ca9f01f34ef882bbf4edb5973d6d~tplv-k3u1fbpfcp-watermark.image?)

**5. 扩展**  
正常情况下，设计师通过 AE、Principle 这类动画制作软件导出的 json 文件里面数值都是具体的，svg 动画实现到 web 端的细节参数是很多的，一个完整的动画，如果存在外层宽高比例变化的情况，哪怕是一个像素的差异，都会导致很多 svg 参数的改变，所以因此 svg 提供 [**preserveAspectRatio**](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/preserveAspectRatio) 这么一个参数去让我们“兼容”。

那这个属性是干嘛用的呢？官方描述是很抽象的，简单来说，可以这么理解：  
它是一个调整元素在 2 维画布中，位置、宽度、高度自适应缩放的一个自定义设置，可以让你的 svg 元素，在外层容器宽高比例出现变化的时候，做出定义的适配变化，也就是说，如果外层和最初的这个比例基准不对等的时候，我们让它怎么处理。

**6. 结束语**  
如果觉得写的不错，有用的话，还请帮我点个赞 O(∩_∩)O  
转发请注明出处，谢谢！

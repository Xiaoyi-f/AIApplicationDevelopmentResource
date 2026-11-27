---
title: "ECharts 从入门到够用（柱状/折线/饼图 + Vue3 实战一次讲透）"
status: 已发布
author: XiaoYi
---

# ECharts 从入门到够用（柱状/折线/饼图 + Vue3 实战一次讲透）

> 数据可视化是前端最「出效果」的技能——一堆冷冰冰的数字，一张图就活了。而 **ECharts** 是国内最主流的图表库，功能全、生态好、上手快。 这一篇不讲花里胡哨的，就讲你**日常够用**的：柱状图、折线图、饼图，配坐标轴、提示框、图例，最后直接封进 Vue3 里用。学完你就能给任何项目配上图表。
>
> Let's use AI to create a better world。

## 一、ECharts 是啥 + 为什么按需引入

**ECharts** 是一个开源的数据可视化图表库，由 Apache 基金会维护，用 JavaScript 写。柱状图、折线图、饼图、散点图、雷达图、地图……几乎你能想到的图它都有，而且是**一个配置对象搞定一切**。

引入方式有两种：

| 方式 | 写法 | 体积 | 适用 |
| --- | --- | --- | --- |
| 全量引入 | `import * as echarts from 'echarts'` | 大（几个 MB） | 原型、简单 demo |
| **按需引入** | `echarts/core` + `echarts.use([...])` | 小（只打包用到的） | **生产环境**（推荐） |

> **核心价值**：按需引入（tree-shaking）只打包你用到的图表和组件，能显著减小包体积。生产项目一定要用按需引入。

---

## 二、环境 + 按需引入

**安装：**

```
  npm install echarts
```

**按需引入：**

```
  import * as echarts from 'echarts/core'
  import { BarChart, LineChart, PieChart } from 'echarts/charts'
  import { CanvasRenderer } from 'echarts/renderers'
  import {
      GridComponent,
      TooltipComponent,
      LegendComponent,
      TitleComponent
  } from 'echarts/components'
  ​
  echarts.use([
      BarChart, LineChart, PieChart,      // 图表
      GridComponent, TooltipComponent,    // 组件
      LegendComponent, TitleComponent,
      CanvasRenderer                      // 渲染器
  ])
```

> **规则**：用了什么图表/组件，就必须 `use` 注册什么，漏注册会报错（比如用了 tooltip 却没注册 TooltipComponent）。

---

## 三、核心概念（一张表）

| 概念 | 一句话理解 |
| --- | --- |
| `option` | 配置对象，所有图表的「说明书」 |
| `xAxis` / `yAxis` | 坐标轴（X 轴 / Y 轴） |
| `series` | 数据系列，真正的「数据 + 图表类型」 |
| `grid` | 直角坐标系的容器（位置、大小） |
| `tooltip` | 悬浮提示框 |
| `legend` | 图例 |
| `echarts.init(dom)` | 初始化，绑定到一个 DOM 元素 |
| `setOption()` | 设置 / 更新配置 |
| `resize()` | 容器尺寸变化时调用 |
| `dispose()` | 销毁实例，释放内存 |

> 记忆：**一个 DOM 容器 + 一个 option 配置 = 一张图**。核心流程就是 `init → setOption → resize/dispose`。

---

## 四、一个最小可跑的例子

```
  <!DOCTYPE html>
  <html>
  <head>
      <meta charset="UTF-8">
  </head>
  <body>
      <!-- 容器必须有明确的宽高 -->
      <div id="chart" style="width: 600px; height: 400px;"></div>
  ​
      <script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
      <script>
          // 1. 初始化：绑定到 DOM
          const chart = echarts.init(document.getElementById('chart'));
  ​
          // 2. 配置
          const option = {
              xAxis: { type: 'category', data: ['周一', '周二', '周三'] },
              yAxis: { type: 'value' },
              series: [{ type: 'bar', data: [120, 200, 150] }]
          };
  ​
          // 3. 渲染
          chart.setOption(option);
  ​
          // 4. 容器变化时自适应
          window.addEventListener('resize', () => chart.resize());
      </script>
  </body>
  </html>
```

这就是 ECharts 的全部骨架。下面各种图，**只是 `series` 和 `xAxis/yAxis` 配置不同**，其余一模一样。

---

## 五、柱状图（堆叠 / 横向 / 样式）

### 5.1 基础柱状图

```
  series: [{
      type: 'bar',
      data: [120, 200, 150, 80, 70]
  }]
```

### 5.2 堆叠柱状图

**关键**：多个系列的 `stack` 值写成**同一个字符串**，它们就会堆叠到一起。

```
  series: [
      { type: 'bar', stack: 'x', data: [120, 200, 150], name: '基础' },
      { type: 'bar', stack: 'x', data: [80, 70, 60],  name: '堆叠' }
  ]
```

`stack: 'x'` 这个 `x` 是你**自己起的组名**，只要两个系列写的是一样的值，就堆叠。想不堆叠，换不同的名字或去掉即可。

### 5.3 横向柱状图（⚠️ 你笔记里这里有个误区）

> 你笔记里「横向柱状图」写的是加 `barWidth: 20`，那是**柱宽**，跟横向没关系。**真正的横向柱状图 = 把 X/Y 轴类型对调**：类别放 yAxis，数值放 xAxis。

```
  // 横向柱状图：yAxis 是类别，xAxis 是数值
  yAxis: {
      type: 'category',
      data: ['苹果', '香蕉', '橙子']
  },
  xAxis: {
      type: 'value'
  },
  series: [{
      type: 'bar',
      data: [120, 200, 150],
      barWidth: 20          // 柱宽（可选）
  }]
```

### 5.4 柱体样式

```
  series: [{
      type: 'bar',
      data: [120, 200, 150],
      barWidth: 20,
      itemStyle: {
          color: 'red',          // 柱体颜色
          borderRadius: 5        // 圆角
      },
      label: {
          show: true,            // 显示数值标签
          position: 'top',       // 标签在柱顶
          color: 'green',
          fontSize: 12
      }
  }]
```

---

## 六、折线图

```
  series: [{
      type: 'line',
      data: [120, 200, 150, 80, 70],
      smooth: true,              // 平滑曲线（false 就是直线折线）
      lineStyle: {
          width: 2,              // 线宽
          color: 'red'           // 线颜色
      }
  }]
```

要点：

- `smooth: true` → 平滑曲线；`false` → 折线。
- `lineStyle` 控制线的样式。

---

## 七、饼图

```
  series: [{
      type: 'pie',
      center: ['50%', '50%'],    // 圆心位置（默认居中）
      data: [
          { value: 1, name: 'A' },
          { value: 2, name: 'B' },
          { value: 3, name: 'C' }
      ],
      label: {
          show: true
      }
  }]
```

要点：

- 饼图的数据每一项是 `{ value, name }` 对象，不是纯数字。
- `center` 是圆心位置，可用百分比或像素。
- 想显示百分比，label 的 formatter 用 `'{b}: {d}%'`（`{b}` 名称、`{d}` 百分比）。

---

## 八、坐标轴配置（xAxis / yAxis）

```
  xAxis: {
      type: 'category',          // 类目轴（离散，如月份、名称）
      data: ['1月', '2月', '3月'],
      name: 'X轴',               // 轴名称
      axisLine: { show: true },  // 轴线
      axisTick: { show: true },  // 刻度线
      axisLabel: { rotate: 45 }, // 标签旋转 45°（标签太长时用）
      boundaryGap: true          // 类目轴两端留白（柱状图要 true，折线图常 false）
  },
  yAxis: {
      type: 'value',             // 数值轴（连续）
      name: 'Y轴',
      min: 0,                    // 最小值
      max: 100,                  // 最大值
      splitLine: { show: true }, // 网格线（背景横线）
      axisLine: { show: true },
      axisTick: { show: true }
  }
```

**两种轴类型对比：**

| 属性 | `category`（类目轴） | `value`（数值轴） |
| --- | --- | --- |
| 数据 | 离散（名称/月份） | 连续（数字） |
| 需要 `data` | ✅ 需要 | ❌ 不需要 |
| 典型用途 | 柱状图 X 轴、折线图 X 轴 | Y 轴 |

> **boundaryGap 坑**：柱状图 X 轴两端要留白（`boundaryGap: true`），柱子才不会「贴边」；折线图常常要 `boundaryGap: false`，让折线顶到两端。

---

## 九、tooltip / legend / grid 三大辅助

### tooltip（悬浮提示）

```
  tooltip: {
      trigger: 'item',           // 'item' 数据项触发 / 'axis' 坐标轴触发
      axisPointer: { type: 'shadow' },  // 柱状图用 shadow，折线图用 line
      formatter: (params) => {
          // trigger: 'axis' 时 params 是数组；'item' 时 params 是单个对象
          const name = params.name        // x 轴类目名
          const value = params.value
          const seriesName = params.seriesName
          return `系列: ${seriesName}<br>类型: ${name}<br>值: ${value}`
      }
  }
```

> ⚠️ **关键坑**（你笔记里的）：`trigger: 'axis'` 时 `params` 是**数组**（多个系列各一项），要用 `params[0].name`；`trigger: 'item'` 时 `params` 是**单个对象**，直接用 `params.name`。两者别混用。

> 💡 `params.formatRes`：这是 ECharts **5.3+** 提供的内置默认格式化函数，`formatter: (params) => params.formatRes` 能拿到默认格式化结果。但不是所有版本都有，**手写 formatter 返回字符串**最通用、最好懂。

### legend（图例）

```
  legend: {
      data: ['基础', '堆叠'],   // 对应 series 里的 name
      top: 10,
      left: 'center',
      orient: 'horizontal',    // horizontal 水平 / vertical 垂直
      icon: 'circle'           // 图例图标形状
  }
```

### grid（布局容器）

```
  grid: {
      top: 10,
      left: 10,
      right: 10,
      bottom: 10,
      containLabel: true        // 包含坐标轴标签，避免被裁剪
  }
```

> **containLabel: true** 很重要——加上后坐标轴文字不会被 `top/left` 挤掉或裁剪。

---

## 十、生命周期（init / setOption / resize / dispose）

```
  const chart = echarts.init(domElement)   // 1. 初始化（DOM 必须有宽高）
  chart.setOption(option)                  // 2. 渲染 / 更新
  chart.resize()                           // 3. 容器尺寸变了，手动调
  chart.dispose()                          // 4. 销毁实例，释放内存
```

**两个必踩的坑：**

1. **容器没有宽高** → 图表不显示。容器（div）必须提前设定 `width/height`。
2. **忘了 `dispose`** → 内存泄漏。组件卸载时一定要销毁实例，否则反复进入页面会越用越卡。

---

## 十一、Vue3 实战封装

你是 Vue3 技术栈，这里直接给一个**可复用的封装组件**，覆盖 `onMounted` 初始化、`resize` 自适应、`onBeforeUnmount` 销毁。

```
  <template>
      <div ref="chartRef" :style="{ width: '100%', height: '400px' }"></div>
  </template>
  ​
  <script setup>
  import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
  import * as echarts from 'echarts/core'
  import { BarChart, LineChart, PieChart } from 'echarts/charts'
  import { CanvasRenderer } from 'echarts/renderers'
  import { GridComponent, TooltipComponent, LegendComponent, TitleComponent } from 'echarts/components'
  ​
  echarts.use([
      BarChart, LineChart, PieChart,
      GridComponent, TooltipComponent, LegendComponent, TitleComponent,
      CanvasRenderer
  ])
  ​
  const props = defineProps({
      option: { type: Object, required: true }
  })
  ​
  const chartRef = ref(null)
  let chart = null
  ​
  const render = () => {
      if (!chart) return
      chart.setOption(props.option)
  }
  ​
  onMounted(() => {
      chart = echarts.init(chartRef.value)   // 初始化
      render()
      window.addEventListener('resize', () => chart.resize())  // 自适应
  })
  ​
  // option 变了自动更新
  watch(() => props.option, render, { deep: true })
  ​
  // 卸载时销毁，防止内存泄漏
  onBeforeUnmount(() => {
      window.removeEventListener('resize', () => chart.resize())
      chart.dispose()
      chart = null
  })
  </script>
```

**父组件用法：**

```
  <template>
      <BaseChart :option="option" />
  </template>
  ​
  <script setup>
  import { ref } from 'vue'
  import BaseChart from './BaseChart.vue'
  ​
  const option = ref({
      xAxis: { type: 'category', data: ['周一', '周二', '周三'] },
      yAxis: { type: 'value' },
      series: [{ type: 'bar', data: [120, 200, 150] }]
  })
  </script>
```

> 这套封装的三个关键点：`onMounted` 里 init（DOM 已挂载）、`watch` 监听 option 变化自动重绘、`onBeforeUnmount` 里 dispose 防泄漏。直接抄走就能用。

---

## 常见坑速查表

| 坑 | 原因 | 解决 |
| --- | --- | --- |
| 图表不显示 | 容器没有宽高 | 给 div 设 `width/height` |
| 报「component not exists」 | 按需引入没注册组件 | `echarts.use([...])` 补注册 |
| tooltip 取不到值 | `item` 触发用了 `params[0]` | item 用 `params`，axis 才用 `params[0]` |
| 横向柱状图没效果 | 以为加 `barWidth` 就能横 | xAxis/yAxis 类型对调 |
| 柱状图柱子贴边 | X 轴没留白 | `xAxis.boundaryGap: true` |
| 坐标轴文字被裁 | grid 没包含 label | `grid.containLabel: true` |
| 页面越用越卡 | 忘了 dispose | `onBeforeUnmount` 里 `dispose()` |
| 窗口缩放图表不变形 | 没监听 resize | `window.addEventListener('resize', chart.resize)` |

---
title: "UniApp 从入门到生产部署（一套代码跑遍小程序 / App / H5）"
status: 已发布
author: XiaoYi
---

# UniApp 从入门到生产部署（一套代码跑遍小程序 / App / H5）

> 上一篇把 Vue3 讲透了，这篇进 **UniApp**——用你刚学的 Vue3 语法，一套代码同时跑微信小程序、App、H5 网页。 从环境搭建、项目结构、页面配置、生命周期，一路讲到打包发布和生产避坑。 Let's use AI to create a better world。

## 一、UniApp 是啥？

一句话：**DCloud 出品的、基于 Vue 的国产跨端框架——一套代码，到处运行（手机端为主）。**

> ```
> 你只写一套 Vue 代码
>         │
>         ├──→ 微信小程序
>         ├──→ App（安卓 / iOS）
>         └──→ H5 网页
> ```

|  |  |
| --- | --- |
| 概念 | 说明 |
| 出品方 | DCloud（国产） |
| 底层 | Vue3 + Vite（uni-app v3） |
| 编辑器 | HBuilderX（DCloud 出品，可视化运行调试） |
| 核心价值 | 写一遍，多端跑，不用分别学小程序/App/H5 三套语法 |

> 你技术栈里「Vue3 + UniApp 做小程序」就是这个意思——学完 Vue3，UniApp 几乎零门槛。

---

## 二、开发环境准备

### 1. 编辑器 + 微信工具

- **HBuilderX**：DCloud 出品，创建、运行、调试、打包一站式

- **微信开发者工具**：微信小程序开发必会，需要小程序 ID 和小程序密钥（AppSecret）

### 2. 打通 HBuilderX → 微信开发者工具

1. HBuilderX 菜单「设置 → 运行配置」，配置微信开发者工具路径

1. 微信开发者工具「设置 → 安全设置」里，**开启服务端口**

1. UniApp 项目在 HBuilderX 里「运行 → 运行到小程序模拟器」

### 3. 更新机制（两个概念）

|  |  |
| --- | --- |
| 更新 | 说明 |
| 热更新 | 小程序端，改代码即时生效，无需重新编译 |
| 冷更新 | 重新编译 / 重新启动，全量加载 |

---

## 三、项目结构

一个 UniApp 项目的核心文件：

|  |  |
| --- | --- |
| 文件 | 作用 |
| `App.vue` | 页面路由文件（应用的「根」，全局生命周期写这里） |
| `main.js` | 项目入口文件 |
| `manifest.json` | 应用配置文件（应用名称、图标、权限、AppID、各端配置） |
| `pages.json` | 全局配置文件（页面注册、导航栏、tabBar、全局样式） |

> **一个 `.vue` 文件就是一个「单文件组件」**——模板、脚本、样式都装在一个文件里，和 Vue3 完全一致。

---

## 四、pages.json 全局配置（重点）

### 1. 页面级 style 常见配置项

```
{
  "pages": [
    {
      "path": "pages/xxx/index",
      "style": {
        "navigationBarTextStyle": "black",      // 导航栏文字颜色 black/white
        "navigationBarTitleText": "页面标题",
        "navigationBarBackgroundColor": "#ddd", // 导航栏背景色
        "navigationStyle": "default",           // default 默认导航栏 / custom 自定义
        "disableScroll": false,                 // 是否禁止页面滚动
        "backgroundColor": "#ddd",              // 页面背景色
        "backgroundTextStyle": "dark",          // 下拉 loading 样式 dark/light
        "enablePullDownRefresh": false,         // 是否开启下拉刷新
        "onReachBottomDistance": 50,            // 触底距离（px），触发上拉加载
        "pageOrientation": "auto",              // 屏幕方向 auto/portrait/landscape
        "usingComponents": {                    // 手动注册组件
          "component-name": "component-path"
        }
      }
    }
  ]
}
```

### 2. 完整形式（pages + tabBar + globalStyle）

```
{
  "pages": [
    { "path": "pages/xxx/index", "style": {} }
  ],
  "tabBar": {
    "backgroundColor": "#fafafa",              // tabBar 背景色
    "list": [
      {
        "pagePath": "path",                    // 页面路径
        "text": "首页",                        // 显示文字
        "iconPath": "iconPath",                // 未选中图标
        "selectedIconPath": "selectedIconPath" // 选中图标
      }
    ]
  },
  "globalStyle": {}                            // 全局样式（作用于所有页面）
}
```

> ⚠️ 优先级：**页面针对性配置（pages 里的 style）> globalStyle**。同一项，页面级覆盖全局。

---

## 五、UniApp 特殊语法（对比 HTML）

小程序端没有 HTML 标签，用 UniApp 的组件替代：

|  |  |  |
| --- | --- | --- |
| HTML | UniApp | 说明 |
| `<div>` | `<view>` | 块级容器，`hover-class` 属性可设按压样式 |
| `<span>` | `<text>` | 文本，`selectable` / `user-select` 控制能否选中复制 |
| `<img>` | `<image>` | 图片，`mode` 控制裁剪缩放 |
| `<a>` | `<navigator>` | 链接，`open-type` 控制跳转方式 |
| `<select>` | `<picker>` | 选择器 |
| 单选组 | `<radio-group>` | 单选按钮组 |

```
<view hover-class="active">等价 div</view>

<text selectable user-select="true">等价 span，可选中</text>

<image src="path" mode="aspectFit"></image>

<navigator url="path" open-type="navigate">等价 a</navigator>

<picker :value="val" :range="options" @change="onChange"></picker>

<radio-group @change="radioChange">
  <label v-for="(item, index) in items" :key="item.name">
    <radio :value="item.name" :checked="item.checked">{{ item.value }}</radio>
  </label>
</radio-group>

// 单选数据
const items = [
  { name: "name", value: "value" },
]

function radioChange(e) {
  console.log(e.detail.value)   // 选中的 name
}
```

---

## 六、样式

### 1. 引入外部样式

```
<style>
@import url("绝对路径");
@import "相对路径";
</style>
```

### 2. 全局 vs 局部

```
<style>
/* 局部样式：组件内写，只作用于本组件 */
</style>
```

> ⚠️ **全局样式和局部样式冲突时，局部样式覆盖对应的全局片段**（就近原则）。

### 3. 响应式单位 rpx（生产必会）

- **750rpx = 屏幕宽度**（任何手机都一样）

- 设计稿如果是 750px 宽，1px = 1rpx，直接照抄不用换算

```
.box {
  width: 750rpx;   /* 永远等于屏幕宽度，自动适配所有机型 */
}
```

---

## 七、生命周期

### 1. 应用生命周期（写在 App.vue）

|  |  |  |
| --- | --- | --- |
| 钩子 | 触发时机 | 用途 |
| `onLaunch` | 应用初始化完成，**全局只触发一次** | 查用户是否授权、获取设备信息 |
| `onShow` | 启动 / 从后台进前台（可多次） | 消息提示、数据刷新 |
| `onHide` | 从前台进后台 | 退出应用时的提示 |
| `onError` | 应用报错 | 监测并处理错误 |

```
export default {
  onLaunch() {
    // 应用初始化，只跑一次
  },
  onShow() {
    // 每次回到前台都触发
  },
  onHide() {
    // 切到后台
  },
  onError(err) {
    // 捕获全局报错
  },
}
```

> 应用什么时候被**真正销毁**？当进入后台超过一定时间、或系统资源占用过高，应用才会被真正销毁，再次打开就是重新初始化启动。

### 2. 页面生命周期

```
export default {
  // options 是页面跳转时 URL 参数组装成的对象
  onLoad(options) {
    console.log(options.key);   // 接收 ?key=value
  },

  // 用户下拉时触发（需在 pages.json 开启 enablePullDownRefresh: true）
  onPullDownRefresh() {
    // 刷新数据...
    setTimeout(() => {
      uni.stopPullDownRefresh()   // 停止下拉刷新动画，必须调
    }, 1000)
  },
}
```

---

## 八、页面导航（五大跳转）

|  |  |  |  |
| --- | --- | --- | --- |
| API | 说明 | 能否跳 tabBar | 能否带参 |
| `uni.navigateTo({url})` | 新页面入栈，可返回（栈最多 10 层） | ❌ | ✅ |
| `uni.redirectTo({url})` | 替换当前页，旧页销毁，返回键回不去 | ❌ | ✅ |
| `uni.reLaunch({url})` | 重启，清空页面栈，返回键直接退出 | ✅ | ✅ |
| `uni.switchTab({url})` | 只能跳 tabBar 页面 | ✅ | ❌ |
| `uni.navigateBack({delta})` | 返回上 N 页，delta 是返回层数 | - | - |

```
uni.navigateTo({ url: "pages/detail/index?id=1" });
uni.redirectTo({ url: "pages/order/index" });
uni.reLaunch({ url: "pages/index/index?key=value" });
uni.switchTab({ url: "pages/home/index" });      // 不带参
uni.navigateBack({ delta: 1 });                  // 返回上一页
```

> 记忆：**navigateTo 入栈、redirectTo 替换、reLaunch 重启、switchTab 切 tabBar、navigateBack 后退。**

---

## 九、easycom 组件自动引入

easycom 是 UniApp 内置的「组件自动扫描、自动引入、自动注册」机制——**不用手动 import，直接用**。

### 默认规则

- 组件放在 `components` 目录下

- 组件名用小写短横线规范（如 `my-header`）

```
// pages.json 开启自动扫描
{
  "easycom": {
    "autoscan": true
  }
}
```

### 自定义规则

```
// 用正则匹配任意路径的组件，比如 uni-ui 组件库
{
  "easycom": {
    "custom": {
      "^uni-(.*)": "@/uni-ui/components/uni-$1/uni-$1.vue"
    }
  }
}
```

> 这样 `<uni-badge>` 之类的组件，直接写标签就能用，不用 import。

---

## 十、创建项目（CLI 方式）

除了 HBuilderX 可视化创建，也能用命令行（Vue3 + Vite 模板）：

```
pnpm degit dcloudio/uni-preset-vue#vite my-app
cd my-app
pnpm install
```

> `#vite` 就是 Vue3 + Vite 版本的模板。

---

## 十一、打包发布

### 1. App（软件）

1. 登录 HBuilderX 账号
2. `manifest.json` 里获取 AppID
3. 菜单「发行 → 云打包」（或其他方式）
4. 选「公共测试证书」快速出包

### 2. 微信小程序

1. HBuilderX 运行到小程序开发工具
2. 在微信开发者工具里「上传」，填版本号和项目备注
3. 到微信公众平台的小程序管理页「提交审核」
4. 审核通过后发布

### 3. H5

1. `manifest.json` 的 web 配置里，设置运行基础路径为 `./`
2. 发行 H5，把生成的静态文件部署到服务器（Nginx）

---

## 十二、组件库与资源推荐（uni-app 适配版）

UniApp 要编译到小程序和 App，这两端**没有浏览器 DOM，SVG / WebGL 也受限**——所以 Web 端很多库（靠操作 DOM、画 SVG 的）在这里**直接失效**。选库前先记住一句判断口诀：

> **这个库要不要操作 DOM / SVG / WebGL？要 → 基本只支持 H5；不用 → 小程序端大概率有替代品。**

### 1. 组件库（小程序 / App 必用）

|  |  |
| --- | --- |
| 组件库 | 说明 |
| **uni-ui** | DCloud 官方组件库，配合 easycom 自动引入（上节讲过），最稳 |
| **uview-plus** | 社区最流行，组件全、颜值高，Vue3 版 |
| **Wot Design Uni** | 高颜值新秀，文档友好 |
| **Vant Weapp** | Vant 的小程序原生版，通过 easycom 引入 |

> 一句话选型：**求稳用 uni-ui，追颜值和组件全用 uview-plus。** Vant（H5 版）、Element Plus 是 Web/PC 组件库，小程序里用不了。

### 2. CSS 原子化

|  |  |
| --- | --- |
| Web 端 | uni-app 替代 |
| TailwindCSS | **UnoCSS + `@uni-helper/unocss-preset-uni`** |

> 原版 Tailwind 在小程序端会「选择性失效」（`p-4` 生效、`bg-blue-300` 失效），因为 `@layer`、CSS 变量 `--tw-*`、`*` 通配符小程序都不支持。用 UnoCSS 就没这问题：**类名跟 Tailwind 一模一样，但构建时静态提取、只生成用到的类，天然适配小程序**，还内置 rpx 转换。

### 3. 图表 / Markdown / 图标 / 动画对照表

|  |  |  |
| --- | --- | --- |
| 用途 | Web 端推荐 | uni-app（小程序 / App）替代 |
| 数据图表 | ECharts | uCharts（qiun-data-charts）、lime-echart |
| Markdown 渲染 | md-editor-v3 | mp-html、towxml（只能渲染，编辑器需自研） |
| 流程图 | Mermaid | 后端渲染成 SVG / 图片，前端 mp-html 展示 |
| 图标库 | Iconfont、IconPark | uni-icons + 阿里 iconfont（字体图标，全端可用） |
| 动画 | Anime.js、Lottie | lottie-miniprogram（微信官方 Lottie 小程序版） |
| 3D | Three.js | 基本只支持 H5；小程序用 webgl canvas（复杂） |
| 滚动动画 | ScrollReveal | onPageScroll 监听 + CSS animation |
| 打字动画 | Typed.js | JS 定时器 + text 组件（setInterval 逐字截取） |
| 页面过渡 | Barba.js | pages.json 配 `animationType`（uni-app 自带） |
| 动效 | Mo.js | CSS animation + lottie-miniprogram |
| 静态资源网站 | 爱给网 aigei.com | 爱给网（通用，素材两端都能用） |

> 记两个最高频的：**组件库用 uni-ui / uview-plus，图表用 uCharts**——这俩覆盖 uni-app 大多数项目，其余用到再查。

---

## 十三、生产开发避坑清单（必看）

|  |  |
| --- | --- |
| 坑 | 说明 |
| **无 DOM 对象** | App / 小程序端**没有 DOM**，不支持 `@resize`、`@mouseover` 等鼠标/窗口事件 |
| **点击用 `@tap`** | 移动端用 `@tap` 替代 `@click`（不是 `@tab`，别拼错） |
| **路由参数过长** | URL 参数过长可能截断/丢失，**保持几百字符内**，大数据用全局变量或 store 传 |
| **样式冲突** | 局部样式覆盖全局同片段，就近原则 |
| **页面必须注册** | 页面要在 `pages.json` 的 `pages` 数组里注册，页面最外层原生容器叫 `page` |

---

## 十四、条件编译实战（多端差异化）

### 1. 什么是条件编译

一句话：**同一套代码里，用「注释」标记某段代码只在指定平台生效，编译到其他平台时这段会被自动删掉。**

比如「微信小程序显示 A、App 显示 B、H5 显示 C」，不用写三套代码，用条件编译标记就行。

核心两条语法：

- `#ifdef 平台` … `#endif`：**只在**指定平台编译

- `#ifndef 平台` … `#endif`：**除了**指定平台，其他平台都编译

### 2. 三个地方都能用

**① 模板里（template）**

```
<view>所有端都显示</view>

<view>只有 H5 网页显示</view>

<view>只有微信小程序显示</view>
```

**② 脚本里（script）**

```
onLoad() {
  // #ifdef APP-PLUS
  uni.showToast({ title: 'App 端专属逻辑' });
  // #endif

  // #ifndef H5
  console.log('除了 H5，其他端都执行');
  // #endif
}
```

**③ 样式里（style）**

```
/* #ifdef MP-WEIXIN */
.box { width: 750rpx; }   /* 小程序用 rpx */
/* #endif */

/* #ifdef H5 */
.box { width: 100%; }     /* H5 用百分比 */
/* #endif */
```

> ⚠️ 注释符号跟着所在位置走：模板用 ``、脚本用 `//`、样式用 `/* */`，混用会失效或报错。

### 3. 常用平台值速查

|  |  |
| --- | --- |
| 平台值 | 含义 |
| `APP-PLUS` | App（安卓 + iOS） |
| `H5` | H5 网页 |
| `MP-WEIXIN` | 微信小程序 |
| `MP-ALIPAY` | 支付宝小程序 |
| `MP-BAIDU` | 百度小程序 |
| `MP-TOUTIAO` | 头条 / 抖音小程序 |
| `MP-QQ` | QQ 小程序 |
| `MP` | 所有小程序（快应用除外） |

|  |  |  |
| --- | --- | --- |
|  |  |  |
| 多个平台用 ` |  | `（或）连接： |

```
<view>微信和 QQ 小程序都会显示</view>
```

### 4. 实战：一个页面，三种顶部逻辑

场景：App 端要自定义沉浸式导航（关系统导航栏），小程序端用原生导航栏，H5 端要一个「返回首页」按钮。

```
<template>
  <view class="page">

    <view class="nav">
      <text>返回</text>
      <text class="title">详情页</text>
    </view>

    <navigator url="/pages/index/index" open-type="navigate">
      返回首页
    </navigator>

    <view class="content">正文内容</view>
  </view>
</template>
```

`pages.json` 里同样能条件编译，让 App 端关掉原生导航栏：

```
<template>
  <view class="page">

    <view class="nav">
      <text>返回</text>
      <text class="title">详情页</text>
    </view>

    <navigator url="/pages/index/index" open-type="navigate">
      返回首页
    </navigator>

    <view class="content">正文内容</view>
  </view>
</template>
```

### 5. 三个注意点

1. **编译期决定，不是运行期判断**：`#ifdef H5` 这段代码，打包成小程序时**根本不会带进去**（不是运行时 `if`）。所以可以放心写各端专属的 API，其他端不会报错。

1. **注释符号别写错**：模板 ``、脚本 `//`、样式 `/* */`，跟着位置走。

1. **`#ifndef` 想清楚**：它表示「除某端之外都编译」，别和 `#ifdef`（只在某端）搞反。

---

## 十五、一页速查

|  |  |
| --- | --- |
| 你想干嘛 | 写法 |
| 容器 | `<view>` |
| 文本 | `<text>` |
| 图片 | `<image mode="aspectFit">` |
| 链接跳转 | `<navigator url="..." open-type="navigate">` |
| 选择器 | `<picker :value :range>` |
| 单选框 | `<radio-group>` + `<radio>` |
| 新页跳转 | `uni.navigateTo({url})` |
| 替换页面 | `uni.redirectTo({url})` |
| 重启应用 | `uni.reLaunch({url})` |
| 切 tabBar | `uni.switchTab({url})` |
| 返回 | `uni.navigateBack({delta})` |
| 应用初始化 | `onLaunch` |
| 进前台 | `onShow` |
| 进后台 | `onHide` |
| 页面加载 | `onLoad(options)` |
| 下拉刷新 | `onPullDownRefresh` + `uni.stopPullDownRefresh()` |
| 自动引组件 | easycom `autoscan: true` |
| 跨端适配 | `750rpx = 屏幕宽度` |
| 多端差异化 | `…` |

---

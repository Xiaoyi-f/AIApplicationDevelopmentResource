---
title: "HTML + Sass + TailwindCSS：从骨架到现代化样式，一篇讲透"
status: 已发布
author: XiaoYi
---

# HTML + Sass + TailwindCSS：从骨架到现代化样式，一篇讲透

> 今天把 HTML、Sass、TailwindCSS 串成一条线，TailwindCSS 讲到「完全够用、能开发一切页面」。 Let's use AI to create a better world。

## 一、三个东西是啥关系？（盖房子比喻）

| 技术 | 一句话 | 比喻 |
| --- | --- | --- |
| **HTML** | 网页的「结构」 | 房子的骨架 |
| **Sass** | 让 CSS 更好写的「增强工具」 | 更省力的装修方案 |
| **TailwindCSS** | 原子化的「样式工具箱」 | 一箱现成的装修积木 |

一句话总结分工：

> **HTML 搭骨架，Sass/Tailwind 负责「长什么样」。** 后面两个是「二选一」或「配合用」的关系，Tailwind 是现在最火的方案。

---

## 二、HTML：网页的骨架（完全够用手册）

HTML 用**标签**把内容结构化。下面按「你要干的每件事」分类，覆盖开发任何页面都用得到的标签。

### 2.1 页面骨架（每次都要写）

```
  <!DOCTYPE html>
  <html lang="zh-CN">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>页面标题</title>
      <link rel="stylesheet" href="style.css">
  </head>
  <body>

      <script src="main.js"></script>
  </body>
  </html>
```

> `meta viewport` 那行**必写**，没有它手机上看网页会缩成一团。

### 2.2 文本标签

```
  <h1>一级标题</h1>
  <h2>二级标题</h2>
  <h3>三级标题</h3>
  ​
  <p>这是一个段落</p>
  <p>段落之间会<strong>自动换行</strong>，<em>强调</em>文字</p>
  ​
  <b>加粗</b> <i>斜体</i> <u>下划线</u> <s>删除线</s>
  <br>
  <hr>
```

### 2.3 列表

```
  ​
  <ul>
      <li>第一项</li>
      <li>第二项</li>
  </ul>
  ​
  ​
  <ol>
      <li>第一步</li>
      <li>第二步</li>
  </ol>
```

### 2.4 链接和图片

```
  ​
  <a href="https://example.com" target="_blank">点我跳转</a>
  ​
  ​
  <img src="logo.png" alt="公司 Logo" width="200" height="100">
```

### 2.5 容器与语义化标签

```
  <div>块级容器，网页排版全靠它</div>
  <span>行内容器，嵌在文字里</span>
  ​
  ​
  <header>页头</header>
  <nav>导航栏</nav>
  <main>主体内容</main>
  <section>一个区块</section>
  <article>一篇文章</article>
  <aside>侧边栏</aside>
  <footer>页脚</footer>
```

> 语义化很重要：该用 `nav` 就别用 `div`，搜索引擎和读屏软件靠它理解你的页面结构。

### 2.6 表格

```
  <table border="1">
      <tr>
          <th>姓名</th>
          <th>年龄</th>
      </tr>
      <tr>
          <td>小翊</td>
          <td>19</td>
      </tr>
  </table>
```

### 2.7 表单（重点！做登录、注册、搜索全靠它）

```
  <form action="/submit" method="post">

      <label for="name">姓名：</label>
      <input type="text" id="name" name="name" placeholder="请输入姓名" required>
  ​
      <label for="pwd">密码：</label>
      <input type="password" id="pwd" name="pwd" placeholder="请输入密码">
  ​

      <input type="email" placeholder="邮箱">
      <input type="number" placeholder="年龄">
      <input type="date">
      <input type="checkbox"> 记住我
      <input type="radio" name="gender"> 男
      <input type="radio" name="gender"> 女
      <input type="file">
  ​

      <select name="city">
          <option value="beijing">北京</option>
          <option value="shanghai">上海</option>
      </select>
  ​

      <textarea rows="5" placeholder="请输入留言"></textarea>
  ​

      <button type="submit">提交</button>
      <button type="reset">重置</button>
  </form>
```

> input 的 `type` 决定它是文本框、密码框、还是日期选择器，这是表单的灵魂。

### 2.8 媒体

```
  ​
  <video src="movie.mp4" controls width="600"></video>
  ​
  ​
  <audio src="music.mp3" controls></audio>
```

### 2.9 块级 vs 行内（新手最容易懵）

|  | 表现 | 常见标签 |
| --- | --- | --- |
| **块级** | 独占一行，能设宽高 | `div` `p` `h1~h6` `ul` `li` `table` `header` `footer` |
| **行内** | 不独占，随内容流 | `span` `a` `img` `strong` `em` `input` `button` |

> 一句话：**块级是「一整行」，行内是「嵌在文字里的一个点」。** 用 Tailwind 的 `flex`/`grid` 布局时，这个区别会直接影响排列效果。

### 2.10 常用属性速查

| 属性 | 用在哪 | 作用 |
| --- | --- | --- |
| `id` / `class` | 所有标签 | 唯一标识 / 分类（CSS、JS 靠它选中） |
| `href` | `a` | 跳转地址 |
| `src` | `img` `video` | 资源地址 |
| `alt` | `img` | 图片说明 |
| `type` | `input` `button` | 类型 |
| `placeholder` | `input` | 灰色提示文字 |
| `name` | 表单元素 | 提交时的字段名 |
| `required` | 表单元素 | 必填校验 |
| `disabled` | 表单元素 | 禁用 |
| `hidden` | 所有标签 | 隐藏元素 |
| `data-*` | 所有标签 | 自定义数据（JS 读取用） |

---

## 三、Sass：更省力的 CSS

CSS 有痛点：**不能嵌套、不能定义变量、代码重复**。Sass（SCSS 语法）就是来解决这些的。

### 3.1 变量 `### 

```
  // 定义一次，到处用
  $primary: #4f6bff;
  $spacing: 16px;
  ​
  .button {
      background: $primary;
      padding: $spacing;
  }
```

改主题色，改一个变量就够了。

### 3.2 嵌套

```
  .card {
      padding: 20px;
  ​
      .title {            // 相当于 .card .title
          font-size: 18px;
      }
  ​
      &:hover {           // & 代表父级，相当于 .card:hover
          box-shadow: 0 0 10px rgba(0,0,0,0.2);
      }
  }
```

### 3.3 混入 mixin（复用一段样式）

```
  @mixin flex-center {
      display: flex;
      align-items: center;
      justify-content: center;
  }
  ​
  .box {
      @include flex-center;   // 引入
  }
```

> Sass 三件套：**变量 `$`、嵌套、mixin。** 有这三样，写 CSS 就不再痛苦了。

---

## 四、TailwindCSS：现成的样式积木

Tailwind 是**原子化 CSS**：它不给你现成的「按钮」「卡片」组件，而是给你一堆**极小的样式类**，像积木一样，你自己拼。

### 4.1 安装（两种方式）

**快速体验（CDN，直接玩）：**

```
  <script src="https://cdn.tailwindcss.com"></script>
```

**正式项目（npm，Vue/React 都用这个）：**

```
  npm install -D tailwindcss postcss autoprefixer
  npx tailwindcss init -p
```

### 4.2 核心用法：像写「样式单词」一样

一个按钮，传统 CSS 要写 5 行，Tailwind 直接堆类名：

```
  <button class="bg-blue-500 text-white px-6 py-3 rounded-lg font-bold hover:bg-blue-600">
      点击我
  </button>
```

翻译成人话：

| 类名 | 意思 |
| --- | --- |
| `bg-blue-500` | 背景蓝色（500 是色阶，数字越大越深） |
| `text-white` | 文字白色 |
| `px-6` | 左右内边距 6（padding x） |
| `py-3` | 上下内边距 3 |
| `rounded-lg` | 圆角大号 |
| `font-bold` | 加粗 |
| `hover:bg-blue-600` | 鼠标悬停时背景变深一号 |

---

## 五、TailwindCSS 完全够用手册（重点！）

下面按「你要干的每件事」分类，记住这些，90% 的页面都能拼出来。

### 5.1 布局（最重要）

```
  ​
  <div class="flex items-center justify-between gap-4">
      <div>左边</div>
      <div>右边</div>
  </div>
  ​
  ​
  <div class="grid grid-cols-3 gap-6">
      <div>1</div>
      <div>2</div>
      <div>3</div>
  </div>
```

常用：`flex`（弹性）、`grid`（网格）、`items-center`（垂直居中）、`justify-center`（水平居中）、`justify-between`（两端对齐）、`gap-4`（间距）、`grid-cols-3`（3 列）。

### 5.2 尺寸与间距

```
  <div class="w-full max-w-md mx-auto p-4 mt-8">
```

- 宽高：`w-full`（100%）、`w-1/2`（50%）、`h-screen`（全屏高）、`max-w-md`（最大宽）
- 间距：`p-4`（四边）、`px-4`（左右）、`py-4`（上下）、`m-4`（外边距）、`mt-8`（上边距）
- 居中大法：`mx-auto`（水平居中）

> 数字规则：`p-4` = `1rem`（16px），数字是 4 的倍数，`p-1`=4px、`p-2`=8px、`p-4`=16px、`p-6`=24px。

### 5.3 颜色

```
  <div class="bg-blue-500 text-white border-red-500">
```

格式：`bg-颜色-色阶`、`text-颜色-色阶`、`border-颜色-色阶`。

色阶从 50（最浅）到 900（最深），常用 500 左右。颜色名：`blue`、`red`、`green`、`gray`、`slate`、`purple`、`pink`、`amber`。

### 5.4 文字

```
  <p class="text-lg font-bold text-center leading-relaxed">
```

常用：`text-sm/lg/xl/2xl`（字号）、`font-bold`/`font-medium`（粗细）、`text-center`（居中）、`leading-relaxed`（行高）。

### 5.5 边框、圆角、阴影

```
  <div class="border border-gray-200 rounded-xl shadow-lg">
```

- `border` + 颜色 + 粗细（`border-2`）
- `rounded` / `rounded-lg` / `rounded-full`（圆形）
- `shadow` / `shadow-lg` / `shadow-xl`

### 5.6 响应式（不同屏幕不同样式）

前缀 `sm:` `md:` `lg:` `xl:`，**从小到大**，屏幕够宽才生效：

```
  ​
  <div class="grid grid-cols-1 md:grid-cols-3">
```

> 记牢：`md:grid-cols-3` = 「屏幕宽度 ≥ 768px 时，变成 3 列」。手机默认堆叠，大屏展开，这就是响应式的核心套路。

### 5.7 暗黑模式

```
  <div class="bg-white dark:bg-gray-900 text-black dark:text-white">
```

`dark:` 前缀 = 系统/用户开了暗黑模式时才生效。

**默认跟随系统，不用配**：只要用户的系统/浏览器开了深色模式，`dark:` 样式就自动生效。验证方法——Windows「设置 → 个性化 → 颜色 → 深色」，或 F12 开发者工具里点「太阳/月亮」图标模拟。

**进阶：网站自带「亮/暗切换按钮」**，要把暗黑模式改成手动控制：

1. 配置成 `class` 策略（CDN 方式）：

```
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
      tailwind.config = {
          darkMode: 'class',   // 默认是 'media'（跟随系统），改成 class 手动控制
      }
  </script>
```

npm 项目则在 `tailwind.config.js` 里写 `darkMode: 'class'`。

1. 开关变成「`<html>` 上有没有 `dark` class」：

```
  <html>
  <html class="dark">
```

1. 用 JS 切换（带记忆功能）：

```
  <button onclick="document.documentElement.classList.toggle('dark')">切换亮/暗</button>
  ​
  <script>
      // 读取上次选择
      if (localStorage.getItem('theme') === 'dark') {
          document.documentElement.classList.add('dark')
      }
  </script>
```

> 两种策略怎么选：**`media`（默认）= 跟随系统，零配置；`class` = 网站自带切换按钮，给用户选择权。**

### 5.8 交互态

```
  <button class="bg-blue-500 hover:bg-blue-600 active:bg-blue-700 focus:ring-2 transition duration-300">
```

- `hover:` 悬停、`active:` 按下、`focus:` 聚焦
- `transition` + `duration-300`：加个过渡动画，交互更顺滑

### 5.9 滤镜：模糊、灰度、透明度

```
  ​
  <img class="blur-sm">
  <img class="blur">
  <img class="blur-lg">
  ​
  ​
  <div class="backdrop-blur-md bg-white/50">
```

- `blur-sm` / `blur` / `blur-md` / `blur-lg` / `blur-xl` / `blur-2xl` / `blur-3xl`：模糊程度从轻到重
- `backdrop-blur-*`：毛玻璃，只模糊**背后**的内容，常配合 `bg-white/50`（半透明白底）
- 其他常用滤镜：`grayscale`（变灰）、`opacity-50`（半透明）、`brightness-75`（变暗）

> 想给图片「打码/弱化」用 `blur`；想做「毛玻璃导航栏」用 `backdrop-blur-md bg-white/50`。**`blur` 是模糊元素本身，`backdrop-blur` 是模糊背后的东西**，别搞混。

---

## 六、实战：一个完整的登录卡片

把上面全串起来，写一个现代登录页：

```
  <body class="bg-gray-100 flex items-center justify-center h-screen">
      <div class="bg-white rounded-2xl shadow-lg p-8 w-full max-w-sm">
          <h2 class="text-2xl font-bold text-center mb-6">欢迎回来</h2>
  ​
          <input type="text" placeholder="邮箱"
                 class="w-full px-4 py-3 border border-gray-300 rounded-lg mb-4
                        focus:border-blue-500 focus:outline-none">
  ​
          <input type="password" placeholder="密码"
                 class="w-full px-4 py-3 border border-gray-300 rounded-lg mb-6
                        focus:border-blue-500 focus:outline-none">
  ​
          <button class="w-full bg-blue-500 text-white py-3 rounded-lg font-bold
                         hover:bg-blue-600 transition duration-300">
              登录
          </button>
      </div>
  </body>
```

这个卡片用了：`flex` 居中、`max-w-sm` 限宽、`rounded-2xl` 圆角、`shadow-lg` 阴影、`focus:` 聚焦变色、`hover:` 悬停、`transition` 过渡。**全是上面手册里的积木拼的。**

---

## 七、和你的 Vue3 结合

你项目是 Vue3，Tailwind 直接写在模板的 `class` 里：

```
  <template>
      <button class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600">
          {{ text }}
      </button>
  </template>
  ​
  <script setup>
  defineProps({ text: String })
  </script>
```

动态类名时，注意**别动态拼接字符串**（`bg-${color}-500` 会被 Tailwind 漏掉），要用完整类名映射：

```
  <button :class="{
      'bg-blue-500': type === 'primary',
      'bg-gray-500': type === 'secondary'
  }">
```

---

## 八、前端资源推荐（收藏！）

这些是开发网站最常用的「轮子」，按用途分类，用到哪个去官网查用法即可（大部分都支持 npm 安装或 CDN 引入）。

| 用途 | 推荐库 / 网站 |
| --- | --- |
| CSS 框架 | TailwindCSS |
| PC 端组件库 | Element Plus、Bootstrap、Ant Design Vue |
| 移动端 / 小程序 | Vant、UniApp |
| 数据图表 | ECharts |
| Markdown 编辑器 | md-editor-v3 |
| 流程图 | Mermaid |
| 图标库 | Iconfont、IconPark |
| 动画 | Anime.js、Lottie |
| 3D | Three.js |
| 滚动动画 | ScrollReveal |
| 打字动画 | Typed.js |
| 页面过渡 | Barba.js |
| 动效 | Mo.js |
| 静态资源网站 | 爱给网 [爱给网\_音效配乐\_3D模型\_视频素材\_免费下载](https://www.aigei.com/ "爱给网_音效配乐_3D模型_视频素材_免费下载") |

> 记三个最高频的：**PC 端用 Element Plus，移动端用 Vant，图表用 ECharts**——这仨基本覆盖大多数项目需求，其余用到再查。

---

## 总结：一页速查

| 需求 | Tailwind 类 |
| --- | --- |
| 横向布局 | `flex items-center` |
| 网格 | `grid grid-cols-3 gap-4` |
| 内边距 | `p-4` `px-4` `py-4` |
| 外边距 | `m-4` `mt-8` `mx-auto` |
| 宽度 | `w-full` `max-w-md` `w-1/2` |
| 颜色 | `bg-blue-500` `text-white` |
| 圆角 | `rounded-lg` `rounded-full` |
| 阴影 | `shadow-lg` |
| 响应式 | `md:grid-cols-3` |
| 悬停 | `hover:bg-blue-600` |
| 暗黑 | `dark:bg-gray-900` |
| 过渡 | `transition duration-300` |

> 一句话记牢：**HTML 搭骨架，Sass 省力写 CSS，Tailwind 用现成积木拼样式。** Tailwind 记住「布局、间距、颜色、响应式」四类就够开发一切页面了。

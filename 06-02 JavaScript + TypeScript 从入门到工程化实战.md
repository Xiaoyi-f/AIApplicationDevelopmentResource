---
title: "JavaScript + TypeScript 从入门到工程化实战"
status: 已发布
author: XiaoYi
---

# JavaScript + TypeScript 从入门到工程化实战

> 前面讲了 HTML（骨架）+ Sass + Tailwind（皮肤），这篇一次把「大脑」讲完——**JavaScript**，以及它的进阶版 **TypeScript**，再加上真实项目里天天用的工程化能力（Axios、SSE、WebSocket、IndexedDB）。 从「会写页面」到「能工程化」，一篇讲透。

## 一、JS 和 TS 是啥？

先记住一句话定位：

```
HTML = 骨架(有什么内容)
CSS/Sass/TailwindCSS/组件库 = 皮肤(长什么样)
JavaScript = 大脑(点按钮会怎样、数据怎么变)
```

**TypeScript** 则是给 JavaScript 加上「类型约束」的进阶版——就像给 JS 装了个「提前查错」的编译器。

|  |  |  |
| --- | --- | --- |
|  | JavaScript | TypeScript |
| 关系 | 基础 | 在 JS 之上加类型 |
| 编译 | 浏览器直接跑 | 先转成 JS 再跑 |
| 查错时机 | 运行时报错 | **写的时候就报错** |
| 类型 | 自由（弱类型） | 严格标注 |

> 一句话：**TS = JS + 类型。** 你现在写 Vue3 + TypeScript 项目，本质就是在写 TS。

---

## 二、环境准备

### 1. 跑起来

不用装一堆东西，两样就够：

- **浏览器**：按 F12 打开控制台（Console），就能直接敲 JS

- **Node.js**：让 JS 脱离浏览器，在命令行跑

```
node -v
```

写一个 `test.js`：

```
console.log("Hello JavaScript")
```

命令行 `node test.js` 就能跑。

### 2. 管理 Node 版本（nvm）

Node 版本一多就乱，用 nvm 管理：

```
nvm list
nvm use 20
```

### 3. 包管理器 pnpm

比 npm 快、省硬盘，推荐用 pnpm：

```
npm install -g pnpm
pnpm install
pnpm add axios
pnpm add -D typescript
```

> 国内网络慢，换淘宝镜像，装包快很多：

```
pnpm config set registry https://registry.npmmirror.com/
```

---

## 三、JS 基础语法

### 变量：let 和 const

```
let age = 19
const name = "小翊"

age = 20
```

---

## 四、字符串与数组（天天用）

### 字符串

```
const str = String(0).trim()
str.length
```

### 数组方法全家桶

```
const = [1, 2, 3]

arr.map((val, idx) => val * 2)
arr.filter((val, idx) => val > 1)
arr.find((val, idx) => val === 2)
arr.reduce((a, b) => a + b, 0)
arr.includes(2)

// 增删六件套
arr.pop() // 去尾
arr.push(3) // 入尾
arr.shift() // 删头
arr.unshift() // 添头
arr.slice(0, 2) // 切片
arr.splice(0, 1, 4) // 从下标0开始删除一个再插入4
```

> `slice` 是「复制一份再截」，`splice` 是「原地动刀」。记不住就记：**slice 温和，splice 暴力**。

---

## 五、对象与类

### 对象（键值对）

```
const user = {
    name: "小翊",
    age: 19,
    sayHi() {
        console.log(`我是 ${this.name}`)
    }
}

const { name, age } = user
```

### 类与继承

```
class User extends Object {
    constructor(name) {
        super()
        this.name = name
    }

    useSuperMethod() {
        super.superMethod()
    }
}

const obj = new User("小翊")
```

### 遍历对象和字符串

```
for (const key in obj) {}
for (const ch of str) {}
```

> `for...in` 拿的是**键**，`for...of` 拿的是**值**。对象用 in，数组/字符串用 of。

---

## 六、运算符与类型判断

```
const arr2 = [...arr, 4, 5]

const msg = `我叫 ${name}, 今年 ${age}岁`

typeof 123
typeof "a"
typeof null
obj instanceof User # 判断是不是某个类实例
```

### JS 的类型体系

```
// 原始类型 number string boolean symbol bigint null undefined
// 引用类型 object
```

> JS 对**不存在的字段**默认返回 `undefined`，不会报错（这是它「宽容」的地方，也是容易出 bug 的地方）。

---

## 七、操作网页（DOM）

JS 最有用的能力，就是**操作页面上的元素**：

```
const btn = document.querySelector("#btn")
const title = document.querySelector(".title")

title.textContent = "新标题"

btn.addEventListener("click", () => {
    alert("你点击了我")
})

title.style.color = "red"
```

> 这就是「大脑」在干活：用户点按钮 → JS 监听到 → 改页面内容。

### 7.1 事件大全：鼠标、键盘、表单、窗口

你后面写 Vue 时，`@click`、`@keyup.enter` 这些 `@` 语法，**本质就是原生事件的简写**。

下面按四类把常用事件一次列全，附 Vue 写法对照。

**① 鼠标事件**

|  |  |  |
| --- | --- | --- |
| 事件 | 触发时机 | Vue 写法 |
| `click` | 单击 | `@click` |
| `dblclick` | 双击 | `@dblclick` |
| `contextmenu` | 右键（弹出菜单） | `@contextmenu` |
| `mousedown` | 鼠标按下 | `@mousedown` |
| `mouseup` | 鼠标松开 | `@mouseup` |
| `mousemove` | 鼠标移动 | `@mousemove` |
| `mouseenter` | 鼠标进入元素 | `@mouseenter` |
| `mouseleave` | 鼠标离开元素 | `@mouseleave` |

> ⚠️ 双击是 `dblclick`（**dbl** = double），别写成 `dbclick`，少了个 `l` 会静默失效。

**② 键盘事件**

|  |  |  |
| --- | --- | --- |
| 事件 | 触发时机 | Vue 写法 |
| `keydown` | 按下键盘（任意键） | `@keydown` |
| `keyup` | 松开键盘 | `@keyup` |

按「具体某个键」触发时，用修饰符：`@keyup.enter`（回车）、`@keyup.esc`（Esc）、`@keyup.up/down/left/right`（方向键）。

```
const input = document.querySelector("input")

input.addEventListener("keydown", (e) => {
    if (e.key === "Enter") console.log("按下回车")
})
```

**③ 表单事件**

|  |  |  |
| --- | --- | --- |
| 事件 | 触发时机 | Vue 写法 |
| `input` | 输入内容时（实时） | `@input` |
| `focus` | 获得焦点（点进去） | `@focus` |
| `change` | 内容改变且失焦后 | `@change` |

```
input.addEventListener("input", (e) => {
    console.log(e.target.value) # 同步监听
})

input.addEventListener("change", (e) => {
    console.log("改完了", e.target.value) # 离开输入框触发
})
```

> `input` 和 `change` 的区别：`input` 是**边打字边触发**，`change` 是**输完走人才触发**。做实时搜索用 `input`，做表单提交校验用 `change`。

**④ 窗口事件**

|  |  |  |
| --- | --- | --- |
| 事件 | 触发时机 | Vue 写法 |
| `resize` | 窗口大小改变 | `@resize` |

```
window.addEventListener("resize", () => {
    console.log(window.innerWidth)
})
```

**⑤ 事件修饰符（.prevent / .stop）**

这两个是 Vue 的修饰符，对应原生里的两个「阻止」操作，最常用：

|  |  |  |
| --- | --- | --- |
| Vue 修饰符 | 原生等价 | 作用 |
| `.prevent` | `e.preventDefault()` | 阻止默认行为（如表单跳转、右键菜单） |
| `.stop` | `e.stopPropagation()` | 阻止事件冒泡（别往上层的元素传） |

> 一句话记：**`.prevent` 管「别干它默认那件事」，`.stop` 管「别惊动上面的父元素」。**

### 7.2 自定义事件（组件间通信）

```
// 派发事件
dispatchEvent(new CustomEvent("eventName", {
    detail: data
}))

// 监听: 收到后从 event.detail 取数据
const handler = (e) => {
    const res = e.detail
}

addEventListener("eventName", handler)
removeEventListener("eventName", handler)
```

---

## 八、迭代器与生成器

### 迭代器（Iterator）

```
const iter = array[Symbol.iterator]()
iter.next() // { value: 1, done: false }
```

### 生成器（Generator）

```
function* generator() {
    yield 0
    yield 1
}

const it = generator()
it.next()
it.next()
it.next() // { value: undefined, done: true }
```

> 生成器是「能暂停的函数」，`yield` 就是暂停点。它和 async/await 底层是一家，理解了它，异步就透了。

---

## 九、异步：Promise + async/await

JS 里「要等一会儿才返回」的操作（请求数据、读文件），用异步处理。

### 1. Promise（手动创建）

```
const promise = new Promise((resolve, reject) => {
    if (成功条件) {
        resolve(data) // 传递数据
    } else {
        reject(new Error("error")) // 传递错误
    }
})

// 回调
promise.then(
    (data) => { // 成功处理逻辑 },
    (e) => { // 失败处理逻辑 }
)
```

### 2. async/await（Promise 的语法糖）

```
async function getUser() {
    const resp = await fetch("https://api.example.com")
    const data = await resp.json()
    console.log(data)
    return data
}

getUser() // 顶层调用, 拿不到返回值
```

- `async`：声明这个函数里有异步操作

- `await`：等这个操作完成，再往下走

- `fetch`：JS 内置的「发请求」方法

> 老写法是 `Promise.then(...)`，现在项目里基本都用 `async/await`，更好读。

### 3. 定时器

```
const timeout = setTimeout(() => {}, 1000) // 1s 后执行一次
const interval = setInterval(() => {}, 1000) // 每一秒执行一次

clearTimeout(timeout)
clearInterval(interval)
```

---

## 十、模块化（import / export）

项目一变大，代码要拆文件，互相引用：

```
// math.js
export function add(a, b) {
    return a + b
}

export const PI = 3.14

// 默认导出的导入无需花括号
import add, { PI } from "./math.js"
console.log(add(1, 2), PI)
```

> Vue 组件里的 `import { ref } from "vue"`，就是同一个道理。

---

## 十一、浏览器 API 全家桶

```
// Cookie
document.cookie = "key=val; other_key=val"

// DOM 节点
document.documentElement
document.querySelector(".box")
document.querySelectorAll(".box")

// 属性
obj.getAttribute("href")
obj.setAttribute("href", "https://x.com")

// JSON
JSON.stringify(obj)
JSON.parse(str)

// localStorage sessionStorage
localStorage.getItem("key")
localStorage.setItem("key", "value")
localStorage.removeItem("key")
localStorage.clear()

// 滚动
window.scrollTo(0, 0)

// URL 编码 传递中文参数必用 框架一般自动封装了
encodeURIComponent("小翊")
decodeURIComponent(str)

// 日期
const now = new Date()
now.getFullYear()
now.getMonth()
now.getDate()
now.getHours()
now.getMinutes()
now.getSeconds()

// 异常处理
try {
    // 可能出错的代码
} catch (e) {
    // 出错执行
} finally {
    // 无论成不成均执行
}
```

---

## 十二、Map 和 Set（更强大的集合）

```
const map = new Map()
map.set("key", "value")
map.has("key")
map.get("key")
map.delete("key")
map.clear()
map.size

const set = new Set([1, 2, 3])
set.add(4)
set.delete(1)
set.size
Array.from(set)
```

> Set 去重数组最快：`[...new Set(arr)]`。

---

## 十三、Reflect 与 Proxy（Vue3 响应式的底层）

```
const user = { age: 20 }

Reflect.has(user, "age")
Reflect.get(user, "age")
Reflect.set(user, "age", 18)
Reflect.deleteProperty(user, "age")

const proxy = new Proxy(user, {
    // receiver 代理对象本身
    get(target, attr, receiver) {
        return Reflect.get(target, attr, receiver) // 读取时触发
    },
    set(target, attr, value, receiver) {
        return Reflect.set(target, attr, value, receiver) // 赋值时触发
    }
})
```

> **这就是 Vue3 响应式的原理**：用 Proxy 拦截数据的读写，一变就通知页面更新。理解了 Proxy，就理解了 Vue 的 `ref`/`reactive` 为什么能自动刷新。

---

## 十四、TypeScript

### 1. 为什么项目都要上 TS？

JS 的痛点：**类型太自由，写错了一时不会报错，等运行了才炸**。

```
function add(a, b) {
    return a + b
}

add(1, "小翊") // 结果: 1小翊 -- 与理想情况不同
```

TS 就是来解决这个的——**提前告诉编译器每个变量是什么类型**：

```
function add(a: number, b: number): number {
    return a + b
}

add(1, "小翊") // 报错
```

> 好处：**写的时候就发现错误、编辑器自动提示、重构更安全**。大项目、团队协作，TS 基本是标配。

### 2. TS 核心类型（完全够用）

**① 基础类型标注**

```
let name: string = "小翊"
let age: number = 19
let isDev: boolean = true
let skills: string[] = ["Vue", "Python", "Java"]
let nothing: null = null
```

**② 联合类型（一个变量可以是几种类型）**

```
let id: number | string
id = 1
id = "abc"
```

**③ 接口 interface（描述对象的形状）**

```
interface User {
    name: string
    age: number
    email?: string
}
```

**④ 类型别名 type（和 interface 类似）**

```
type Status = "pending" | "success" | "error"

let s: Status = "success" // ok
// s = "其他" 报错
```

**⑤ 泛型（让类型也「参数化」）**

```
function first<T>(arr: T[]): T {
    return arr[0]
}

first<number>([1, 2, 3])
first<string>(["a", "b"])
```

> 泛型在 Vue3 里处处可见（比如 `ref<number>(0)`），理解「把类型当参数传」就行。

### 3. TS 实战：配合 Vue3

你技术栈里 TS 和 Vue3 是一起用的，看一个最小例子：

```
import { ref } from "vue"

const count = ref<number>(0)

interface User {
    name: string
    age: number
}

const user = ref<User>({ name: "小翊", age: 19 })

function add(): void {
    count.value++
}
```

> `ref<T>` 里的 `<T>` 就是泛型，`.value` 是 Vue3 响应式的固定用法。看到这里，你应该能把 TS 和 Vue3 对上号了。

---

## 十五、工程化实战（真实项目必用）

### 1. IndexedDB（浏览器里的本地数据库）

localStorage 只能存字符串，存大文件、结构化数据要用 IndexedDB。核心是「打开数据库 → 开事务 → 读写」：

```
const DB_NAME = "dbName"
const DB_VERSION = 1
const STORE_NAME = "documents"
const DOC_KEY = "currentDoc"

function openDB() {
    return new Promise((resolve, reject) => {
        const req = indexedDB.open(DB_NAME, DB_VERSION)

        req.onupgradeneeded = () => {
            const db = req.result
            if (!db.objectStoreNames.contains(STORE_NAME)) {
                db.createObjectStore(STORE_NAME, { keyPath: "id" })
            }
        }

        req.onsuccess = () => resolve(req.result)
        req.onerror = () => reject(req.error)
        req.onblocked = () => reject(new Error("IndexedDB upgrade blocked"))
    })
}

// 更新保存(新增或覆盖)
async function saveToDB(json, html) {
    const db = await openDB()
    return new Promise((resolve, reject) => {
        const tx = db.transaction(STORE_NAME, "readwrite")
        const store = tx.objectStore(STORE_NAME)
        const record = {
            id: DOC_KEY,
            json,
            html,
            updatedAt: new Date().toISOString()
        }
        store.put(record)
        tx.oncomplete = () => {
            db.close();
            resolve();
        }
        tx.onerror = () => {
            db.close();
            reject(tx.error);
        }
        tx.onabort = () => {
            db.close();
            reject(tx.error ?? new Error("aborted"))
        }
    })
}

// 读取
async function loadFromDB() {
    const db = await openDB()
    return new Promise((resolve, reject) => {
        const tx = db.transaction(STORE_NAME, "readonly")
        const req = tx.objectStore(STORE_NAME).get(DOC_KEY)
        req.onsuccess = () => {
            db.close();
            resolve(req.result ?? null);
        }
        req.onerror = () => {
            db.close();
            reject(req.error)
        }
    })
}

// 删除
async function deleteFromDB() {
    const db = await openDB()
    return new Promise((resolve, reject) => {
        const tx = db.transaction(STORE_NAME, "readwrite")
        tx.objectStore(STORE_NAME).delete(DOC_KEY)
        tx.oncomplete = () => { db.close(); resolve(); }
        tx.onerror = () => { db.close(); reject(tx.error); }
        tx.onabort = () => { db.close(); reject(tx.error ?? new Error("aborted")) }
    })
}
```

> 三个关键：**`onupgradeneeded` 建表、`transaction` 开事务、`put/get/delete` 读写**。IndexedDB 的 API 是回调式的，所以都包成了 Promise。

### 2. Axios 拦截器（真实项目必配）

```
import axios from 'axios'

const myAxios = axios.create({
  baseURL: process.env.NODE_ENV === 'development' ? '' : 'https://codefather.cn',
  timeout: 10000,
  withCredentials: true
})

myAxios.interceptors.request.use(
  (config) => config,
  (error) => Promise.reject(error)
)

myAxios.interceptors.response.use(
  (response) => {
    const { data } = response
    if (data.code === 40100) {
      if (
        !response.request.responseURL.includes('user/current') &&
        !window.location.pathname.includes('/user/login')
      ) {
        window.location.href = `/user/login?redirect=${window.location.href}`
      }
    }
    return response
  },
  (error) => Promise.reject(error)
)

export default myAxios
```

> 拦截器的价值：**统一处理 token、统一处理未登录跳转、统一报错**，不用在每个请求里重复写。`withCredentials: true` 呼应你之前 Nginx 那篇的 Cookie 跨域坑。

### 3. SSE 前端接收（AI 对话流式输出）

后端用 `text/event-stream` 往外推，前端用 EventSource 接：

```
const eventSource = new EventSource("https://example.com/sse")

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data)
}

eventSource.addEventListener("customEvent", (event) => {
  const data = JSON.parse(event.data)
})

eventSource.onerror = (err) => {

}

if (eventSource) eventSource.close()
```

> 这正好接上你 FastAPI 那篇的 SSE 后端——AI 一个字一个字往外蹦，前端就用这个接。

### 4. socket.io 实时通信（接你的 Netty-SocketIO）

```
import { io } from "socket.io-client"

const socket = io("http://example.com?token=TOKEN", {
  reconnection: true,
  reconnectionDelay: 1000,
  reconnectionAttempts: 5,
  transports: ["websocket"],
  timeout: 10000,
  auth: { token: "TOKEN" }
})

socket.on("connect", () => console.log(socket.id))
socket.on("connect_error", (err) => console.log(err.message))
socket.on("disconnect", (reason) => console.log(reason))

socket.emit("customEvent", msg, (res) => {})

socket.on("customEvent", (msg) => {})

socket.once("customEvent", (msg) => {})

socket.off("customEvent")
```

> 你后端是 Netty-SocketIO，前端就是 socket.io-client，版本要配对（v2.x 后端配 v4.x 前端）。聊天、实时推送、在线状态，都靠这一套。

---

## 十六、一页速查

|  |  |
| --- | --- |
| 你想干嘛 | 写法 |
| 声明变量 | `let x = 1` / `const y = 2` |
| 箭头函数 | `(a, b) => a + b` |
| 数组遍历 | `map` / `filter` / `find` / `reduce` |
| 数组增删 | `push` / `pop` / `shift` / `unshift` / `splice` |
| 类继承 | `class X extends Y` + `super()` |
| 拼字符串 | `` `名字 ${name}` `` |
| 类型判断 | `typeof` / `instanceof` |
| 选元素 | `document.querySelector("#id")` |
| 绑定点击 | `el.addEventListener("click", fn)` |
| 自定义事件 | `dispatchEvent(new CustomEvent(...))` |
| 发请求 | `await fetch(url)` / axios + 拦截器 |
| 异步函数 | `async function fn() {}` |
| 定时 | `setTimeout` / `setInterval` |
| 去重 | `[...new Set(arr)]` |
| 拦截读写 | `new Proxy(obj, handler)` |
| 本地数据库 | IndexedDB |
| 流式接收 | EventSource（SSE） |
| 实时通信 | socket.io-client |
| 导入导出 | `import { x } from "./m.js"` |
| TS 类型标注 | `let n: number` |
| 接口 | `interface User { name: string }` |
| 联合类型 | `number | string` |
| 泛型 | `function f<T>(x: T): T` |

---

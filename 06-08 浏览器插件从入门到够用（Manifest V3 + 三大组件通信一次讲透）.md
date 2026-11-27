---
title: "浏览器插件从入门到够用（Manifest V3 + 三大组件通信一次讲透）"
status: 已发布
author: XiaoYi
---

# 浏览器插件从入门到够用（Manifest V3 + 三大组件通信一次讲透）

> 浏览器插件（扩展）是前端很「香」的技能：会 HTML/JS 就能写，做个小工具自己用、开源涨粉、甚至商业化。你用的油猴脚本、广告拦截、密码管理器，本质都是插件。
> 这一篇按最新 Manifest V3 规范，把插件的四大件、三大组件通信、存储、权限一次讲透，最后手把手做一个能跑的完整插件。学完你就能写出自己的浏览器插件。
> Let's use AI to create a better world。

## 一、浏览器插件是啥 + Manifest V3

**浏览器插件（扩展）** 就是一段跑在浏览器里的代码，用 `manifest.json` 声明「我是谁、我能干什么」，用 JS 实现功能。

**为什么现在学必须用 Manifest V3**：

| 版本 | 状态 | 关键区别 |
| --- | --- | --- |
| Manifest V2 | ❌ 已废弃 | 用 background page（常驻页面） |
| **Manifest V3** | ✅ 当前标准 | 用 **service worker**（按需唤醒，更省资源更安全） |

> ⚠️ **Chrome 已停止支持 V2**，新插件必须用 V3。V3 最大的变化：后台从「常驻页面」变成「service worker」，会休眠、不常驻（这是个坑，后面讲）。

**支持范围**：你笔记里也提了——**Chrome 和 Edge 都支持**（两者内核相同，插件通用）。

---

## 二、插件结构（manifest + 四大件）

一句话：**浏览器插件 = manifest（声明）+ scripts（实现）**。

一个完整插件的四大件：

```
  my-extension/
  ├─ manifest.json           # 声明一切（入口）
  ├─ popup.html              # 工具栏弹窗页
  ├─ popup.css               # 弹窗样式
  ├─ popup.js                # 弹窗逻辑
  ├─ background.js           # 后台服务（service worker）
  ├─ content.js              # 注入网页的脚本
  └─ icons/
     ├─ icon16.png
     ├─ icon48.png
     └─ icon128.png
```

| 文件 | 作用 | 类比 |
| --- | --- | --- |
| `manifest.json` | 插件的「身份证 + 说明书」 | 项目配置文件 |
| `popup.*` | 点击工具栏图标弹出的页面 | 一个迷你网页 |
| `background.js` | 后台服务，常驻监听 | 后端服务 |
| `content.js` | 注入到目标网页里运行 | 油猴脚本 |

---

## 三、manifest.json 详解

```
  {
      "manifest_version": 3,
      "name": "extension",
      "version": "1.0.0",
      "description": "description",
      "icons": {
          "16": "icons/icon16.png",
          "48": "icons/icon48.png",
          "128": "icons/icon128.png"
      },
      "action": {
          "default_title": "打开插件设置",
          "default_popup": "popup.html"
      },
      "background": {
          "service_worker": "background.js"
      },
      "content_scripts": [
          {
              "matches": ["https://*/*", "http://*/*"],
              "js": ["content.js"]
          }
      ],
      "permissions": ["storage", "activeTab", "scripting"],
      "host_permissions": ["https://*/*", "http://*/*"]
  }
```

**逐字段讲**：

| 字段 | 作用 |
| --- | --- |
| `manifest_version` | 版本，**必须是 3** |
| `name` / `version` / `description` | 名称、版本号、描述 |
| `icons` | 图标（16/48/128 三个尺寸） |
| `action` | 工具栏按钮配置：`default_popup` 指定弹窗页 |
| `background` | 后台：`service_worker` 指定后台脚本 |
| `content_scripts` | 注入网页的脚本（我补充的，核心） |
| `permissions` | API 权限（storage、scripting 等） |
| `host_permissions` | 站点权限（能访问哪些网站） |

---

## 四、popup 弹窗页

点击工具栏图标弹出的迷你页面，就是一个普通的 HTML。

**popup.html：**

```
  <!DOCTYPE html>
  <html>
  <head>
      <meta charset="UTF-8">
      <link rel="stylesheet" href="popup.css">
  </head>
  <body>
      <h3>我的插件</h3>
      <button id="btn">切换背景色</button>
      <script src="popup.js"></script>
  </body>
  </html>
```

**popup.js：**

```
  document.getElementById('btn').addEventListener('click', async () => {
      // 向 background 发消息
      const res = await chrome.runtime.sendMessage({ type: 'toggle-bg' })
      console.log('background 返回:', res)
  })
```

---

## 五、background 后台服务

后台用 **service worker** 实现，负责监听事件、中转消息、持久化数据。

**background.js：**

```
  // 插件安装/更新时触发
  chrome.runtime.onInstalled.addListener(() => {
      console.log('插件安装成功')
      chrome.storage.local.set({ enabled: true })
  })
  ​
  // 读取存储
  async function getKey() {
      const res = await chrome.storage.local.get('enabled')
      return res.enabled
  }
  ​
  // 监听来自 popup / content script 的消息
  chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
      if (msg.type === 'toggle-bg') {
          // 转发给当前标签页的 content script
          chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
              chrome.tabs.sendMessage(tabs[0].id, { type: 'toggle-bg' })
          })
          sendResponse('ok')
      }
      return true   // ★ 异步 sendResponse 必须 return true
  })
```

> 你笔记里就是 `chrome.runtime.onInstalled` + `chrome.storage.local`，这里补全了**消息监听**（通信的灵魂）。

---

## 六、content script 注入网页

**content script** 是插件最核心的部分——它能**读写目标网页的 DOM**，实现「操作网页」的能力。

```
  // content.js（注入到网页里运行）
  chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
      if (msg.type === 'toggle-bg') {
          document.body.style.background = 'pink'   // 改网页背景色
          sendResponse('已修改背景')
      }
  })
```

**关键点**：

- content script 通过 manifest 的 `content_scripts` 字段声明注入，`matches` 指定哪些网址生效。
- 它**运行在网页的上下文**里，能操作 DOM，但和网页本身的 JS 是隔离的。

---

## 七、三大组件通信（重点）

这是插件开发最绕的地方。三大组件之间靠 `chrome.runtime.sendMessage` 和 `chrome.tabs.sendMessage` 通信：

```
  popup  ←→  background  ←→  content script
     chrome.runtime        chrome.tabs
     .sendMessage          .sendMessage
```

### 通信方式对比（必背）

| 通信方向 | API | 说明 |
| --- | --- | --- |
| popup → background | `chrome.runtime.sendMessage()` | 都走 runtime |
| content → background | `chrome.runtime.sendMessage()` | 都走 runtime |
| background → popup | 通过 `sendResponse` 返回 | popup 发的消息能拿到返回 |
| background → content | `chrome.tabs.sendMessage(tabId, msg)` | 需要指定 tabId |

### 完整链路实战（切换背景色）

```
  ① popup.js：点击按钮 → chrome.runtime.sendMessage({ type: 'toggle-bg' })
  ② background.js：onMessage 收到 → 查当前标签页 → chrome.tabs.sendMessage(tabId, {type:'toggle-bg'})
  ③ content.js：onMessage 收到 → 改 document.body 背景色
  ④ content.js：sendResponse('已修改') → 返回给 background
  ⑤ background.js：sendResponse('ok') → 返回给 popup
```

> 记忆：**runtime 管「自己人」（popup/background），tabs 管「网页」（content script）**。background 是中转站，两头都能连。

---

## 八、storage 存储

`chrome.storage` 是插件的持久化存储（类似 localStorage，但能跨 popup/background/content 共享）。

```
  // 存（异步）
  await chrome.storage.local.set({ key: 'value' })
  ​
  // 取（异步，返回对象）
  const res = await chrome.storage.local.get('key')
  console.log(res.key)   // 'value'
  ​
  // 删
  await chrome.storage.local.remove('key')
```

**两种存储对比**：

| API | 说明 |
| --- | --- |
| `chrome.storage.local` | 本地存储，容量较大（约 10MB） |
| `chrome.storage.sync` | 同步存储，跟着账号跨设备（容量小，约 100KB） |

> ⚠️ 注意：`chrome.storage` 是**异步**的（返回 Promise），别当同步用。

---

## 九、权限详解

**permissions vs host\_permissions** 是最容易混的：

| 权限类型 | 管什么 | 例子 |
| --- | --- | --- |
| `permissions` | **API 权限**（能调用哪些浏览器 API） | `storage`（存储）、`activeTab`（当前标签）、`scripting`（注入脚本）、`tabs`、`notifications` |
| `host_permissions` | **站点权限**（能访问哪些网站） | `https://*/*`、`http://*/*` |

> 记忆：**permissions 管「用什么能力」，host\_permissions 管「访问哪些网站」。**

**最小权限原则**：只申请你真正用到的权限，权限越多，上架审核越严、用户越警惕。

---

## 十、加载 + 打包发布

### 本地加载（开发调试）

1. 打开 `chrome://extensions`（Edge 是 `edge://extensions`）。
2. 右上角打&#x5F00;**「开发者模式」**。
3. &#x70B9;**「加载已解压的扩展程序」**。
4. 选择你的插件文件夹。

> 改代码后，点插件卡片上的**刷新按钮**重新加载即可。

### 打包

在 `chrome://extensions` 开发者模式 → &#x70B9;**「打包扩展程序」** → 选择插件目录 → 生成 `.crx` 和 `.pem` 文件。

### 发布到商店

上架 Chrome Web Store 需要注册开发者账号（一次性 $5），按流程提交 `.zip` 包审核即可。

---

## 十一、常见坑

| # | 坑 | 原因 | 解决 |
| --- | --- | --- | --- |
| 1 | background 变量丢失 | service worker **会休眠**，不常驻，全局变量清空 | 重要数据存 `chrome.storage`，别依赖全局变量 |
| 2 | 异步 sendResponse 不生效 | onMessage 里异步返回没标记 | 监听器末尾 `return true` |
| 3 | 不能用 `eval` / 远程代码 | V3 的 CSP 限制 | 所有代码本地打包，别动态加载远程脚本 |
| 4 | content script 操作不到页面 | 没在 manifest 声明注入 | 配好 `content_scripts.matches` |
| 5 | 改了 manifest 不生效 | manifest 改动要重载 | 重新加载扩展 |

### 坑 1 展开：service worker 休眠

V3 的 background 是 service worker，**空闲会被休眠**（不常驻内存）。所以：

```
  // ❌ 错误：依赖全局变量，休眠后丢失
  let count = 0
  ​
  // ✅ 正确：存 storage，休眠也不丢
  await chrome.storage.local.set({ count: 0 })
```

### 坑 2 展开：return true

```
  chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
      // 同步处理，可以不用 return true
      // 但如果是异步（比如 await 了），必须 return true 才能 sendResponse
      setTimeout(() => sendResponse('异步返回'), 100)
      return true   // ★ 不写这行，sendResponse 会失效
  })
```

---

## 速查表

| 需求 | 操作 |
| --- | --- |
| 声明插件 | `manifest.json`（`manifest_version: 3`） |
| 工具栏弹窗 | `action.default_popup` 指定 popup.html |
| 后台服务 | `background.service_worker` 指定 background.js |
| 注入网页 | `content_scripts` 声明 content.js |
| popup/background 通信 | `chrome.runtime.sendMessage` + `onMessage` |
| background → content 通信 | `chrome.tabs.sendMessage(tabId, msg)` |
| 持久化存储 | `chrome.storage.local.set/get` |
| API 权限 | `permissions` 字段 |
| 站点权限 | `host_permissions` 字段 |
| 本地调试 | `chrome://extensions` → 开发者模式 → 加载已解压 |

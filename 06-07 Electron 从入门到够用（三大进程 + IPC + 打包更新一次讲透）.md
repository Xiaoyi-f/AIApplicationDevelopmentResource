---
title: "Electron 从入门到够用（三大进程 + IPC + 打包更新一次讲透）"
status: 已发布
author: XiaoYi
---

# Electron 从入门到够用（三大进程 + IPC + 打包更新一次讲透）

> 会 Vue 就能做桌面应用？对，Electron 让你用 HTML/CSS/JS 写出 Windows、Mac、Linux 都能跑的桌面软件。VS Code、钉钉、飞书、Notion 桌面版，全是 Electron 做的。
> 这一篇不讲虚的，就讲你日常够用的：三大进程架构、IPC 通信、electron-vite 配置、打包 + 自动更新，最后把 5 个必踩的坑一次讲清。学完你就能把 Vue 项目打包成桌面 App。
> Let's use AI to create a better world。

## 一、Electron 是啥 + 为什么用它

**Electron** 是一个用 Web 技术（HTML/CSS/JavaScript）构建**跨平台桌面应用**的框架。一句话理解：

> **Electron = Chromium（网页引擎）+ Node.js（后端能力）合体**，给网页套上一个「桌面壳子」。

**为什么用它**（对前端特别友好）：

|  |  |
| --- | --- |
| 优势 | 说明 |
| 技术栈复用 | 会 Vue/React 就能写桌面应用，不用学 C++/C# |
| 跨平台 | 一套代码，Windows / Mac / Linux 都能跑 |
| 生态成熟 | VS Code、钉钉、飞书、Slack 都在用 |
| 能力全 | 文件系统、系统托盘、通知、自动更新全都有 |

> 对比：想做桌面应用，如果你只会前端，Electron 是**成本最低**的选择。

---

## 二、三大进程架构（重点）

这是 Electron 最核心、也最绕的概念。Electron 应用分**三个进程**：

|  |  |  |  |
| --- | --- | --- | --- |
| 进程 | 是什么 | 能做什么 | 类比 |
| **主进程 Main** | Node.js 环境 | 管窗口、文件、系统能力、菜单、托盘 | 「管家 / 后端」 |
| **渲染进程 Renderer** | Chromium 环境 | 跑网页（Vue/React），负责界面 | 「前端页面」 |
| **预加载脚本 Preload** | 桥接层 | 安全地暴露 API 给渲染进程 | 「翻译 / 安全桥」 |

**关键理解**：

- **主进程**是应用的「大脑」，只有一个，用 Node.js 写，`require('electron')` 拿到 `app`、`BrowserWindow` 等能力。

- **渲染进程**一个窗口一个，就是普通的网页，**默认没有 Node 权限**（为了安全）。

- **问题来了**：渲染进程（网页）想存文件、读系统信息，但没权限，怎么办？→ 通过 **preload** 这个「安全桥」中转。

```
渲染进程（网页） --invoke--> preload（暴露 API） --ipcRenderer.invoke--> 主进程（有 Node 权限）
      ↑                                                                      ↓
      └────────────────────── 返回结果（Promise）────────────────────────────┘
```

> 记忆：**主进程管系统，渲染进程管界面，preload 是它们之间唯一的「安全通道」。**

---

## 三、环境 + 项目初始化

```
# 安装 Electron（开发依赖）
pnpm add electron --save-dev

# 同时安装 electron-vite（构建工具）和 vue 插件
pnpm add -D electron-vite @vitejs/plugin-vue
pnpm add vue
```

**package.json 里的脚本**（常见配置）：

```
{
  "scripts": {
    "dev": "electron-vite dev",
    "build": "electron-vite build",
    "build:win": "electron-vite build && electron-builder --win"
  }
}
```

---

## 四、目录结构解析

```
electron-vue-demo/
├─ package.json                 # 依赖 + 脚本
├─ electron.vite.config.js      # electron-vite 配置
├─ index.html                   # 渲染进程入口 HTML
├─ src/
│  ├─ main/index.js             # 主进程（管窗口/系统）
│  ├─ preload/index.js          # 预加载脚本（安全桥）
│  └─ renderer/                 # 渲染进程（Vue 前端）
│     ├─ main.js                # Vue 入口
│     └─ App.vue
└─ build/
   └─ icon.ico                  # 应用图标
```

**三块对应三大进程**，一一对上：

|  |  |
| --- | --- |
| 目录 | 对应进程 |
| `src/main/` | 主进程 |
| `src/preload/` | 预加载脚本 |
| `src/renderer/` | 渲染进程（Vue） |

---

## 五、electron.vite.config.js 配置

```
import { defineConfig, externalizeDepsPlugin } from 'electron-vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
    main: {
        plugins: [externalizeDepsPlugin()]
    },
    preload: {
        plugins: [externalizeDepsPlugin()]
    },
    renderer: {
        plugins: [vue()],
        base: './'          // ★ 关键：相对路径，防止打包后白屏
    }
})
```

**逐段讲**：

|  |  |
| --- | --- |
| 段 | 作用 |
| `main` | 主进程构建配置 |
| `preload` | 预加载脚本构建配置 |
| `renderer` | 渲染进程构建配置（这里才用 vue 插件） |

**两个关键点**：

1. **`externalizeDepsPlugin()`**：把 `main`/`preload` 里 `require` 的依赖**外部化**——不打包进 bundle，运行时直接从 `node_modules` 加载（因为主进程本身就有 Node 环境）。

1. **`base: './'`**：渲染进程静态资源用**相对路径**，否则打包后加载不到资源会**白屏**（这是坑点第 1 条）。

---

## 六、主进程 main/index.js

```
const { app, BrowserWindow, ipcMain } = require('electron')
const path = require('path')
const { autoUpdater } = require('electron-updater')
const log = require('electron-log')

log.initialize()

function createWindow() {
    const win = new BrowserWindow({
        width: 1200,
        height: 800,
        webPreferences: {
            preload: path.join(__dirname, '../preload/index.js'),
            contextIsolation: true,   // 隔离上下文（安全）
            nodeIntegration: false,    // 禁用渲染进程的 Node（安全）
            sandbox: true              // 沙箱（安全）
        }
    })

    if (!app.isPackaged) {
        // 开发模式：加载 Vite 开发服务器
        win.loadURL('http://localhost:5173')
    } else {
        // 生产模式：加载打包后的 HTML
        win.loadFile(path.join(__dirname, '../renderer/index.html'))
    }
}

app.whenReady().then(() => {
    createWindow()
    autoUpdater.checkForUpdatesAndNotify()   // 启动时检查更新

    app.on('activate', () => {
        // Mac 上点击 Dock 图标且没有窗口时，重新创建
        if (BrowserWindow.getAllWindows().length === 0) createWindow()
    })
})

app.on('window-all-closed', () => {
    // 非 Mac 平台，关闭所有窗口就退出
    if (process.platform !== 'darwin') app.quit()
})

// IPC 示例：invoke/handle 双向异步通信
ipcMain.handle('save-file', async (_, content) => {
    log.info('收到内容：', content)
    return 'ok'
})

// 自动更新事件
autoUpdater.on('update-available', () => {
    log.info('发现新版本')
})
autoUpdater.on('download-progress', (progress) => {
    log.info('下载进度', progress.percent)
})
autoUpdater.on('update-downloaded', () => {
    autoUpdater.quitAndInstall()   // 下载完，重启安装
})
```

### 关键点讲透

**1. 安全三件套**（必须开）：

|  |  |
| --- | --- |
| 配置 | 作用 |
| `contextIsolation: true` | 隔离渲染进程上下文，防止网页直接碰 Node |
| `nodeIntegration: false` | 禁止渲染进程用 `require` 等 Node 能力 |
| `sandbox: true` | 沙箱隔离，进一步限制权限 |

> ⚠️ 这三项**默认推荐全开**。有些人图省事把 `nodeIntegration` 设 `true`，等于网页能直接读写你的电脑，有安全隐患。

**2. 开发 vs 生产**：`app.isPackaged` 判断是否打包——开发时加载 `localhost:5173`（热更新），生产时加载本地 HTML 文件。

**3. Mac 特殊处理**：Mac 关闭所有窗口不退出应用（要留在 Dock），所以 `window-all-closed` 里非 darwin 才 `quit`。

---

## 七、preload + IPC 通信（重点）

渲染进程要调主进程的能力，全靠 **IPC（进程间通信）**。

### preload/index.js（安全桥）

```
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('appAPI', {
    saveFile: (content) => ipcRenderer.invoke('save-file', content)
})
```

### 渲染进程里调用

```
// 在 Vue 组件里
const result = await window.appAPI.saveFile('要保存的内容')
console.log(result)   // 'ok'（主进程返回的）
```

### 完整链路

```
渲染进程:  window.appAPI.saveFile(content)
   ↓
preload:   contextBridge 暴露的 saveFile → ipcRenderer.invoke('save-file', content)
   ↓
主进程:    ipcMain.handle('save-file', async (_, content) => { ... return 'ok' })
   ↓
返回 'ok' → preload → 渲染进程拿到 result
```

### IPC 两种通信方式对比（考点）

|  |  |  |
| --- | --- | --- |
| 方式 | 是否返回结果 | 用途 |
| `invoke` / `handle` | ✅ 双向，能拿到返回值 | 查数据、存文件（要结果） |
| `send` / `on` | ❌ 单向，只发不收 | 通知、触发（不要结果） |

> 你笔记里用的就是 **`invoke`/`handle` 双向异步通信**——渲染进程发请求，主进程处理完把结果 Promise 回来。

---

## 八、渲染进程

渲染进程就是普通的 Vue 项目，`main.js` 挂载 App：

```
// src/renderer/main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

> 重点：渲染进程里**不要直接 `require` 任何 Node 模块**（被安全策略禁了），要用系统能力一律走 `window.appAPI`。

---

## 九、打包 + 自动更新

### 打包

```
pnpm build:win   # 打包 Windows 安装包
```

打包依赖 `electron-builder`，配置文件（package.json 里的 `build` 字段）控制图标、安装包格式等。

### 自动更新（electron-updater）

主进程里已经配好了，三个事件驱动整个更新流程：

|  |  |  |
| --- | --- | --- |
| 事件 | 触发时机 | 动作 |
| `update-available` | 发现新版本 | 记日志（可提示用户） |
| `download-progress` | 下载中 | 更新进度条 |
| `update-downloaded` | 下载完成 | `quitAndInstall()` 重启安装 |

```
autoUpdater.on('update-downloaded', () => {
    autoUpdater.quitAndInstall()   // 下载完自动重启安装
})
```

> ⚠️ Mac 平台自动更新需要**签名 + 公证**，否则会被系统拦截（这是坑点第 5 条）。

---

## 十、5 个必踩的坑（重点）

|  |  |  |  |
| --- | --- | --- | --- |
| # | 坑 | 原因 | 解决 |
| 1 | 打包后**白屏** | 静态资源用了绝对路径 | `renderer.base: './'` |
| 2 | 路由刷新 404 | history 模式在 file 协议下刷新找不到文件 | 路由用 `createWebHashHistory` |
| 3 | Electron 下载慢/失败 | 从 GitHub 下载，国内慢 | 设 `ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/` |
| 4 | 原生 `.node` 模块失效 | 打包进 asar 后路径变了 | 配置 `asarUnpack: ["**/*.node"]` |
| 5 | Mac 自动更新被拦 | 没签名+公证 | Mac 更新必须签名 + 公证 |

### 坑 2 展开：路由为什么必须用 Hash

```
// ❌ history 模式：打包后刷新页面 404
createWebHistory()

// ✅ hash 模式：file 协议下刷新正常
createWebHashHistory()
```

原因：生产环境 `loadFile` 加载本地文件（`file://` 协议），history 路由刷新时浏览器去找真实文件路径，找不到就 404；hash 模式路由在 `#` 后面，不触发真实请求。

### 坑 3 展开：加速 Electron 下载

```
# Windows（PowerShell）
$env:ELECTRON_MIRROR="https://npmmirror.com/mirrors/electron/"

# Mac / Linux
export ELECTRON_MIRROR="https://npmmirror.com/mirrors/electron/"
```

设完再 `pnpm install`，Electron 二进制会走国内镜像，快很多。

---

## 速查表

|  |  |
| --- | --- |
| 需求 | 操作 |
| 初始化窗口 | 主进程 `new BrowserWindow({...})` |
| 渲染进程调主进程 | preload 暴露 → `ipcRenderer.invoke` → `ipcMain.handle` |
| 双向通信（要结果） | `invoke` / `handle` |
| 单向通信（不要结果） | `send` / `on` |
| 安全三件套 | `contextIsolation:true` + `nodeIntegration:false` + `sandbox:true` |
| 开发/生产加载 | `app.isPackaged` 判断，loadURL / loadFile |
| 打包 Windows | `pnpm build:win` |
| 自动更新 | `electron-updater` + 三个事件 |

---

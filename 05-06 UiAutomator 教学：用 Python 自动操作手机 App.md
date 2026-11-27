---
title: "UiAutomator 教学：用 Python 自动操作手机 App"
status: 已发布
author: XiaoYi
---

# UiAutomator 教学：用 Python 自动操作手机 App

> 上次讲了网页爬虫（requests + Playwright），这次上一个更硬核的uiautomater：**让你的代码去操作手机 App**。 自动签到、批量点赞、抓 App 数据、跑 UI 测试，全都能干。 Let's use AI to create a better world。

## 一、UiAutomator 是啥？

一句话：**一个「操作手机 App 的机器人」**。

你平时玩手机，是手指在屏幕上点、滑、输入。UiAutomator 就是把这些动作**用代码替你做**。

对比一下你就懂了：

|  |  |
| --- | --- |
| 自动化对象 | 用什么工具 |
| 网页 | Playwright / Selenium |
| 电脑桌面 | PyAutoGUI 之类 |
| **手机 App** | **UiAutomator / uiautomator2** |

> 本质都一样：**找到控件 → 对它做动作**。只是这次控件在手机屏幕上。

---

## 二、先分清两个名字（别搞混）

这俩名字长得像，但不是一回事：

### ① Android 官方 UI Automator（Java）

谷歌官方出的 **Android 原生 UI 测试框架**，用 Java/Kotlin 写，跑在测试环境里，主要用于**自动化测试**（比如 App 上线前的回归测试）。

- 优点：官方、稳定、不需要装额外东西

- 缺点：要会 Android 开发，写起来重

### ② uiautomator2（Python 第三方库）

一个 **Python 库**，底层调用 Android 的 UiAutomator 接口，但让你用**几行 Python** 就能操作手机。

- 优点：语法简单、上手快，适合做**自动化脚本 / 爬 App 数据 / 自动签到**

- 缺点：需要手机开启调试、装个守护程序

> **这篇文章主推 ② uiautomator2**，因为你做 Python 自动化，它最对味；① 作为背景了解。

---

## 三、环境准备（3 步）

### 第 1 步：装 Python 库

```
pip install uiautomator2
```

### 第 2 步：手机开启调试

用一根 USB 线连电脑，手机上做两件事：

1. **打开「开发者选项」**（设置 → 关于手机 → 连点「版本号」7 次）
2. 进入开发者选项，**打开「USB 调试」**

> 弹窗问「允许 USB 调试吗」，点**允许**。

### 第 3 步：验证连接

```
adb devices
```

看到你的设备序列号，就说明连上了。没看到就换个 USB 口 / 数据线。

---

## 四、连接手机（第一步代码）

```
# ADB = Android Debug Bridge（安卓调试桥）
import uiautomator2 as u2

# 一台手机不用参数
d = u2.connect()
# 多台手机指定设备序列号
# u2.connect("设备序列号")

# 无线连接
# adb tcpip 5555
d = u2.connect("192.168.1.100:5555")
```

`d` 就是你的「机器人」，后面所有操作都靠它。

> 第一次连接时，uiautomator2 会**自动往手机装两个东西**（atx-agent + 一个测试 App），等它装完即可，只装一次。
>
> Tip: 下载MT管理器

---

## 五、核心操作（照着抄就能跑）

### 1. 启动 / 关闭 App

```
d.app_start("com.tencent.mm") # 打开微信 包名
d.app_stop("com.tencent.mm") # 关掉软件

d.app_list_running() # 查看正在跑的app包名
```

> **包名怎么找？** 每个 App 有个唯一包名，微信是 `com.tencent.mm`、淘宝是 `com.taobao.taobao`。不知道就 `d.app_list_running()` 看正在跑的，或者网上搜「XX App 包名」。

### 2. 找元素 + 点击

```
# 按照文字找
d(text="登录").click()

# 按照文字模糊匹配
d(textContains="手机号").click()

# 按照控件 id 找
d(resourceId="com.xxx:id/btn_login").click()
```

### 3. 输入文字

```
d(text="请输入手机号").set_text("xxxxxxxxxxx")

d(text="请输入手机号").click()
d.send_keys("xxxxxxxxxxx")
```

### 4. 滑动

```
# 从下往上滑
d.swipe(500, 1500, 500, 500)

# 从上往下滑
d.swipe(500, 500, 500, 1500)

# 带时长
d.swipe(500, 1500, 500, 500, duration=0.5)

# swipe(sx, sy, ex, ey, duration=n) n 单位 秒
```

### 5. 按键

```
d.press("home") # 主页键
d.press("back") # 返回键
d.press("recent") # 最近任务
```

### 6. 截图

```
d.screenshot("screen.png") # 存在当前目录
```

---

## 六、元素定位（自动化最关键的一步）

找到控件是核心，常用这几种：

```
# ① 按文字（text）
d(text='确定').click()

# ② 按描述（desc，图标按钮常靠它）
d(description='返回').click()

# ③ 按资源 id（resourceId，最稳定）
d(resourceId='com.xxx:id/search').click()

# ④ 按控件类型 + 文字组合
d(className='android.widget.Button', text='提交').click()

# ⑤ XPath 定位（最灵活）
d.xpath('//*[@text="确定"]').click()
```

### 怎么知道一个控件的 id / 文字？

用 **weditor**（可视化查看器），把手机屏幕投到电脑上，鼠标点哪就显示哪的控件信息：

```
pip install weditor
python -m weditor
```

浏览器会打开一个页面，选你的设备，就能实时看控件树、点选元素、自动生成定位代码。**这是新手最该装的神器。**

---

## 七、实战：一个自动签到脚本

假设某个 App 每天签到领积分，签到按钮文字是「签到」。完整脚本：

```
import uiautomator2 as u2
import time

d = u2.connect()

# 包名（换成你目标的包名）
APP = 'com.example.app'

# 1. 启动 App
d.app_start(APP)
time.sleep(3)   # 等它加载

# 2. 找到签到按钮并点击
if d(text='签到').exists:
    d(text='签到').click()
    print('签到成功')
else:
    print('没找到签到按钮，可能已签到')

# 3. 截图留证
d.screenshot('sign_result.png')

# 4. 关闭 App
d.app_stop(APP)
```

> 两个实用判断：
>
> - `d(text='签到').exists` 返回布尔值，判断按钮在不在
>
> - `d(text='签到').wait(timeout=10)` 会**等**这个元素出现（最多 10 秒），比写死 sleep 更稳

---

## 八、能干嘛（赚钱 / 价值场景）

|  |  |
| --- | --- |
| 场景 | 说明 |
| 自动签到 | 每天固定时间帮自己/家人签，省事 |
| 爬 App 数据 | 有些数据只有 App 有、没网页版，用它采 |
| UI 自动化测试 | 写脚本回归测试，比手动点省人力 |
| 批量操作 | 群发、批量点赞、批量设置（注意平台规则） |
| 监控 | 定时截图/抓取，监控价格、库存、上新 |

---

## 九、合规红线（必看）

App 自动化比网页爬虫**更容易踩线**，三条务必记住：

1. **账号规则**：很多平台明确禁止自动化，封号风险自负。先看目标 App 的用户协议。
2. **频率克制**：别高频刷，模拟真人节奏，加随机延时。
3. **只碰你有权碰的**：别拿它做撞库、刷单、薅羊毛，这些可能违法。

> 学技术是为了提效和测试，别拿它去碰灰色地带。

---

## 十、一页速查

|  |  |
| --- | --- |
| 你想干嘛 | 代码 |
| 连接手机 | `d = u2.connect()` |
| 打开 App | `d.app_start('包名')` |
| 关 App | `d.app_stop('包名')` |
| 按文字点击 | `d(text='登录').click()` |
| 输入 | `d(text='手机号').set_text('138')` |
| 滑动 | `d.swipe(500,1500,500,500)` |
| 返回键 | `d.press('back')` |
| 截图 | `d.screenshot('x.png')` |
| 判断在不在 | `d(text='x').exists` |
| 等待出现 | `d(text='x').wait(timeout=10)` |

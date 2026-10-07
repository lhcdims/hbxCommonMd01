---
name: "uniautotest"
description: "HBuilderX CLI uni-app x 自动测试：环境准备、cli uniapp.test 执行、编译失败查证、常见卡住根因、调试方法。"
---

# UniAutoTest

uni-app x 项目自动测试（CLI）标准流程。

## Trigger

收到「跑测试」「自动测试」「cli uniapp.test」或类似请求时触发。

## 前置条件确认

1. 项目已云打包（自定义基座，非本地编译）
2. 手机已安装该基座，且 USB 调试已连接
3. **安卓真机和 HBuilderX 所在电脑必须在同一 WiFi 网络下**（否则 sync data 无法建立 WebSocket 通信）
4. 项目路径存在且可访问

## 环境特点

- **测试框架**：`C:\HBuilderX\plugins\hbuilderx-for-uniapp-test-lib\node_modules\jest`
- **Node 路径**：`C:\HBuilderX\plugins\node\node.exe`
- **CLI**：`C:\HBuilderX\cli.exe`
- **项目 node_modules**（如有）会干扰测试，建议删除（见步骤 3）

## 测试命令

**使用 PowerShell**（Windows 推荐）：

```powershell
# 完整命令
C:\HBuilderX\cli.exe uniapp.test app-android --project "C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP005" --vapor true

# 如果已 cd 到 HBuilderX 安装目录，可以简写
.\cli.exe uniapp.test app-android --project "C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP005" --vapor true
```

> ⚠️ 使用 **PowerShell** 或 **cmd.exe** 均可，但不要用 Git Bash / WSL，会路径解析出错。

## 标准流程

### 步骤 1：确认设备和网络

```powershell
& "C:\HBuilderX\plugins\launcher-tools\tools\adbs\adb.exe" devices
```

确保设备在线，且手机和电脑在同一 WiFi 网络下。

### 步骤 2：清理项目的 node_modules（关键）

项目的 `node_modules`、`package.json`、`package-lock.json` 可能包含测试框架依赖（jest、playwright、puppeteer、adbkit 等），会与 HBuilderX CLI 的测试环境冲突，导致 sync data 卡住。

**删除顺序：**
1. 删除 `node_modules` 文件夹
2. 删除 `package.json`
3. 删除 `package-lock.json`

```powershell
# 在项目根目录执行
Remove-Item -Recurse -Force "node_modules"
Remove-Item -Force "package.json"
Remove-Item -Force "package-lock.json"
```

### 步骤 3：确认 main.uts 存在（HBuilderX CLI 判断 uni-app-x 的必要文件）

`main.uts` 是 `cli uniapp.test` 的必要文件。`cli uniapp.test` 内部通过 `isUniAppX()` 判断是否为 uni-app-x 项目，同时检查：
1. `main.uts` 是否存在于项目根目录
2. `manifest.json` 的 `uni-app-x.vapor == true`

**两者皆需满足**，才会设置 `UNI_APP_X_DOM2: true`、`UNI_APP_X_VAPOR_RENDER_TARGET: "bytecode"`、`UNI_CLI_PATH` → `uniapp-cli-vite`。

**如果 `main.uts` 缺失**：这些环境变量不会被设置，导致 Jest 报 `Validation Error: Test environment "...environment.js" cannot be found`，即使 APK 已正确安装也如此。

**检查**：
```powershell
Test-Path "C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP005\main.uts"
```

**如果缺失，创建 `main.uts`**：
```uts
import App from './App.uvue'
import { createSSRApp } from 'vue'
export function createApp() {
    const app = createSSRApp(App)
    return { app }
}
```

### 步骤 4：确认 uni.connectSocket（自定义基座必加）

自定义基座的 App，必须在业务逻辑里建立 WebSocket 通信管道，否则 sync data 会卡住。

**检查位置：** `App.uvue` 的 `onLaunch` 或 `onAppShow` lifecycle 里。

**如果缺少，在 App.uvue 的 onLaunch 里加：**
```javascript
onLaunch(() => {
    uni.connectSocket({ url: 'ws://127.0.0.1:0' })
})
```

> 注意：普通基座（未修改的 DCloud 基座）不需要此步骤。

### 步骤 5：执行测试

```powershell
# 跑全部测试
C:\HBuilderX\cli.exe uniapp.test app-android --project "<项目根目录>" --vapor true
```

### 步骤 6：读取输出，识别编译失败

**Every `cli uniapp.test` invocation** — scan the output for these patterns:

| Pattern | Meaning | Action |
|---|---|---|
| `kotlin编译失败` | `.uvue` file has Kotlin-incompatible code (e.g., unresolved plugin type references) | Copy error block; report to user with file:line evidence |
| `发送同步资源数据` followed by **no `automator:runtime connected`** for >30s | App initializes but WebSocket channel never establishes | Run `adb logcat -d \| Select-String "initWebSocket\|crash\|Exception"` to diagnose |
| `Build failed with errors` | Compile stage failed | Same as `kotlin编译失败` |

**Key observed pattern (2026-09-10):** `kotlin编译失败` with `Unresolved reference 'ReadDcimImageResult'` in `pages/index/index.uvue` — the `.uvue` imports a UTS plugin type that Kotlin cannot resolve. This manifests as a hang at "发送同步资源数据" because the app code is fundamentally broken at compile level. **Do not retry `cli uniapp.test` until the `.uvue` code is fixed.**

### 步骤 7：等待结果

正常输出：
```
[DONE] Build complete. Watching for changes...
[connected]              ← 测试框架已连上 App
[program ready]          ← App 已就绪
[index.test.js 测试开始！]  ← 测试逻辑开始执行
Test Suites: 1 passed, 1 total
Time: xx s
```

## 只跑某个测试文件

在项目根目录新建 `jest.config.js`：

```javascript
// jest.config.js
module.exports = {
  // 只跑指定的测试文件（glob 模式）
  testMatch: ['**/pages/index/index.test.js'],
  // 调大超时（默认 30s，测试跑得久可以调大）
  testTimeout: 60000,
}
```

> 注意：加了 `testMatch` 后，只跑匹配的文件，其他测试文件会被忽略。

## .test.js 怎么写

参考文档：
- [uni-app 自动化测试 API](https://uniapp.dcloud.io/collocation/auto/quick-start)（DCloud 官方）
- [jest 官方文档](https://www.jestjs.cn/)

**基础示例**（来自 P005，已验证可跑）：

```javascript
jest.setTimeout(60000)

describe('[P005] 项目测试', () => {
  let page

  beforeAll(async () => {
    await new Promise(r => setTimeout(r, 10000))
    page = await program.currentPage()
  })

  it('确认 index.uvue 页面加载成功', async () => {
    await program.reLaunch('/pages/index/index')
    await page.waitFor(3000)
    const indexPage = await program.currentPage()
    console.log('页面路径：' + indexPage.path)
    if (indexPage.path.includes('index')) {
      console.log('测试成功！')
    } else {
      console.log('测试失败！')
    }
  })
})
```

**常用 API：**
- `jest.setTimeout(ms)` — 设置全局超时
- `describe(name, fn)` — 测试套件分组
- `beforeAll(fn)` — 所有测试前执行一次
- `beforeEach(fn)` — 每个测试前执行一次
- `afterAll(fn)` — 所有测试后执行一次
- `it(name, fn)` 或 `test(name, fn)` — 单个测试用例
- `expect(value)` — 断言
- `program.currentPage()` — 获取当前页面信息
- `program.navigateTo(options)` — 跳转到指定页面
- `program.callMethod(name, args)` — 调用 App 内的 method
- `page.callMethod(name, args)` — 调用当前页面的 method

## 常见失败模式

### program.screenshot() 卡住 / 截图无数据返回

**症状：** `program.screenshot()` 发出后一直收到 pong 心跳，无截图数据，60s 超时。

**根因：** 项目缺少 `jest-setup.js` 和 `jest.config.js` 里的 `setupFilesAfterEnv` 注册。

**修复步骤：**
1. 从 hello-uni-app-x 模板复制 `jest-setup.js` 到项目根目录
2. 在 `jest.config.js` 加入 `setupFilesAfterEnv: ['<rootDir>/jest-setup.js']`

```powershell
# 1. 复制 jest-setup.js
Copy-Item "<HBuilderX安装目录>\plugins\hbuilderx-ai-chat\uni-agent\knowledges\uni-app-x\samples\hello-uni-app-x\jest-setup.js" "<项目根目录>\jest-setup.js"

# 2. jest.config.js 中确保有 setupFilesAfterEnv
```

### sync data 卡住

**症状：** `发送同步资源数据` 后无响应，CLI 挂起超时。

**排查顺序：**
1. 确认手机和电脑在同一 WiFi 网络
2. 确认 App.uvue 有 `uni.connectSocket`（自定义基座必加）
3. 确认手机上的基座是最新版本（云打包后需重新安装）
4. 删除项目的 `node_modules`、`package.json`、`package-lock.json`

### Unable to find package

**症状：** log 显示 `Unable to find package: com.xxx.xxx`

**排查：**
1. 确认手机已安装该 app
2. 确认 `adb devices` 能看到设备
3. 确认 app 包名与测试命令中的 `--project` 路径匹配

### client connection close: 1006

**症状：** WebSocket 断开，测试框架退出。

**原因：** App 主动关闭了连接，或 App crash 了。

**排查：** 用 `adb logcat` 读 App 日志确认 crash 原因。

## 测试结果

结果保存在：
```
C:\Users\lichiukenneth\AppData\Roaming\HBuilder X\hbuilderx-for-uniapp-test\<项目名>\android\
```

文件名为 `ZY<deviceId>-<timestamp>.json`。

## 调试：读 device log

不触发新编译，直接读取 device 上现有 log：

```powershell
# 全部 log
& "C:\HBuilderX\plugins\launcher-tools\tools\adbs\adb.exe" logcat -d

# 过滤 plugin:uts 错误
& "C:\HBuilderX\plugins\launcher-tools\tools\adbs\adb.exe" logcat -d | Select-String -Pattern "plugin:uts"

# 过滤项目前缀
& "C:\HBuilderX\plugins\launcher-tools\tools\adbs\adb.exe" logcat -d | Select-String -Pattern "001\.01|013\.01"
```

## 重要：uni_modules 修改后必须改版本号

每次修改 `uni_modules` 后重新云打包，必须在 `manifest.json` 里修改版本号（如 `versionName` + `versionCode`），否则手机上的旧基座不会更新，测试会用旧代码跑。

## 注意事项

- **不要在测试时开 HBuilderX GUI**：同一项目同时只能有一个调试会话，否则端口冲突
- **使用 PowerShell 或 cmd.exe**：不要用 Git Bash / WSL，会路径解析出错

## 当前环境

| 项目 | 路径 |
|---|---|
| P004 | `C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP004` |
| P005 | `C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP005` |
| P006 | `C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP006` |

| 设备 | ID |
|---|---|
| Motorola XT2201-2 | `ZY22FCSF4W` |

> 测试结果保存在 `C:\Users\lichiukenneth\AppData\Roaming\HBuilder X\hbuilderx-for-uniapp-test\<project>\android\ZY<deviceId>-<timestamp>.json`

---
name: "uniautotest"
description: "HBuilderX CLI uni-app x automated testing: environment prerequisites, cli uniapp.test execution, compile failure diagnosis, common hang root causes, and debug methods."
---

# UniAutoTest

Automated testing for uni-app x projects via HBuilderX CLI.

## 路径代号解析（执行前必做）

本文档中的代号必须在执行前换成本机实际路径：

- `<HBX_CLI>` / `<HBX_HOME>`：读 `%USERPROFILE%\.hbx-common.json` 的 `hbxCli` / `hbxHome`
- `<PROJECT_ROOT>`：当前任务的专案根目录（由派工 brief 指定，不写死）
- `<OPENCLAW_HOME>`：OpenClaw 状态目录（通常是 `%USERPROFILE%\.openclaw`）
- `%APPDATA%`：使用者 AppData 目录

## Trigger

User asks to "跑测试", "自动测试", "cli uniapp.test", or similar.

## Prerequisites

1. Project has been cloud-packaged (custom test base, not local build)
2. Phone has the test base installed and USB debugging is connected
3. **Android device and HBuilderX PC must be on the same WiFi network** (WebSocket sync requires it)
4. Project path is accessible

## Environment

- **Test framework**: `<HBX_HOME>\plugins\hbuilderx-for-uniapp-test-lib\node_modules\jest`
- **Node**: `<HBX_HOME>\plugins\node\node.exe`
- **CLI**: `<HBX_CLI>`
- **Project node_modules** (if present) interfere with the test environment — delete before testing (see Step 2)

## Test Command (PowerShell)

```powershell
<HBX_CLI> uniapp.test app-android --project "<PROJECT_ROOT>" --vapor true
```

Do NOT use Git Bash / WSL — path resolution will fail. PowerShell or cmd.exe are both fine.

## Standard Workflow

### Step 1: Confirm Device and Network

```powershell
& "<HBX_HOME>\plugins\launcher-tools\tools\adbs\adb.exe" devices
```

Device must be online and on the same WiFi as the PC.

### Step 2: Delete Project node_modules (critical)

Project `node_modules`, `package.json`, and `package-lock.json` may contain test framework deps (jest, playwright, puppeteer, adbkit) that conflict with HBuilderX CLI's test environment and cause sync to hang.

Delete in order:
1. `node_modules` folder
2. `package.json`
3. `package-lock.json`

### Step 3: Confirm main.uts Exists

`main.uts` is the gate file `cli uniapp.test` uses to identify a uni-app-x project. Without it, Jest fails with `Validation Error: Test environment "...environment.js" cannot be found` even if the APK is correctly installed.

```powershell
Test-Path "<PROJECT_ROOT>\main.uts"
```

If missing, create it:
```uts
import App from './App.uvue'
import { createSSRApp } from 'vue'
export function createApp() {
    const app = createSSRApp(App)
    return { app }
}
```

### Step 4: Confirm uni.connectSocket (custom base only)

Custom test bases must open a WebSocket pipe in the app logic, otherwise sync data hangs.

Check in `App.uvue` `onLaunch` or `onAppShow` lifecycle. If missing, add:

```javascript
onLaunch(() => {
    uni.connectSocket({ url: 'ws://127.0.0.1:0' })
})
```

(Standard DCloud bases do NOT need this.)

### Step 5: Run the Test

```powershell
<HBX_CLI> uniapp.test app-android --project "<project-root>" --vapor true
```

### Step 6: Read Output — Identify Failures

**Every invocation** — scan for these patterns:

| Pattern | Meaning | Action |
|---|---|---|
| `kotlin编译失败` | `.uvue` file has Kotlin-incompatible code (e.g., unresolved plugin type references) | Copy error block with file:line evidence; do NOT retry until .uvue code is fixed |
| `发送同步资源数据` followed by no `automator:runtime connected` for >30s | App initializes but WebSocket never establishes | Run `adb logcat -d \| Select-String "initWebSocket\|crash\|Exception"` |
| `Build failed with errors` | Compile stage failed | Same as `kotlin编译失败` |

**Known pattern (2026-09-10):** `kotlin编译失败` with `Unresolved reference 'ReadDcimImageResult'` in `pages/index/index.uvue` — the `.uvue` imports a UTS plugin type that Kotlin cannot resolve. This hangs at "发送同步资源数据" because the app code is broken at compile level. Do not retry `cli uniapp.test` until the `.uvue` is fixed.

### Step 7: Wait for Results

Normal output:
```
[DONE] Build complete. Watching for changes...
[connected]              ← test framework connected to App
[program ready]          ← App is ready
[index.test.js 测试开始！] ← test logic starts
Test Suites: 1 passed, 1 total
Time: xx s
```

## Running a Specific Test File

Create `jest.config.js` in the project root:

```javascript
module.exports = {
  testMatch: ['**/pages/index/index.test.js'],  // only run this file
  testTimeout: 60000,
}
```

## Writing .test.js

Reference:
- [uni-app automation API](https://uniapp.dcloud.io/collocation/auto/quick-start) (DCloud official)
- [jest docs](https://jestjs.io/)

**Example (verified):**

```javascript
jest.setTimeout(60000)

describe('[<project>] 项目测试', () => {
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

**Common APIs:**
- `jest.setTimeout(ms)` — global timeout
- `describe(name, fn)` — test suite grouping
- `beforeAll(fn)` — run once before all tests
- `beforeEach(fn)` — run before each test
- `afterAll(fn)` — run once after all tests
- `it(name, fn)` / `test(name, fn)` — single test case
- `expect(value)` — assertion
- `program.currentPage()` — current page info
- `program.navigateTo(options)` — navigate to a page
- `program.callMethod(name, args)` — call an App method
- `page.callMethod(name, args)` — call current page method

## Common Failures

### program.screenshot() hangs / no data returned

**Symptom:** `program.screenshot()` sends pong heartbeats but never returns screenshot data, times out after 60s.

**Root cause:** Project is missing `jest-setup.js` and `setupFilesAfterEnv` in `jest.config.js`.

**Fix:**
1. Copy `jest-setup.js` from the hello-uni-app-x template to project root:
   ```
   Copy-Item "<HBuilderX安装目录>\plugins\hbuilderx-ai-chat\uni-agent\knowledges\uni-app-x\samples\hello-uni-app-x\jest-setup.js" "<项目根目录>\jest-setup.js"
   ```
2. In `jest.config.js`, ensure `setupFilesAfterEnv: ['<rootDir>/jest-setup.js']` is set.

### sync data hangs

**Symptom:** `发送同步资源数据` appears then nothing for >30s.

**Troubleshoot in order:**
1. Confirm device and PC are on the same WiFi
2. Confirm `App.uvue` has `uni.connectSocket` (custom base)
3. Confirm the test base on the phone is up to date (reinstall after cloud packaging)
4. Delete project's `node_modules`, `package.json`, `package-lock.json`

### Unable to find package

**Symptom:** log shows `Unable to find package: com.xxx.xxx`

**Troubleshoot:**
1. Confirm app is installed on the device
2. Confirm `adb devices` shows the device
3. Confirm app package name matches the `--project` path

### client connection close: 1006

**Symptom:** WebSocket disconnects, test framework exits.

**Cause:** App crashed or closed the connection actively.

**Troubleshoot:** Use `adb logcat` to read App logs and find the crash reason.

## Test Results

Saved to:
```
%APPDATA%\HBuilder X\hbuilderx-for-uniapp-test\<project>\android\
```

Filename: `ZY<deviceId>-<timestamp>.json`.

## Debug: Read Device Log (no new compile)

```powershell
# all logs
& "<HBX_HOME>\plugins\launcher-tools\tools\adbs\adb.exe" logcat -d

# filter plugin:uts errors
& "<HBX_HOME>\plugins\launcher-tools\tools\adbs\adb.exe" logcat -d | Select-String -Pattern "plugin:uts"

# filter project prefix
& "<HBX_HOME>\plugins\launcher-tools\tools\adbs\adb.exe" logcat -d | Select-String -Pattern "001\.01|013\.01"
```

## Important: Bump Version After uni_modules Changes

After modifying `uni_modules` and re-cloud-packaging, increment `versionName` and `versionCode` in `manifest.json`. Otherwise the phone's old base will not update and tests run against stale code.

## Notes

- **Do not open HBuilderX GUI for the same project** while testing — only one debug session allowed at a time, port conflicts will occur
- Use PowerShell or cmd.exe — not Git Bash / WSL

## Current Environment

| Project | Path | Status |
|---|---|---|

| Device | ID |
|---|---|
| Motorola XT2201-2 | `ZY22FCSF4W` |

Test results: `%APPDATA%\HBuilder X\hbuilderx-for-uniapp-test\<project>\android\ZY<deviceId>-<timestamp>.json`

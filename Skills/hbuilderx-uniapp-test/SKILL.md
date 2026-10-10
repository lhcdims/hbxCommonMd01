---
name: "hbuilderx-uniapp-test"
description: "Run uniapp Android tests via HBuilderX CLI and report results to main agent."
---

# HBuilderX uniapp test execution

Run uniapp Android tests via HBuilderX CLI, report results to main agent.

## Task entry points

- Inter-session message from `agent:main:telegram:direct:<id>` with test task
- Manual request to run tests for a project

## Procedure

### 1. Identify project and test file

```
Project path: from task (e.g. C:\Users\...\xLearnAiP005)
Test file:    autotest/<test-name>.test.js
```

If no test file exists in `autotest/`, create one before running.

### 2. Update jest.config.js

```js
// In project's jest.config.js, update testMatch:
testMatch: ["C:/Users/lichiukenneth/Documents/HBuilderProjects/Learn/xLearnAiP005/autotest/p005-delete-permanent.test.js"],
```

Replace path with actual project and test file paths.

### 3. Run test command

```
C:\HBuilderX\cli.exe uniapp.test app-android --project "<project-path>" --vapor true
```

Use `& "C:\HBuilderX\cli.exe"` in PowerShell if bare path fails.

### 4. Wait for completion

Typical output lines when complete:
```
Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Time:        94.784 s
测试用例总计：1，运行通过 1，运行失败 0，运行异常 0
测试运行结束。
```

Exit code 0 = success. Poll with `process(action="poll", timeout=120000)` in 30s increments.

### 5. Report results to main agent

Use `message(action="send", channel="telegram", target="942566572")`:
- Pass/Fail summary with test counts and total time
- Key findings or failures
- Test report file location

## Known result locations

```
AppData\Roaming\HBuilder X\hbuilderx-for-uniapp-test\<project>\android\<device-id>-<timestamp>.json
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "cli.exe not recognized" | Use full path `C:\HBuilderX\cli.exe` |
| No approved executables | Request operator add `C:\HBuilderX\cli.exe` to approved list |
| Test file not found | Create in `autotest/` directory, update jest.config.js testMatch |
| Long compile time | First run compiles; subsequent runs skip unchanged files |

## Verification check

Confirm all three test phases completed (compile → install → run) before reporting pass/fail.

---
name: "forward-uniagent"
description: "Dispatches prompts to HBuilderX uni-agent CLI. After edits to uni_modules utssdk, runs `cli compile` verification. Full `cli uniapp.test` workflow → uniautotest skill."
---

# Forward uni-agent Instructions via OpenClaw

Forwards `uniagent:`-prefixed user messages (and PM-initiated fix-loop dispatches) to HBuilderX IDE's uni-agent CLI. **`cli.exe` is a deterministic Windows native program** (not AI); n=1 PASS = complete verify.

## Workflow (Step 0 → 4)

### Step 0: Prepend PM Context Header (REQUIRED — every prompt)

First line of the prompt must be (literal Chinese text — copy exactly):

```
我是openclaw PM，正在用 cli.exe 和你对话，你回覆我并需要问我问题时，不要使用QUESTION工具，否则你的系统会弹窗并等待，我无法看见你的问题，因此请改用文字模式发问！
```

Why (3 reasons, all observed failure modes if skipped):
1. uni-agent MUST know it is talking to OpenClaw PM (not a human in HBuilderX GUI) — affects its response style
2. CRITICAL: uni-agent QUESTION tool pops a UI dialog in HBuilderX GUI — OpenClaw runtime is headless and CANNOT see or respond to that dialog (fix loop would block). Text-mode questions are visible in stdout and PM can answer next turn
3. PM replies are batched (not real-time), so uni-agent should phrase questions to be answerable in 1 prompt

Order is strict: Step 0 → Step 1 (escape `"` to `\"` on WHOLE combined content) → Step 2 (write file) → Step 3 (invoke PS1) → Step 4 (read response) → decide next turn or report.

### Step 1: Escape `"` to `\"` in WHOLE prompt content

When writing the prompt (header + body), replace every literal `"` with exactly `\"` (backslash + double-quote, 2 chars).

Why: PowerShell auto-escapes `"` to `\"` when invoking native commands. If prompt already contains `"`, double-processing corrupts the command line.

Concrete example:
- Original: `He said "hello" to me`
- Write to file: `He said \"hello\" to me`

### Step 2: Write Prompt File (use OpenClaw `write` tool)

- Tool: `write`
- Path: `C:\Users\lichiukenneth\.openclaw\workspace\forward-prompt.txt`
- Content: prompt string from Step 1, with `\"` in place of every `"`. Encoding: UTF-8 with BOM.

Do NOT use PS1 string interpolation, here-strings, or Python `open(path,'w')` to write the prompt — all break on quotes or encoding.

### Step 3: Invoke PS1 (use OpenClaw `exec` tool)

PS1 content (write this EXACTLY to `C:\Users\lichiukenneth\.openclaw\workspace\forward.ps1`):

```powershell
param(
    [ValidateSet('P004', 'P005')]
    [string]$Project = 'P004'
)
$map = @{
    P004 = 'C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP004'
    P005 = 'C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP005'
}
$promptFromFile = Get-Content "C:\Users\lichiukenneth\.openclaw\workspace\forward-prompt.txt" -Raw
& 'C:\HBuilderX\cli.exe' uni-agent --prompt $promptFromFile --project $map[$Project] --continue true --output-format stream 2>&1
```

Execute:

- Tool: `exec`
- Command: `powershell -ExecutionPolicy Bypass -File "C:\Users\lichiukenneth\.openclaw\workspace\forward.ps1" -Project P005`
- Timeout: 120 seconds (uni-agent response can take 30-90s)

Pass `-Project P005` (or `-Project P004`) as argument — the PS1 uses a param, not hardcoded path.

Do NOT modify any of: `& 'C:\HBuilderX\cli.exe'` (full path required), `--continue true` (Hard rule #0), `2>&1` (stderr capture), or use BAT/CMD wrapper (cmd splits at newlines).

### Step 4: Read Response

In the exec output, look for `HBuilderX Version: 5.24` then read uni-agent's response paragraphs.

If response contains `参数 project 值不能为空`, see Common Errors below.

### Post-Dispatch: Verify Compile

After uni-agent reports "done" for any prompt that edited `uni_modules/<plugin>/utssdk/app-android/*.uts`, run compile verification:

```powershell
& 'C:\HBuilderX\cli.exe' compile app-android --project 'C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP005' --uni_module 'uni_modules/<plugin-name>' 2>&1 | Out-String -Width 4096
```

- `编译成功` → dispatch next turn or report success
- `kotlin编译失败` → copy error block (file paths + line numbers) into next forward-prompt.txt as evidence; do NOT ask uni-agent to guess without showing the actual error

Full `cli uniapp.test` workflow (including `main.uts` prerequisite, `uni.connectSocket` requirement, and test result reading) → see **`uniautotest`** skill.

### Step 5: Multi-turn Pattern

For multi-turn dialogue (fix loop), use `--continue true` to keep the same session.

- **Turn 1**: write `forward-prompt.txt` with first message (with `\"`), run PS1.
- **Turn 2+**: write `forward-prompt.txt` with follow-up (with `\"`), run SAME PS1 — DO NOT change `--continue true` to `--continue false`.

Both turns use the same PS1 file. `--continue true` is fixed.


---

## 🔴 Hard Rule #0: MUST ONLY USE `--continue true`

| Flag value | Behavior | Result |
|---|---|---|
| `--continue true` | Continue most recent CLI session | ✅ Multi-turn context preserved |
| `--continue false` | Start new session | ❌ Kills previous session context |

User 原话 (2026-08-14): "这样你会杀掉我之前的对话". Absolute rule. If `cli.exe` returns no output (session dead), only the user can restart HBuilderX GUI.


---

## Pre-flight Checklist (perform IN ORDER)

1. **Verify HBuilderX session healthy** — Open the HBuilderX GUI window and confirm previous forward's output is visible (not stuck loading). If stuck, ask user to restart HBuilderX GUI.
2. **Verify mmx-cli quota > 10%** — Run `mmx quota --output json`, parse `model_remains[0].current_interval_remaining_percent`. If ≤ 10%, reply `配额紧张（剩 X%），先不发，5h 重置于 HH:MM UTC+8` and stop.
3. **Verify project path exists** — Run `Test-Path "C:\Users\lichiukenneth\Documents\HBuilderProjects\Learn\xLearnAiP004"`. Expected: `True`.
4. **Verify prompt file written correctly** — Run `Get-Content "C:\Users\lichiukenneth\.openclaw\workspace\forward-prompt.txt" -Encoding utf8` and verify content matches prompt with `\"` in place of every `"`.


---

## Common Errors + Fixes

| Error (exact text) | Cause | Fix |
|---|---|---|
| `cli.exe : The term 'cli.exe' is not recognized` | Used `cli.exe` without full path (`cli` alone is PowerShell's `Clear-Item` alias) | Use `& 'C:\HBuilderX\cli.exe' uni-agent ...` with `&` + single-quoted full path |
| `[hbuilderx-ai-chat] 参数 project 值不能为空` | Prompt content contains literal `"` without `\"` escape | Re-do Step 1: replace every `"` with `\"`, re-do Step 2 (overwrite prompt file), re-do Step 3 (re-run PS1) |
| `The string is missing the terminator: '` (PS1 parse error) | Single quotes in `& 'C:\HBuilderX\cli.exe' ... --project '...' ...` are unbalanced — must be exactly 4 (1 pair around cli.exe path, 1 pair around project path) | Re-write PS1 file with balanced quotes |
| HBuilderX session hangs (no exec output for 60+ seconds) | Session died from previous forward | Ask user to restart HBuilderX GUI (only user can do this) |


---

## Gotchas (key pitfalls, reference)

- **PowerShell auto-escapes `"`**: Write prompt to file via OpenClaw `write` tool (Step 2), have PS1 read it — do not embed prompt in PS1 strings or here-strings
- **`cli` bare word triggers alias**: PowerShell `Clear-Item` alias matches `cli` — always use `& 'C:\HBuilderX\cli.exe'` with `&` + full path
- **BAT/CMD splits at newlines**: Direct PS1 invocation only; no `cmd /c bat.bat` wrappers
- **User refuses HBuilderX GUI compile output** (2026-09-07, 3 times): Always use `cli compile` (Post-Dispatch step) instead of asking user to read GUI output
- **Quota gate**: `mmx quota --output json` → `model_remains[0].current_interval_remaining_percent` ≤ 10% → refuse forward; see `references/quota-gate.md` for background

---

## `cli uniapp.test` vs Cloud Packaging

`cli uniapp.test` always uses the **LOCAL** APK at:
```
<project>\unpackage\debug\android_debug_vapor.apk
```

Cloud packaging produces a **separate** APK on DCloud's servers. Cloud packaging success does NOT mean the local build passes Kotlin compilation. If local compile fails, `cli uniapp.test` will fail regardless of cloud packaging status.

Before running `cli uniapp.test`, ensure the local project compiles cleanly — Kotlin errors in `.uvue` files will block the test before the WebSocket channel is even established. Full workflow → **`uniautotest`** skill.

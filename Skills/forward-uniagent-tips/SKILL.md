---
name: "forward-uniagent-tips"
description: "Guidance for dispatching tasks to HBuilderX uni-agent via the forward-uniagent skill. Covers prompt truncation workarounds, multi-turn dialogue handling, and Telegram-bound session key patterns."
---

# forward-uniagent Tips

Practical guidance for communicating with HBuilderX uni-agent via `forward-uniagent` skill. Based on observed failures and successful workarounds.

## 路径代号解析（执行前必做）

本文档中的代号必须在执行前换成本机实际路径：

- `<HBX_CLI>` / `<HBX_HOME>`：读 `%USERPROFILE%\.hbx-common.json` 的 `hbxCli` / `hbxHome`
- `<PROJECT_ROOT>`：当前任务的专案根目录（由派工 brief 指定，不写死）
- `<OPENCLAW_HOME>`：OpenClaw 状态目录（通常是 `%USERPROFILE%\.openclaw`）
- `%APPDATA%`：使用者 AppData 目录

## When to Use

- Using `forward-uniagent` skill to dispatch tasks to HBuilderX uni-agent
- uni-agent reports "task description truncated" or "incomplete information"
- Multi-turn dialogue with uni-agent (asking questions, clarifications)

## Prompt Truncation Workaround

### Symptom
uni-agent reports the prompt is truncated, especially content containing:
- Long text (> ~800 characters)
- Special characters like `→` (arrow symbol)
- Chinese punctuation or complex formatting

### Solution
1. **Simplify the prompt**: Remove special characters, shorten descriptions
2. **Write to file first**: Always use `write` tool to write prompt to `forward-prompt.txt` (not inline in PS1)
3. **Test with minimal version**: Start with the core research question, add context only after confirmed working

### Example
```
# ❌ Truncated
"拍照→上传到 Windows→立即永久删除"

# ✅ Works
"Android 15 永久删除照片：现有 File.delete() 只进回收筒，需要研究 MediaStore.createDeleteRequest 方案"
```

## Multi-turn Dialogue Pattern

When uni-agent asks questions or requests clarification:

### Step-by-step
1. **Read uni-agent's question** from exec output
2. **Update `forward-prompt.txt`** with the additional information
3. **Re-run the SAME PS1** — do NOT create a new PS1 file
4. **Keep `--continue true`** — never change to `--continue false`

### Key Rule
```
Same PS1 file + Same session = context preserved
New PS1 or --continue false = context killed
```

### Example Flow
```
# Turn 1: Initial task (truncated by uni-agent)
write forward-prompt.txt → exec PS1 → uni-agent asks for clarification

# Turn 2: Provide missing info
update forward-prompt.txt → exec SAME PS1 → uni-agent proceeds with research

# Turn 3: Continue
update forward-prompt.txt → exec SAME PS1 → uni-agent completes task
```

## Completion Check

After uni-agent reports "research done" or "task complete":
- [ ] Confirm report file was written to the expected path
- [ ] Read the report to verify content

## sessions_send Session Key Pattern for Telegram-bound Agents

### Symptom
`sessions_send` to a Telegram-bound agent (e.g., team-reviewer, team-chief-of-staff) fails with:
```
"Session creation publication owner is no longer current"
```

### Root Cause
Telegram-bound agents create sessions per (agent, channel, user) tuple. Using `agent:team-reviewer:main` creates a session in the wrong context.

### Solution
Use the Telegram-bound session key pattern:
```
agent:<agent-id>:telegram:direct:<user-id>
```

**Examples:**
| Agent | Correct sessionKey |
|-------|-------------------|
| team-reviewer | `agent:team-reviewer:telegram:direct:942566572` |
| team-chief-of-staff | `agent:team-chief-of-staff:telegram:direct:942566572` |

**Wrong:** `agent:team-reviewer:main` — fails for Telegram-bound agents

### Decision Rule
```
Is target agent bound to Telegram?
├── YES → agent:<agent-id>:telegram:direct:<user-id>
└── NO  → agent:<agent-id>:<collaboration-name>
```

## Troubleshooting Reference

| Symptom | Cause | Fix |
|---------|-------|-----|
| "Task description truncated" | Prompt too long or has `→` symbol | Simplify, remove special chars |
| "Missing information: platform/uniCloud/etc" | uni-agent needs clarification | Update prompt with missing info, re-run |
| No output for 60+ seconds | Session dead or HBuilderX stuck | Ask user to restart HBuilderX GUI |
| `cli.exe not recognized` | Missing full path | Use `& '<HBX_CLI>'` with `&` and full path |
| sessions_send "publication owner" error | Wrong sessionKey for Telegram agent | Use `agent:<agent-id>:telegram:direct:<user-id>` pattern |
| Accidentally overwrote a file | `write` tool overwrites without confirmation | Use `git checkout HEAD -- <file>` to restore |

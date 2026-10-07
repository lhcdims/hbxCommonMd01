---
name: "crossagentcommunication"
description: "Enable communication between two OpenClaw agents using sessions_send with a persistent session key, similar to uni-agent's --continue true flag."
---

# CrossAgentCommunication Skill

Enable communication between two OpenClaw agents (e.g., main agent and a subordinate/assistant agent) using a persistent session.

## When to Use

- You need to coordinate with another agent (e.g., chief-of-staff, assistant, coordinator)
- You want the target agent to remember context across multiple message exchanges
- You want to delegate tasks to another agent while maintaining conversation continuity

## Core Concept

OpenClaw agents communicate via the `sessions_send` tool using a **persistent session key**. This is analogous to `uni-agent`'s `--continue true` flag — the session maintains context across multiple exchanges.

## Prerequisites

- Two or more OpenClaw agents configured on the same Gateway
- Access to the `sessions_send` tool
- Optionally: Telegram bot channel configured (if you want to expose dialogue to users via Telegram)

## Steps

### Step 1: Identify the Target Agent ID

List all configured agents to find the target agent's ID:

```bash
gateway agents list
```

Or via OpenClaw dashboard (Agents section).

**Example:** `chief-of-staff`, `assistant`, `coordinator`

### Step 2: Choose a Session Key Pattern

Use a consistent session key format:

```
agent:<target-agent-id>:<collaboration-name>
```

Where `<collaboration-name>` is a descriptive name for this collaboration (e.g., `main`, `collaboration`).

**Example:** `agent:chief-of-staff:main`

### Step 3: Send Messages with sessions_send

Use the `sessions_send` tool with the identical `sessionKey` every time:

```
Tool: sessions_send
{
  "agentId": "<target-agent-id>",
  "sessionKey": "agent:<target-agent-id>:<collaboration-name>",
  "message": "<your-message-content>",
  "timeoutSeconds": 60
}
```

**Key rule:** Use the **identical** `sessionKey` for every message to the same agent. Even a slight difference creates a new session.

### Step 4: Receive Responses

The response includes:
- `reply`: The target agent's reply text
- `sessionKey`: Confirms which session was targeted
- `runId`: Unique run identifier

### Step 5: Continue the Conversation

Call `sessions_send` again with the **same `sessionKey`** to continue. The target agent remembers previous context.

**Example flow:**
```
// Turn 1
{"agentId": "chief-of-staff", "sessionKey": "agent:chief-of-staff:main", "message": "Hello, I need information about the weather."}

// Turn 2
{"agentId": "chief-of-staff", "sessionKey": "agent:chief-of-staff:main", "message": "What about tomorrow?"}
// chief-of-staff remembers "weather" context from Turn 1
```

### Step 6: View Dialogue History

View session dialogues in:
- **OpenClaw dashboard** → Select target agent → Main Session
- **Sessions history** → Use `sessions_search` or `sessions_history` tools

## Viewing Dialogue in Telegram (Optional)

If you want users to see agent-to-agent dialogue in Telegram:

1. Create a Telegram group with:
   - The user
   - The main agent's Telegram bot
   - The subordinate agent's Telegram bot

2. Configure routing so the main agent can see group messages (must be @mentioned)

3. Relay strategy: When the subordinate agent responds, the main agent forwards relevant parts to the Telegram group using:

```
Tool: message
{
  "action": "send",
  "channel": "telegram",
  "target": "<group-chat-id>",
  "message": "<content>"
}
```

## Key Differences from uni-agent's --continue true

| Aspect | uni-agent `--continue true` | OpenClaw sessions_send with sessionKey |
|--------|----------------------------|---------------------------------------|
| Mechanism | CLI flag | Tool parameter |
| Context | Automatic when flag is set | Explicit via identical sessionKey |
| Specification | `--continue true` argument | `sessionKey` field in tool call |
| Session identification | Same CLI process | Session key string |

## Troubleshooting

**Problem:** Agent doesn't remember previous messages.
**Solution:** Ensure you're using the **identical** `sessionKey` every time (check for whitespace or case differences).

**Problem:** `sessions_send` returns error or is unavailable.
**Solution:** Check that `sessions_send` is available in your agent's tool list.

**Problem:** Response starts fresh every time.
**Solution:** The target agent may have restarted. Start a new collaboration with a fresh `sessionKey`.

## Summary Checklist

- [ ] Identify the target agent ID
- [ ] Choose a consistent `sessionKey` pattern
- [ ] Use `sessions_send` with identical `sessionKey` for all messages
- [ ] View dialogue in OpenClaw dashboard if needed
- [ ] Optionally relay to Telegram group for user visibility

---
**Last updated:** 2026-10-06

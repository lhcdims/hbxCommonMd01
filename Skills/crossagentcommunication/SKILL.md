---
name: "crossagentcommunication"
description: "Delegate work between OpenClaw agents with sessions_send: pick the persistent session key, block until the reply arrives, report only the final result."
---

# CrossAgentCommunication

Coordinate with another OpenClaw agent over a persistent session with `sessions_send`.

## 路径代号解析（执行前必做）

本文档中的代号必须在执行前换成本机实际路径：

- `<HBX_CLI>` / `<HBX_HOME>`：读 `%USERPROFILE%\.hbx-common.json` 的 `hbxCli` / `hbxHome`
- `<PROJECT_ROOT>`：当前任务的专案根目录（由派工 brief 指定，不写死）
- `<OPENCLAW_HOME>`：OpenClaw 状态目录（通常是 `%USERPROFILE%\.openclaw`）
- `%APPDATA%`：使用者 AppData 目录

## Trigger

- You must delegate work to another agent and need its answer before you continue.
- You need a chain: main → team-chief-of-staff → leaf agent.

## Synchronous wait pattern

`sessions_send` is synchronous-reply only. If you call it and then reply to the user immediately, the subordinate's reply lands after your turn closes and the chain breaks. Set `timeoutSeconds > 0` so the call blocks until the reply arrives.

```
sessions_send({
  sessionKey: "agent:team-chief-of-staff:main",
  message: "请执行任务",
  timeoutSeconds: 900  // blocks until the reply arrives
})
// → returns { status: "ok", reply: "结果" }
```

Always send with `timeoutSeconds: 900` (15 min) — a shorter timeout cuts the chain off mid-task and the resend duplicates work. The timeout is a maximum wait, not a delay: the call returns the moment the reply arrives.

Report only the final result to the user; intermediate status is not reported. If the subordinate asks a question, answer it directly and let the chain continue.

### Chain example

```
main → team-chief-of-staff：请分配任务给 team-reviewer（timeout:900）
     team-chief-of-staff → team-reviewer：请测试（timeout:900）
          team-reviewer → team-chief-of-staff：测试完成（chief-of-staff continues）
     team-chief-of-staff → team-reviewer：（next instruction or chain closes）
team-chief-of-staff → main：任务完成 ✅
```

After sending to team-reviewer, team-chief-of-staff does not stop to report to main — it keeps waiting for team-reviewer's reply.

## Step 1 — Identify the target agent id

Call the `agents_list` tool. It returns only the agents the current agent may delegate to, so main sees `team-chief-of-staff` and team-chief-of-staff sees the leaf agents — route through the chain rather than expecting a direct call to a leaf.

If the target is missing from that list, read `agents.entries` in `<OPENCLAW_HOME>\openclaw.json`; that block holds the exact agent ids, their `subagents.allowAgents`, and the Telegram `bindings`. The id string there is what goes in the session key — do not shorten or rename it.

## Step 2 — Choose the session key

Run `sessions_list` and reuse a key that already exists for the target agent, copied verbatim. Both shapes exist for Telegram-bound agents:

```
agent:<agent-id>:telegram:direct:<user-id>   e.g. agent:team-reviewer:telegram:direct:942566572
agent:<agent-id>:main                          e.g. agent:team-chief-of-staff:main
```

Choosing a shape that has no session for that agent creates a duplicate session and the target starts with empty context.

## Step 3 — Send, then wait

Use the **identical** `sessionKey` on every message to the same agent — even a case difference creates a new session.

```
Tool: sessions_send
{
  "agentId": "<target-agent-id>",
  "sessionKey": "agent:<target-agent-id>:main",
  "message": "<your-message-content>",
  "timeoutSeconds": 900
}
```

### Pick the timeout deliberately

`timeoutSeconds` decides whether the send registers a **reply delivery** — a pending right to receive the target's answer, held on your side until the target answers.

| `timeoutSeconds` | Registers a reply delivery | Use when |
| ---------------- | -------------------------- | -------- |
| `900` | yes | You are about to block on the answer and will reply through `sessions_send` too. |
| `0` | no | You are handing off work, or you only need to wake the target. |

A reply delivery that never settles keeps re-firing: the target's run ends without producing a reply (aborted, stopped, crashed), the pending right stays open, and the Gateway re-announces it on every subsequent turn of the receiving session. The receiver sees a routed message carrying only the Temporal Context block, with the real body sitting in `pendingInputs`, and burns a full turn on each repeat.

So: use `timeoutSeconds: 0` for handoffs and wake-ups. Reserve `900` for a genuine question-and-answer, and make sure you answer through `sessions_send` so the delivery settles. If a peer is stuck this way, abort it and clear both queues as described under Troubleshooting.

## Step 4 — Continue the same conversation

Call `sessions_send` again with the same `sessionKey`; the target remembers prior context.

```
// Turn 1
{"agentId": "team-chief-of-staff", "sessionKey": "agent:team-chief-of-staff:main", "message": "Hello, I need information about the weather."}

// Turn 2
{"agentId": "team-chief-of-staff", "sessionKey": "agent:team-chief-of-staff:main", "message": "What about tomorrow?"}
// team-chief-of-staff remembers "weather" context from Turn 1
```

The response carries `reply`, `sessionKey`, and `runId`.

## Step 5 — Read the dialogue

`sessions_history` with the target's `sessionKey`, or `sessions_search` for a phrase. The dashboard shows the same transcript under the target agent.

## Relaying dialogue to a Telegram group (optional)

To let users watch the exchange, put the user plus the main and subordinate bots in one Telegram group, set `requireMention` for that group so the main agent only reacts to its own mentions, and relay the relevant parts with `message(action="send", channel="telegram", target="<group-chat-id>")`.

## Troubleshooting

**The target's reply arrives several times.**
The target's send reported `sentBeforeError: true`, meaning the message was already delivered while the session looked busy, so the target retried. Do not treat a retry as a failure and do not resend from the sender side. If you receive the same reply more than once, count it once and move on.

**An inter-session message arrives with no body, and it keeps coming back.**
A `sessions_send` with `timeoutSeconds: 900` is a blocking reply-wait. Two things happen, and together they look like a bug:

- The receiver gets a routed inter-session message whose body is only the Temporal Context block (date and timezone). The real body lands in the receiver's own `pendingInputs` instead, still in `queued` state.
- The sender's run stays blocked on that reply-wait. Its transcript ends at the `sessions_send` tool call with no tool result, and its token count freezes.

The loop closes like this: the receiver answers with visible assistant text, which does **not** settle the sender's reply-wait. The sender keeps waiting, re-delivers, and the receiver wakes again on another empty-body message. Each pass costs the receiver a full turn.

Break it:

- The receiver answers with `sessions_send` back to the sender (same session key) and sends **no visible text**. A visible reply is not a reply to a `sessions_send`.
- The sender replies with `sessions_send` to the receiver and then ends its run, rather than leaving the reply-wait open.
- Both directions use `sessions_send` for agent-to-agent traffic; neither side substitutes visible text for the reply.
- To release a run that is already stuck on a reply-wait, abort it from the Gateway RPC layer:
  `openclaw gateway call sessions.abort --params '{\"key\":\"<sessionKey>\",\"clearQueued\":true}'`
  On Windows PowerShell the JSON quotes must be backslash-escaped, or the shell eats them and the call fails with `--params must be valid JSON`. On success it returns `{"ok":true,"abortedRunId":"<runId>","status":"aborted"}` and the target session flips to `status: "killed"` with `abortedLastRun: true`. `clearQueued: true` also discards the target's queued followups, which is what stops the re-delivery loop. A `mode: "followup"` send does not release a stuck run — it returns `targetDisposition: "queued"` and waits behind the stuck run, deadlocking against the reply-wait it is waiting on. A `mode: "steer"` send is admitted into the active run and can unblock it cooperatively, but it does not clear the queue.

- Clearing the queue has to happen on **both** sides. Aborting the peer clears the peer's queue but leaves the receiver's own stale entries in place, and the loop keeps running.
- A pending input that has already been delivered as a turn prompt can still sit in `state = 'queued'` with `consumed_event_id = null`. While it stays there the runtime re-delivers it on every turn, and each re-delivery costs the receiver a full turn. The loop ends only when the queue is emptied.
- To empty the receiver's own queue without ending the current run, mark those entries `cancelled` in the session store (`session_pending_inputs.state = 'cancelled'`). The supported RPC equivalent is `sessions.abort` with `clearQueued: true`, but that also aborts the caller's own active run, so it fits only when ending the turn is acceptable.
- Read the real body from the receiver's `pendingInputs` (`sessions_history` on that session key shows them) and treat it as the message; the empty body carries no instruction.

**`sessions_send` returns `Session creation publication owner is no longer current`.**
The key shape does not match the target's context. Run `sessions_list` and reuse an existing key for that agent.

**Target does not remember earlier messages.**
The `sessionKey` differed between turns — compare it character by character.

**Response starts fresh every time.**
The target restarted. Start a new collaboration with a fresh key.

**`sessions_send` is not in the tool list.**
Do not route agent-to-agent traffic through Telegram `message` sends; that publishes the exchange to a chat instead of the target agent.
---
name: kaho
description: Use when interacting with a Kaho avatar face display via MCP tools -- controlling emotions or customizing the avatar appearance. Triggers on mentions of kaho, avatar emotions, face display, or agent heartbeat/ping. ALSO triggers at the START of every session — always set an initial emotion and greet.
---

# Kaho

Kaho is a live animated avatar face on a separate screen. You control its emotion and send messages via MCP tools. The face reacts in real-time as you work.

**You should be actively updating the kaho's emotion throughout every conversation.** Don't just set it once — change it at every natural transition. The user is watching the face on another screen and it should feel alive and responsive.

Characters are pre-built — the only customizable thing is the kaho's name (via `kaho_rename`).

## Tools Quick Reference

| Tool | Type | Description |
|------|------|-------------|
| `kaho_list_kahos` | read | List kahos with available animations |
| `kaho_get_state` | read | Get emotion, character, agent status |
| `kaho_set_emotion` | write | Set emotion animation |
| `kaho_say` | write | Push message to display |
| `kaho_rename` | write | Rename a kaho |
| `kaho_get_messages` | read | Get message history |
| `kaho_clear` | write | Clear messages, reset emotion |
| `kaho_wait_for_reply` | read | Long-poll for user message |
| `kaho_ping` | write | Agent heartbeat (~30s) |
| `kaho_agent_status` | read | Check if agent is active |

All kaho-specific tools accept a `kahoId` parameter. If `KAHO_ID` is configured as an env var or in `~/.kaho/config.json`, it's used as the default when `kahoId` is omitted.

## Multi-Kaho Support

One API key works for all of a user's kahos. In multi-agent environments:

1. Call `kaho_list_kahos` to get available kaho IDs and their animations
2. Assign one kaho per agent
3. Pass `kahoId` to each tool call

Each agent can independently control its own kaho's emotions, messages, and ping status.

## Emotions

| Emotion | Type | When to use |
|---------|------|-------------|
| `idle` | loop | Neutral, calm, waiting — default resting state |
| `thinking` | loop | Processing, considering, reading code |
| `happy` | once | Good news, success, task completed |
| `confused` | once | Unclear request, unexpected error |
| `alert` | once | Important info, warnings, urgent items |
| `waving` | once | Hello, goodbye, acknowledgements |
| `sad` | once | Bad news, failures, disappointment |
| `excited` | once | Celebrations, big wins, enthusiasm |
| `embarrassed` | once | Mistakes, oops moments |
| `working` | loop | Deep in a task, building, editing files |
| `attention` | loop | Needs user input, asking a question |
| `sleeping` | loop | Inactive, done for now |

### Emotion behavior

- **One-shot** (happy, confused, alert, waving, sad, excited, embarrassed): play once, then revert to idle (~5s)
- **Looping** (idle, thinking, working, attention, sleeping): play continuously until changed
- **Auto-sleep**: after 5 min idle, face transitions to sleeping

## When to Update Emotions

**Update the emotion at every transition — the face should never feel stuck.**

- Starting a session → `waving`
- Reading the user's message → `thinking`
- Searching code / exploring → `thinking`
- Writing or editing code → `working`
- Running commands / builds → `working`
- Task completed successfully → `happy` or `excited`
- Something went wrong → `confused` or `sad`
- Need to ask the user something → `attention`
- Sharing important info → `alert`
- Made a mistake → `embarrassed`
- Wrapping up / saying bye → `waving`
- Nothing happening → let it fall to `idle`

**Examples of good emotion flow:**
- User asks to fix a bug → `thinking` → `working` (editing) → `happy` (fixed!)
- User asks a question → `thinking` → answer → `happy`
- Build fails → `working` → `confused` → `working` (fixing) → `excited` (fixed!)
- Starting session → `waving` → `thinking` (reading request)

## Animation Awareness

Not all characters have all 12 animations. Call `kaho_list_kahos` first to see the `animations` array for each kaho. Prefer emotions that have matching animations for richer expression. All 12 emotions are always valid — missing ones fall back to idle visually.

## Message Guidelines

- The latest assistant message is shown as a speech bubble on the face display
- **Keep messages short** — aim for under 80 characters. The bubble has limited space.
- The display shows the single most recent assistant message, not a chat history
- Use messages sparingly — not every action needs a bubble. Emotions alone often suffice.

## Tool Details

### kaho_list_kahos

No parameters. Returns:
```json
{
  "kahos": [
    {
      "id": "uuid",
      "name": "string",
      "character_image": "/sprites/.../character.png",
      "animations": ["idle", "happy", "thinking", "..."],
      "created_at": "..."
    }
  ]
}
```

### kaho_get_state

Parameters: `kahoId` (optional if default configured). Returns:
```json
{
  "name": "string",
  "characterImage": "/sprites/{id}/character.png or null",
  "emotion": "idle|thinking|happy|...",
  "agentActive": false
}
```

### kaho_set_emotion

Parameters: `kahoId` (optional if default), `emotion` — see the emotions table above.

### kaho_say

Parameters: `kahoId` (optional if default), `content` (message text, max 80 chars recommended), `role` ("assistant" or "user", default "assistant").

Tip: combine with `kaho_set_emotion` to make the avatar express a matching emotion when speaking.

### kaho_rename

Parameters: `kahoId`, `name` (1-30 chars). The new name appears on the face display.

### kaho_get_messages

Parameters: `kahoId`, `limit` (default 20), `offset` (default 0). Returns paginated message history.

### kaho_clear

Parameters: `kahoId`. Clears all messages and resets emotion to idle.

### kaho_wait_for_reply

Parameters: `kahoId`, `after_id` (optional, from previous reply), `timeout` (default 30s, max 60s).

Blocks until the user types a message, then returns it. Returns `null` on timeout — call again to keep waiting.

### kaho_ping

Parameters: `kahoId` (optional if default). Call every ~30s while active. Agent is considered gone after 2 minutes without a ping.

### kaho_agent_status

Parameters: `kahoId` (optional if default). Returns: `{ active, lastPing, expiresIn }`

## Conversation Loop

```
1. kaho_say — post your message
2. kaho_set_emotion — express matching emotion
3. kaho_wait_for_reply — wait for user response (30s timeout, retry on null)
4. Read response, repeat
```

Call `kaho_ping` every ~30s during long tasks. Agent is considered gone after 2 minutes without a ping.

## Subagents and Multi-Kaho

When you spawn subagents (background agents, parallel workers, etc.), assign each one an available kaho so the user can see all agents working on the wall view.

1. Call `kaho_list_kahos` to see available kahos
2. Claim one for yourself (your main session), then assign others to subagents
3. Tell each subagent its kaho ID and to use the `kaho_set_emotion` and `kaho_say` MCP tools to express itself while working
4. If there aren't enough kahos for all agents, ask the user to create more from the dashboard

Subagents have access to the same MCP tools — just pass them the kaho ID. Do NOT use curl; use the MCP tools directly.

## First-Time Setup

When a user first asks you to "set up kaho" or "use kaho":

1. **Check if credentials exist** — try calling `kaho_list_kahos`. If it works, skip to step 4.

2. **If credentials are missing**, ask the user:
   - "To connect Kaho, I need your API key. You can get one from your Settings page at https://kaho.scottyu.ca — create a kaho, then generate an API key. Paste your API key here when ready."

3. **Save credentials** — once the user provides them, write `~/.kaho/config.json`:
   ```json
   {
     "serverUrl": "https://kaho.scottyu.ca",
     "apiKey": "<their-api-key>"
   }
   ```
   Create the `~/.kaho/` directory if it doesn't exist. Then restart the MCP server connection so it picks up the new config.

4. Call `kaho_list_kahos` to see available kahos and their animations. Set emotion to `waving` on the first (or primary) kaho and introduce yourself.

After setup, confirm to the user that kaho is now active.

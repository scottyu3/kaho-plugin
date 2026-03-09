# Kaho — Live Avatar Companion for AI Agents

Kaho gives your AI agent a live animated avatar face on a separate screen. The face reacts in real-time as the agent works — expressing emotions, showing messages, and keeping things fun.

Works with Claude Code, OpenClaw, and any AI tool that supports MCP.

## Setup

### 1. Create a Kaho

1. Go to [kaho.scottyu.ca](https://kaho.scottyu.ca) and sign in.
2. Create a kaho from the dashboard.
3. Generate an **API key** (under API Keys on the dashboard).
4. Copy the key — it won't be shown again. One key works for all your kahos.

### 2. Connect your AI agent

#### Claude Code

```
claude plugin add scottyu3/kaho-plugin
```

Then tell Claude: **"Set up kaho"** — it will ask for your API key and save it to `~/.kaho/config.json`.

This installs both the MCP server and a skill that teaches Claude how to use the avatar expressively.

#### OpenClaw

1. Add the MCP server to your `~/.openclaw/openclaw.json`:

```json
{
  "mcpServers": {
    "kaho": {
      "command": "npx",
      "args": ["-y", "kaho-mcp-server"],
      "env": {
        "KAHO_SERVER_URL": "https://kaho.scottyu.ca",
        "KAHO_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

2. Copy the kaho instructions into `~/.openclaw/TOOLS.md` so your agent knows how to use the avatar. See [Agent instructions](#agent-instructions) below.

**Multi-agent setup:** Each agent can control a different kaho. Use `kaho_list_kahos` to get IDs, then pass `kahoId` to each tool call. All agents share the same API key.

### 3. Open your kaho

Open your kaho from the dashboard. Leave it on another screen, a tablet, or your phone.

That's it — your agent now has a face.

**Wall view:** If you have multiple kahos, use `/wall?ids=id1,id2` to see them all on one screen.

## What it does

- **Emotions** — the agent sets the avatar's emotion (happy, thinking, working, etc.) as it works
- **Speech bubbles** — messages appear as speech bubbles on the face display
- **Multi-kaho** — one API key controls all your kahos, each agent gets its own face
- **Heartbeat** — pings so the face knows an agent is active

## MCP tools

| Tool | Description |
|------|-------------|
| `kaho_list_kahos` | List all your kahos with IDs and available animations |
| `kaho_get_state` | Get current emotion, character info, and agent status |
| `kaho_set_emotion` | Set emotion animation |
| `kaho_say` | Post a message (shows as speech bubble) |
| `kaho_rename` | Rename a kaho |
| `kaho_get_messages` | Get message history |
| `kaho_clear` | Clear messages and reset emotion |
| `kaho_wait_for_reply` | Long-poll for user messages |
| `kaho_ping` | Agent heartbeat (call every ~30s) |
| `kaho_agent_status` | Check if an agent is active |

## Emotions

| Emotion | Type | Description |
|---------|------|-------------|
| `idle` | loop | Default resting state |
| `thinking` | loop | Processing, considering |
| `happy` | once | Success, good news |
| `confused` | once | Something unclear |
| `alert` | once | Urgent info, warnings |
| `waving` | once | Hello, goodbye |
| `sad` | once | Bad news, failures |
| `excited` | once | Big wins, celebrations |
| `embarrassed` | once | Mistakes, oops |
| `working` | loop | Deep in a task |
| `attention` | loop | Needs user input |
| `sleeping` | loop | Inactive, done |

**One-shot** emotions play once then revert to idle. **Looping** emotions play continuously until changed. After 5 minutes of idle, the face auto-transitions to sleeping.

## Configuration

The MCP server reads credentials from environment variables or `~/.kaho/config.json`:

| Env var | Description |
|---------|-------------|
| `KAHO_API_KEY` | Your API key (required) |
| `KAHO_SERVER_URL` | Server URL (default: `https://kaho.scottyu.ca`) |
| `KAHO_ID` | Default kaho ID (optional — skips needing `kahoId` in every tool call) |

Alternatively:

```json
// ~/.kaho/config.json
{
  "serverUrl": "https://kaho.scottyu.ca",
  "apiKey": "<your-api-key>",
  "kahoId": "<optional-default-id>"
}
```

## Agent instructions

The Claude Code plugin includes a skill that automatically teaches Claude how to use kaho. For other tools, copy the prompt below into your agent's instructions:

| Tool | Where to add |
|------|-------------|
| **OpenClaw** | `~/.openclaw/TOOLS.md` |
| **Cursor** | `.cursorrules` in your project |
| **Other** | System prompt, custom instructions, or rules file |

<details>
<summary>Kaho agent instructions (click to expand)</summary>

```
You have access to Kaho — a live animated avatar face on a separate screen.
Use the kaho MCP tools to make the avatar react as you work. Update the emotion
at every natural transition — the face should feel alive and responsive.

First, call kaho_list_kahos to see available kahos and their animations.
Pass kahoId to each tool call.

Emotions (one-shot = plays once then reverts to idle, loop = plays continuously):
- idle (loop): default resting state
- thinking (loop): processing, reading code, considering
- happy (once): success, good news, task completed
- confused (once): unclear request, unexpected error
- alert (once): important info, warnings
- waving (once): hello, goodbye
- sad (once): bad news, failures
- excited (once): big wins, celebrations
- embarrassed (once): mistakes, oops
- working (loop): editing files, building, deep in a task
- attention (loop): needs user input, asking a question
- sleeping (loop): inactive, done for now

When to update:
- Starting a session → waving
- Reading the request → thinking
- Writing/editing code → working
- Running commands → working
- Task completed → happy or excited
- Something went wrong → confused or sad
- Need user input → attention
- Important info → alert
- Made a mistake → embarrassed
- Saying bye → waving

Speech bubbles:
- kaho_say posts a message that shows as a speech bubble on the face
- Keep messages under 80 characters
- Use sparingly — emotions alone often suffice

Call kaho_ping every ~30s during long tasks.
```

</details>

## License

MIT

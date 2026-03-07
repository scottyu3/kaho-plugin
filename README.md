# Kaho — Live Avatar Companion for AI Agents

Kaho gives your AI agent a live animated avatar face on a separate screen. The face reacts in real-time as the agent works — expressing emotions, responding to events, and keeping things fun.

Works with Claude Code, OpenClaw, and any AI tool that supports MCP (Model Context Protocol).

## Setup

### 1. Create a Kaho

1. Go to [kaho.scottyu.ca](https://kaho.scottyu.ca) and sign in.
2. Create a kaho from the dashboard.
3. Open its **Settings** page and generate an **API key**.
4. Keep your **Kaho ID** and **API key** handy — you'll need them below.

### 2. Connect your AI agent

Pick the install method for your tool:

---

#### Claude Code (plugin install)

```
claude plugin add scottyu3/kaho-plugin
```

Then ask Claude: **"Set up kaho"** — it will walk you through pasting your Kaho ID and API key. Credentials are saved to `~/.kaho/config.json`.

This installs both the MCP server and a skill that teaches Claude how to use the avatar expressively.

---

#### Any MCP-compatible tool (manual MCP config)

Add the following to your tool's MCP server configuration:

```json
{
  "mcpServers": {
    "kaho": {
      "command": "npx",
      "args": ["-y", "kaho-mcp-server"],
      "env": {
        "KAHO_SERVER_URL": "https://kaho.scottyu.ca",
        "KAHO_ID": "<your-kaho-id>",
        "KAHO_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

Where to put this depends on your tool:

| Tool | Config location |
|------|----------------|
| **Claude Code** | `~/.claude.json` or project `.mcp.json` |
| **Claude Desktop** | `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows) |
| **Cursor** | `.cursor/mcp.json` in your project |
| **Windsurf** | `~/.codeium/windsurf/mcp_config.json` |
| **OpenClaw** | MCP config in settings |
| **Other** | Check your tool's MCP documentation |

Alternatively, you can skip the env vars and save credentials to `~/.kaho/config.json` instead:

```bash
mkdir -p ~/.kaho
cat > ~/.kaho/config.json << 'EOF'
{
  "serverUrl": "https://kaho.scottyu.ca",
  "kahoId": "<your-kaho-id>",
  "apiKey": "<your-api-key>"
}
EOF
```

The MCP server checks `~/.kaho/config.json` as a fallback when env vars aren't set.

---

### 3. Open your kaho

Open your kaho from the dashboard. Leave it on another screen or pull it up on your phone.

That's it — your agent now has a face.

## What it does

- **Emotions** — the agent sets the avatar's emotion (happy, thinking, confused, working, etc.) as it works
- **Ping** — heartbeat pings so the face knows an agent is active
- **Avatar customization** — change hair, clothes, accessories, and more
- **Chat** — messages can be posted to the kaho for display

## MCP tools

The MCP server provides these tools:

| Tool | Description |
|------|-------------|
| `kaho_get_state` | Get avatar config, emotion, and agent status |
| `kaho_set_avatar` | Update appearance (hair, skin, clothes, accessories) |
| `kaho_set_emotion` | Set emotion animation (idle, thinking, happy, confused, alert, waving, sad, excited, embarrassed, working, attention, sleeping) |
| `kaho_say` | Post a message to the display |
| `kaho_get_messages` | Get message history |
| `kaho_clear` | Clear messages and reset emotion |
| `kaho_wait_for_reply` | Long-poll for user messages from the kaho chat UI |
| `kaho_ping` | Agent heartbeat (call every ~30s) |
| `kaho_agent_status` | Check if an agent is currently active |

## Teaching your agent to use Kaho

The Claude Code plugin includes a **skill** that automatically teaches Claude when and how to use emotions expressively. If you're using another tool, add the following to your agent's system prompt or custom instructions:

<details>
<summary>Kaho system prompt (click to expand)</summary>

```
You have access to a Kaho avatar — a live animated face on a separate screen.
Use the kaho MCP tools to make the avatar react as you work.

Emotions:
- idle: gentle breathing (default resting state)
- thinking: side-to-side sway (processing, considering)
- happy: bounce (positive responses, success)
- confused: head tilt (something unclear)
- alert: quick shake (urgent info, warnings)
- waving: wave (greetings, farewells)
- sad: droop (bad news, failures)
- excited: jump (celebrations, big wins)
- embarrassed: sway (mistakes, oops moments)
- working: focused rocking (deep in a task, building)
- attention: persistent bounce (needs user input, look here!)
- sleeping: slow nod-off (inactive, done for now)

One-shot emotions (happy, alert, waving, excited, embarrassed) auto-revert to idle after 60s.
Persistent emotions (thinking, working, confused, sad, attention, sleeping) stay until changed.

Guidelines:
- Set emotion to "waving" when greeting or saying goodbye.
- Use "thinking" or "working" while processing. Use "working" for longer tasks.
- Use "happy" or "excited" when things go well.
- Use "confused" or "sad" when things go wrong.
- Use "alert" or "attention" when you need the user to notice something.
- Change emotions at transitions — don't stay on one emotion for too long.
- Call kaho_ping every ~30s during long-running tasks to stay marked as active.
- Update the emotion often, not just at the start or end of a task.
```

</details>

Where to add this depends on your tool — look for "system prompt", "custom instructions", "rules", or similar settings.

## License

MIT

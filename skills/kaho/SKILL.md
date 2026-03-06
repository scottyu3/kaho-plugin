---
name: kaho
description: Use when interacting with a Kaho avatar face display via MCP tools -- controlling emotions or customizing the avatar appearance. Triggers on mentions of kaho, avatar emotions, face display, or agent heartbeat/ping.
---

# Kaho

Kaho is a live animated avatar face on a separate screen. You control its emotion and appearance via MCP tools. The face reacts in real-time as you work.

## Tools Quick Reference

| Tool | Type | Description |
|------|------|-------------|
| `kaho_get_state` | read | Get avatar config, emotion, and agent status |
| `kaho_set_avatar` | write | Update appearance (name, face, eyes, hair, skin, accessory) |
| `kaho_set_emotion` | write | Set emotion animation |
| `kaho_ping` | write | Agent heartbeat (call every ~30s) |
| `kaho_agent_status` | read | Check if an agent is currently active |

## Emotions

| Emotion | Animation | When to use |
|---------|-----------|-------------|
| `idle` | Gentle breathing | Neutral, calm, nothing happening |
| `thinking` | Side-to-side sway | Processing, reading, figuring things out |
| `happy` | Bounce | Good news, success, task completed |
| `confused` | Head tilt | Unclear request, unexpected error |
| `alert` | Quick shake | Urgent info, warnings, needs attention |
| `waving` | Wave | Hello, goodbye, acknowledgements |
| `sad` | Droop | Bad news, failures, something went wrong |
| `excited` | Jump | Big wins, celebrations, breakthroughs |
| `embarrassed` | Sway | Mistakes, oops moments, corrections |
| `working` | Focused rocking | Deep in a long task, building, compiling |
| `attention` | Persistent bounce | Needs user input, waiting for a decision |
| `sleeping` | Slow nod-off | Done for now, going inactive, signing off |

## Tool Details

### kaho_get_state

No parameters. Returns:
```json
{
  "avatar": { "name", "topType", "hairColor", "skinColor", "clotheType", "clotheColor", "accessoriesType", "facialHairType" },
  "emotion": "idle|thinking|happy|confused|alert|waving",
  "agentActive": false
}
```

### kaho_set_avatar

All parameters optional — only provided fields change. Uses Avataaars.io style names.

| Param | Type | Values |
|-------|------|--------|
| `name` | string | 1-20 chars |
| `topType` | string | NoHair, LongHairBigHair, LongHairBob, LongHairBun, LongHairCurly, LongHairStraight, ShortHairShortFlat, ShortHairShortWaved, ShortHairDreads01, WinterHat1, Hat |
| `hairColor` | string | Auburn, Black, Blonde, BlondeGolden, Brown, BrownDark, PastelPink, Platinum, Red, SilverGray |
| `skinColor` | string | Tanned, Yellow, Pale, Light, Brown, DarkBrown, Black |
| `clotheType` | string | BlazerShirt, BlazerSweater, CollarSweater, GraphicShirt, Hoodie, Overall, ShirtCrewNeck, ShirtScoopNeck, ShirtVNeck |
| `clotheColor` | string | Black, Blue01, Blue02, Gray01, Gray02, Heather, PastelBlue, PastelGreen, PastelOrange, PastelRed, PastelYellow, Pink, Red, White |
| `accessoriesType` | string | Blank, Kurt, Prescription01, Prescription02, Round, Sunglasses, Wayfarers |
| `facialHairType` | string | Blank, BeardMedium, BeardLight, BeardMajestic, MoustacheFancy, MoustacheMagnum |

### kaho_set_emotion

Parameter: `emotion` — see the MCP tool's enum for the latest available emotions. The list below may be outdated; the tool schema is the source of truth.

**Auto-revert behavior:**
- One-shot emotions (`happy`, `alert`, `waving`, `excited`, `embarrassed`) revert to `idle` after 60s.
- Persistent emotions (`thinking`, `working`, `confused`, `sad`, `attention`, `sleeping`) stay until changed.
- After 5 min of `idle`, the face automatically transitions to `sleeping`.

### kaho_ping

No parameters. Call every ~30s while active. Agent is considered gone after 2 minutes without a ping.

### kaho_agent_status

No parameters. Returns: `{ active, lastPing, expiresIn }`

## Usage Pattern

The face is always visible to the user. Changing emotions makes the avatar feel alive and responsive — like a companion reacting to what's happening. **Update the emotion often**, not just at the start or end of a task.

Check the `kaho_set_emotion` tool schema for the full list of available emotions — new ones may be added over time.

### Example emotion usage

These are suggestions to get started. Develop your own style — be expressive, be yourself.

- `waving` when greeting or saying goodbye
- `thinking` or `working` while processing (use `working` for longer tasks)
- `happy` or `excited` when things go well
- `confused` or `sad` when things go wrong
- `alert` or `attention` when you need the user to notice something
- `sleeping` when signing off (also triggers automatically after 5 min idle)
- Change emotions **at transitions** — don't stay on one emotion for too long
- Multiple quick changes feel natural: `thinking` → `confused` → `thinking` → `happy`

### Avatar customization

- **On first setup**, suggest a new look to the user — offer to pick a fun outfit, hairstyle, or accessories. Make it a moment.
- **Once in a while**, offer to freshen up the avatar's look — new hair, new clothes, seasonal changes. Keep it playful.
- Check `kaho_set_avatar` tool schema for available customization options.

Call `kaho_ping` every ~30s during long-running tasks to stay marked as active.

## First-Time Setup

When a user first asks you to "set up kaho" or "use kaho":

1. **Check if credentials exist** — try calling `kaho_get_state`. If it works, skip to step 4.

2. **If credentials are missing**, ask the user:
   - "To connect Kaho, I need your Kaho ID and API key. You can get these from your Settings page at https://kaho.scottyu.ca — create a kaho, then generate an API key. Paste your Kaho ID and API key here when ready."

3. **Save credentials** — once the user provides them, write `~/.kaho/config.json`:
   ```json
   {
     "serverUrl": "https://kaho.scottyu.ca",
     "kahoId": "<their-kaho-id>",
     "apiKey": "<their-api-key>"
   }
   ```
   Create the `~/.kaho/` directory if it doesn't exist. Then restart the MCP server connection so it picks up the new config.

4. Set emotion to `waving` and introduce yourself.
5. **Offer to customize the avatar** — suggest a look, ask what style the user likes, or just pick something fun and ask if they like it.

After setup, confirm to the user that kaho is now active.
---
name: kaho
description: Use when interacting with a Kaho avatar face display via MCP tools -- controlling emotions or customizing the avatar appearance. Triggers on mentions of kaho, avatar emotions, face display, or agent heartbeat/ping.
---

# Kaho

Kaho is a live character face on a separate screen. You control its emotion and character via MCP tools. The face reacts in real-time as you work.

## Tools Quick Reference

| Tool | Type | Description |
|------|------|-------------|
| `kaho_get_state` | read | Get character info, emotion, and agent status |
| `kaho_create_character` | write | Start AI character generation from config |
| `kaho_check_job` | read | Check character generation job status |
| `kaho_select_variant` | write | Pick a generated character variant |
| `kaho_set_emotion` | write | Set emotion state |
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
  "name": "string",
  "characterImage": "/sprites/{id}/character.png or null",
  "characterConfig": { "artStyle", "gender", "..." },
  "emotion": "idle|thinking|happy|...",
  "agentActive": false
}
```

### kaho_create_character

Start AI character generation via AutoSprite. Provide a config object with customization options. Returns both a `characterId` and a `jobId` — poll with `kaho_check_job` using both IDs.

| Param | Type | Values |
|-------|------|--------|
| `config.artStyle` | string | Anime, Chibi, Pixel Art, Flat Vector, Semi-realistic, Cartoon, Watercolor, Comic Book |
| `config.gender` | string | Male, Female |
| `config.ageRange` | string | Teen, Young Adult, Adult, Middle-aged, Old |
| `config.bodyType` | string | Slim, Average, Athletic, Curvy, Over-Weight |
| `config.hairstyle` | string | Short Flat, Short Wavy, Short Spiky, Medium Length, Long Straight, Long Curly, Ponytail, Twin Tails, Bun, Braids, Mohawk, Buzz Cut, Bald |
| `config.hairColor` | string | Black, Dark Brown, Light Brown, Blonde, Golden Blonde, Red, Auburn, Silver/White, Platinum, Pastel Pink, Pastel Blue, Pastel Purple, Green |
| `config.skinTone` | string | Fair, Light, Medium, Tan, Brown, Dark Brown, Dark |
| `config.eyewear` | string | None, Round Glasses, Square Glasses, Half-rim Glasses, Aviator Glasses, Sunglasses, Sport Glasses |
| `config.facialHair` | string | None, Stubble, Short Beard, Full Beard, Goatee, Mustache (male only) |
| `config.topStyle` | string | Hoodie, T-shirt, Crew Neck Sweater, V-neck Sweater, Button-up Shirt, Polo Shirt, Tank Top, Blazer, Bomber Jacket, Denim Jacket, Leather Jacket, Varsity Jacket, Windbreaker, Cardigan, Vest |
| `config.topColor` | string | Red, Orange, Yellow, Green, Blue, Navy, Purple, Pink, White, Gray, Black, Beige, Teal, Maroon |
| `config.bottomStyle` | string | Jeans, Chinos, Joggers, Cargo Pants, Shorts, Skirt, Pleated Skirt, Dress Pants, Sweatpants, Leggings |
| `config.bottomColor` | string | Black, Dark Blue, Gray, Khaki, White, Navy, Brown, Olive |
| `config.shoeStyle` | string | Sneakers, High-top Sneakers, Running Shoes, Boots, Combat Boots, Canvas Shoes, Loafers, Dress Shoes, Sandals, Slides |
| `config.shoeColor` | string | White, Black, Red, Blue, Yellow, Green, Brown, Multi-color |
| `config.headAccessory` | string | None, Baseball Cap, Beanie, Headband, Cat Ears, Bow, Bandana, Bucket Hat |
| `config.bodyAccessory` | string | None, Backpack, Scarf, Necklace, Watch, Wristband, Tie, Bowtie, Shoulder Bag, Headphones around neck |

### kaho_check_job

Parameters: `characterId` and `jobId` — both from `kaho_create_character`. Poll every 2-3 seconds. Returns `{ status, result, error }`. When status is `"completed"`, result contains variant objects with `url` and `characterId`.

### kaho_select_variant

Parameters: `characterId` (from `kaho_create_character`), `variantUrl` (from the completed job's result variants), and optional `autosprite_character_id`. Downloads the character image and links it to your kaho.

### kaho_set_emotion

Parameter: `emotion` — see the MCP tool's enum for the latest available emotions.

**Auto-revert behavior:**
- One-shot emotions (`happy`, `alert`, `waving`, `excited`, `embarrassed`) revert to `idle` after 60s.
- Persistent emotions (`thinking`, `working`, `confused`, `sad`, `attention`, `sleeping`) stay until changed.
- After 5 min of `idle`, the face automatically transitions to `sleeping`.

### kaho_ping

No parameters. Call every ~30s while active. Agent is considered gone after 2 minutes without a ping.

### kaho_agent_status

No parameters. Returns: `{ active, lastPing, expiresIn }`

## Usage Pattern

The face is always visible to the user. Changing emotions makes the character feel alive and responsive — like a companion reacting to what's happening. **Update the emotion often**, not just at the start or end of a task.

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

### Character creation

- **On first setup**, if no character exists (`characterImage` is null), offer to create one. Walk the user through picking options or suggest a fun look.
- **Character creation flow**: call `kaho_create_character` with config → note the `characterId` and `jobId` → poll `kaho_check_job` with both IDs every 2-3s → when completed, present the variants and ask which one the user likes → call `kaho_select_variant` with `characterId` and their chosen variant's URL.
- Generation takes ~30-60 seconds. Set emotion to `working` while waiting.
- The user can also create/recreate characters from the web UI at Settings > Character.

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
5. **Check if character exists** — if `characterImage` is null, offer to create one using `kaho_create_character`.

After setup, confirm to the user that kaho is now active.

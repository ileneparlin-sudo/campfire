# Story Games for Two

Two apps, one folder:

- **`campfire.html` — Campfire**: two ways to tell a story together. *This is the main event.*
  - **📖 Story adventure** (the core experience): an authored tale — *The Midnight Train* built in — where the plot is real and the voices are yours. Each beat: the story sets the scene → the prompted player writes their part in their own words (1–3 sentences) → the partner plays **Fate** and chooses how the world answers from authored consequences → the story remembers (invisible marks unlock different paths later) → roles swap. The finished playthrough — scenes, your words, Fate's rulings — saves to the library. The title screen has a dropdown listing every story baked into campfire.html — to add one permanently, paste a new `<script type="application/json" class="embedded-tale">` block after the existing one (it appears in the dropdown automatically, no other code changes needed). Or drop one in for a single session via **Load a story file…** (the engine validates it and adds it to the dropdown as "(loaded)"). Format: `meta {id,title,blurb,roles[2],start}` + `beats {scene, role, prompt, icon?, fates[{text,goto,mark?,requireAny?}]}` with `ending: true` beats. `icon` is optional and must match a `<symbol id="icon-...">` in the sprite sheet near the top of `<body>` in campfire.html (ticket-moon, lit-window, ledger, coat, lantern, token, receipt, constellation, platform-moon so far) — it renders as a small line-art picture above that beat's scene text. See `stories/campfire-template.json` for an annotated skeleton.
  - **🔥 Freeform**: alternating 1–3 sentence turns sparked by prompt cards, fixed length plus closing lines. Pure collaborative writing, no authored plot.

  Both modes play same-device, by send-a-code, or in a live room (host shares a 5-letter code once; turn locking and live updates are automatic).
- **`player.html` — Adventure Player**: the choose-your-own-adventure engine with swappable story files (`stories/*.json`), for authored branching stories with stats, dice, endings, and achievements.

## Campfire quick start

Double-click `campfire.html` → **Start a story** → names, length, pick an opening line → write. After your lines, pass the device or send the code. For real-time on separate devices: **Host a live room**, share the 5-letter code, partner joins — then it's automatic, with turn locking and lines appearing live.

---

# The Adventure Player (`player.html`)

A reusable game engine plus swappable story files (`stories/*.json`).

## Quick start

1. Double-click `player.html` — it opens in your browser with the demo story (*The Clockwork Tower*) already loaded.
2. To play a different story, click **Load Story…** or drag any story `.json` onto the page.

No installs, no server. Progress autosaves; endings and achievements you discover are remembered per story.

## 🌐 Live Together — real-time on separate devices

For playing at the same time (e.g., on a call): title screen → **Live Together — Separate Devices**.

1. One of you clicks **Host a room** and reads out the 5-letter code (once — that's the only thing you ever share).
2. The other enters it and clicks **Join room**. The story and game state sync automatically — the joiner doesn't even need the story file.
3. From then on it's seamless: on your turn you choose; on theirs, your buttons lock ("Waiting for…") and their choice appears on your screen the instant they make it, with a note showing what they picked.

Endings, the journey recap, and any achievements earned land on **both** devices. The host's device runs the game (including dice rolls); if the connection drops, the host re-hosts and you rejoin with a new code. Both devices need internet. The connection is direct peer-to-peer (PeerJS) — no accounts, nothing stored anywhere.

## 💌 Two-Player Pass & Play (asynchronous — across days)

On the title screen, choose **💌 Two-Player Pass & Play**, enter both names, and pick who goes first. After each choice, the game produces a code — send it to your partner over text/WhatsApp/anything. They paste it under **"Have a game code from your partner?"** and the story continues on their screen, with a recap of what you chose. Take turns until you reach an ending together.

If you're in the same room, the **"pass the device"** button skips the codes entirely.

### Upgrade: clickable links instead of codes

If you host the game at a web address, the handoff becomes a clickable link (the game state rides along after the `#` — nothing is uploaded anywhere). Free, five-minute options:

1. **GitHub Pages**: make a free GitHub account → create a repository → upload `player.html` (rename it `index.html`) → Settings → Pages → enable. Your game lives at `https://yourname.github.io/yourrepo/`.
2. **Netlify Drop** (netlify.com/drop): drag the folder onto the page. Done.

Both of you bookmark the same address; links sent between you "just work."

## Writing a new story

Copy `stories/template.json`, rename it, and edit. A story is one JSON file:

```json
{
  "meta": { ... },
  "scenes": { "scene-id": { ... }, ... }
}
```

### meta

| Field | Required | Description |
|---|---|---|
| `id` | yes | Unique slug. Saves, endings, and achievements are keyed to this — change it and players start fresh. |
| `title` | yes | Shown on title screen and header. |
| `start` | yes | Scene id where the story begins. |
| `author`, `blurb` | no | Title-screen flavor. |
| `stats` | no | Starting stats, e.g. `{ "nerve": 0, "health": 3 }`. |
| `items` | no | Starting inventory, e.g. `["lantern"]`. |
| `hardcore` | no | `true` disables the Back/undo button. Choices stick. |
| `statDeath` | no | `{ "health": "death-scene" }` — if that stat hits 0, the player is sent to that scene immediately. |
| `achievements` | no | `{ "badge-id": { "name", "desc", "secret" } }`. Secret ones show as "???" until earned. |

### scenes

Each key is a scene id. Each scene has:

- `text` (required) — the prose. `\n\n` makes paragraph breaks.
- `mood` (optional) — `danger`, `calm`, `mystery`, or `victory`. Tints the scene panel.
- `choices` — array of choice objects (see below). Required unless the scene is an ending.
- `onEnter` (optional) — effects applied automatically when the scene loads.
- `ending` (optional) — `{ "id": "unique-id", "name": "Display Name" }`. Marks a final scene.

### choices

```json
{
  "text": "Button label",
  "goto": "target-scene-id",
  "require": {
    "items": ["must-have"],
    "missingItems": ["must-not-have"],
    "stats": { "nerve": { "min": 2, "max": 5, "eq": 3 } }
  },
  "effects": {
    "gainItems": ["thing"],
    "loseItems": ["thing"],
    "stats": { "nerve": 1 },
    "setStats": { "nerve": 0 },
    "award": ["badge-id"]
  },
  "lockedText": "why it's locked (shown on disabled button)",
  "hideIfLocked": true
}
```

Only `text` and `goto` are required (or `roll` instead of `goto`). `stats` effects add the value (use negatives to subtract); `setStats` overwrites; `award` grants achievements.

### Write-in scenes (you write the move, your partner judges it)

Give a scene a `writeIn` instead of `choices`. The active player writes their action in their own words ("I challenge the guardian to a riddle contest"). Then it becomes the **other player's** turn — they see the written move and decide what it earns by picking one of the author's outcomes. The engine never interprets the text; the person who knows you does. Written moves and rulings both appear in the journey recap, so every playthrough's recap is partly in your own words. Solo players write (optionally) and judge themselves.

```json
"writeIn": {
  "prompt": "The guardian blocks the stairs. What do you do?",
  "outcomes": [
    { "text": "It works — the guardian yields", "goto": "workshop", "effects": { "stats": { "nerve": 1 } } },
    { "text": "It fails — the gears take them", "goto": "end-gears" }
  ]
}
```

Both two-player stories have one: the summit shout in *Keeper & Wanderer*, and the final log entry in *The Perennial*.

### Dice-roll choices

Replace `goto` with a `roll` to make a choice risky. The button shows the live odds (🎲 62%):

```json
{
  "text": "Leap across the chasm",
  "roll": {
    "chance": 0.4,
    "perStat": { "agility": 0.1 },
    "success": "other-side",
    "failure": "the-fall",
    "successEffects": { "award": ["leaper"] },
    "failureEffects": { "stats": { "health": -1 } }
  }
}
```

`chance` is the base probability (0–1); `perStat` adds per point of a stat; final odds are clamped to 5–95% so nothing is ever certain. The player sees the roll result banner on the next scene.

### Tips

- Item and stat names are displayed prettified: `brass-key` → "Brass Key".
- The endings tracker counts every scene with an `ending` field; give each a unique id.
- Use `hideIfLocked: true` for secret choices players only see when qualified — great for replays.
- The engine validates on load and reports broken links and dead ends (browser console has the full list).

## Files

- `player.html` — the engine, with the demo story embedded so it works on open.
- `stories/the-clockwork-tower.json` — demo story: 4 endings, 3 achievements, a dice roll, moods.
- `stories/the-keeper-and-the-wanderer.json` — a two-player tale written for pass & play: you each embody one character (the Keeper at the lighthouse, the Wanderer on the mountain), alternating scenes. 6 endings, 4 achievements, a shared hope stat that can fade to the saddest ending if either of you gives in to doubt.
- `stories/the-perennial.json` — a dilemma story for two: alternating watch officers on a colony ship, communicating only through the ship's log, inheriting each other's hardest calls. No right answers, no ranked endings — 6 different ones, 3 gated by how you played. Built for the post-game conversation.

At every ending, the engine now shows **"The road you took"** — a full recap of every choice made (and by whom, in two-player), so the conversation about *why* starts itself.
- `stories/template.json` — annotated template showing every feature.

## Swapping the default story

Replace the JSON inside the `<script id="embedded-story">` block at the bottom of `player.html` with your story's contents.

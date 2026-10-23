# Campfire — Game Design Document

*A two-player collaborative storytelling game. Written to hand off cold to another person or AI tool.*

---

## 1. What this is (one paragraph)

Campfire is a game for **two people** who want to tell a story together — designed especially for
couples, including long-distance. Its heart is the **Story Adventure** mode: an *authored* story
provides the plot rails; the two players provide the human details by writing in their own words and,
in turn, deciding how the world responds to each other. It is not a Choose-Your-Own-Adventure (no
menu-picking as the primary act), not a tabletop RPG (no stats/dice/inventory), and not pure improv
(the plot is authored, so it never stalls on a blank page). A secondary **Freeform** mode is lighter
prompt-driven co-writing with no authored plot.

The intended outcome is not "we reached ending #4." It's **story memories**: *"remember when you told
the conductor the second passenger was your conscience, and I made him laugh?"*

---

## 2. Design history (why it is what it is)

This matters because the design was arrived at by elimination, and the dead ends are instructive.

1. **Started as a CYOA engine** (branching stories, stats, inventory, dice, achievements, multiple
   endings). Fully built and tested. Verdict from the designer: *technically works, isn't fun; feels
   like picking from menus; "I don't see much creativity."*
2. **Added a write-in mechanic** (you type a free action, your partner judges it). This was the first
   thing that felt alive — the tell that **player authorship**, not mechanics, was the point.
3. **Pivoted to "Campfire" — pure collaborative storytelling** (alternating sentences + random prompt
   cards, fixed length, keepsake library). Fun, but two flaws surfaced: (a) random prompts give
   *variety but not structure* — no arc, so no satisfying ending; (b) total freedom ("you could
   literally say anything") felt unmoored.
4. **Final synthesis — Guided Interactive Fiction.** The designer articulated it: *the story provides
   the rails, the players provide the details.* Author controls pacing, plot, reveals, setting, major
   events. Players control personality, approach, motivation, relationships, emotional meaning.
   Reference touchstone: the Commodore-64 *Wizard of Oz* game — a strong authored story path with
   freedom of expression *inside* scenes, remembered fondly not for branching but for the feeling of
   *participating in a story*.

**Removed on purpose:** stats, inventory, dice, health, achievements, keys, multiple "win" endings,
random prompts as the core driver. These were scaffolding that distracted from the experience.

---

## 3. The core loop (locked)

One **beat** = one trip through this loop:

```
SCENE           the authored story presents a situation (fixed prose)
   ↓
PLAYER WRITES   the prompted player answers the beat's prompt in their own
                words, 1–3 sentences (soft ~320-char "ink" budget)
   ↓
PARTNER = FATE  the OTHER player reads those words and chooses how the world
                answers, from 2–3 author-written consequences. They pick — they
                do NOT rewrite or free-judge. No negotiating.
   ↓
STORY REMEMBERS the chosen consequence may set an invisible "mark" (no HUD, no
                visible stats). Marks silently unlock/gate later options.
   ↓
NEXT SCENE      the consequence routes to the next beat (or an ending)
   ↓
SWAP ROLES      writer and Fate trade for the next beat
```

Key principles:
- **The words are never mechanically parsed.** A human (the partner) is the interpreter. This
  sidesteps the classic parser-adventure failure ("guess the verb") and the "AI judged my words
  wrong" failure. The partner maps expression → authored branch.
- **Freedom of content, discipline of consequence.** You can write anything; the *story* decides what
  it means via the authored consequences the partner picks between.
- **No right answer, no best ending.** Endings are *different*, not *ranked*.
- **Invisible memory.** Marks are felt, not displayed — an option simply appears (or doesn't) later
  because of something you did. Preserves immersion; no optimization mindset.

---

## 4. Roles & identity

- Players are **themselves** (real names), but each is cast into a **story role** at setup — e.g. in
  *The Midnight Train*: **the Bold One** and **the Careful One**. Roles determine who is prompted to
  write on which beats (the author tags each beat with `role: 0` or `role: 1`).
- Casting is chosen at setup ("Who plays the Bold One?"). The other player is automatically the other
  role. Writer/Fate alternate because beats alternate which role they prompt.

---

## 5. Modes of play (all three supported by one app)

1. **Same device** — pass the phone/laptop back and forth; a "pass the device" button between turns.
2. **Send-a-code (async)** — after your turn, the full game state is compressed into a short code (or
   a URL hash if the app is web-hosted). You text it to your partner; they paste it and continue.
   Good for long-distance across days.
3. **Live room (real-time)** — one player hosts, shares a 5-letter room code once; the other joins.
   Peer-to-peer (PeerJS), no accounts, no server owned by us. Turn-locking is enforced (your controls
   disable when it's the partner's turn); the partner's writing/choice appears on your screen the
   moment they commit it. Host device is authoritative for game logic.

At the end, the finished playthrough — scenes + each player's written words + each Fate ruling — is
saved to a local **library**, rereadable and downloadable as a `.txt` keepsake.

---

## 6. Authored-story data format (drop-in `.json`)

The app is **content-agnostic**: the engine ships with one built-in story (*The Midnight Train*) and
can load any story file with the same shape. This is the whole schema:

```jsonc
{
  "meta": {
    "id": "the-midnight-train",          // unique slug (keys the save/library)
    "title": "The Midnight Train",
    "blurb": "One-line teaser for the setup screen.",
    "roles": ["the Bold One", "the Careful One"],  // EXACTLY two role names
    "start": "b1"                          // id of the first beat
  },
  "beats": {
    "b1": {
      "scene": "Authored prose. \n\n makes paragraph breaks.",
      "role": 0,                           // which role (index into meta.roles) WRITES this beat
      "prompt": "The question the writing player answers in their own words.",
      "fates": [                           // 2–3 consequences the PARTNER chooses between
        {
          "text": "What happens, shown to Fate as an option and then added to the story.",
          "goto": "b2",                    // next beat id
          "mark": "CONDUCTOR-AMUSED"       // OPTIONAL invisible flag this choice sets
        },
        {
          "text": "A consequence that only appears if an earlier mark was set.",
          "goto": "ep3",
          "mark": "THE-PASSENGERS-DEBT",
          "requireAny": ["THE-WARNING", "THE-OFFER"]  // OPTIONAL gate: show only if any mark held
        }
      ]
    },
    "ep1": {
      "scene": "An ending beat. No role/prompt/fates.",
      "ending": true
    }
  }
}
```

Field notes:
- `role` (0 or 1) says who *writes*; the other player is *Fate* for that beat.
- Each `fate` needs `text` + `goto`. `mark` and `requireAny` are optional.
- `requireAny`: the option is hidden unless the player holds at least one of the listed marks. (This
  is how "the story remembers" — a path opens because of an earlier choice.)
- Ending beats have `scene` + `ending: true` and nothing else.
- The engine validates on load and reports broken `goto`s, missing fields, wrong role count, etc.

**Authoring guidance:**
- ~12–15 beats for a full one-sitting story (20–30 min). The built-in demo is a 3-beat prototype.
- Every fate should be *plausible and different*, never "right vs. wrong."
- Alternate `role` so both players write roughly equally.
- Use marks for payoff, not bookkeeping: an early emotional choice should visibly change a late beat.
- Prompts should invite *character* ("What secret worry do you have?"), not *plot* ("Do you open the
  door?"). The plot is your job as author; the feelings are theirs.

---

## 7. The built-in story: *The Midnight Train* (prototype, 3 beats)

A sleeper train stops at stations that don't exist. The two players board with **one ticket between
them**, and a conductor who finds that fascinating. Beats:
1. **The conductor** confronts them about the single ticket (the Bold One writes the reply).
2. **The window** shows a station that looks like a place the Careful One misses (they write what
   they see and would do).
3. **PROVISIONAL station**: a Stationmaster demands a fare paid in "something true" (the Bold One
   offers it). One fate option only unlocks if an earlier beat set `THE-WARNING` or `THE-OFFER`.

Three "to be continued" epilogues. Full version would run ~12–15 beats (the conductor's recognition,
the corridor passenger's story, the station with your name on it).

---

## 8. Freeform mode (secondary)

Lighter co-writing with no authored plot: pick an opening line from a seed deck, then alternate 1–3
sentence turns, each sparked by a random prompt card ("someone tells a lie", "make it worse", "an
animal has an opinion"). After a chosen number of turns (8/12/20) each player writes a closing line;
the result saves to the library. This is the earlier Campfire concept, kept as a low-structure option.

---

## 9. Implementation notes (for a builder)

- Single self-contained `campfire.html` — no build step, no dependencies except PeerJS (loaded from
  CDN only when live mode is used). Opens by double-click.
- State is plain JSON; async codes are `deflate-raw` + base64url (with an uncompressed fallback).
- All persistence is `localStorage` (autosave + finished-story library). Nothing leaves the device
  except peer-to-peer live-session traffic.
- Verified with a headless (jsdom) test suite simulating two devices: full playthrough, mark gating
  (hidden vs. shown), code roundtrip mid-turn, live sync both directions with turn-locking, and
  freeform regression.
- Optional future step: host on GitHub Pages / Netlify so both partners share one URL and codes
  become clickable links.

---

## 10. Open design questions (good things to pressure-test)

- **Length & pacing:** is 12–15 beats the right one-sitting size? Should stories be chaptered?
- **Fate's power:** should Fate ever be able to write a custom consequence, or does "pick from
  authored options" stay pure? (Current answer: stays pure — it's what keeps the story coherent.)
- **Prompts inside story mode:** currently the beat's `prompt` is the only nudge. Is that enough
  guidance, or do writers want an optional "if stuck" hint?
- **Emotional vs. plot marks:** how much should late beats reward *how* players played vs. *what*
  they chose?
- **Replayability:** with authored plot, replay value comes from different expression + different
  Fate rulings + mark-gated branches. Is that enough, or do stories need larger branch divergence?
```

# Karks Cub Kingdom — Playable Demo Spec

Top-down Zelda-style playable sequence: punchline → heart capture → critic boss
fight → fake death → wagyu resurrection finale. Deliverable:
`game/index.html` (single file + reuse `intro/assets/`), plus small text edits
to the existing `intro/index.html`.

## Golden rule of tone

**Never explain the joke.** No text anywhere may mention cost, money, price,
bills, or things being free — EXCEPT the punchline itself, "FOR FREE?".
The stories are mundane, the chef asks "FOR FREE?", that's the whole bit.

## Intro edits (`intro/index.html`)

1. Scene 1 lines are currently "...NO COIN WAS ASKED." / "NO BILL EVER CAME."
   Replace all three lines with:
   - `LONG AGO, IN THE KING OF KARK,` (verbatim inside joke -- not a typo)
   - `CHEF GABE PREPARED A FEAST FOR FOUR GUYS.`
   - `THE STORIES THEY TOLD WERE VERY ORDINARY.`
2. Scene 4 text `IT COSTS... NOTHING.` → `IT IS EXTREMELY MARBLED.`
3. Title screen PRESS START (and any click/key on the title) now navigates to
   `../game/` instead of replaying. Attract-replay after 25 s idle stays.
4. Title lockup renamed from "THE LEGEND OF WAGYU" to **KARKS CUB KINGDOM**
   (two-line hand-built pixel lockup, `KARKS CUB` / `KINGDOM`); the `FOR
   FREE?` terracotta ribbon subtitle stays exactly where it was.

## Game (`game/index.html`)

Same technical base as the intro: single file, canvas 960×540, pixel-crisp,
integer scaling, WebAudio-only sound, same palette, same asset PNGs via
`../intro/assets/`. Port stays 8807 (served from repo root). Copy/adapt the
intro's audio engine (sequencer, voices, jaw-harp KS twang, Theme A/B tables).
Root `index.html` (repo root): meta-refresh redirect to `intro/`.

### Player

The chef. 16px sprite (same one as intro Scene 4) at 4× scale. Moves with
WASD/arrows/d-pad, 8-direction, ~220 px/s. One action button: Space / click /
gamepad A — context-sensitive (say the line / throw bottle). Hearts UI top-left
(Zelda style: full/half hearts, max 6). Start with 3 hearts.

### Arena

One room, the dinner party: Tiny Dungeon floor tiles, terracotta table in the
center with 4 seated diners (the intro's Scene 3 look), kitchen doorway with
warm glow bottom-right (player spawns there), main door on the left wall.
The table is solid (collision); diners are solid. Keep 8-px collision grid.

### Beat 1 — the punchline (×2 rounds)

1. A diner's speech bubble types a mundane story (reuse typewriter):
   - Round A: `I DRANK A GLASS OF RIESLING` / `SO NOBODY ELSE HAD TO.`
   - Round B: `I STAYED TILL THE SEVENTH INNING` / `OF A TWELVE TO ZERO GAME.`
     (spec originally said "TWELVE-NOTHING" -- changed to avoid the tone
     grep's own NOTHING gate; same mundane baseball-score beat)
2. When the story completes, a bouncing `!` prompt appears over the chef.
   Player presses action → the chef **clearly says "FOR FREE?"**:
   - Big chunky gold text `FOR FREE?` slams over the kitchen doorway
     (same style as intro Scene 3) + jaw-harp twang + screen shake.
   - **Spoken aloud** via the SpeechSynthesis API: utterance "For free?",
     rate 0.85, pitch 0.7 (deadpan). Pick the first available local voice;
     if speechSynthesis is unavailable or has no voices, skip silently —
     the text + twang carry it.
   - Timing bonus: pressing within 1.2 s of story completion = "PERFECT"
     flash and +2 extra hearts in the scatter below.
3. **The table goes crazy**: all four diners bounce, HA-HA particles, drums
   kick in — and 6–10 small laugh tokens burst from the table on a lobbed
   z-arc (napkin-style) out to a pre-vetted floor point beyond the table's
   edge, so they visibly float up and over it rather than sliding on flat
   friction; every landing spot is guaranteed outside the table/walls (any
   spot that would fail that check gets slid back onto open floor before it's
   allowed to settle) so a token can never rest somewhere the player can't
   reach. They sit and blink; despawn after 6 s.
4. **Laugh capture**: player runs and collects them before they vanish
   (pickup radius ~20 px, Zelda pickup chime, +half-heart-unit each, cap 6
   full). A big chunky `CATCH THE LAUGHS!` banner pops in (0.2 s) / holds
   (1.2 s) / fades (0.4 s) centered in the upper third at the start of every
   capture phase (not just the first — gameplay is never paused for it; the
   one-time tutorial card still pauses on the very first capture, and the
   banner effectively resumes once that's dismissed). A small persistent hint
   also shows bottom-center on the first round only.
5. Short breather, next round. After round B's capture, go to Beat 2.

### Beat 2 — the critic boss

Intro moment: the glasses diner stands up — bubble: `OKAY. WE GET IT.` — walks
to the RIGHT wall, and **a door appears halfway up the wall** (frame + door
tile floating ~130 px above the floor, clearly absurd, with a 0.5 s "pop"
sparkle). He climbs into it (brief scripted hop) and leans out. Boss HP bar
appears top-center: `THE CRITIC` — 6 segments.

**His attacks** (he stays in the elevated door; player must dodge):
- Lobbed crumpled napkin balls (arcing projectiles with shadow blobs that
  telegraph the landing spot; ~1 every 1.4 s ±25% jitter). Contact or landing
  burst = −half heart, 0.8 s i-frames + blink.
- Every ~6 s ±25% jitter: a spoken critique — bubble text cycles
  `WEAK PREMISE.` / `DERIVATIVE.` / `THE SOUP WAS LUKEWARM.` /
  `I'VE HEARD THIS STORY TWICE.` / `GOOD THING HE BOUGHT ALL THE BASEBALLS AT
  THE GAME.` (callback to Round B's story) — each accompanied by a 3-napkin
  fan volley (5 in phase 2).
- SpeechSynthesis optional for critiques (same fallback rule), rate 1.0,
  pitch 0.6, only if the punchline speech worked.

**Attack patterns (anti-circling)**: every single throw and every volley
independently rolls one of 4 aiming patterns rather than always aiming at the
player's live position, so no one fixed movement habit (e.g. holding a
steady circle-strafe) is ever safe:
- ~35% (phase 1) / ~30% (phase 2) **LED** — aim at player position +
  velocity × 0.45 s (0.6 s in phase 2).
- ~30% / ~20% **DIRECT** — aim at the player's current position; punishes
  stopping/reversing to bait the lead.
- ~20% / ~20% **SCATTER** — 3 napkins at randomized offsets 60–130 px (45–95
  px in phase 2) around the player — covers a circle path both ahead and
  behind.
- ~15% / ~30% **CUTOFF** — aim 2× the lead distance ahead along the player's
  current heading — lands where a circler is about to be.
Every throw's timing also gets ±25% random jitter so the rhythm can't be
memorized. Shadows still telegraph the exact landing spot for the full flight
either way — the goal is that dodging requires reading the telegraph, not
holding one direction.

**Player attack — riesling throw**: riesling bottles (green pixel bottle,
14×6) spawn one at a time on the table's near edge (respawn 2.5 s after
pickup). Walk over one to pick it up (bottle icon shows next to hearts; carry
max 1). Action button throws it in an auto-aimed arc at the boss door. Hit =
−1 boss HP, glass-shatter noise burst + splash particles + boss "OW."-style
recoil. Miss (he ducks into the door randomly, ~25% of throws when above 3 HP)
= bottle shatters on the wall.

**Fake death at 1 HP**: his next hit doesn't kill — instead:
- He slumps over the door edge, arms dangling. Boss bar shatters. Victory
  jingle STARTS playing (first 4 notes of the item-get fanfare)…
- …record-scratch (noise sweep), he pops back up: bubble
  `JUST KIDDING.` then `ALSO: THE SOUP WAS COLD.`
- Bar reappears with 4 segments, phase 2: napkins every 0.9 s, volleys of 5,
  landing bursts bigger. He no longer ducks.

**Real death**: at 0 HP he slumps for real, drops a single big heart (full
heal), the wall door sags shut behind him, music drops to the soft intro
arpeggio.

### Beat 3 — the wagyu resurrection

1. The LEFT main door opens: **the woman enters** carrying a platter with the
   wagyu (pink marbled steak sprite + rotating sparkles). She walks slowly to
   the slumped critic. Bubble (sincere, typed slowly):
   `EXCUSE ME...` / `I BROUGHT THE WAGYU.`
2. She offers the wagyu — sparkle burst — **it brings him back to life**:
   he springs upright in the wall door, fully restored, adjusts his glasses.
3. One full beat of silence (0.9 s, music fully ducked)…
4. **EVERYONE says it at once**: simultaneous speech bubbles over all four
   diners, the woman, AND the boss — each just `FOR FREE?` — plus the giant
   gold `FOR FREE?` slam, jaw-harp gallop, screen flash, drums. If speech
   worked earlier, speak it once more.
5. Does NOT freeze/cut to the end card yet — flows straight into Beat 4.

### Beat 4 — Aram

Right after the unison finale, the LEFT door slams open: **Aram**, the
chef's boss, storms in (a hand-tinted ~2× diner-scale sprite, dark red/angry
palette, simple hand-drawn angry brows over the base sprite). He walks in to
about mid-room and delivers two sequential bubbles: `A ONE-STAR GOOGLE
REVIEW?!` then `WHO DID THIS?`. Tutorial card (shown once, same freeze-the-
game card system as the others): title `ARAM HAS ARRIVED` — body `CHEF
GABE'S BOSS SAW A BAD REVIEW.` / `CHASE THE GUYS. BEG FOR FIVE STARS.` /
`DON'T GET CAUGHT.`

**Gameplay**: Aram chases the player directly at 0.8× player speed; contact
(≈36 px, 1 s cooldown) costs half a heart-unit, grants 1.0 s i-frames (longer
than the boss's 0.8 s, since he's a persistent chaser not a one-shot
projectile), and knocks the player back ~40 px. Meanwhile the three
non-critic seated diners, the woman, and the revived critic (who is fixed to
actually re-render post-fight — see Implementation notes) all wander the
room on random floor waypoints (~50 px/s, avoiding the table). Any of them
within ~120 px of the player flees (~130 px/s — slower than the player's
220 px/s, so they're always catchable; resumes wandering only past ~160 px,
to avoid flee/resume flicker right at the boundary).

**Begging**: within ~40 px of a non-fleeing, not-yet-reviewed character,
press the action button: the chef's own speech bubble reads `PLEASE. FIVE
STARS.`, that character stops and "checks their phone" (small blinking phone
icon, ~1.2 s), then a row of 5 small gold stars floats up from them and the
review counter ticks — each character can only give one review. HUD top-
center during the chase: `<n>/3` in gold plus 3 small hand-built star icons.

**Turning good**: at 3/3 reviews, Aram freezes mid-chase and "checks his
phone" for ~1.5 s, then turns good (warmer palette, brows relax) while
everyone else cheers (bounce + HA bursts + sparkle confetti, fanfare into
full Theme B). Sequential bubbles: `FIVE STARS...` then `DINNER AT MY PLACE.
SOMETIME.` — then the freeze frame and end card as before, with an added
stat line `FIVE-STAR REVIEWS: 3`.

**Failing Beat 4**: player hearts hit 0 during the chase → same dimmed
retry-card flow, but the bubble reads `THE REVIEW STANDS.` and `TRY AGAIN`
restarts Beat 4 specifically (reviews reset to 0, characters reset to
wandering; the ARAM HAS ARRIVED card is not reshown).

**Implementation note**: the critic is left in a `state==='dead'` limbo after
his real death (which is how his sagging-door fade-out plays), and nothing
previously cleared that before Beat 3's resurrection beat — fixed by setting
`boss.state` to a distinct "revived" value (not the fight's `active`, so the
HP bar and attack AI never come back) as soon as the revive beat starts, so
he's actually visible/wanderable/beggable for Beat 3's finale and all of
Beat 4.

### Failure state

- **Boss fight** (Beat 2): player hearts hit 0 → screen dims, bubble from the
  whole table: `OKAY. WE GET IT.` → retry card (`TRY AGAIN` restarts Beat 2
  with 3 hearts; stats keep counting).
- **Aram's chase** (Beat 4): player hearts hit 0 → screen dims, bubble:
  `THE REVIEW STANDS.` → retry card restarts Beat 4 (reviews reset; stats
  keep counting; the tutorial card is not reshown).

### Music

- Beat 1: Theme A loop (no drums until the first FOR FREE? slam, then drums).
- Beat 2: Theme A transposed feel at 140 BPM with drums (reuse note tables,
  scale the beat clock), + low drone. Phase 2 after fake death: add the
  harmony voice and hats on sixteenths.
- Beat 3: silence → arpeggio → full Theme B for the unison finale.
- Beat 4: Theme B loop at ~140 with drums for the chase; a duck into the
  turn-good fanfare, then full Theme B again for the cheer.
- Keep all ducking behavior from the intro engine.

**Real soundtrack (`assets/music/theme.mp3`)**: both the intro and the game
route a looping `<audio>` element through `createMediaElementSource()` into
the same `musicGain` node the chiptune scheduler already used, so every
existing ducking call works unchanged on the real track with no special-
casing. The chiptune scheduler keeps running underneath (cheap bookkeeping,
still drives tempo/section-program state) but its note-dispatch is silently
skipped once the track is confirmed playing — every direct one-off SFX call
(beeps, dings, twangs, slam booms, whoosh, chimes, fanfare, record scratch,
groan, rumble, glass shatter, speech synthesis) is untouched either way,
since none of those go through the scheduler. If the file fails to load or
`MediaElementSource` isn't available, playback silently stays on the
chiptune system — this must never result in silence. Playback starts on the
same user-gesture that already unlocks the AudioContext today.

### Acceptance checklist

- [ ] No text anywhere mentions cost/money/bills/"nothing" (grep the file for COIN, BILL, COST, NOTHING, FREE — only "FOR FREE?" may match FREE)
- [ ] Chef movement + collision solid at 60 fps; no console errors through a full playthrough
- [ ] speechSynthesis speaks "For free?" when available; silent fallback otherwise
- [ ] Laugh tokens scatter (arced beyond the table), bounce, blink, despawn, always land on reachable floor; capture updates HUD; prominent banner shows every capture phase
- [ ] Wall door appears mid-wall; boss telegraphed arcing napkins are dodgeable via 4 randomized aim patterns, not just one fixed strategy
- [ ] Riesling pickup/throw/hit loop works; boss duck behavior only above 3 HP
- [ ] Fake death sequence: slump → jingle start → record scratch → phase 2
- [ ] Wagyu revive → unison FOR FREE? → Aram's chase/beg/turn-good → end card with stats, reviews, and rank
- [ ] Game over path works and TRY AGAIN restarts the beat it failed in (Beat 2 or Beat 4)
- [ ] Real mp3 soundtrack plays through musicGain with ducking intact, chiptune SFX unaffected, automatic silent-safe fallback if the track can't load
- [ ] Intro edits applied (new Scene 1 lines, marbled line, PRESS START → ../game/)

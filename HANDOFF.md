# morse-tutor — Handoff

## What this is

A single-file Morse code trainer. The entire application — HTML, CSS, JavaScript, data tables, and favicon — lives in **`index.html`** (~3,210 lines). There is **no build step, no server, no dependencies**: open `index.html` in any modern browser and it runs.

`README.md` is the user-facing documentation (features, lesson table, per-mode how-to, prosign table, Farnsworth formula). This file is the developer-facing map.

---

## Quick start

```sh
open index.html        # macOS — or just double-click it
```

Audio requires a user gesture to start (browser autoplay policy); the first **Play**/keypress resumes the `AudioContext`.

---

## Three kinds of mode

There are **eight modes**, split into three fundamentally different shapes:

1. **Copy-practice quizzes** (`char`, `word`, `sent`, `prosign`, `custom`, `random`) — the app plays Morse, the user types what they heard, and the answer is graded. These drive the Session stats and the adaptive-weighting system.
2. **Keyboard mode** (`keyboard`) — a **sender/encoder**, *not* a quiz. The user types text and the app plays the corresponding Morse, highlighting each character as it sounds. No grading, no stats.
3. **Send mode** (`send`) — the reverse of the quizzes: the **user** keys Morse with a straight key (on-screen pad or held **Space**), and the app either grades it against a prompted character (`sendType: 'copy'`, shares `S.level`/`LESSONS` with Character mode) or just decodes whatever's keyed with no prompt (`sendType: 'free'`). Added most recently.

`S.mode` is the single source of truth for which mode is active. Many functions branch on it.

---

## Architecture (all inside `index.html`)

### Layout
- `<style>` block: all CSS, including per-mode `#main-display.mode-*` font/color rules and the `.kb-*` classes for keyboard-mode rendering.
- `<body>`: Settings card → Mode card (tabs + one `.sub-panel` per mode) → Practice card (display, feedback, the three input rows, transport controls) → Stats card → two collapsible reference panels.
- `<script>` block: data tables → state → audio/timing engine → per-mode round management → UI helpers → builders → event wiring → `init()`.

### Data tables (top of script, ~line 584+)
| Const | Purpose |
|---|---|
| `MORSE` | char → dot/dash code. The authority for "is this character sendable" — used everywhere via `MORSE[ch]` |
| `LESSONS` | character-mode levels, ordered by element count |
| `WORD_CATS` | word-mode categories |
| `SENT_CATS` | sentence-mode categories; index 4 = `All` (built from the others), index 5 = `Wikipedia` (`source: 'wikipedia'`) |
| `PROSIGNS` | prosign name → merged code + description |
| `RANDOM_POOLS` / `RANDOM_CATS` | random-mode char pools and length presets |
| `HAM_REF` | data for the Ham Radio Quick Reference panel |

### State — `const S` (~line 1077)
Single mutable object holding all runtime state: settings (`charWPM`, `textWPM`, `freq`, `outputMode`, `noiseLevel`), current `mode`, per-mode selection indices, `currentTarget` (the string being played), playback flags (`isPlaying`, `isPaused`, `waitingInput`), resume bookkeeping (`playArgs` = the current `playString` call's `{text, onCharStart, onDone}`; `playPos` = global index of the most recently started character), session counters, and `stats` (`key → {correct, total}`).

### Audio + timing engine (~line 1107–1345)
- **`timings()`** — the Farnsworth formula. `tc = 1200/charWPM`, `tw = 60000/textWPM`, `Ts = max(tc, (tw − 31·tc)/19)`. Returns `{dit, dah, elemGap, charGap: 3·Ts, wordGap: 7·Ts}`. Read fresh on every play, so speed changes apply immediately.
- **Session gain node** (`newSession`/`killSession`): every playback connects its oscillators through a dedicated gain node. Disconnecting it instantly mutes all queued/in-flight tones — this is how Replay/Skip/Pause silence audio with zero bleed-over. `killSession` also stops any active `S.noiseSource` (see below).
- **`playCode(code, onDone)`** — plays one character's elements (used by single-symbol modes: char, prosign, custom).
- **`playString(text, onCharStart, onDone, startChar = 0)`** — plays a full space-separated string with Farnsworth char/word gaps. `onCharStart(ch, code, ci, wi, nChars, nWords)` fires ~25 ms before each character. Used by word, sentence, random, **and keyboard** modes. It stashes its args in `S.playArgs` and records the live position in `S.playPos` (set inside each character's scheduled callback) so a paused round can be resumed. `startChar` is the global index of the first playable character to sound — earlier characters **and their leading gaps** are skipped, so playback begins immediately at that character (used by Resume).
- **Silent mode**: both players have a `silent` branch driving the `#lamp` flash + `navigator.vibrate` instead of tones, selected by `S.outputMode`.
- **Pause / Resume** (`pauseRound`/`resumeRound`, the `playString` modes only): Pause tears down the scheduled playback but leaves `S.playPos` intact. Resume calls `playString(...S.playArgs, S.playPos)` to restart the *same* playback at the character that was sounding when paused (not from the beginning — that's what **Replay** does), then returns focus to the input box (`keyboard-input` in keyboard mode, else `multi-input`). char/prosign/custom modes don't pause.
- **Band noise / fading** (`noiseBuffer`, `startNoise`, `scheduleFading`): when `S.noiseLevel > 0` and output isn't silent, `playCode`/`playString` compute their own start time and total duration (`t0`, `at - t0`) after scheduling all tones, then call `scheduleFading(sessionGain, t0, dur)` — a random-walk series of `linearRampToValueAtTime` calls on the session gain, simulating QSB — and `startNoise(t0, dur)` — a looping white-noise buffer through a bandpass filter centered on `S.freq`, simulating band-limited QRN static. Both are scoped to the session: `killSession` stops `S.noiseSource` immediately on cancel/pause so noise never bleeds into the next round. The straight-key sidetone in Send mode (`ST_OSC`/`ST_GAIN`) is a separate audio path and is unaffected by this setting — noise only applies to *received* (played) Morse.

### Per-mode round management
Each mode has a `start<Mode>Round()` (and the quizzes have an `on<Mode>Answer()`). `startRound()` (~line 1934) dispatches on `S.mode`. `replayRound()` re-plays `S.currentTarget` and also branches on mode.

### Keyboard mode specifics (~line 1853–1930)
- `normalizeKeyboardText(s)` — uppercases, normalizes smart quotes/dashes/ellipsis, collapses whitespace (incl. newlines), trims.
- `prettyCode(code)` — renders a code as `·`/`–` for the feedback line.
- `buildKeyboardMarkup(text, hlWord, hlChar)` — returns HTML: each character is a span; the currently-sounding one gets `.kb-cur` (blue), characters not in `MORSE` get `.kb-skip` (struck through). Pass `-1, -1` for no highlight (idle/preview/finished).
- `playKeyboard(text)` — shared by Play/Skip and Replay; calls `playString`, updates the highlight + feedback per character. Deliberately keeps the base `mode-keyboard` (yellow) class while playing so the blue highlight stands out (it does **not** add `.playing`).
- `startKeyboardRound()` — reads the textarea, validates at least one sendable char, sets `currentTarget`, plays.
- A live-preview `input` listener on `#keyboard-input` re-renders the display as the user types (when idle).

### Send mode specifics (~line 2120–2400)
- **Sidetone** (`sidetoneStart`/`sidetoneStop`, `ST_OSC`/`ST_GAIN` module-level vars, not on `S`): a dedicated oscillator+gain, ramped in/out with `setTargetAtTime` on key-down/up, so it can start and stop instantly on real user input without the click artifacts a hard on/off would cause.
- **`KEY`**: the keyer state object — `dit` (adaptive element duration, seeded from `S.charWPM` by `keyerAnchorSpeed()` and then drifted toward the user's actual keying via a running average), `down`/`downAt`/`upAt`, `marks` (accumulated `.`/`-` for the character in progress), `elems` (raw press durations, used to re-classify dit vs dah as `KEY.dit` adapts), and `charTimer`/`wordTimer` (gap-based auto-flush of the current character/word, mirroring how a real operator's pauses delimit characters).
- **Input sources**: the on-screen `#send-pad` button (pointer down/up) and holding **Space** both drive the same `keyDown()`/`keyUp()` pair — there's no separate code path per input device.
- **`classifyChar(elems)`** classifies a run of press durations into dits/dahs (via `DAH_RATIO * KEY.dit`) and looks up the resulting code against `MORSE`.
- **`sendType`**: `'copy'` prompts a character from `LESSONS[S.level]` (shared with Character mode) and grades the next keyed character (`startSendRound`/its answer handler); `'free'` just decodes and displays whatever's keyed, no grading, no `S.level` UI.
- **`btn-send-hear`/`btn-send-skip`/`btn-send-clear`** (`send-controls`, shown only in Send mode) play the correct code, skip to the next prompt, and clear the in-progress character respectively.

### UI helpers
- `resetPracticeUI(mode)` — shows the correct input row (`char-input-row`, `multi-input-row`, or `keyboard-input-row`), sets the hint text, and resets the display. Keyboard mode is special-cased here (it is neither the single-key nor the multi-input quiz layout).
- `syncButtons()` — enables/disables Play/Replay/Skip and shows Pause/Resume. `isMulti` (anything that uses `playString`, including keyboard) controls Pause visibility.
- `syncStats()` — updates the Session card and progress bar.

### Event wiring (`initEvents`, ~line 2415)
Mode tabs, output toggle, transport buttons, the three inputs, and global keyboard shortcuts. **Important:** the global `keydown` handler returns early when the event target is one of the inputs (`charInp`, `multiInp`, `kbInp`, `inText`) so typing — including Space and Enter — is not hijacked. In the keyboard textarea, plain **Enter** sends and **Shift+Enter** inserts a newline.

---

## Key implementation details / gotchas

- **`MORSE` is the validation authority.** Wikipedia sentence filtering, custom-set building, random generation, and keyboard skip-detection all gate on `MORSE[ch]`.
- **Farnsworth caveat (known, intentional):** Text Speed is calibrated to the PARIS reference word. Real prose with longer-than-average words (e.g. Wikipedia) *sounds* faster than the nominal Text Speed because only the gaps are stretched while the dits/dahs stay at Character Speed. This is correct Farnsworth behavior, not a bug — the same `timings()` path is used by every mode.
- **Display uses `textContent` in quiz modes but `innerHTML` in keyboard mode** (for the per-character spans). Switching modes via `resetPracticeUI` resets `textContent`, which clears any prior innerHTML.
- **CSS specificity:** `.kb-cur`/`.kb-skip` win over the inherited `#main-display.mode-keyboard` color because they match the child span directly. Adding `.playing` to the parent during keyboard playback would wash out the highlight — hence it's deliberately omitted.
- **Wikipedia mode** fetches random article intros (`WIKI_API_URL`), extracts sentences whose every character is in `MORSE`, and buffers them. Single-flight guarded by `WIKI_FETCH_PROMISE`; degrades gracefully offline.
- **Favicon** is an inline SVG data URI in `<head>` (an `SOS` pattern) — keeps the project a single file.
- **Noise/fading duration is read, not precomputed.** `playCode`/`playString` increment `at` (an `AudioContext` timestamp) synchronously through their `forEach` scheduling loops; by the time the loop finishes, `at` holds the exact end time, so `scheduleFading`/`startNoise` are called *after* the loop with `at - t0` as the duration rather than summing timings up front separately.

---

## File layout

```
morse-tutor/
├── index.html     the entire application (HTML + CSS + JS + data + favicon)
├── README.md      user-facing docs
└── HANDOFF.md     this file — developer map
```

---

## Adding a new mode (checklist)

1. Add a `<button class="mode-tab" data-mode="x">` and a `.sub-panel#sub-x`.
2. Add an input row in the Practice card if the existing ones don't fit.
3. Add a `start<X>Round()` and wire it into `startRound()` (and `replayRound()` if replay differs).
4. Handle the mode in `resetPracticeUI()` (input-row visibility, hint text) and `syncButtons()` (`isMulti` if it uses `playString`).
5. If it has its own input, exclude that element from the global `keydown` handler so typing isn't hijacked.
6. Update `README.md` and the `S.mode` comment.

The **keyboard mode** is the cleanest recent example to copy from — it touched exactly these spots.

---

## Pending / ideas

| Idea | Notes |
|---|---|
| Persist settings | `localStorage` for speed/tone/last-mode across sessions |
| Keyboard-mode history | Recall recently sent text |
| Adjustable tone envelope | Expose the 6 ms ramp / waveform |
| Export session stats | Download per-character accuracy as CSV |

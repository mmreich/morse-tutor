# Morse Code Tutor

A single-page Morse code copy-practice application using the **Farnsworth method** for timing.

## Features

- **Five practice modes**: Character, Word, Sentence, Prosign, and Custom copy practice
- **Farnsworth timing**: character speed and overall text speed are independently configurable, stretching inter-character and inter-word gaps while keeping element timing fast
- **Lessons ordered by element count**: progress from simple 1-element characters (E, T) through 6-element punctuation
- **Full character set**: 26 letters, 10 digits, and 14 punctuation marks
- **Pause/resume**: word and sentence playback can be paused mid-sequence with the **⏸ Pause** button or **Space** — **▶ Resume** or **↺ Replay** restart from the beginning of the current item
- **Master gain node**: replay and skip immediately silence any in-progress audio with no bleed-over
- **Adaptive repetition**: missed characters and words are weighted more heavily in selection — weight = `(errors+1)/(total+1)` — so struggling items appear more often without completely crowding out others
- **Per-character accuracy display**: lesson characters are color-coded after each answer (grey = unseen, green ≥ 80%, yellow ≥ 50%, red < 50%); hover any character to see the exact percentage
- **Prosign recognition**: 9 standard CW procedure signals sent as merged code (no inter-character gap); answer by typing the 2-letter abbreviation
- **Custom drill set**: hand-pick any combination of characters and prosigns to practice; group quick-selectors (A–Z, 0–9, Punct, Prosigns) toggle entire groups on or off
- **Answer comparison**: per-character highlighting for words, per-word highlighting for sentences
- **Clickable reference chart**: hear any character or prosign on demand; prosigns appear in a dedicated section below the character table
- **Ham Radio Quick Reference**: expandable panel listing all prosigns, 21 Q-codes, and 39 common CW abbreviations with definitions
- **Session stats**: correct, total, accuracy percentage, and streak tracking; Reset Stats clears the weight history

## Lessons

| Level | Elements | Characters |
|-------|----------|------------|
| Lv 1  | 1        | E T |
| Lv 2  | 2        | A I M N |
| Lv 3  | 3        | D G K O R S U W |
| Lv 4  | 4        | B C F H J L P Q V X Y Z |
| Lv 5  | 5        | 0 1 2 3 4 5 6 7 8 9 |
| Lv 6  | 5 (punct)| / ( & = + |
| Lv 7  | 6 (punct)| . , ? ! ) : ; - @ |
| Mix A | —        | All letters |
| Mix B | —        | All characters |

## How to Use

Open `index.html` in any modern browser — no build step or server required.

### Character mode
1. Select a lesson level (ordered by element count)
2. Click **Play** or press **Enter** — a random character from the lesson is sent
3. Type the character you heard — the answer submits immediately on the first keystroke
4. Press **Space** or **↺ Replay** to hear the character again

### Word mode
1. Pick a word category (3-letter, 4–5 letter, 6+ letter, Ham radio, All)
2. Click **Play** — a random word is sent; the display shows `Letter X / Y` as each character plays
3. Press **Space** or **⏸ Pause** to pause; press again or click **▶ Resume** / **↺ Replay** to restart
4. Type the word you heard and press **Enter** or **Submit**
5. Correct and incorrect characters are highlighted letter by letter

### Sentence mode
1. Pick a sentence category (Short, Medium, Long, Ham radio, All)
2. Click **Play** — the display shows `Word X / N · Letter Y / M` during playback
3. Press **Space** or **⏸ Pause** to pause; press again or click **▶ Resume** / **↺ Replay** to restart
4. Type the sentence you heard and press **Enter** or **Submit**
5. Correct and incorrect words are highlighted word by word

### Custom mode
1. Click individual symbols in the grid to toggle them in or out of the drill set; use the quick-select buttons (**A–Z**, **0–9**, **Punct**, **Prosigns**, **All**, **Clear**) to add or remove whole groups at once
2. Click **Play** — a symbol is drawn at random from your set, weighted toward ones you've missed
3. Type what you heard — the answer submits after **1 keystroke** for a character or **2 keystrokes** for a prosign, adapting automatically each round
4. Feedback shows the correct symbol, its code, and (for prosigns) its description on a wrong answer

### Prosign mode
1. Choose **Basic** (AR SK BT KN AS) or **All** (9 prosigns)
2. Click **Play** — the prosign is sent as a single merged sequence with no inter-character gap
3. Type the 2-letter abbreviation (e.g. `AR`, `SK`) — the answer submits automatically on the second keystroke
4. Feedback shows the prosign name, its description, and its code on a wrong answer

## Prosigns

| Prosign | Code | Meaning |
|---------|------|---------|
| `<AR>` | `.-.-. ` | End of message |
| `<SK>` | `...-.-` | End of contact |
| `<BT>` | `-...-` | Break / separator |
| `<KN>` | `-.--. ` | Go ahead (specific station) |
| `<AS>` | `.-...` | Wait / stand by |
| `<SN>` | `...-. ` | Understood |
| `<CT>` | `-.-.-` | Start of message |
| `<CL>` | `-.-..-..` | Closing station |
| `<HH>` | `........` | Error in keying |

## Farnsworth Timing Formula

```
tc = 1200 / charWPM          # ms per element unit
tw = 60000 / textWPM         # ms per word (target)
Ts = (tw − 31·tc) / 19      # stretched space unit
```

Inter-character gap = 3·Ts, inter-word gap = 7·Ts. When charWPM equals textWPM the formula reduces to standard PARIS timing.

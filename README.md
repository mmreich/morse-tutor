# Morse Code Tutor

A single-page Morse code copy-practice application using the **Farnsworth method** for timing.

## Features

- **Three practice modes**: Character, Word, and Sentence copy practice
- **Farnsworth timing**: character speed and overall text speed are independently configurable, stretching inter-character and inter-word gaps while keeping element timing fast
- **Lessons ordered by element count**: progress from simple 1-element characters (E, T) through 6-element punctuation
- **Full character set**: 26 letters, 10 digits, and 14 punctuation marks
- **Real-time pattern animation**: dots and dashes light up in sync with audio playback, tracking through multi-character words and sentences character by character
- **Master gain node**: replay and skip immediately silence any in-progress audio with no bleed-over
- **Answer comparison**: per-character highlighting for words, per-word highlighting for sentences
- **Clickable reference chart**: hear any character on demand
- **Session stats**: correct, total, accuracy percentage, and streak tracking

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
3. Type the word you heard and press **Enter** or **Submit**
4. Correct and incorrect characters are highlighted letter by letter

### Sentence mode
1. Pick a sentence category (Short, Medium, Long, Ham radio, All)
2. Click **Play** — the display shows `Word X / N · Letter Y / M` during playback
3. Type the sentence you heard and press **Enter** or **Submit**
4. Correct and incorrect words are highlighted word by word

## Farnsworth Timing Formula

```
tc = 1200 / charWPM          # ms per element unit
tw = 60000 / textWPM         # ms per word (target)
Ts = (tw − 31·tc) / 19      # stretched space unit
```

Inter-character gap = 3·Ts, inter-word gap = 7·Ts. When charWPM equals textWPM the formula reduces to standard PARIS timing.

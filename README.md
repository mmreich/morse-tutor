# Morse Code Tutor

A single-page Morse code copy-practice application using the **Farnsworth method** for timing.

## Features

- **Farnsworth timing**: character speed and overall text speed are independently configurable, stretching inter-character and inter-word gaps while keeping element timing fast
- **Lessons ordered by element count**: progress from simple 1-element characters (E, T) through 6-element punctuation
- **Full character set**: 26 letters, 10 digits, and 14 punctuation marks
- **Real-time pattern animation**: dots and dashes light up in sync with audio playback
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

1. Select a lesson level
2. Adjust character WPM and Farnsworth text WPM to your target speed
3. Click **Play** (or press **Enter**) to hear a character
4. Type the character you heard — the answer submits automatically
5. Press **Space** or **↺ Replay** to hear the character again before answering

## Farnsworth Timing Formula

```
tc = 1200 / charWPM          # ms per element unit
tw = 60000 / textWPM         # ms per word (target)
Ts = (tw − 31·tc) / 19      # stretched space unit
```

Inter-character gap = 3·Ts, inter-word gap = 7·Ts. When charWPM equals textWPM the formula reduces to standard PARIS timing.

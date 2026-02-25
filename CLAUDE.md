# CLAUDE.md — Speaking-Practice Repository Guide

This file provides comprehensive guidance for AI assistants working with this codebase.

---

## Project Overview

**英検スピーキングチャレンジ — 語句整序ゲーム**
(EIKEN Speaking Challenge — Word Ordering Game)

An interactive English speaking practice application for Japanese learners preparing for the EIKEN (英検) proficiency test. Users listen to and/or read English sentences, then speak them aloud. The Web Speech API captures the speech, compares it to the expected sentence, and provides immediate feedback.

---

## Architecture: Single-File Application

**This entire application lives in one file: `index.html` (923 lines).**

There is:
- No build system
- No package manager (no `package.json`, no `node_modules`)
- No framework (React, Vue, etc.)
- No backend or API server
- No external dependencies whatsoever

To run the application, simply open `index.html` in a browser. No installation or server required.

---

## File Structure

```
Speaking-Practice/
└── index.html    # The entire application (HTML + CSS + JavaScript)
```

### Sections within `index.html`

| Lines     | Content                                             |
|-----------|-----------------------------------------------------|
| 1–15      | DOCTYPE, `<head>` meta tags, viewport, charset     |
| 16–250    | Inline `<style>` block (CSS — partially minified)  |
| 251–923   | `<body>`: HTML structure + inline `<script>` block |

---

## Technology Stack

| Layer           | Technology                          |
|-----------------|-------------------------------------|
| Markup          | HTML5                               |
| Styling         | CSS3 (inline, partially minified)   |
| Logic           | Vanilla JavaScript (ES6+)           |
| Speech Input    | Web Speech API (`SpeechRecognition`)|
| Speech Output   | Web Speech API (`SpeechSynthesis`)  |
| Data Persistence| None (all state is in-memory)       |

Browser API requirements:
- `window.SpeechRecognition` or `window.webkitSpeechRecognition`
- `window.speechSynthesis` with `SpeechSynthesisUtterance`

**Browser compatibility:**
- Chrome/Chromium: Full support (recommended)
- Edge: Full support
- Firefox: Limited (speech recognition may not work)
- Safari/iOS: Partial (WebKit-prefixed APIs, inconsistent)

---

## Application Screens

The app has four screens managed via CSS `hidden` class toggling:

| Screen ID        | Purpose                                                  |
|------------------|----------------------------------------------------------|
| `screen-select`  | Grade and difficulty level selection (home screen)       |
| `screen-game`    | Active gameplay: displays question, timer, record button |
| `screen-result`  | Per-question result: correct/incorrect feedback          |
| `screen-score`   | Final score summary after all 5 questions                |

Two overlay elements also exist for animation effects:
- `#bomb-overlay` — Bomb explosion animation on wrong answers
- `#explosion-overlay` — Supplementary explosion effect

---

## Data Structures

### Question Bank (`QUESTIONS`)

Defined near the top of the `<script>` block. A nested object:

```javascript
QUESTIONS[grade][levels][level][questionIndex]
// e.g. QUESTIONS["5"][levels][1][0]
```

Each question object:
```javascript
{
  en: "I like apples.",          // English sentence (the answer)
  jp: "私はりんごが好きです。",      // Japanese translation shown to user
  grammar: "「I like ～.」は..."   // Japanese grammar explanation shown on result screen
}
```

**Scale:** 5 grades × 5 levels × 5 questions = **125 total questions**

Grades: `"5"`, `"4"`, `"3"`, `"準2"`, `"2"` (EIKEN grade names)
Levels: `1` through `5` (difficulty within each grade)

### Game State (`state`)

A single mutable object tracks all runtime state:

```javascript
state = {
  grade: null,        // Selected EIKEN grade string (e.g. "5")
  level: null,        // Selected difficulty level number (1–5)
  questions: [],      // Shuffled array of question objects for the session
  currentQ: 0,        // Index of the current question (0–4)
  totalQ: 5,          // Questions per session (always 5)
  score: 0,           // Count of correct answers
  results: [],        // Array of { question, spoken, correct } per question
  timer: null,        // setInterval ID for the countdown
  timeLeft: 30,       // Seconds remaining on current question
  isRecording: false, // Whether speech recognition is active
  recognition: null,  // SpeechRecognition instance
  answered: false     // Whether the current question has been answered
}
```

---

## Key Functions Reference

### Screen Navigation
| Function               | Purpose                                      |
|------------------------|----------------------------------------------|
| `showScreen(screen)`   | Show one screen, hide all others             |
| `selectGrade(grade)`   | Store grade, show level buttons              |
| `selectLevel(level)`   | Store level, start game                      |
| `backToGrades()`       | Return to grade selection                    |
| `goHome()`             | Reset state, return to home screen           |
| `retryLevel()`         | Restart current grade/level combination      |

### Game Flow
| Function                      | Purpose                                           |
|-------------------------------|---------------------------------------------------|
| `startQuestion()`             | Initialize question display and timer             |
| `nextQuestion()`              | Advance to next question or show final score      |
| `timeUp()`                    | Handle timer expiration (counts as wrong)         |
| `showResultScreen(correct)`   | Display per-question feedback screen              |
| `showFinalScore()`            | Display score summary after all 5 questions       |

### Speech
| Function                   | Purpose                                          |
|----------------------------|--------------------------------------------------|
| `initRecognition()`        | Create and configure SpeechRecognition instance  |
| `toggleRecognition()`      | Start or stop speech capture                     |
| `startRecognition()`       | Begin recording                                  |
| `stopRecognition()`        | End recording                                    |
| `checkAnswer(spoken)`      | Compare spoken string to expected answer         |
| `speakAnswer()`            | Read correct answer aloud via SpeechSynthesis    |

### Utilities
| Function               | Purpose                                           |
|------------------------|---------------------------------------------------|
| `normalizeText(text)`  | Lowercase, strip punctuation for comparison      |
| `shuffleArray(arr)`    | Fisher-Yates in-place shuffle                    |
| `startTimer()`         | Begin 30-second countdown and update UI          |
| `stopTimer()`          | Clear the countdown interval                     |
| `updateTimerDisplay()` | Animate the timer bar width                      |
| `updateProgressDots()` | Refresh the progress indicator dots              |
| `buildProgressDots()`  | Create DOM elements for progress dots            |

### DOM Helper
```javascript
const $ = id => document.getElementById(id);
```
Used throughout for concise element access.

---

## Coding Conventions

- **No classes** — purely functional, procedural style
- **camelCase** for all functions and variables
- **Arrow functions** preferred in newer code; `function` declarations used in older sections
- **Direct state mutation** — `state.score++`, `state.currentQ++`, etc.
- **Inline `onclick` attributes** on HTML elements rather than `addEventListener`
- **No event delegation** — each button has its own handler
- **Section comments** use `// ============================================================` style dividers

---

## How to Add New Questions

1. Open `index.html` and locate the `QUESTIONS` object (around line 253)
2. Find the target grade key (e.g. `"5"`) and level number (e.g. `1`)
3. Add a new object to that level's array:
   ```javascript
   { en: "She is my friend.", jp: "彼女は私の友達です。", grammar: "「She is ～.」は..." }
   ```
4. The game randomly selects 5 questions per session via `shuffleArray`, so additional questions increase variety

> Each level currently has exactly 5 questions. Adding more is safe — any number ≥ 5 works.

---

## How to Add a New Grade or Level

**New level within existing grade:**
- Add a new numeric key inside the grade's `levels` object with an array of ≥ 5 questions
- Add a corresponding level button in the HTML `screen-select` section

**New grade:**
- Add a new top-level key in `QUESTIONS` with nested `levels`
- Add a grade button in the HTML with `onclick="selectGrade('X')"` where X matches the key

---

## Development Workflow

1. **Edit** `index.html` in any text editor
2. **Open** (or refresh) `index.html` in Chrome for immediate feedback
3. **Test** speech recognition manually — there is no automated test suite
4. **Commit** and **push** changes directly (no build step needed)

```bash
git add index.html
git commit -m "Your message"
git push -u origin <branch-name>
```

---

## Testing

**There is no automated test suite.** All testing is manual:

1. Open `index.html` in Chrome
2. Click through grade/level selection
3. Test speech recognition by speaking the displayed sentence
4. Verify timer counts down and triggers "time up" correctly
5. Verify scores are tallied and displayed on the final screen
6. Test "Retry" and "Home" navigation buttons

Browser Console is the primary debugging tool. Use `console.log` freely during development.

---

## Known Limitations

- **No data persistence** — scores and progress reset on page refresh
- **No user accounts** — nothing is saved between sessions
- **Speech recognition accuracy** varies significantly by browser, microphone, and accent
- **Firefox** has limited or no support for `SpeechRecognition`
- **iOS Safari** support is inconsistent
- **All questions are hardcoded** — no CMS or data-loading mechanism
- **Japanese UI, English content** — the app is designed specifically for Japanese-speaking learners

---

## Important Notes for AI Assistants

- **Do not introduce a build system** unless the user explicitly requests it. The zero-dependency nature is intentional.
- **Do not split into multiple files** without user direction. The single-file approach simplifies deployment.
- **Preserve the inline CSS** style — do not move styles to external files unless asked.
- **Respect the `state` object** as the single source of truth for runtime data.
- **Answer correctness** is determined by `normalizeText()` comparison — when modifying answer checking logic, test with realistic speech inputs.
- **Grammar explanations** (`grammar` field) are written in Japanese — maintain that language when adding questions.
- **Never add external CDN links or npm dependencies** without explicit user approval.

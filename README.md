# Game Glitch Investigator

## Original Project Summary

The original project (Modules 1-3) was called the Game Glitch Investigator. It was an AI-generated Streamlit number guessing game that was intentionally shipped with three bugs: backwards comparison hints, a broken New Game button, and a state-persistence failure that caused the secret number to reset on every button click. The goal was to give students a realistic broken codebase to diagnose and repair, to show what it looks like to inherit AI-generated code in a real project.

---

## Summary

Game Glitch Investigator is a Streamlit web application where users play a number guessing game across three difficulty levels. The game was originally broken by design. The core of this project is the debugging, refactoring, and reliability-testing pipeline built around it.

This project matters because AI code generation is now common in real software teams, and trusting AI output without verification is a real risk. This system shows how to apply test-driven development, separation of concerns, and automated regression testing to catch bugs that AI introduced and prove they are fixed.

### Advanced AI Feature: Reliability and Testing System

The project includes a pytest test suite with a targeted regression test designed to catch the specific type-comparison bug that AI introduced. The tests directly validate the logic layer (`logic_utils.py`) that the live application imports and runs. If the bug were ever reintroduced, pytest would fail immediately. This is a reliability system that is fully integrated into the application logic, not a standalone script.

---

## Architecture Overview

The system uses a three-layer architecture separating UI, game logic, and tests:

```text
+--------------------------------------------------+
|               app.py  (UI Layer)                 |
|  Streamlit interface, session state, rendering   |
|  Imports: check_guess, parse_guess,              |
|           get_range_for_difficulty, update_score  |
+--------------------+-----------------------------+
                     | function calls
+--------------------v-----------------------------+
|           logic_utils.py  (Logic Layer)          |
|  Pure functions, no UI dependencies              |
|  check_guess  parse_guess                        |
|  get_range_for_difficulty  update_score          |
+--------------------+-----------------------------+
                     | imported by
+--------------------v-----------------------------+
|       tests/test_game_logic.py  (Test Layer)     |
|  pytest suite, regression tests                  |
|  4 tests: win / too-high / too-low /             |
|  numeric-vs-string comparison bug                |
+--------------------------------------------------+
```

All game logic lives in `logic_utils.py` as pure functions with no Streamlit imports. This makes every function independently testable without spinning up a web server or simulating UI interactions.

Session state in Streamlit is used to persist the secret number, attempt count, score, and game status across reruns, because Streamlit re-executes the entire script on every user interaction.

---

## Setup Instructions

### Prerequisites

Python 3.8 or higher.

### 1. Clone or download the project

```bash
git clone <your-repo-url>
cd applied-ai-system-project
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv
# Windows
venv\Scripts\activate
# macOS or Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Run the application

```bash
python -m streamlit run app.py
```

The app will open in your browser at `http://localhost:8501`.

### 5. Run the test suite

```bash
pytest
```

All 4 tests should pass. You will see output like:

```text
tests/test_game_logic.py ....                     [100%]
4 passed in 0.12s
```

---

## Sample Interactions

### Interaction 1: Successful game on Normal difficulty

```text
Difficulty: Normal (range: 1-100, 8 attempts)
Secret number (hidden): 42

Guess 1: 70   "Too High, Go LOWER"
Guess 2: 35   "Too Low, Go HIGHER"
Guess 3: 50   "Too High, Go LOWER"
Guess 4: 42   "Correct, You won. Final score: 60"
```

Streamlit launches a balloons animation. Score is calculated as `100 - 10*(4+1) = 50` plus accumulated deltas.

### Interaction 2: Invalid input handling

```text
User types: "banana"
System response: "That is not a number."
Attempt count: unchanged (bad input does not consume an attempt)

User types: "13.7"
System response: parsed as integer 13, guess proceeds normally.
```

The `parse_guess` function handles empty strings, non-numeric text, and floats without crashing. No unhandled exceptions reach the UI.

### Interaction 3: Game over on Hard difficulty

```text
Difficulty: Hard (range: 1-50, 5 attempts)
Secret number (hidden): 37

Guess 1: 25   "Too Low, Go HIGHER"
Guess 2: 40   "Too High, Go LOWER"
Guess 3: 33   "Too Low, Go HIGHER"
Guess 4: 38   "Too High, Go LOWER"
Guess 5: 36   "Too Low, Go HIGHER"
Result: "Out of attempts. The secret was 37. Score: -25"
```

The game locks and shows an error message. Clicking New Game resets all session state and starts a fresh round.

### Interaction 4: pytest regression test catching the original bug

The test that specifically targets the AI-introduced bug:

```python
# test_hints_use_numeric_not_string_comparison
outcome, _ = check_guess(7, 42)
assert outcome == "Too Low"
```

```text
PASSED  [100%]
```

With the old broken code, `7 > 42` evaluated as `True` because the secret was cast to a string and `"7" > "42"` is `True` alphabetically. The test catches this regression permanently.

## Design Decisions

**Separating `logic_utils.py` from `app.py`**
Streamlit cannot run outside a live session, so game logic lives in pure Python functions that pytest can import and test directly.

**Using pytest instead of only manual testing**
Manual testing found the bug but does not prevent it from returning. Automated tests run in milliseconds and fail loudly if the string-conversion bug is reintroduced.

**Streamlit session state**
Session state is scoped to one browser tab, resets cleanly, and needs no external services — the right fit for a single-player game.

**Three difficulty levels**
`get_range_for_difficulty` is a parameterized, testable function rather than hardcoded constants scattered through the UI.

**Debug info expander**
Shows the secret number in a collapsible panel so students can verify hints during development. A production game would remove it.

## Testing Summary

| Test | Input | Expected | Result |
| ---- | ----- | -------- | ------ |
| `test_winning_guess` | `check_guess(50, 50)` | `"Win"` | PASSED |
| `test_guess_too_high` | `check_guess(60, 50)` | `"Too High"` | PASSED |
| `test_guess_too_low` | `check_guess(40, 50)` | `"Too Low"` | PASSED |
| `test_hints_use_numeric_not_string_comparison` | `check_guess(7, 42)` | `"Too Low"` | PASSED |

4 out of 4 tests passed. The regression test (`check_guess(7, 42)`) was written before the fix, failed immediately, then passed once the string-conversion block was removed. Separating logic into `logic_utils.py` kept every test to a single import and a direct assertion — no mocking or fixtures needed.

**Screenshots:**

![Winning game](win.png)

![All 4 pytest tests passing](pytest.png)

## Ethical Reflection

### Limitations and biases

The game logic is deterministic. The broader limitation is that this project exposes students to only one class of AI bug: type comparison errors. Real AI-generated bugs are more varied, and familiarity with one pattern does not generalize broadly.

### Potential misuse

Minimal surface for a number game. The larger concern is deploying AI-generated code without review. The test suite addresses this by creating a structural check before trusting generated output.

### What surprised me while testing

The broken code passed visual inspection. The bug was not in the comparison itself but in a silent string cast applied to `secret` before it ran. AI bugs often hide in the wiring between pieces, not inside the pieces themselves.

**AI collaboration**

Claude identified the bug correctly and suggested `check_guess(7, 42)` as the regression test. Its first explanation used undefined terms and had to be simplified before I could act on it — a reminder that AI output is calibrated to an average reader, not a specific one.

## Reflection

This project changed how I think about AI-generated code. Fluent syntax does not mean correct logic. The most important habit I am taking forward is writing a failing test before applying a fix — it turns "did I fix it" from a guess into something provable. AI is better at diagnosis than explanation; getting a useful answer sometimes requires follow-up prompts to make the output actionable.

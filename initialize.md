# Puzzmo Solvers — Product / Build Prompt (Organized)

## Goal
Build a new **React + Next.js** web app named **Puzzmo Solvers**, deployed on **Vercel**. The app provides solver tools for four Puzzmo games:

- **Typeshift**
- **Memoku**
- **Wordbind**
- **Spelltower**

A shared requirement across the word-based solvers:
- **All words returned by solvers must exist in the Wordnik dictionary**, since that is the same dictionary Puzzmo uses.
- The Wordnik API key/config will be provided later.

---

## Global UI / Layout Requirements

### Navigation
- A **top navbar** appears on every page.
- Navbar links:
  - Home
  - Typeshift
  - Memoku
  - Wordbind
  - Spelltower

### Footer
- A footer appears on every page and must include:
  - **“Not affiliated with Puzzmo.”**

### Routing
Each game must be on its **own page/route**:
- `/` (landing page)
- `/typeshift`
- `/memoku`
- `/wordbind`
- `/spelltower`

---

## Landing Page (`/`)
Create a default landing page that includes:
- A brief overview of **Puzzmo** (high-level: daily puzzle games platform).
- A short description of each supported game and what this site does (it provides “solver” utilities).
- A clear call-to-action to pick a game from the navbar.

(Keep it brief, informational, and non-affiliated.)

---

## External Dependency: Wordnik
### Usage
- Wordnik will be used to validate candidate words:
  - If Wordnik says the word exists in the dictionary, it is eligible.
  - If not, it must not appear in results.

### Implementation Expectations (high-level)
- A shared dictionary/word-validation module so all solvers can call the same logic.
- Cache results to reduce API calls (especially important for large search spaces).

---

# Game Pages

## 1) Typeshift (`/typeshift`)

### Game Model
- The puzzle is represented as **N columns**, each containing **M letters** (columns may be jagged; i.e., not all columns are the same height).
- In the actual game, you “shift” each column up/down to line letters up into valid words.
- A valid word is formed by taking **one letter from each column**, **left-to-right**, producing a word of length **N**.

**Example columns**
- `MWSB`, `LAIT`, `SGOE`, `AIOE`, `CRDK`
- One valid solution word might be `WAGER` (one letter from each column, in order).

### “Core Solution” Requirement
- Define the **core solution** size as **X = height of the tallest column**.
- The “minimal core solution” is **X words** that collectively **use all letters** (in the sense intended by the puzzle: each column’s letters are consumed across the set of words).
  - Example: if all columns are height 4, the minimal core solution is **4 words**.

### UI Requirements
- Step 1: User enters/selects **number of columns (N)**.
- Step 2: Render **N column inputs** where the user populates each column with letters (top-to-bottom).
  - Support jagged columns (allow variable length per column).
  - Validate entries as letters only.
- Step 3: A **Solve** button runs the solver.

### Solver Output Requirements
- Find **all possible valid words** that can be formed from the columns under the shifting/selection rules.
- Return the **minimal core solution set** (X words) that satisfies “uses all letters” as described above.
- All output words must be **Wordnik-valid**.

---

## 2) Memoku (`/memoku`)

### Game Model
- Memoku is a “fancy sudoku,” but **for this solver treat it as standard Sudoku**.
- Board is a classic **9x9** Sudoku grid with digits 1–9.

### UI Requirements
- Display a **blank 9x9 grid**.
- User enters the given numbers (prefilled clues).
- Provide a **Solve** button that solves the Sudoku.

### Star Marking Feature
- User can optionally mark **exactly 3 squares** as “starred,” each with a distinct color:
  - Gold
  - Purple
  - Green
- When the Sudoku is solved:
  - The **numbers in the starred squares** should be rendered in the **same corresponding color**.

### Solver Output Requirements
- Must return a valid solved Sudoku grid (or an error if unsolvable).
- Star coloring must reflect the user’s chosen star placements.

---

## 3) Wordbind (`/wordbind`)

### Input
- A **free text field** where the user enters **2–3 words** (source words).

### Constraints / Rules
1. **Exact-word exclusion**
   - The final solution **must not include** the **exact source words** as outputs.

2. **Ordered-letter rule (left-to-right)**
   - Output words must be formed by selecting letters from the source text **in the order they appear** (a subsequence constraint).
   - Example: source `SAMPLE CARD`
     - `SAMPLER` is valid (letters appear in order in the source stream)
     - `DREAM` is invalid if it requires using letters out of order (e.g., `D` comes “too late” relative to other required letters)

3. **Double-letter allowance**
   - A letter may be used **twice** to create a **double letter** in the output.
   - Example: source `SAMPLE`
     - `APPLE` is valid by using the `P` twice as a double letter.

### UI Requirements
- A single input for the source phrase (2–3 words).
- A **Solve** button to generate solutions.
- A results area that lists:
  - The found words (Wordnik-valid)
  - Optionally counts/metrics (e.g., total words found)

### Solver Output Requirements
- Goal: **make as many valid words as possible** from the source input under the constraints above.
- All output words must be **Wordnik-valid**.
- Must not output the exact source words.

---

## 4) Spelltower (`/spelltower`)

### Board Model
- Grid size is **9 columns wide x 13 rows tall**.
- Each cell can be:
  - A **letter tile**
  - **blank** (empty) tile
  - **red** letter tile (special clearing effect)
  - **starred** letter tile (special scoring effect)

### Word Formation Rule
- A word can start on **any letter tile**.
- The next letter must be **adjacent** to the previous letter, continuing as a path that “snakes” through the grid.
  - “Adjacent” should be implemented as both up/down/left/right, as well as all 4 diagonals.
- **Blank tiles cannot be used** in a word.
- A letter cannot be used twice (the word cannot loop back on itself).

### Clearing / Gravity Rules
When a word is selected:
1. **Primary removal**
   - All tiles used in the word are removed from the board.

2. **Adjacent removal (conditional)**
   - Normally, **all tiles adjacent** to the word’s tiles are also removed.
   - Exception: if the word length is **4 or fewer letters**, adjacent tiles are **not** cleared.
   - Additional exception: **blank tiles adjacent to a cleared word should still be cleared**, even for short words (≤ 4).

3. **Red tile effect**
   - If a **red letter tile** is used in the word:
     - The **entire row** containing that red tile is cleared.

4. **Blank tile clearing rule**
   - Blank tiles cannot be selected into a word.
   - To clear blanks, they must be **adjacent to a word** when that word is cleared (including the short-word exception behavior noted above).

5. **Gravity**
   - After clearing, everything **shifts down** to fill empty spaces.

### Scoring / Optimization Goal
- The goal is to use all letters and **maximize score**.
- **Starred tile scoring**
  - If a **starred letter tile** is used in a word, the **score for that word is doubled**.
- Generally:
  - **Longer words are better** (higher score).

### UI Requirements
- Render an editable **9x13 grid**.
- Each cell should support:
  - Setting a letter
  - Marking as blank
  - Marking a letter as red
  - Marking a letter as starred
- Provide a **Solve** button.
- Output should include:
  - A proposed sequence of words (a play path)
  - The resulting score (or scoring breakdown)
  - A visualization or step list is strongly preferred (at minimum, list words in order)

### Solver Output Requirements
- Must generate valid word sequences that follow adjacency rules.
- Must apply clearing/gravity rules correctly to simulate the board.
- Must prioritize maximizing score, leveraging:
  - Star multipliers
  - Longer words
  - Efficient clearing

All words must be **Wordnik-valid**.

---

## Non-Functional Requirements / Expectations

### Performance & Practicality
- Some solvers (especially Spelltower and Wordbind) can explode combinatorially.
- Implement:
  - Caching of Wordnik lookups
  - Reasonable search pruning / heuristics
  - Guardrails to prevent UI lockups (e.g., web workers or incremental search if needed)

### Code Organization (suggested)
- Shared components:
  - Navbar, footer, layout
  - Reusable grid components (9x9, 9x13)
- Shared logic:
  - Wordnik validation + caching
  - Common “word search” utilities where applicable

---

## Branding / Legal Note
- This project is a fan/utility tool.
- Footer must clearly state:
  - **Not affiliated with Puzzmo.**

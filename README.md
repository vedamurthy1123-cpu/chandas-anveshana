# 📿 Chandas Identifier — Sanskrit Metrical Analyzer

> A beautiful, frontend-only web application that identifies Sanskrit poetic meters (Chandas) from transliterated or Unicode verse input — with Laghu/Guru syllable visualization, strict rule-based detection, and a glassmorphism UI.

---

## 🔭 Overview

**Chandas Identifier** is an interactive browser-based tool for analyzing the metrical structure of Sanskrit verses. Users enter a Sanskrit verse in either Unicode (Devanāgarī transliteration with diacritics) or ITRANS-style romanized text, and the application:

1. **Normalizes** the input across multiple encoding conventions
2. **Extracts** the Laghu (light) / Guru (heavy) syllable pattern
3. **Matches** the pattern against classical Sanskrit chandas rules
4. **Visualizes** the result as color-coded metrical badges with an animated results panel

It is designed for students, scholars, developers, and enthusiasts of Sanskrit prosody (Chandasśāstra), and runs entirely in the browser — no backend, no server, no installation required.

---

## ✨ Features

### 🔤 Input & Preprocessing
- **Multi-encoding support**: Accepts Unicode diacritics (`ā`, `ī`, `ū`, `ṛ`, `ṃ`, `ḥ`) and ITRANS-style uppercase shortcuts (`A`, `I`, `U`, `M`, `H`, `R`)
- **Automatic normalization**: Converts double-vowel sequences (`aa → ā`, `ii → ī`, `uu → ū`) to their long vowel equivalents
- **Diphthong collapsing**: `ai` → internal `E`, `au` → internal `O` for uniform iteration
- **Punctuation stripping**: Removes `. , ? ! ; : | " ' \` characters before analysis
- **Aspirated consonant normalization**: Treats digraph aspirates (`kh`, `gh`, `ch`, `jh`, `ṭh`, `ḍh`, `th`, `dh`, `ph`, `bh`, `sh`) as single consonant units (`C`), preventing incorrect conjunct-cluster detection

### 📏 Line Validation
- **Maximum 4 lines** enforced in real-time; input is automatically truncated at the 4th newline
- **Live line counter** (`Lines: X / 4`) displayed beneath the textarea
- **Warning state**: Counter turns red when the 4-line limit is reached
- **Empty input handling**: Gracefully returns "Unknown Chandas" with an appropriate message when no syllables are detected

### 🧠 Syllable Extraction (Laghu / Guru Logic)
- Iterates character-by-character over the cleaned text
- Identifies every vowel position as one syllable
- Applies **lookahead consonant counting** across word boundaries (spaces, hyphens, newlines are skipped to support cross-word sandhi weight rules)
- **Guru (G) conditions** — a syllable is heavy if:
  - It contains a long vowel (`ā`, `ī`, `ū`, `e`, `o`, diphthongs `E`/`O`)
  - It is followed by **two or more consonants** (conjunct cluster)
  - It is followed by **Anusvara** (`ṃ`) or **Visarga** (`ḥ`) — immediately treated as position-2 consonant, forcing Guru status
  - The syllable ends a word with `m` or `h` as the final character (word-end heuristic)
- **Laghu (L) conditions**: A short vowel followed by zero or one consonant before the next vowel

### 🎯 Chandas Detection (Pattern Matching)
- **Anuṣṭubh** (Śloka): 8-syllable pādas
  - 5th syllable must be Laghu (`L`)
  - 6th syllable must be Guru (`G`)
  - 7th syllable: Guru (`G`) in odd pādas (pathyā), Laghu (`L`) in even pādas
  - Supports multi-pāda verses (16, 24, 32 syllables…)
- **Indravajrā**: 11-syllable pādas matching `G G L G G L L G L G G`
- **Upendravajrā**: 11-syllable pādas matching `L G L G G L L G L G G`
- **Padānte guru normalization**: The final (11th) syllable of each pāda is normalized to `G` per Piṅgala's *padānte go vikalpena* rule before strict matching
- **Unknown Chandas**: Returned with syllable count when no rule matches

### 🖼️ Visualization & UI
- **Syllable badges**: Each syllable rendered as a color-coded badge
  - `L` (Laghu) — blue-tinted badge with blue border
  - `G` (Guru) — red-tinted badge with red border
- **Pāda grouping**: Badges are organized into rows, one row per pāda (chunk of 8 or 11)
- **Pattern toggle**: "Show / Hide Metrical Pattern" button reveals/collapses the badge display with smooth CSS height + opacity animation
- **Animated results panel**: Slides up with a `slideUp` keyframe animation on each analysis
- **Re-trigger animation**: Panel is briefly hidden and re-shown on each click to replay the animation
- **Success / Unknown color coding**: Detected chandas name shown in green; unknown in red

### 🎨 UI / UX Design
- **Glassmorphism card**: `backdrop-filter: blur(16px)` frosted-glass container with semi-transparent dark background
- **Gradient background**: Full-page radial/linear gradient from `#0f172a` (dark navy) to `#1e1b4b` (deep indigo)
- **Ambient glow**: Fixed radial glow orb centered on the page for atmospheric depth
- **Gradient heading**: Title rendered with a `linear-gradient` clip from blue `#60a5fa` to purple `#a78bfa`
- **Hover animations**: Analyze button lifts (`translateY(-2px)`) on hover and presses down on click
- **Focus glow**: Textarea glows blue (`box-shadow` with primary color) on focus
- **Google Fonts**: Uses the `Outfit` typeface (weights 300, 400, 600, 700)
- **Mobile responsive**: Padding and font sizes adjust at `≤480px` viewport width

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| **HTML5** | Semantic document structure, textarea input, button elements |
| **CSS3** | Glassmorphism, gradients, animations, responsive layout, CSS variables |
| **Vanilla JavaScript (ES6+)** | Input validation, text normalization, syllable extraction, pattern matching, DOM rendering |
| **Google Fonts (Outfit)** | Typography |
| **No frameworks / No backend** | Entirely frontend; works offline after first load |

---

## ⚙️ How It Works

### Step 1 — Input & Validation
The user types a Sanskrit verse (up to 4 lines) into the textarea. As they type, JavaScript listens for `input` events and:
- Counts newlines; truncates to 4 lines if exceeded
- Updates the live counter below the textarea
- Turns the counter red at the limit

### Step 2 — Text Normalization (`extractPattern`)
When "Analyze Verse" is clicked:

```
Input text
  → Replace ITRANS uppercase shortcuts (A→ā, I→ī, U→ū, M→ṃ, H→ḥ, R→ṛ)
  → Strip punctuation, convert to lowercase
  → Normalize doubled vowels (aa→ā, ii→ī, uu→ū)
  → Collapse diphthongs (ai→E, au→O)
  → Collapse aspirated digraphs (kh/gh/ch/… → C)
```

### Step 3 — Syllable Weight Calculation
The cleaned string is iterated character-by-character. At every vowel position:

```
Is the vowel long (ā/ī/ū/e/o/E/O)?  → Guru
Is the vowel short?
  → Scan ahead (skipping spaces/hyphens/newlines) until next vowel
  → Count consonants encountered
  → If anusvara/visarga found → force consonantCount = 2
  → If word-ending m/h found → force consonantCount = 2
  → If consonantCount ≥ 2 → Guru
  → Otherwise → Laghu
```

Result: a string of `G` and `L` characters — the **metrical pattern**.

### Step 4 — Chandas Matching (`matchChandas`)

```
pattern.length % 8 === 0?
  → Split into 8-char chunks (pādas)
  → Check each: p[4]==='L' && p[5]==='G'
  → Check 7th: G for odd pādas, L for even pādas
  → Match → "Anuṣṭubh"

pattern.length % 11 === 0?
  → Normalize last char of each 11-chunk to 'G'
  → Check each chunk vs "GGLGGLLGLGG" → "Indravajrā"
  → Check each chunk vs "LGLGGLLGLGG" → "Upendravajrā"

No match → "Unknown Chandas"
```

### Step 5 — Rendering
- The chandas name is injected into the heading with success/error colour
- A description string explains the match (syllable count, pāda count, rules applied)
- Badges are rendered by `renderBadges()` — one `<div class="badge-container">` per pāda, one `<span class="syllable-badge badge-L/G">` per syllable

---

## 📐 Supported Chandas

| Chandas | Syllables/Pāda | Pattern | Key Rules |
|---|---|---|---|
| **Anuṣṭubh** (Śloka) | 8 | Flexible | 5th=L, 6th=G, 7th=G (odd pāda) / 7th=L (even pāda) |
| **Indravajrā** | 11 | `G G L G G L L G L G G` | All pādas identical; last syllable normalized to G |
| **Upendravajrā** | 11 | `L G L G G L L G L G G` | All pādas identical; last syllable normalized to G |

---

## 🔬 Algorithm Deep Dive

### Laghu (L) — Light Syllable
A syllable is Laghu when:
- Its vowel is **short** (`a`, `i`, `u`, `ṛ`, `ḷ`)
- It is followed by **at most one consonant** before the next vowel

### Guru (G) — Heavy Syllable
A syllable is Guru when **any** of the following hold:
1. Its vowel is **inherently long**: `ā`, `ī`, `ū`, `e`, `o`, `ai` (→E), `au` (→O)
2. It is followed by a **conjunct cluster** (2+ consonants) — even across word boundaries
3. It is followed by **Anusvara** (`ṃ`) — immediately forces Guru
4. It is followed by **Visarga** (`ḥ`) — immediately forces Guru
5. Its word ends with `m` or `h` as the absolute last character before a space/newline

### Aspirated Consonant Collapsing
Digraphs like `kh`, `gh`, `th`, `dh`, `ph`, `bh`, `sh`, `ch`, `jh`, `ṭh`, `ḍh` are replaced with a single placeholder `C` before consonant counting. This prevents `th` in a cluster being miscounted as two separate consonants.

### Cross-Word Sandhi Weight
Spaces, hyphens, and newlines between words are **skipped during lookahead**. This reflects the Sanskrit prosodic principle that word boundaries do not reset consonant cluster counting — a short vowel at the end of one word can become Guru due to consonants starting the next word.

### Anuṣṭubh Strict Validation
- The 5th and 6th syllables of every pāda are mandatory fixed positions.
- The 7th syllable follows the **pathyā** cadence: Guru in odd pādas, Laghu in even pādas.
- Single-pāda input accepts either G or L at position 7 (no odd/even context).

### Indravajrā / Upendravajrā Normalization
Per Piṅgala's rule *padānte go vikalpena* ("at the end of a verse-foot, Guru is optional"), the 11th syllable of each pāda is forced to `G` before comparison. This allows verses where the last syllable happens to be short to still match correctly.

---

## 🎨 UI / UX Highlights

| Feature | Implementation |
|---|---|
| Glassmorphism card | `backdrop-filter: blur(16px)` + semi-transparent `rgba` background |
| Gradient page background | `linear-gradient(135deg, #0f172a → #1e1b4b)` |
| Ambient glow orb | Fixed `position` div with `radial-gradient`, `pointer-events: none` |
| Gradient title text | `linear-gradient` + `-webkit-background-clip: text` |
| Slide-up animation | `@keyframes slideUp` on `.result-panel` — opacity 0→1, translateY 20px→0 |
| Animation re-trigger | Panel's `visible` class removed, browser reflow allowed (50ms timeout), then re-added |
| Button hover lift | `transform: translateY(-2px)` + enhanced box-shadow on hover |
| Button press | `transform: translateY(1px)` on `:active` |
| Focus glow | `box-shadow: 0 0 15px var(--primary-glow)` on textarea focus |
| Laghu badge | Blue background `rgba(59,130,246,0.15)`, blue border, blue text `#60a5fa` |
| Guru badge | Red background `rgba(239,68,68,0.15)`, red border, red text `#f87171` |
| Pattern expand/collapse | CSS `height: 0 → auto` + `opacity 0 → 1` transition on `.expanded` class |
| Responsive breakpoint | `@media (max-width: 480px)` — reduced padding and heading font size |
| Warning line counter | `.warning` class changes counter color to `var(--error)` red |

---

## 📁 Project Structure

Since this is a single-file project, everything lives in `index.html`:

```
index.html
│
├── <head>
│   ├── Meta tags (charset, viewport)
│   ├── Google Fonts link (Outfit)
│   └── <style> — All CSS
│       ├── CSS Custom Properties (:root variables)
│       ├── Body & layout styles
│       ├── Ambient glow effect
│       ├── Glassmorphism container
│       ├── Header & typography
│       ├── Input group & textarea
│       ├── Line counter styles
│       ├── Analyze button styles & animations
│       ├── Result panel & slideUp animation
│       ├── Pattern toggle button
│       ├── Pattern display (collapsed/expanded)
│       ├── Syllable badge styles (L / G)
│       └── Responsive media query
│
├── <body>
│   ├── .ambient-glow (decorative background orb)
│   └── .container (glassmorphism card)
│       ├── <header> (title + subtitle)
│       ├── .input-group (textarea + line counter)
│       ├── #analyze-btn
│       └── #result-panel
│           ├── #chandas-name (detected meter heading)
│           ├── #explanation (rule description)
│           ├── #toggle-pattern (show/hide button)
│           └── #pattern-display (dynamically populated badges)
│
├── <!-- Architecture pseudocode comment block -->
│
└── <script>
    ├── DOM references
    ├── Input event listener (validation + line counter)
    ├── Toggle pattern button listener
    ├── extractPattern(text) — normalization + Laghu/Guru extraction
    ├── matchChandas(pattern) — rule-based chandas identification
    ├── renderBadges(pattern) — dynamic badge DOM generation
    └── analyzeBtn click listener — orchestrates the full pipeline
```

---

## 🚀 Installation & Usage

### Prerequisites
None. This is a pure HTML/CSS/JS application.

### Running Locally

```bash
# Clone the repository
git clone https://github.com/your-username/chandas-identifier.git

# Navigate into the directory
cd chandas-identifier

# Open in your browser — no server needed
open index.html
# or on Linux:
xdg-open index.html
# or just double-click index.html in your file manager
```

### Using Online
If deployed (e.g., GitHub Pages):
1. Visit the live URL
2. Enter a Sanskrit verse in the textarea
3. Click **Analyze Verse**
4. View the detected chandas name and description
5. Click **Show Metrical Pattern** to expand the Laghu/Guru badge visualization

---

## 📝 Example Input & Output

### Example 1 — Anuṣṭubh (Śloka)

**Input:**
```
dharmo rakṣati rakṣitaḥ
```

**Expected Output:**
- **Detected Chandas**: Anuṣṭubh ✅
- **Description**: Matched 8-syllable pādas (1 line). 5th=L, 6th=G, 7th follows odd/even rule.
- **Pattern**: `L G L L L G G L` (badges displayed in a row)

---

### Example 2 — Indravajrā

**Input (transliterated, 4 lines of 11 syllables each):**
```
tvam eva mātā ca pitā tvam eva
tvam eva bandhuś ca sakhā tvam eva
tvam eva vidyā draviṇaṃ tvam eva
tvam eva sarvaṃ mama deva deva
```

**Expected Output:**
- **Detected Chandas**: Indravajrā ✅
- **Pattern per line**: `G G L G G L L G L G G`

---

### Example 3 — Unknown Chandas

**Input:**
```
hello world
```

**Expected Output:**
- **Detected Chandas**: Unknown Chandas ❌
- **Description**: No match found. Detected N syllables unsupported by strict rules.

---

### Example 4 — ITRANS-style Input

**Input (using uppercase shortcuts):**
```
dharmо rakSati rakSitaH
```
`H` → `ḥ`, `A` → `ā` etc. are normalized automatically before processing.

---

## ✅ Validation Rules

| Rule | Behavior |
|---|---|
| Maximum 4 lines | Input is hard-truncated to 4 lines in real time |
| Empty input | Returns "Unknown Chandas — No valid syllables detected" |
| Line counter | Shows `Lines: X / 4`; turns red at 4 |
| Unknown pattern | Shows syllable count and "No match found" message |
| Punctuation | `.`, `,`, `?`, `!`, `;`, `:`, `\|`, `"`, `'`, `\` are stripped |
| Uppercase ITRANS | `A I U M H R` pre-converted before `toLowerCase()` |

---

## 🔮 Future Improvements

- 📚 **More Chandas Support**: Triṣṭubh, Jagatī, Vasantatilakā, Mālinī, Śārdūlavikrīḍita, Mandākrāntā, and others
- 🤖 **AI-Based Suggestions**: Suggest closest matching meter when strict match fails
- 🎙️ **Voice Input**: Speak the verse using the Web Speech API for hands-free analysis
- 📷 **Sanskrit OCR**: Upload a photograph of a manuscript or printed text and detect chandas from extracted text
- 🔊 **Audio Recitation**: Play back the verse with correct Laghu/Guru timing (short/long beats)
- 🌐 **Devanāgarī Script Input**: Accept native Sanskrit script (not just transliteration)
- 💾 **Export Options**: Download analysis as PDF or image
- 📖 **Meter Learning Mode**: Interactive tutorial explaining each chandas rule with examples
- 🧩 **Mixed-Meter Detection**: Identify verses with pādas in different meters (e.g., Upajāti — a mix of Indravajrā and Upendravajrā)
- 🖥️ **Backend API**: REST endpoint for batch processing of large texts (corpora, grantha analysis)
- 🗃️ **History**: Save and revisit previously analyzed verses within the session
- 🌍 **Multilingual UI**: Interface translations for Hindi, Sanskrit, and other Indian languages

---

## 🤝 Contributing

Contributions are warmly welcome! Here's how to get involved:

### Getting Started

1. **Fork** the repository on GitHub
2. **Clone** your fork:
   ```bash
   git clone https://github.com/your-username/chandas-identifier.git
   ```
3. **Create a branch** for your feature or fix:
   ```bash
   git checkout -b feature/add-tristubh-support
   ```
4. **Make your changes** — all logic is in `index.html`
5. **Test thoroughly** with known Sanskrit verses
6. **Commit** with a clear message:
   ```bash
   git commit -m "feat: add Triṣṭubh chandas detection (11-syllable fixed pattern)"
   ```
7. **Push** and open a **Pull Request**

### Contribution Guidelines

- Follow the existing code style (vanilla JS, no frameworks)
- Each new chandas rule should be added inside `matchChandas()` with a clear comment
- Add test verses in the PR description demonstrating the new feature
- Keep the UI consistent with the current glassmorphism design language
- Document any new Sanskrit prosody rules in comments within the code

### Good First Issues

- Add support for a new chandas (e.g., Triṣṭubh, Vasantatilakā)
- Improve the ITRANS normalization to handle more encoding variants
- Add aria-labels and keyboard accessibility improvements
- Write a test suite (Jest or plain JS) for `extractPattern` and `matchChandas`

---

## 📄 License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2024 Chandas Identifier Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Acknowledgements

- **Piṅgala** — Ancient Indian mathematician and prosodist, author of the *Chandaḥśāstra* (c. 3rd–2nd century BCE), the foundational treatise on Sanskrit meter from which all chandas rules in this project derive
- **Chandasśāstra tradition** — The classical Sanskrit prosodic tradition that formalized Laghu/Guru syllable weight, pāda structure, and the rules encoded in this analyzer
- **Sanskrit prosody scholars** — Whose modern commentaries and digital resources make it possible to implement these ancient rules in software
- The open-source community for inspiration in building accessible, educational tools for classical languages

---

*Built with ❤️ for Sanskrit, linguistics, and open-source learning.*

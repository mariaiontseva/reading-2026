# Mrigendratantra Yogapada — Interactive Sanskrit Reader

A sentence-by-sentence interactive reader for the **Yogapada** of the **Mrigendratantra** with the commentary (*vrtti*) of **Bhatta Narayanakantha**. Designed as a study tool for Sanskrit students working through Shaiva Siddhanta philosophical texts.

**[Live Demo](https://mariaiontseva.github.io/reading-2026/)** (enable GitHub Pages to activate)

---

## Overview

Each sentence of the text is presented as an interactive card with:

- Color-coded Sanskrit with hover tooltips
- Word-by-word translation (English word order)
- Smooth literary translation
- Collapsible panels: semantic tree, sandhi resolution, vocabulary, notes

The reader distinguishes between **mula** (root verses of the tantra) and **vrtti** (Narayanakantha's commentary), visually nesting commentary under the verse it explains.

---

## Architecture

### Single-file HTML

The entire application is a single `index.html` file with no build step, no dependencies, and no framework. It loads two Google Fonts (`Crimson Pro` for body text, `Inter` for UI elements) and is otherwise fully self-contained.

```
index.html
├── <style>        — all CSS (~340 lines)
├── <body>         — semantic HTML cards with data attributes
├── #globalTip     — single tooltip element (fixed-position)
└── <script>       — tab switching + tooltip positioning (~65 lines)
```

### Why single-file?

Sanskrit study tools need to work offline, be trivially shareable, and avoid toolchain complexity. A single HTML file can be opened from a USB stick, emailed, or hosted on GitHub Pages with zero configuration.

---

## Visual Design System

### Card Hierarchy

There are two levels of cards reflecting the structure of a Sanskrit text with commentary:

```
┌─── verse-card (blue left border) ──────────────────────┐
│  [1] Mangalashloka                          MULA       │
│                                                         │
│  ┌─── comm-card (purple left border, indented) ───────┐ │
│  │  VRTTI  Commentary on the verse above              │ │
│  │  ──── connected by a horizontal line ────          │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

- **`verse-card`** — blue accent (`#1e40af`), header background `#eff6ff`. Used for mula (root text of the tantra) and standalone commentary passages.
- **`comm-card`** — purple accent (`#7c3aed`), header background `#f5f3ff`. Always nested inside a `verse-card`. A CSS `::before` pseudo-element draws a connector line from the parent verse.

When a `verse-card` is itself commentary (not a mula verse), the border color is overridden to purple via inline styles.

### Tags

Each card header carries a tag:

| Tag | Color | Meaning |
|-----|-------|---------|
| `MULA` | Blue `#dbeafe` | Root verse of the Mrigendratantra |
| `COMM.` | Purple `#f3e8ff` | Narayanakantha's vrtti (commentary) |

---

## Color Coding — Grammatical Categories

Every Sanskrit word (token) is color-coded by grammatical category. The system uses 7 categories, each with a text color, underline color, and hover background:

| Category | CSS class | Text color | Underline | Hover bg | Examples |
|----------|-----------|------------|-----------|----------|----------|
| **Finite Verb** | `.verb` | `#b91c1c` (red) | `#fca5a5` | `#fee2e2` | prabhavati, smarami, mocayati |
| **Noun** | `.noun` | `#1d4ed8` (blue) | `#93c5fd` | `#dbeafe` | dhama, bandhaat, pratibha |
| **Abstract Noun** (-tva/-ta) | `.abstr` | `#be185d` (pink) | `#f9a8d4` | `#fce7f3` | sarvajnatvam, akhilakartrita |
| **Compound** | `.cmpd` | `#6d28d9` (violet) | `#c4b5fd` | `#ede9fe` | nirupadhi, agocarah, yogapadah |
| **Participle / Adjective** | `.ptcp` | `#047857` (emerald) | `#6ee7b7` | `#d1fae5` | para, shantam, pravitatat |
| **Pronoun / Correlative** | `.pron` | `#0e7490` (cyan) | `#67e8f9` | `#cffafe` | yatah, tad, asau, asmaat |
| **Indeclinable** | `.indecl` | `#b45309` (amber) | `#fcd34d` | `#fef3c7` | atha, ca, eva, api, tatha |

### Why these categories?

The division reflects what a Sanskrit student needs to parse a sentence:

- **Verbs** (red) are the structural backbone — find them first
- **Nouns vs. abstract nouns** — abstracts (-tva, -ta) are separated because they signal philosophical concepts and are often the hardest to parse
- **Compounds** (violet) — the single biggest challenge in Sanskrit prose; highlighting them warns the reader that multiple words are fused
- **Participles** (green) — occupy an ambiguous zone between verbs and adjectives; separating them helps identify relative clauses and attributive constructions
- **Pronouns** (cyan) — critical for tracking anaphora (yatah...tad, yad...asau)
- **Indeclinables** (amber) — particles, conjunctions, emphatics; they control sentence logic (eva = "only", api = "even", ca = "and")

### Grammar Superscripts

Each token may carry a superscript label showing its grammatical form:

```html
<span class="t noun" data-tip="...">bandhaat<span class="g">abl</span></span>
```

Common labels: `nom` `acc` `gen` `loc` `abl` `inst` `dat` `voc` `1sg` `3sg` `3sg.pass` `3sg.caus` `gen.pl`

---

## Tooltip System

### Problem solved

CSS tooltips using `position: absolute` inside `overflow: hidden` containers get clipped. This is especially bad for Sanskrit readers where the tooltip might need to appear above the first line of a block.

### Solution

A single global `<div class="tip" id="globalTip">` lives at the end of `<body>`. It uses `position: fixed` and is positioned via JavaScript on `mouseover`:

```javascript
document.addEventListener('mouseover', function(e) {
  const token = e.target.closest('.t');
  if (!token) return;
  const data = token.getAttribute('data-tip');
  // ... parse and display
  const rect = token.getBoundingClientRect();
  // position above the word, flip below if no room
});
```

### Tooltip data format

Each token stores its tooltip data in a `data-tip` attribute using `|` as delimiter:

```
data-tip="word|grammar|root|meaning"
```

Example:
```
data-tip="prabhavati|verb · 3sg pres. indic.|√bhu + pra- (1P)|arises, originates, comes forth"
```

The tooltip renders four lines:
1. **Word** (gold `#fbbf24`) — the lemma form
2. **Grammar** (grey `#a8a29e`) — part of speech, case, number, gender
3. **Root** (green `#86efac`) — dhatu with gana and prefixes
4. **Meaning** (light `#e7e5e4`) — English translation

### Auto-positioning

- Centers horizontally over the token
- Places above by default (`rect.top - height - 10px`)
- Flips below if there's no room above (`rect.bottom + 10px`)
- Clamps horizontally to stay within viewport (8px margin)

---

## Three Layers of Translation

Each sentence card presents three layers, from most detailed to most readable:

### 1. Sanskrit Block (`.sk-block`)

The original Sanskrit in IAST transliteration with color-coded tokens, grammatical superscripts, and hover tooltips. Verse breaks (`.vbr`) and dandas (`.danda`) preserve the metrical structure.

### 2. Word-by-Word (`.wbw`)

English translation **in English word order** (not Sanskrit word order) with Sanskrit terms in parentheses after each English word:

```
Initiation (diksha) alone (eva) liberates (mocayati) [the soul] from this (asmat)
extended (pravitatat) bondage (bandhat)...
```

Key conventions:
- **English word order** — the student reads natural English but can trace each word back to the Sanskrit
- **Implied words in [square brackets]** — words not present in the Sanskrit but required by English: `[the soul]`, `[arises]`, `[is]`, `[Because,]`
- Sanskrit terms in `<span class="sk-inline">` (bold) inside regular parentheses

### 3. Smooth Translation (`.transl`)

A literary English translation with no Sanskrit interpolations. Larger font (1.25rem), italic, with a left border accent. This is what the reader would cite in an essay.

---

## Tab System

### Behavior

Tabs within a card are **mutually exclusive** — clicking one hides the others (radio-button style, not accordion). This prevents information overload.

### Implementation

Each card has a `.tabs-bar` containing `<button class="tab-btn">` elements, followed by `.tab-panel` divs in the same order:

```html
<div class="tabs-bar">
  <button class="tab-btn active" onclick="switchTab(this, 'v1')">Semantic Tree</button>
  <button class="tab-btn" onclick="switchTab(this, 'v1')">Sandhi</button>
  <button class="tab-btn" onclick="switchTab(this, 'v1')">Vocabulary</button>
  <button class="tab-btn" onclick="switchTab(this, 'v1')">Notes</button>
</div>
<div class="tab-panel active" data-parent="v1">...</div>
<div class="tab-panel" data-parent="v1">...</div>
<div class="tab-panel" data-parent="v1">...</div>
<div class="tab-panel" data-parent="v1">...</div>
```

The `switchTab()` function:
1. Finds the clicked button's index in its `.tabs-bar`
2. Walks through sibling `.tab-panel` elements
3. Activates only the panel matching that index

This approach avoids IDs for individual panels and works with nested cards (verse + commentary each have independent tab sets).

### Available tabs

| Tab | Content | Visual style |
|-----|---------|-------------|
| **Semantic Tree** | Dependency tree with color-coded nodes | Indented list with CSS connector lines |
| **Sandhi** | Sandhi resolution showing joined → split forms | Amber background box |
| **Compound Analysis** | Breakdown of long tatpurusha/bahuvrihi chains | Amber background box (same style as sandhi) |
| **Vocabulary** | Grid of vocabulary entries (word, root, meaning) | 2-column responsive grid |
| **Notes** | Doctrinal, grammatical, or structural commentary | Green background box |

Not every card has all tabs — commentary cards may only have 2-3.

---

## Semantic Tree

The dependency tree uses pure CSS for the connector lines:

```css
.tree ul li::before {   /* vertical line */
  content: ''; position: absolute;
  left: -1.2rem; top: 0;
  height: 100%; width: 1px; background: #d6d3d1;
}
.tree ul li::after {    /* horizontal branch */
  content: ''; position: absolute;
  left: -1.2rem; top: 0.82em;
  width: 0.9rem; height: 1px; background: #d6d3d1;
}
.tree ul li:last-child::before {
  height: 0.82em;       /* cut vertical line at last child */
}
```

Each node shows:
- **Word** (`.nw`) — color-coded pill matching the grammatical category
- **Role** (`.nr`) — syntactic role: SUBJ, PRED, OBJ, ATTR, GEN, LOC, ABL, DET, CONJ, RESTRICT, etc.
- **Gloss** (`.ng`) — short English translation in italics

Subscript numbers (PRED1, PRED2) distinguish multiple predicates in a single verse.

---

## Sandhi Resolution

Each sandhi entry shows:

```
dīkṣaiva  →  dīkṣā eva  · vowel sandhi (ā + e → ai)
```

- **Joined form** (bold) — as it appears in the manuscript
- **Arrow** (amber) — visual separator
- **Split form** — the underlying words
- **Rule** (small grey) — the sandhi rule applied

For compounds, the same visual style is used for compound breakdown, with each member explained below.

---

## Adding New Sentences

To add a new sentence, follow this pattern:

### For a mula (root verse):

```html
<div class="verse-card" id="v4">
  <div class="verse-header">
    <div class="verse-num">4</div>
    <div class="verse-label">Verse description <span class="type-tag tag-verse">mula</span></div>
  </div>
  <div class="verse-body">
    <div class="sk-block">
      <!-- tokens here -->
    </div>
    <div class="wbw">
      <div class="wbw-title">Word-by-word</div>
      <!-- English word order with (sanskrit) -->
    </div>
    <div class="transl">
      <!-- Smooth translation -->
    </div>
    <div class="tabs-bar">
      <button class="tab-btn active" onclick="switchTab(this, 'v4')">Semantic Tree</button>
      <!-- more tab buttons -->
    </div>
    <div class="tab-panel active" data-parent="v4"><!-- tree --></div>
    <!-- more tab panels -->
  </div>
</div>
```

### For nested commentary:

Place a `.comm-card` inside the parent `.verse-card`, after the `.verse-body`:

```html
<div class="verse-card" id="v4">
  <div class="verse-body">
    <!-- verse content -->
  </div>

  <div class="comm-card" id="v4c">
    <div class="comm-header">
      <div class="comm-marker">VRTTI</div>
      <div class="comm-label">Commentary on the verse above</div>
    </div>
    <div class="comm-body">
      <!-- same structure as verse-body -->
    </div>
  </div>
</div>
```

### Token format:

```html
<span class="t verb" data-tip="word|grammar line|root line|meaning">
  word<span class="g">3sg</span>
</span>
```

Classes: `verb`, `noun`, `abstr`, `cmpd`, `ptcp`, `pron`, `indecl`

---

## File Structure

```
reading-2026/
├── index.html    — the complete reader (single file, ~850 lines)
└── README.md     — this documentation
```

No build tools. No package.json. No node_modules. Just open `index.html` in a browser.

---

## Hosting with GitHub Pages

1. Go to **Settings > Pages** in this repository
2. Set Source to **Deploy from a branch**
3. Select **main** branch, **/ (root)** folder
4. The site will be live at `https://mariaiontseva.github.io/reading-2026/`

---

## License

Educational use. The Mrigendratantra text is in the public domain. Translations and commentary analysis are original work.

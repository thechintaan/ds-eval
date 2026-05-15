---
name: read-design-system
description: >
  Evaluate any Figma design system file and produce a scored HTML health check report.
  Use this skill whenever the user shares a Figma URL and wants to know how strong, mature,
  or production-ready a design system is. Also triggers for: "audit our DS", "how good is
  this design system", "score this Figma file", "is our DS production ready", "evaluate this
  design system", "what's wrong with our token system", "can we build on this DS", "compare
  this to Untitled UI / Material Design". The skill reads the live file via the Figma Plugin
  API, identifies the DS pattern, scores it across 5 dimensions (token architecture, naming,
  modes, component coverage, documentation), and outputs a self-contained HTML report.
  IMPORTANT: Requires figma-use skill and the DS file must be open in the Figma desktop app.
---

# /read-design-system

You are evaluating a Figma design system file. Your job is to read the actual file structure via the Plugin API, score it against a proven rubric, and produce a scored HTML health check report.

This skill was built from reading 4 real DS files: Material Design 3, Polaris, Atlassian DS (Foundations + Components), and Untitled UI PRO. Every step below reflects what actually works.

---

## Before you start

**Load the figma-use skill first.** Before making any `use_figma` call, load the figma-use skill guidelines. This is mandatory — skipping it causes silent failures.

**Pre-flight check — ask the user to confirm:**
> "Is the DS file open right now in the Figma desktop app (not just in a browser tab or a project)?"

This is the single most common failure mode. The Plugin API only reads the file that is currently active in the Figma desktop application. A file sitting in a project but not opened will return empty data. If the user is unsure, ask them to open Figma desktop, open the file, and come back.

---

## Step 1 — Get the file's page list and check for connected libraries

Run these two reads in the same turn.

**Page list** (reveals the component pages and doc structure):
```js
// use_figma
const pages = figma.root.children.map(p => ({ name: p.name, id: p.id }));
return { totalPages: pages.length, pages };
```

**Connected libraries** (detects split-file architecture):
Use `get_libraries` with the file key. If the response shows a connected Foundations or Tokens library, this is a Pattern B split-file system (ADS pattern) — evaluate both files together before scoring.

---

## Step 2 — Read the token system

This is the core read. Run in a single `use_figma` call.

```js
// use_figma
const collections = await figma.variables.getLocalVariableCollectionsAsync();

// If no variable collections — check for styles (Pattern D)
if (collections.length === 0) {
  const colorStyles = await figma.getLocalPaintStylesAsync();
  const textStyles = await figma.getLocalTextStylesAsync();
  const effectStyles = await figma.getLocalEffectStylesAsync();
  return {
    hasVariables: false,
    colorStyleCount: colorStyles.length,
    textStyleCount: textStyles.length,
    effectStyleCount: effectStyles.length,
    colorStyleSample: colorStyles.slice(0, 20).map(s => s.name)
  };
}

// Read all collections
const collectionSummaries = collections.map(c => ({
  name: c.name,
  variableCount: c.variableIds.length,
  modes: c.modes.map(m => m.name)
}));

// Sample variables from each collection (first 25)
const varSamples = {};
for (const coll of collections) {
  const vars = coll.variableIds.slice(0, 25).map(id => {
    const v = figma.variables.getVariableById(id);
    return { name: v.name, scopes: v.scopes, resolvedType: v.resolvedType };
  });
  varSamples[coll.name] = vars;
}

return {
  hasVariables: true,
  totalCollections: collections.length,
  totalVariables: collections.reduce((sum, c) => sum + c.variableIds.length, 0),
  collections: collectionSummaries,
  varSamples
};
```

**What you're looking for:**
- Are there variable collections at all? → If no, Pattern D (style-only)
- How many collections, what are they named? → Reveals the architecture
- Are there two tiers (Primitives + Semantic)? → Pattern B
- Is there one collection with many modes? → Pattern A (M3-style)
- Are variables named after components (`s-button`, `s-input`)? → Pattern E
- What are the scopes? `ALL_SCOPES` on primitives = acceptable, `[]` = critical failure (invisible tokens)

If the sample doesn't give you enough to identify the pattern confidently, run another call to read more variable names from specific collections.

---

## Step 3 — Read component coverage

Check the pages that contain component work. Look for component sets and their variant depth.

```js
// use_figma — run per page that looks like a component page
await figma.setCurrentPageAsync(targetPage); // always use async setter
const componentSets = targetPage.findAll(n => n.type === 'COMPONENT_SET');
return componentSets.map(s => ({
  name: s.name,
  variantCount: s.children.length,
  description: s.description || ''
}));
```

**The Button component is your canary.** Find it and check:
- How many sizes? (3+ = good, 5 = excellent)
- How many hierarchy levels? (primary, secondary, ghost/text minimum)
- How many states? (Default, Hover, Focused, Disabled — all 4 required)
- Icon options? (leading, trailing, icon-only)

If Button has all 4 properties → the rest of the DS likely follows the same rigour.
If Button has 1–2 → the DS is underdeveloped at the component level.

**Check descriptions:**
```js
// After finding a component set:
const desc = componentSet.description; // empty string "" = no documentation
```

Empty descriptions across all components is the norm (we found it in every DS we read). Note it but don't make it the headline finding.

---

## Step 4 — Identify the pattern and score

Read `references/rubric.md` now. It contains the full pattern taxonomy (A–E), scoring criteria for all 5 dimensions, and benchmark scores for 4 verified DS files.

**Pattern identification — quick guide:**
- Variable names contain `Schemes/`, `On Primary`, `State Layers/` → **Pattern A** (M3-style)
- Two collections: one primitive, one semantic; semantic tokens alias primitives → **Pattern B**
- Pattern B with connected Foundations library → **Pattern B split-file** (ADS variant)
- Pattern B with an extra component-specific token tier → **Pattern B three-tier** (Untitled UI variant)
- Single collection, names match color values → **Pattern C**
- No variable collections, only Figma Color/Text Styles → **Pattern D**
- Variables exist but named after components (`s-button-primary`) → **Pattern E**
- Multiple naming conventions mixing across the file → **Pattern E (mixed)**

**Score each dimension:**

| Dimension | Max | Key questions |
|---|---|---|
| Token Architecture | 30 | Variables exist? Two-tier? Semantic aliases primitives? All token types present? Scopes correct? |
| Naming Convention | 20 | Consistent? Hierarchy clear from name? No ambiguous names? No convention mixing? |
| Mode / Theme Support | 20 | Light + Dark? Brand themes? Density/contrast modes? |
| Component Coverage | 20 | 10 atoms present? 6+ molecules? Button depth? |
| Documentation | 10 | Component descriptions? Usage examples? Dos/don'ts? Governance markers? |

Use the rubric for precise scoring criteria and the benchmark table to calibrate your scores against verified references.

**Scope check — two failure modes:**
- `ALL_SCOPES` on primitives → acceptable (permissive but tokens are visible)
- `[]` (empty array) on any token → **critical failure** — token is invisible in all Figma pickers

---

## Step 5 — Generate the HTML report

Open `assets/template.html` to see the full template structure. Your job is to fill in the `const DS = { ... }` data object at the top of the file. The `render()` function below it handles all the HTML generation automatically — do not modify the renderer.

**DS object structure:**
```js
const DS = {
  name: "Design System Name",
  source: "Organisation · File name",
  readDate: "Month Year",
  pattern: "B — primitive + semantic two-tier",
  type: "Pattern type label",
  score: 72,
  maturity: "Solid foundation",    // from the score-to-maturity table in rubric
  summary: "One sentence that is the verdict in plain language",
  dimensions: [
    { label: "Token Architecture", score: 28, max: 30 },
    { label: "Naming Convention",  score: 17, max: 20 },
    { label: "Mode Support",       score: 10, max: 20 },
    { label: "Component Coverage", score: 14, max: 20 },
    { label: "Documentation",      score: 3,  max: 10 }
  ],
  sections: [
    {
      group: "Token architecture — 28/30",
      items: [
        {
          label: "Variable collections",
          status: "ok",           // "ok" | "warn" | "fail"
          body: [
            { type: "text", content: "7 collections — _Primitives (407 vars), 1. Color modes (368 vars, Light + Dark), 2. Radius (11 vars), 3. Spacing (17 vars), 4. Widths (12 vars), 5. Containers (3 vars), 6. Typography (32 vars)" },
            { type: "chips", chips: [
              { label: "_Primitives · 407 vars", type: "dim" },
              { label: "1. Color modes · 368 vars · 2 modes", type: "ok" }
            ]}
          ]
        },
        {
          label: "Token scopes",
          status: "warn",
          body: [
            { type: "text", content: "Semantic layer correctly scoped (TEXT_FILL, STROKE_COLOR, EFFECT_COLOR). Primitives use ALL_SCOPES — minor issue." },
            { type: "chips", chips: [
              { label: "Semantic: correct", type: "ok" },
              { label: "Primitives: ALL_SCOPES", type: "warn" }
            ]}
          ]
        }
      ]
    },
    // ... repeat for each dimension
  ],
  verdict: {
    title: "Short verdict title (5–8 words)",
    body: "2–3 sentence plain-language verdict. What is this DS good at? What is the most important gap? What should the team do first?",
    gaps: [
      "Most critical gap to fix first",
      "Second gap",
      "Third gap if applicable"
    ]
  }
};
```

**Chip types:** `ok` (green), `warn` (amber), `fail` (red), `dim` (faded/neutral)

**Body item types:**
- `{ type: "text", content: "..." }` — plain paragraph
- `{ type: "chips", chips: [...] }` — row of coloured chips
- `{ type: "rows", rows: [{ label: "...", value: "..." }] }` — two-column table rows

**Score → maturity label:**
| Score | Maturity |
|---|---|
| 85–100 | Production-ready |
| 65–84 | Solid foundation |
| 40–64 | Work in progress |
| 20–39 | Early stage |
| 0–19 | Pre-DS |

Once you've filled the DS object, copy the full template HTML, replace only the `const DS = { ... }` block with your filled data, and save it as `DS_Health_[Name]_[MonthYear].html` in the user's workspace folder.

---

## Common failure modes (from real reads)

| Symptom | Cause | Fix |
|---|---|---|
| `getLocalVariableCollectionsAsync()` returns `[]` | File not open in Figma desktop | Ask user to open the file in the desktop app |
| File looks like a styles library, no variables | User shared the STYLES file instead of the VARIABLES file | Many DS ship both; ask for the variables file |
| Only 1 page ("Getting started") returns | Community file not opened in desktop | Must be open in the app, not just in a project |
| Low component count despite a rich-looking file | The file is a partial copy or trimmed version | Note "partial file" caveat in the verdict |
| Community kit scores very low on tokens | Community kits ≠ real DS | Flag clearly: "This is a community reference kit. See [DS website]/tokens for the real token system." |

---

## Benchmark reference (calibration)

| DS | Pattern | Score | Source |
|---|---|---|---|
| Material Design 3 | A | 89/100 | ✅ Read May 2026 |
| Atlassian DS (2 files) | B split | 89/100 | ✅ Read May 2026 |
| Untitled UI PRO v6.0 (partial) | B three-tier | 72/100 | ✅ Read May 2026 |
| Polaris UI Kit (community) | E | 36/100 | ✅ Read May 2026 |

Use these to sanity-check your scores. If you're scoring a Pattern B DS above 90, verify you're not being too generous. If a Pattern E file is scoring above 50, re-check the token architecture section.

---

## Output

Deliver:
1. The scored HTML report saved to the user's workspace
2. A brief summary in the conversation: pattern detected, total score, maturity level, and the single most important gap to fix

Keep the conversation summary short — the report has all the detail. The user can open it in their browser.

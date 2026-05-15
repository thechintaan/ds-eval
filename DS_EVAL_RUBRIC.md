# Design System Evaluation Rubric
**Version:** 1.2  
**Built from:** Material Design 3 (✅ read), Polaris UI Kit (✅ read), Atlassian DS — Foundations + Components (✅ read), Untitled UI PRO VARIABLES v6.0 (✅ read), Figma Simple DS, Apple HIG  
**Purpose:** Standard evaluation framework for `/read-design-system` skill. Scores any DS regardless of philosophy.

---

## How to Use This Rubric

When reading a new design system file:
1. Identify which **Pattern** the file follows (Section 1)
2. Score it across the **five dimensions** (Sections 2–6)
3. Output a health summary using the **verdict template** (Section 7)

Scoring: ✅ Present and correct · ⚠️ Partial or inconsistent · ❌ Missing

---

## Section 1 — Identify the DS Pattern

Before scoring, classify the file into one of these known patterns. This sets the bar for what "complete" looks like.

### Pattern A — Role-based / Semantic-first
*Examples: Material Design 3, Apple HIG*

- Tokens describe **roles** (what the color does), not values (what color it is)
- No primitive collection in the file — raw palette lives externally (e.g. M3 Theme Builder plugin)
- Single semantic collection with role names: `Schemes/Primary`, `Schemes/On Primary`, `Schemes/Surface`, `Schemes/Error`, `Schemes/Outline`
- State layers are part of the same collection as opacity variants: `State Layers/Primary/Opacity-08`
- Modes are first-class and extensive — M3 ships with **32 modes**: Light, Dark, High/Medium Contrast variants, and 12 dynamic color themes (Pink, Rose, Red, Orange, Yellow, Green, Teal, Cyan, Blue, Indigo, Purple, Chartreuse)
- Shape, Typography, and Font are separate collections with single modes
- Component pages are one-per-component (25 components across dedicated pages)

**Verified M3 numbers (read May 2026):**
- Collections: 4 — `M3` (197 vars, 32 modes), `Typescale` (90 vars), `Shape` (10 vars), `Font theme` (7 vars)
- Pages: 33 total — 8 foundation/utility + 25 component pages
- Shape tokens: Corner/None → Corner/Full (10 steps)
- Type tokens: Display, Headline, Title, Body, Label scales × Font, Weight, Size, Line Height, Tracking

**Signal to detect it:** Variable names contain `Schemes/`, `On Primary`, `Surface Container`, `State Layers/`. Single collection with many modes. No separate primitives collection.

---

### Pattern B — Primitive + Semantic two-tier
*Examples: Untitled UI, Figma Simple DS, Radix, Shadcn, Atlassian DS*

- Tier 1: **Primitive tokens** — named by value (`gray/500`, `blue/600`, `spacing/4`)
- Tier 2: **Semantic tokens** — named by purpose (`color/text/primary`, `color/border/brand`)
- Semantic tokens reference primitive tokens (aliases), not raw values
- Light/dark modes live at the semantic tier (primitives don't change)
- Well-scoped: text tokens go to TEXT_FILL, border tokens to STROKE_COLOR

**Pattern B variant — Three-tier (Untitled UI pattern):**
A mature evolution of Pattern B where the semantic layer is further split into a semantic sub-layer and a component-specific sub-layer:
- Tier 2a: Semantic roles — `Colors/Text/text-primary`, `Colors/Border/border-brand`
- Tier 2b: Component tokens — `Component colors/Components/Avatars/avatar-bg`, `Component colors/Utility/Brand/utility-brand-50`
This gives component designers pre-resolved token choices without going back to primitives, while keeping semantic roles clean.

**Untitled UI PRO VARIABLES v6.0 verified numbers (read May 2026):**
- Collections: 7 — `_Primitives` (407 vars, single mode), `1. Color modes` (368 vars, Light + Dark), `2. Radius` (11 vars), `3. Spacing` (17 vars), `4. Widths` (12 vars), `5. Containers` (3 vars), `6. Typography` (32 vars)
- Total: 850 variables
- Color modes semantic categories: Text (21), Border (9), Foreground (22), Background (31), Alpha (20), Effects (23), Utility (150), Components (92)
- Aliasing: ✅ Confirmed — every semantic token aliases a primitive, no raw values in semantic layer
- Scopes: ✅ Exemplary on semantic layer — TEXT_FILL, STROKE_COLOR, EFFECT_COLOR, CORNER_RADIUS, GAP, FONT_* all correctly scoped. ⚠️ Primitives use ALL_SCOPES.
- Naming convention: Mixed — slash hierarchy for colours (`Colors/Text/text-primary (900)`), flat kebab for dimensions (`spacing-md`, `radius-xl`). Consistent within each category but two conventions across the file.
- Parenthetical hints on semantic tokens (`text-primary (900)`) — designers see the primitive step without leaving the variable picker. Recommended pattern.
- Button component: 380 variants — 5 sizes × 7 hierarchies × 4 states × 3 icon configs. Best button variant depth across all reads.

**Pattern B variant — Split-file architecture (ADS pattern):**  
Foundations and Components live in two separate published Figma files. The Foundations file owns all variable collections (primitives + semantic). The Components file consumes them as a subscribed library. This is a maturity signal, not a gap — it enforces boundaries between token ownership and component consumption.

- Signal: `get_libraries` shows a published Foundations library connected to the Components file
- Evaluate as a single system — read both files together before scoring
- Token Architecture score should reflect the combined variable system, not each file independently

**Atlassian DS verified numbers (read May 2026):**
- Two files: ADS Foundations (variables + tokens) + ADS Components (consumes Foundations library)
- Semantic naming with interaction states baked into variable names: `color/text/interactive/hovered`, `color/border/focused/subtle`
- Internal token governance: `⛔️ DST team only` markers on restricted/internal tokens — not for consumer use
- Beta/Deprecated markers on components in active transition
- Strong documentation: usage examples, dos/don'ts, component descriptions filled across all major components

**Signal to detect it:** Two separate variable collections — one for raw values, one for semantic roles. Slash-separated naming with category prefixes.

---

### Pattern C — Flat / Primitive-only
*Examples: early-stage DS, legacy files, quick-build projects*

- Single tier of tokens — raw values with descriptive names (`brand-blue`, `text-dark`, `spacing-16`)
- No semantic layer — components reference raw values directly
- No or minimal mode support
- Common in teams that started in Figma without a token strategy

**Signal to detect it:** Single variable collection or no variables. Color names reference the color itself, not its role.

---

### Pattern D — No token system / Style-only
*Examples: pre-variables Figma files, very old DS files*

- Uses Figma **Color Styles** and **Text Styles** instead of variables
- No variable collections present
- Styles may be well-organised but lack mode switching capability
- Often found in files built before Figma Variables (pre-2023)

**Signal to detect it:** No variable collections. Styles present. Components use fills that reference styles.

---

### Pattern E — Component-level token system (shallow semantic layer)
*Examples: Polaris UI Kit (community), organically evolved real-world DS*

Distinct from Pattern C (flat/primitive-only) — Pattern E files DO have variables, but they are named after specific components rather than semantic roles. No primitive collection exists. The "semantic" layer is component-specific, making reuse and mode support nearly impossible.

- Variables are named after the components they serve: `s-button`, `s-button-primary`, `s-color-bg-surface-selected`
- Prefix conventions tied to component identity (`s-` = surface, `g-` = global) rather than role hierarchy
- Single mode throughout — no light/dark switching
- Collections often fragmented by component or concern (8 collections, 65 vars in Polaris community kit)
- Token scopes frequently set to `[]` (empty array) — **worse than ALL_SCOPES** because tokens are invisible in every Figma property picker; designers cannot discover or apply them
- Collection names may include non-standard characters or emoji (tooling friction for CI/export pipelines)
- Some components use variable bindings; others use hardcoded fills — mixed within the same file

**Verified Polaris numbers (community UI kit, read May 2026):**
- Collections: 8 total, 65 variables total — no primitives collection, no semantic collection
- Modes: Single mode only across all collections
- Naming: Component-level (`s-button-primary-background`, `s-badge-icon-color-emphasis-caution`)
- Scopes: Many variables have `scopes: []` — invisible to designers in all Figma property pickers
- Components: Good coverage (atoms + molecules) but poorly connected to token system
- Score: **36/100** — component library value only, not a token foundation
- ⚠️ Critical note: The community kit does not represent the real Polaris token system. For actual Polaris tokens, see polaris.shopify.com/tokens

**Signal to detect it:** Variables exist but follow component-level naming. No primitive collection. Single mode. Many variables with empty scopes array.

---

## Section 2 — Token Architecture (30 points)

| Check | Pattern A | Pattern B | Pattern C | Pattern D | Pattern E |
|---|---|---|---|---|---|
| Variable collections exist | ✅ req | ✅ req | ⚠️ partial | ❌ none | ⚠️ partial |
| Two-tier structure (primitive + semantic) | ✅ req | ✅ req | ❌ | ❌ | ⚠️ |
| Semantic tokens alias primitives (not raw values) | ✅ req | ✅ req | ❌ | ❌ | ⚠️ |
| Color tokens present | ✅ | ✅ | ⚠️ | ❌ | ⚠️ |
| Spacing tokens present | ✅ | ✅ | ⚠️ | ❌ | ⚠️ |
| Radius/shape tokens present | ✅ | ✅ | ⚠️ | ❌ | ⚠️ |
| Typography tokens present | ✅ | ⚠️ | ❌ | ❌ | ⚠️ |
| Effect/shadow tokens present | ⚠️ | ✅ | ❌ | ⚠️ | ⚠️ |
| Token scopes correct (not ALL_SCOPES) | ✅ req | ✅ req | ❌ | ❌ | ⚠️ |

**What to look for when reading:**
- `getLocalVariableCollectionsAsync()` → list collection names and count
- Check if any collection name suggests primitives vs semantic (e.g. "1. Primitives" vs "2. Color modes")
- Check scopes on a sample of tokens — two red flags at different ends of the spectrum:
  - `ALL_SCOPES` → token appears in every picker; too permissive, creates noise at scale
  - `[]` (empty array) → **worse than ALL_SCOPES** — token is completely invisible in Figma UI; designers cannot find or apply it. Often a build/export error. Flag immediately.
- For split-file DS (Pattern B variant): run `get_libraries` to confirm Foundations file is connected as a published library to the Components file

**Scoring:**
- All 9 checks pass → 30 pts (excellent)
- 6–8 pass → 20 pts (good)
- 3–5 pass → 10 pts (early stage)
- 0–2 pass → 0 pts (no token system)

---

## Section 3 — Naming Convention (20 points)

### Known naming patterns (all valid, score on *consistency*, not choice)

| Convention | Example | Common in |
|---|---|---|
| Slash hierarchy | `color/text/primary` | Figma-native, Untitled UI, Simple DS |
| Role-based camelCase | `sys.color.primary` | Material Design 3 |
| Kebab with prefix | `color-text-primary` | CSS token pipelines |
| T-shirt sizing | `spacing-sm`, `spacing-2xl` | Most modern DS |
| Numeric scale | `gray/500`, `spacing/4` | Tailwind-influenced |

### What to check

| Check | Signal | Score |
|---|---|---|
| Convention is consistent across the file | All tokens follow one naming pattern | +8 |
| Hierarchy is clear from name alone | You can tell the category from reading the name | +6 |
| No ambiguous names | No tokens named `color1`, `blue2`, `text` without context | +3 |
| No mixing of conventions | Not `color/primary` AND `primary-color` in same collection | +3 |

**Red flags:**
- Token names that are purely decorative (`Jewel`, `Scandal`, `Mystic`) with no role indication
- Multiple naming styles in the same collection
- Names that describe the value, not the role (`bright-blue`, `dark-text`) at the semantic tier
- **Component-level naming at the root** — `s-button`, `s-button-primary` are component tokens, not semantic tokens; a system built entirely on these cannot scale or support theming
- **Naming collision** — token name matches the component it serves exactly (e.g. a token named `button` in the same system as a component named `button`); creates ambiguity in design tools and code
- **Non-standard prefixes without a published legend** — `s-`, `g-`, `ds-` are valid if documented; undocumented prefixes are a maintenance hazard
- **Emoji or special characters in collection names** — causes CI/export pipeline failures in most token tooling (Style Dictionary, Tokens Studio)

---

## Section 4 — Mode / Theme Support (20 points)

| Level | Description | Score |
|---|---|---|
| None | Single mode only, no switching | 0 |
| Light + Dark | Two modes on semantic collection | 10 |
| Light + Dark + Brand | Brand themes on top of light/dark | 15 |
| Full multi-mode | Density, platform, contrast modes | 20 |

### What to check
- Does the semantic color collection have more than one mode?
- Are primitive tokens single-mode? (They should be — values don't change)
- Do components respond to mode switching without overrides?

**Two valid approaches for interaction states — both are correct:**
1. **State layer modes** (M3 pattern): Interaction states live as separate opacity layers in the token system. Components apply state layers on top of base tokens.
2. **State-suffix naming** (ADS pattern): Interaction states are baked into variable names at the semantic tier (`color/text/interactive/hovered`, `color/border/focused`). Single mode, but state coverage is encoded in naming hierarchy.

Do not penalise a DS for choosing approach 2 — it is a valid architecture. The question is whether states are covered at all, not how they are structured.

**Material 3 benchmark:** Ships with light, dark + 12 dynamic color theme modes  
**Atlassian DS benchmark:** Light + dark modes, interaction states encoded in variable names  
**Untitled UI benchmark:** Light and dark modes on semantic collection  
**Minimum viable:** Light and dark modes on semantic collection

---

## Section 5 — Component Coverage (20 points)

### Required atoms (must-have for any DS)
- [ ] Button (with size, hierarchy/variant, state properties)
- [ ] Input / Text field
- [ ] Checkbox
- [ ] Radio
- [ ] Toggle / Switch
- [ ] Badge / Tag
- [ ] Avatar
- [ ] Icon (system)
- [ ] Tooltip
- [ ] Divider

**Score: 2 pts per atom present and component-ified (not just a frame)**

### Required molecules (should-have)
- [ ] Card
- [ ] Modal / Dialog
- [ ] Dropdown / Select
- [ ] Navigation (top bar, sidebar, or bottom nav)
- [ ] Form group (label + field + helper text)
- [ ] Table row / List item
- [ ] Alert / Toast / Snackbar
- [ ] Tabs

**Score: presence of 6+ → bonus +5 pts**

### Variant completeness (per component)
Check the primary Button component — it's the canary for variant quality across the system.

| Property | Values to expect | Signal |
|---|---|---|
| Size | 3+ sizes (sm, md, lg minimum) | Basic |
| Hierarchy/Type | 3+ (primary, secondary, ghost/text) | Basic |
| State | Default, Hover, Focused, Disabled | Required |
| Icon | With icon, icon only, no icon | Good |

If Button has all four properties → likely rest of DS follows same rigour  
If Button only has 1–2 properties → DS is underdeveloped at the component level

---

## Section 6 — Documentation Quality (10 points)

| Signal | Score |
|---|---|
| Component descriptions filled in Figma | +3 |
| Usage example frames on component pages | +3 |
| Dos/don'ts annotation frames | +2 |
| Variant property descriptions filled in | +2 |

**What to look for when reading:**
- Frame names like "Usage examples", "Dos & don'ts", "Notes and documentation"
- `component.description` not empty
- Annotation frames alongside component frames on each page

**Bonus signals of mature documentation (add +1–2 pts if present):**
- `⛔️ DST team only` or equivalent internal governance markers on restricted tokens — signals the team controls the token surface deliberately (ADS pattern)
- `Beta` / `Deprecated` labels on components in active transition — signals lifecycle management
- Published changelog or version notes in the file (often as a text frame on a meta/About page)

---

## Section 7 — Verdict Template

Use this format when outputting a DS health check:

```
DS HEALTH CHECK — [File name]
Pattern detected: [A / B / C / D / E] — [one-line description]

TOKEN ARCHITECTURE    [score]/30
  Variable collections: [list collection names]
  Two-tier structure: [✅ / ⚠️ / ❌]
  Spacing tokens: [✅ / ⚠️ / ❌]
  Radius tokens: [✅ / ⚠️ / ❌]
  Scopes correct: [✅ / ⚠️ / ❌]

NAMING CONVENTION     [score]/20
  Pattern used: [e.g. slash hierarchy, camelCase]
  Consistency: [✅ / ⚠️ / ❌]
  Notable issues: [or "None"]

MODE SUPPORT          [score]/20
  Modes found: [list, or "None"]
  Verdict: [e.g. "Light only — no dark mode support"]

COMPONENT COVERAGE    [score]/20
  Atoms present: [X/10]
  Molecules present: [X/8]
  Button variant depth: [e.g. "4 properties — comprehensive"]
  Missing: [list gaps]

DOCUMENTATION         [score]/10
  Usage examples: [✅ / ⚠️ / ❌]
  Descriptions: [✅ / ⚠️ / ❌]

TOTAL SCORE: [X]/100

VERDICT:
[1–2 sentence plain-language summary of DS maturity and the most important gap to fix first]
```

---

## Section 8 — Benchmark Scores (reference)

| Design System | Pattern | Token | Naming | Modes | Components | Docs | Total | Source |
|---|---|---|---|---|---|---|---|---|
| Material Design 3 | A | 25 | 17 | 20 | 20 | 7 | **89/100** | ✅ Read May 2026 |
| Atlassian DS (2 files) | B (split) | 27 | 18 | 15 | 19 | 10 | **89/100** | ✅ Read May 2026 |
| Untitled UI PRO VARIABLES v6.0 (partial file, 11 pages) | B (three-tier) | 28 | 17 | 10 | 14 | 3 | **72/100** | ✅ Read May 2026 |
| Untitled UI PRO (full file, estimated) | B (three-tier) | 28 | 17 | 10 | 19 | 6 | **~80/100** | Partial read May 2026 |
| Figma Simple DS | B | 26 | 19 | 15 | 18 | 8 | **86/100** | Knowledge |
| Polaris UI Kit (community) | E | 5 | 8 | 0 | 18 | 5 | **36/100** | ✅ Read May 2026 |
| Typical early-stage DS | C | 5 | 8 | 0 | 8 | 0 | **~21/100** | Knowledge |
| Style-only legacy file | D | 0 | 10 | 0 | 10 | 3 | **~23/100** | Knowledge |

**Verified data (all ✅ Read May 2026):**

*Material Design 3:* 4 variable collections · 304 total variables · 32 color modes · 33 pages · 25 component pages · 10 shape tokens · 90 typography tokens

*Atlassian DS:* Two-file system (Foundations + Components) · Semantic two-tier with interaction-state naming · Internal governance markers (⛔️ DST team only) · Beta/Deprecated lifecycle markers · Strong usage documentation across all major components

*Untitled UI PRO VARIABLES v6.0 (partial copy, 11 pages read):* 7 collections · 850 total variables · Light + Dark modes · Three-tier colour system (primitives → semantic → component-specific) · Exemplary scoping on semantic layer · Mixed naming conventions (slash for colour, kebab for dimensions) · Parenthetical primitive hints on semantic tokens (`text-primary (900)`) — recommended pattern · Button: 380 variants, 5 sizes, 7 hierarchies, 4 states — best button depth across all reads · Component descriptions empty across all checked components · Full file estimated at ~80/100

*Polaris UI Kit (community):* 8 collections · 65 variables · Single mode · Component-level naming (`s-button-primary-background`) · Critical: many variables with `scopes: []` (invisible in all pickers) · Community kit only — does not represent real Polaris token system (see polaris.shopify.com/tokens)

---

## Section 9 — What the Score Means

| Score | Maturity level | What it means |
|---|---|---|
| 85–100 | Production-ready | DS is comprehensive, consistent, and scalable. Safe to build on. |
| 65–84 | Solid foundation | Major patterns are right. Gaps exist but won't block delivery. |
| 40–64 | Work in progress | Core components present but token system or modes are incomplete. |
| 20–39 | Early stage | DS has components but no real token architecture. Needs rebuild, not patch. |
| 0–19 | Pre-DS | Component library at best. Not a design system yet. |

---

*Last updated: May 2026 (v1.2). DS files read to date: M3, Polaris UI Kit (community), ADS Foundations + ADS Components, Untitled UI PRO VARIABLES v6.0 (partial copy). Add new patterns as new DS files are read.*

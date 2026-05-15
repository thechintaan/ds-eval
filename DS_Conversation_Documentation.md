# Documentation: Building a Design System Evaluation Framework

**Session date:** May 2026  
**Participants:** Chintan (product designer / DS owner) + Claude  
**Duration:** Multi-session conversation  
**Status:** Active — skill build (`/read-design-system`) is next step

---

## Context

Chintan posed a foundational question: design systems are built with wildly different philosophies — role-based, primitive+semantic, flat, style-only. How do you evaluate one objectively when the structure itself changes depending on the team's approach?

The conversation turned into a live research session: instead of relying on Claude's training knowledge, we read real DS files directly via the Figma Plugin API, extracted ground truth, and built an evidence-based evaluation rubric from scratch. Three DS files were read and verified: Material Design 3, Polaris UI Kit (community), and Atlassian DS (Foundations + Components).

---

## Decisions Made

**Decision: Read real files — don't trust knowledge alone**
- Why: Chintan challenged Claude's initial estimates with "you're saying you have knowledge — I don't have a way to verify that." Claude's training-knowledge estimate of M3 was 97/100. The actual read produced 89/100 — a significant gap due to missing spacing tokens, missing effect tokens, and empty component descriptions.
- Trade-off: Requires files to be open in Figma desktop. Community files must be moved to a project AND opened in the app before the API can read variables. More work upfront.
- Owner: Chintan (opens files) + Claude (reads via `use_figma`)
- Constraint: The Figma Plugin API only reads whatever file is currently open in the desktop app. If the wrong file is open, you get wrong data.

**Decision: Identify the DS pattern before scoring**
- Why: A role-based system (M3) should not be penalised for lacking a primitives collection — that's intentional. A pattern-first classification step means the rubric scores against the right bar for each architecture.
- 5 patterns identified: A (role-based/semantic-first), B (primitive+semantic two-tier), C (flat/primitive-only), D (style-only/pre-variables), E (component-level token system)
- Trade-off: More complexity in the rubric. Worth it — without this, you'd incorrectly fail good systems.

**Decision: Data-driven HTML template (not hardcoded report)**
- Why: The evaluation output structure never changes — only the data does. A `DS = { ... }` object at the top, rendered by a static function, means: write once, reuse for every DS evaluated. No copy-pasting HTML per report.
- Trade-off: Slightly more setup upfront. Eliminated immediately when the value became clear.
- Owner: Claude (build) + Chintan (data input per evaluation)

**Decision: Build the rubric from 3 verified real reads before building the skill**
- Why: Rubric built from knowledge has unverifiable gaps. Rubric built from real file reads is defensible in front of a team. Chintan's explicit need: "when I present this to my team, I want to show this is how it's done."
- DS files read: M3, Polaris (community UI kit), ADS Foundations + ADS Components
- DS files deferred (next session): Untitled UI, Figma Simple DS

---

## Patterns Identified

**Pattern: Community kit ≠ real design system**
- Evidence: Polaris UI Kit (community) scored 36/100 — component library value, but near-zero token architecture. The actual Polaris token system at polaris.shopify.com/tokens would score 90+.
- Implication: When evaluating, distinguish between a community Figma kit (for reference/inspiration) and a production design system (for building on). They are different things. State this explicitly in any evaluation output.

**Pattern: Knowledge estimates drift significantly from verified reads**
- Evidence: M3 estimated at 97/100 (knowledge). Actual read: 89/100. The gap was caused by spacing tokens missing from the variable system, no effect tokens, and empty component descriptions — details Claude's training data didn't capture accurately.
- Implication: For any DS evaluation presented to a team, you must read the file. Don't estimate.

**Pattern: The file must be open in Figma desktop, not just in a project**
- Evidence: First attempt on M3 community file — moved to project, but not opened. `get_metadata` returned only 1 page ("Getting started"). `search_design_system` returned nothing. After opening the file in the desktop app, `use_figma` returned the full variable system.
- Implication: Add this as a pre-flight step in the skill: "Is the file open in Figma desktop right now?"

**Pattern: Scopes are the hidden failure mode**
- Evidence: Polaris community kit had many variables with `scopes: []` (empty array). This makes tokens completely invisible in every Figma property picker — designers cannot find or apply them. Worse than `ALL_SCOPES` in practice.
- Implication: Token scopes must be part of the health check. Two failure modes in opposite directions: `ALL_SCOPES` = too permissive, `[]` = invisible.

**Pattern: Split-file architecture is a maturity signal, not a gap**
- Evidence: ADS ships as two separate published libraries — Foundations (all variables/tokens) and Components (consumes Foundations). Initially appeared to be a missing piece. On closer look, it enforces clean ownership boundaries.
- Implication: `get_libraries` must be called to detect connected library files. Evaluate split-file systems together as one system before scoring.

**Pattern: Interaction states have two valid architectures**
- Evidence: M3 uses state layer modes (opacity layers applied over base tokens). ADS encodes state into variable names (`color/text/interactive/hovered`). Both are correct.
- Implication: Rubric must not penalise one approach over the other. What matters: are states covered at all? Not which pattern handles them.

---

## Principles

**Principle: Read first, score second**
- Evidence conversations that assume structure produce unreliable evaluations. The rubric starts with "Identify the pattern." Scoring only makes sense relative to what the DS is trying to be.

**Principle: Evidence over estimation — especially for team-facing work**
- Chintan's driving concern: "you're saying you have knowledge — I don't have a way to verify that." When presenting to a team, every finding needs to be backed by what you actually read, not what you inferred.

**Principle: The rubric scores consistency, not style choice**
- Naming conventions are all valid (slash hierarchy, kebab, camelCase) as long as they're consistent. A DS that uses slash naming everywhere gets full marks. One that mixes three conventions fails — regardless of which convention it chose.

**Principle: Distinguish what's in the Figma file from what's in the real DS**
- The community kit is a snapshot, often outdated, often simplified. The canonical DS lives in code (polaris.shopify.com/tokens, m3.material.io). Figma file evaluations should be scoped appropriately and caveated.

---

## Approach Improvements (discovered mid-conversation)

**From: static hardcoded HTML reports → To: data-driven template**
- Current: Write separate HTML per DS evaluation (all values hardcoded in markup)
- Better: Single template with `const DS = { ... }` data object at top + static `render()` function. Change only the data object per evaluation.
- Why: Same structure every time. One template to maintain. Instantly reusable.

**From: relying on knowledge for DS scores → To: reading real files**
- Current: Claude estimates scores from training knowledge, presents them as facts
- Better: `use_figma` + `getLocalVariableCollectionsAsync()` + `get_metadata` for page/component counts. Real numbers, real API calls.
- Why: Defensible in front of a team. Catches real gaps training knowledge misses.

**From: evaluating a DS in isolation → To: checking for connected library files first**
- Current: Read one file, score it
- Better: Run `get_libraries` first to detect whether the open file is part of a multi-file system. If yes, note the connected files and evaluate as a single system.
- Why: Split-file DS like ADS would be severely undersold if evaluated file-by-file.

---

## Key Technical Findings (per DS)

### Material Design 3 — Score: 89/100 (✅ Verified)
- Pattern A: Role-based / semantic-first
- 4 collections: `M3` (197 vars, 32 modes), `Typescale` (90 vars), `Shape` (10 vars), `Font theme` (7 vars)
- 304 total variables, 32 color modes, 33 pages, 25 component pages
- Gap: No spacing tokens in variable system. No effect/shadow tokens. Component descriptions empty.
- Notable: 32 modes is exceptional — light, dark, contrast variants, + 12 dynamic color themes

### Polaris UI Kit (community) — Score: 36/100 (✅ Verified)
- Pattern E: Component-level token system
- 8 collections, 65 variables, single mode throughout
- Naming: `s-button`, `s-button-primary-background` — tied to components, not semantic roles
- Critical issue: Many variables with `scopes: []` — invisible in all Figma property pickers
- Component quality is decent (atoms + molecules), but disconnected from token system
- ⚠️ This is a community kit, not the real Polaris. See polaris.shopify.com/tokens for actual token system.

### Atlassian DS (Foundations + Components) — Score: 89/100 (✅ Verified)
- Pattern B (split-file variant): Primitive + semantic two-tier across two published library files
- Foundations file owns all variables. Components file consumes as subscribed library.
- Interaction states as variable name suffixes: `color/text/interactive/hovered`, `color/border/focused/subtle`
- Internal governance: `⛔️ DST team only` markers on restricted tokens
- Beta/Deprecated lifecycle markers on components in transition
- Strong documentation: usage examples, dos/don'ts, component descriptions filled

---

## Skill Opportunities

**Skill: `/read-design-system`**
- What it does: Opens a Figma DS file, reads its structure via the Plugin API, scores it against the rubric, outputs a scored HTML health check report
- Inputs: Figma file URL (file must be open in desktop app)
- Output: DS_Health_Check.html — accordion report with scores, findings, verdict
- When to use: Before adopting a new DS, before presenting DS maturity to a team, when auditing your own DS
- Current status: Rubric complete (v1.1), HTML template built. Skill file build is next step.

**Skill: `/ds-health-report`** (derived output step)
- What it does: Takes a filled DS data object and renders it into the standard HTML template
- Inputs: DS evaluation data object (name, pattern, score, dimensions, verdict)
- Output: Rendered HTML file saved to workspace
- Note: This is the output step of `/read-design-system`, could be split into its own skill if rendering needs to be reused independently

---

## Prompt Structures

**For reading a variable collection:**
```
const collections = await figma.variables.getLocalVariableCollectionsAsync();
const results = collections.map(c => ({
  name: c.name,
  id: c.id,
  varCount: c.variableIds.length,
  modes: c.modes.map(m => m.name)
}));
return results;
```

**For pattern detection from variable names:**
Look for:
- `Schemes/`, `On Primary`, `Surface Container`, `State Layers/` → Pattern A
- Two collections with one named "primitives" + one named "semantic" or "modes" → Pattern B
- Single collection, names match color values (`brand-blue`, `text-dark`) → Pattern C
- No variable collections, only styles → Pattern D
- Variables exist but names match component names (`s-button`, `s-input`) → Pattern E

**For checking if a file is part of a split-file system:**
```
Use get_libraries to detect connected published libraries.
If Foundations library is connected → evaluate as Pattern B split-file.
```

---

## Files Built This Session

| File | Purpose | Status |
|---|---|---|
| `DS_EVAL_RUBRIC.md` | Master rubric v1.1 — 5 patterns, 5 dimensions, 3 verified benchmarks | ✅ Complete |
| `DS_Health_Check_Template.html` | Data-driven output template (DS object + renderer) | ✅ Complete |
| `DS_Health_Check_Report.html` | One-off M3 report (pre-template, superseded) | Archived |
| `DS_Conversation_Documentation.md` | This file | ✅ New |

---

## What Comes Next

1. **Build `/read-design-system` skill** using `/skill-creator` — this is the end goal of the entire session
2. **Read Untitled UI** — next reference DS to add to benchmark table
3. **Read Figma Simple DS** — another Pattern B reference to verify
4. **Populate template with Polaris and ADS data** — create `DS_Health_Check_Polaris.html` and `DS_Health_Check_ADS.html` as additional verified report examples

---

*Documented: May 2026. From multi-session conversation between Chintan and Claude.*

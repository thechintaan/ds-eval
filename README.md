# Design System Eval

A skill and rubric for evaluating any Figma design system — how mature is it, where are the gaps, and how does it compare to industry benchmarks like Material Design 3 and Atlassian DS.

Built from reading 4 real DS files via the Figma Plugin API. Scores are evidence-based, not estimated.

---

## What's in this repo

| File / Folder | What it is |
|---|---|
| `read-design-system.skill` | Install this. Gives you `/read-design-system` in Cowork. |
| `read-design-system/` | Skill source — SKILL.md, rubric, HTML template. |
| `DS_EVAL_RUBRIC.md` | The scoring rubric. Read this to understand how scores work. |
| `reports/` | Example health check reports from previous evaluations. |

---

## How to run an evaluation

**Requirements:** Cowork desktop app · Figma desktop app · Figma MCP connected in Cowork

To connect Figma MCP: open Cowork → Settings → Plugins → find Figma → connect your account. Without this, the skill cannot read Figma files.

**Step 1 — Install the skill**
Double-click `read-design-system.skill`. Cowork installs it automatically.

**Step 2 — Open the DS file in Figma desktop**
The file must be actively open in the Figma desktop app — not just in a browser tab or sitting in a project. This is important: the API only reads whichever file is currently open.

**Step 3 — Run it**
Open Cowork. Type:
```
/read-design-system [paste Figma URL here]
```
Claude reads the file, scores it across 5 dimensions, and saves an HTML report to your workspace folder.

**Step 4 — Add your report to this repo**
Save your report as `DS_Health_[DSName]_[MonthYear].html` and add it to the `reports/` folder. Add a row to the benchmark table in `DS_EVAL_RUBRIC.md`. Push.

---

## Scoring dimensions

| Dimension | Max | What it measures |
|---|---|---|
| Token architecture | 30 | Variables, two-tier structure, aliasing, scopes |
| Naming convention | 20 | Consistency, hierarchy clarity, no ambiguity |
| Mode support | 20 | Light/dark, brand themes, density modes |
| Component coverage | 20 | Atom/molecule presence, variant depth |
| Documentation | 10 | Descriptions, usage examples, governance |

**Score → maturity:**
- 85–100 · Production-ready
- 65–84 · Solid foundation
- 40–64 · Work in progress
- 20–39 · Early stage
- 0–19 · Pre-DS

Full criteria in `DS_EVAL_RUBRIC.md`.

---

## Verified benchmarks

| Design System | Pattern | Score | Evaluated |
|---|---|---|---|
| Material Design 3 | A — role-based | 89/100 | May 2026 |
| Atlassian DS (2 files) | B — split-file | 89/100 | May 2026 |
| Untitled UI PRO VARIABLES v6.0 | B — three-tier | 72/100 | May 2026 |
| Polaris UI Kit (community) | E — component-level | 36/100 | May 2026 |

---

## Known constraints

**The Figma API reads whatever file is open in the desktop app.** If you get empty results, the file isn't open — open it in Figma desktop and try again.

**Community kits ≠ real design systems.** The Polaris community UI kit scored 36/100. The actual Polaris token system (polaris.shopify.com/tokens) would score significantly higher. Always note which file you evaluated.

**Partial file copies score lower on component coverage.** If the file URL says "Copy" or has fewer pages than expected, flag it in your report.

---

## Contributing

Run an evaluation → add the HTML report to `reports/` → add a row to the benchmark table in `DS_EVAL_RUBRIC.md` → open a PR.

If you find a new DS pattern not covered by the rubric (Patterns A–E), add it to the rubric with verified evidence and bump the version number.

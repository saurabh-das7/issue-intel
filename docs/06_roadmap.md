# 06 — Roadmap

*llm-issue-categorizer · Last updated: April 2026*

---

## Constraints

**Time budget:** ~8 hours per week on personal time. Target: live 
URL in 4 weeks from environment setup.

**Skill level:** Comfortable with Python and the Streamlit + Gemini 
API stack from the LLM Eval Toolkit project. New territory this 
time: multi-file input parsing (CSV/Excel/TXT), batch LLM processing 
with stateful category accumulation, Plotly chart rendering, and 
a two-pass LLM architecture (main run + consolidation).

**Solo build:** No collaborators. All decisions, code, and testing 
are single-threaded.

**Definition of done:** A public Streamlit Community Cloud URL that 
a hiring manager can open, run against a sample dataset with two 
clicks, and receive a complete categorisation output with chart, 
review panel, and downloadable CSV — without any setup or 
instructions.

---

## Milestone Plan

---

### Milestone 0 — Environment Setup
**Target: Week 1, Days 1–2 | ~2 hours**

Stand up the local development environment, confirm all dependencies 
install cleanly, and verify the Gemini API key works end-to-end 
before writing any application code. This milestone is complete when 
a single test API call returns a valid response.

Deliverables:
- Personal machine Python environment confirmed (Python 3.9+)
- `requirements.txt` created with all 5 dependencies
- Virtual environment created and activated
- Gemini API key obtained from Google AI Studio (free, no credit card)
- Test script sends one prompt to Gemini 2.5 Flash-Lite and receives 
  a valid JSON response
- GitHub repo `llm-issue-categorizer` cloned locally with correct 
  folder structure: `app/`, `docs/`, `requirements.txt`

**Done when:** `python test_api.py` returns a valid Gemini response 
with no errors.

---

### Milestone 1 — Categorisation Engine
**Target: Week 1, Days 3–5 | ~4 hours**

Build and validate the core engine: file parsing, batch processing, 
prompt design, and category accumulation logic. This is the 
highest-risk milestone because it requires solving the two hardest 
technical problems — batch consistency and the consolidation pass — 
before any UI exists. Building the engine in isolation makes it 
testable independently of Streamlit rendering.

Deliverables:
- `app/engine.py` — standalone Python module with:
  - `parse_file(file)` — accepts CSV, Excel, TXT; returns clean DataFrame
  - `run_categorisation(df, context, mode, categories)` — batch 
    processing loop, 10 rows per call, accumulates category list 
    across batches, returns DataFrame with new_category, confidence, 
    reasoning columns
  - `run_consolidation(df, context)` — single call returning 
    one-liners, auto-merges, merge flags, and uncategorised analysis
  - `run_multi_tag(df, context, categories)` — second pass 
    returning multi_tags column
- `app/samples.py` — three sample datasets as Python dictionaries, 
  loadable without file I/O
- Prompt templates for all three call types documented in 
  `engine.py` as module-level constants — not buried in function 
  bodies
- Batch consistency test: run No Suggestions mode on UPI sample, 
  verify category labels are consistent across batches (no 
  near-duplicate names)
- Consolidation pass test: verify auto-merge fires correctly on 
  known merge candidates in sample data
- Batch size confirmed at 10 rows or adjusted based on test results

**Done when:** `engine.py` run directly against all three sample 
datasets returns categorisation results with ≥80% accuracy and 
consistent category labels, with no unhandled exceptions.

---

### Milestone 2 — Evaluate Tab: Single-Bucket Flow End-to-End
**Target: Week 2, Days 1–4 | ~5 hours**

Build the complete Streamlit UI for the primary flow: data input, 
row validation, mode selection (No Suggestions only at this stage), 
progress bar, results table, bar chart, and download. This 
milestone delivers the full zero-friction demo path — sample tile 
→ run → results — before the more complex mode configurations are 
added.

A test deployment to Streamlit Community Cloud happens at the end 
of this milestone to catch any environment issues early.

Deliverables:
- `app/main.py` — Streamlit application with:
  - Step 1: three sample tiles + upload zone, context field, 
    row count validation
  - Step 2: mode selection (No Suggestions selected, others visible 
    but inactive at this stage)
  - Step 3: progress bar with batch count and ETA
  - Step 4: results table with confidence filter toggle
  - Step 4: Plotly horizontal bar chart
  - Step 4: Download CSV button (top and bottom)
  - Start over button wired and functional
- Zero-friction path tested: UPI tile → run → results in ≤2 clicks
- Confidence filter toggle tested: Low confidence rows filter 
  correctly
- Download CSV tested: output file contains all original columns 
  plus new_category, confidence, reasoning
- Test deployment to Streamlit Community Cloud — app loads 
  and runs against sample data on the live URL

**Done when:** The live Streamlit Community Cloud URL returns 
complete results for the UPI sample dataset in ≤90 seconds with 
no errors.

---

### Milestone 3 — Manual List and Auto-Suggest Modes
**Target: Week 2, Day 5 – Week 3, Day 2 | ~4 hours**

Add the Manual List and Auto-Suggest mode configurations to the 
UI. The engine already supports both modes from Milestone 1 — 
this milestone wires the UI controls to the engine correctly.

Deliverables:
- Manual List mode:
  - Category textarea with live counter
  - Minimum 2 / maximum 15 validation logic
  - Run button disabled until validation passes
  - Correctly passes user categories to `run_categorisation()`
  - User-defined categories with zero matches shown at 0% in chart 
    with tooltip
- Auto-Suggest mode:
  - "Generate suggestions" button triggers 25% sample call
  - Progress indicator during suggestion generation
  - Editable chip list renders suggested categories
  - Add / rename / delete chip functionality working
  - "Run full categorisation" button triggers main run with 
    confirmed taxonomy
  - Mode correctly disabled and greyed out for datasets ≤50 rows
- All three modes tested against all three sample datasets
- Upload your own file path tested: CSV, Excel, and TXT formats 
  all parse correctly, context field clears when switching from 
  sample to upload path

**Done when:** All three modes produce correct categorisation 
output on all three sample datasets with no unhandled exceptions.

---

### Milestone 4 — Review Panel, Multi-Tag View, and Post-Run UX
**Target: Week 3, Days 3–5 | ~4 hours**

Add the post-run review panel (transparency log, category 
summaries, merge flags, uncategorised analysis) and the advanced 
multi-tag view. These are the features that differentiate the tool 
from a simple classification script — this milestone is where the 
product quality becomes visible.

Deliverables:
- Post-run review panel:
  - Collapsed by default, expands on click
  - Transparency log renders auto-merges correctly (or confirms 
    none occurred)
  - Per-category summaries render for all final categories
  - Merge flags section hidden when none found
  - Uncategorised bucket analysis renders themes (or confirmation 
    note if bucket empty)
  - All content generated by `run_consolidation()` — no 
    hardcoded strings
- Advanced multi-tag view:
  - Run button card renders below review panel
  - Second progress bar runs on click
  - On completion: multi-tag card disappears, toggle appears in 
    table header, view switches to multi-tag automatically
  - Bar chart relabels correctly for multi-tag view
  - Toggle switches between views instantly (no re-processing)
  - multi_tags column populated in downloaded CSV after multi-tag run
- Token budget for consolidation call confirmed and documented 
  (open question from PRD resolved)

**Done when:** Review panel and multi-tag view work correctly on 
all three sample datasets. All four review panel sections render 
accurately. Multi-tag toggle behaviour matches the UX spec.

---

### Milestone 5 — Polish, Edge Cases, and Pre-Launch Validation
**Target: Week 4, Days 1–3 | ~4 hours**

Systematic validation against the pre-launch checklist in 
`docs/08_launch_and_retro.md`. Fix all blocking issues. Polish 
UI copy, error messages, and edge case handling. No new features 
at this milestone — only quality.

Deliverables:
- All edge cases in the UX doc tested and handled:
  - File >100 rows: hard block with correct message
  - Missing columns: named error
  - TXT wrong format: example shown
  - Empty context field: button stays disabled
  - Upload after sample: context field clears
  - All rows Uncategorised: correct message shown
  - API batch failure: partial results returned, error flagged per row
- Accuracy validation: all three sample datasets run in No 
  Suggestions mode, ≥80% agreement with expected output in 
  `02b_sample_data_bank.md`
- Performance validation: 100-row run completes in ≤90 seconds 
  (median of 3 runs)
- Data warning shown on upload path before file is processed
- `requirements.txt` versions pinned to confirmed working versions
- README updated with live URL and final setup instructions
- Build log (`07_build_log.md`) entries complete for all milestones

**Done when:** All items in the pre-launch checklist pass. Zero 
unhandled exceptions on valid input across all tested flows.

---

### Milestone 6 — Launch
**Target: Week 4, Days 4–5 | ~1 hour**

Final smoke test on the live URL. Share. Update project status 
in README and LinkedIn.

Deliverables:
- Final smoke test sequence executed on live URL (not localhost):
  all three sample tiles, all three modes, multi-tag run, 
  review panel expand, download CSV
- Live URL added to README
- Project status updated from "🔨 Actively building" to 
  "✅ Live" in README
- Build log final entry written
- `08_launch_and_retro.md` pre-launch checklist signed off
- LinkedIn post or profile update referencing the live tool

**Done when:** The live URL is shareable and the full zero-friction 
path works end-to-end on the deployed app.

---

## Timeline Summary

| Milestone | Description | Week | Hours |
|-----------|-------------|------|-------|
| M0 | Environment setup | 1, Days 1–2 | ~2h |
| M1 | Categorisation engine | 1, Days 3–5 | ~4h |
| M2 | Primary UI flow end-to-end | 2, Days 1–4 | ~5h |
| M3 | Manual list + auto-suggest modes | 2–3 | ~4h |
| M4 | Review panel + multi-tag view | 3, Days 3–5 | ~4h |
| M5 | Polish + pre-launch validation | 4, Days 1–3 | ~4h |
| M6 | Launch | 4, Days 4–5 | ~1h |
| **Total** | | **4 weeks** | **~24h** |

At 8 hours per week, this is a 3-week build with one week of buffer. 
The buffer exists for Milestone 1 — if the batch consistency or 
consolidation pass requires significant prompt iteration, that time 
comes from the buffer before any downstream milestone slips.

---

## Critical Path

The minimum dependency chain to a shareable URL:

```
M0 (API works)
    ↓
M1 (engine produces correct output)
    ↓
M2 (primary UI flow live on Streamlit Cloud)
    ↓
M5 (validation passes)
    ↓
M6 (launch)
```

M3 and M4 are not on the critical path — the app is demonstrable 
after M2 with the No Suggestions mode alone. If time runs short, 
M3 and M4 can slip to a v1.1 release without blocking the live URL.

---

## Post-MVP Backlog

In priority order based on product value and build effort:

| Feature | Why it matters | Estimated effort |
|---------|---------------|-----------------|
| Auto-suggest taxonomy mode improvements — smarter sampling strategy | Current 25% random sample may miss rare but important issue types | Medium |
| Trend analysis — requires date field, surfaces fastest-growing categories | High PM value for teams with longitudinal ticket data | High |
| Filter and drill-down — click a bar to see the tickets in that category | Reduces manual table scanning post-run | Medium |
| Progress bar for multi-tag run visible in table section, not just the run card | Minor UX improvement | Low |
| Support for JSON input format | Useful for teams exporting from APIs rather than spreadsheets | Low |
| Batch size auto-tuning based on dataset size | Small datasets (20–30 rows) would benefit from smaller batches | Low |
| Jira / Zendesk CSV export format auto-detection | Removes need for manual column mapping for common ticket tools | Medium |

---

*Previous: [05 — Risk & Cost Plan](./05_risk_and_cost.md)*  
*Next: [07 — Build Log](./07_build_log.md)*

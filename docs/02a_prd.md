# 02a — Product Requirements Document (PRD)

*llm-issue-categorizer · Version 2.0 · Last updated: April 2026*

---

## Product Overview

A Streamlit web application that accepts operational issue data — customer 
support tickets, vendor logs, monitoring alerts — re-categorises every row 
against a structured taxonomy using an LLM, and surfaces the patterns a PM 
needs to make product decisions. Supports three categorisation modes, dual 
output views (single-bucket and multi-tag), and a self-cleaning uncategorised 
bucket. Designed to process up to 100 rows in under 90 seconds.

---

## Primary User

A PM or TPM who manages a vendor operations or support function, has an 
export of ticket data with vague or inconsistent labels, and needs to 
understand what is actually breaking and how often — without a data 
scientist to do it for them.

---

## Goals — What v1 Accomplishes

1. Accept operational issue data in CSV, Excel, or TXT format and validate 
   it before any processing begins
2. Enforce a hard 100-row limit — no partial runs, no silent truncation
3. Set data context upfront so the LLM makes domain-aware categorisation 
   decisions throughout
4. Offer three categorisation modes — no suggestions, manual list, 
   auto-suggest — with auto-suggest gated on dataset size
5. Process all rows in optimised batches with a live progress bar
6. Deliver a primary single-bucket output and an advanced multi-tag output 
   on demand
7. Surface a post-run review panel with per-category summaries, merge 
   flags, and an uncategorised bucket analysis
8. Produce a downloadable CSV with original data plus all new fields appended
9. Allow a first-time user to experience the full output with zero typing 
   using built-in sample datasets

---

## Non-Goals — What v1 Explicitly Does Not Do

**Trend analysis over time** — v1 does not require a date field and does 
not surface fastest-growing categories. Trend analysis is only meaningful 
when data spans multiple time periods with consistent volume. Deferring 
keeps v1 focused on the categorisation value before adding temporal 
complexity.

**Category merge UI** — merging categories via UI interaction is not 
supported. High-confidence merges are handled automatically by the final 
consolidation pass and logged transparently. Uncertain merge cases are 
flagged in plain English in the review panel for user awareness. There 
is no in-app merge action — the output the user downloads already 
reflects all high-confidence consolidations.

**Filter and drill-down** — v1 shows the full distribution. Filtering by 
category, confidence tier, or date range is post-MVP.

**Integration with ticketing systems** — v1 works from a file export only. 
No Jira API, no Zendesk webhook, no live database connection.

**Excel or JSON output** — output is CSV only. Input accepts CSV, Excel, 
and TXT; output is standardised to CSV for simplicity and portability.

**Processing more than 100 rows** — this is a hard limit, not a soft 
warning. Files above 100 rows are blocked at upload. The user is shown 
a clear message with the row count and instructed to trim the file.

---

## Feature Specification

---

### Step 1 — Data Context and File Input

The landing screen presents two paths side by side.

**Path A — Sample datasets (zero-friction entry)**

Three tiles, each representing a different data domain:

| Tile | Domain | File |
|------|--------|------|
| 📱 UPI Customer Care | Customer support tickets for a UPI payments app | `samples/upi_tickets.csv` |
| 📊 Ads System Monitoring | Tickets created by a monitoring tool tracking Ads platform metrics | `samples/ads_monitoring.csv` |
| 🤖 Copilot Support | Support tickets for a Copilot-style AI assistant product | `samples/copilot_tickets.csv` |

Clicking a tile selects both the context description and the associated 
file in a single action. No upload required. The context description 
auto-populates in an editable text field below — the user can leave it 
as-is or refine it before proceeding.

**Path B — Upload your own file**

File uploader accepts `.csv`, `.xlsx`, `.txt`. Below the uploader, a 
required text field: "What kind of data is this?" — the user writes a 
one-line description (e.g. "Support tickets for a logistics ops team in 
Southeast Asia"). This context is passed to the LLM with every prompt 
so categorisation decisions are domain-aware.

**Expected columns (case-insensitive, whitespace-trimmed on import):**

| Column | Required | Description |
|--------|----------|-------------|
| `ticket_id` | Yes | Unique identifier |
| `issue_description` | Yes | Free-text description of the issue |
| `current_label` | Yes | Vendor-assigned or system-generated label |
| `resolution_notes` | No | Free-text resolution summary |

For TXT files: pipe-delimited, one row per line, format: 
`ticket_id | issue_description | current_label | resolution_notes`

---

### Step 2 — Row Count Validation

Immediately after file selection or upload, the app reads and counts rows 
before any other action.

| Row count | Behaviour |
|-----------|-----------|
| >100 | Hard block. Message: "This file has [N] rows. The tool supports up to 100 rows. Please trim the file and re-upload." No further action until a valid file is provided. |
| 51–100 | All three categorisation modes available |
| 1–50 | Auto-suggest mode disabled. Greyed out with tooltip: "Auto-suggest requires more than 50 rows to sample meaningfully." |
| 0 | Error: "No data rows found. Please check the file format." |

Row count and column validation both run at this step. All errors are 
surfaced together — not one at a time.

---

### Step 3 — Categorisation Mode Selection

Three modes presented as a radio button group. Default selection: 
No Suggestions.

---

**Mode 1 — No Suggestions**

The LLM reads all rows and generates its own category names from the 
data. No user input required at this step.

*How it works:* Rows are processed in batches of 10. The first batch 
establishes an initial category set. Each subsequent batch receives the 
current category list as context — the LLM maps to existing categories 
where appropriate and creates new ones only when no existing category 
fits and the pattern would reach the minimum bucket threshold. This 
enforces label consistency across the run without a separate 
consolidation pass.

Rows that do not fit any existing category and cannot justify a new 
one are assigned to Uncategorised.

---

**Mode 2 — Manual List**

The user provides an initial set of category names — one per line in 
a text area. The LLM maps rows to these categories and may create 
additional categories if the data warrants them.

*New category creation rule:* A new category is only created when the 
LLM identifies a pattern that does not fit any user-defined category 
AND that pattern would reach the minimum bucket threshold across the 
full dataset. Rows that cannot justify a new category go to Uncategorised.

User-defined categories with zero matches are retained in the output 
at 0%, shown with a greyed bar and tooltip: "No tickets matched this 
category." They are not hidden — zero is informative.

Maximum user-defined categories: 15. A warning is shown if this is 
exceeded before the run proceeds.

---

**Mode 3 — Auto-Suggest** *(available for datasets >50 rows only)*

The app samples 25% of rows (rounded up, minimum 13 rows), sends them 
to the LLM, and receives a proposed taxonomy. The user reviews the 
suggestions in an editable list — they can rename, delete, and add 
categories before the full run begins.

Flow:
1. User clicks "Generate suggestions" → progress indicator shown → 
   proposed taxonomy appears (typically 5–10 categories)
2. User edits the taxonomy as needed
3. User clicks "Run full categorisation" → main run begins on all rows 
   using the confirmed taxonomy

In Auto-suggest mode, the confirmed taxonomy is the starting point for 
the full run — not a ceiling. The LLM maps rows to confirmed categories 
where they fit, and creates new categories during the full run when it 
identifies patterns the 25% sample did not surface, subject to the 
minimum bucket threshold. This ensures the output reflects the full 
dataset, not just what was visible in the sample. New categories created 
during the full run are handled identically to all other categories in 
the final consolidation pass.

---

### Step 4 — Processing and Progress Bar

On run initiation, a progress bar appears. Rows are processed in batches 
of 10 per API call. Progress increments per completed batch. Estimated 
time remaining is shown beneath the bar.

**Performance target:** 100 rows in ≤90 seconds on Gemini free tier 
(15 RPM). At 10 rows per call = 10 calls, well within rate limits with 
headroom for the post-run review call.

**Error handling during processing:**
- If an API call fails for a batch, those rows are flagged with an error 
  status and the run continues — partial results are always returned
- If the entire run fails, the error message names the most likely cause 
  (rate limit, invalid API key) with a suggested fix
- The progress bar does not freeze on batch failure — it skips and continues

---

### Step 5 — Bucket Threshold Logic

**Minimum bucket threshold:** `max(5 rows, 5% of total rows)`

| Total rows | Threshold |
|-----------|-----------|
| 100 | 5 rows |
| 80 | 5 rows |
| 50 | 5 rows |
| 30 | 5 rows |

The threshold is constant at 5 rows for all valid dataset sizes in v1 
since 5% of 100 rows = 5 rows exactly, and smaller datasets only raise 
the percentage equivalent. 5 rows is the floor in all cases.

**Uncategorised bucket — two triggers:**
1. LLM assigns low confidence to a row during the run
2. A named category falls below 5 rows after the full run — all its 
   rows are moved to Uncategorised

The Uncategorised bucket is always present in the output. If empty, 
it shows at 0% with a note: "All tickets were categorised with 
sufficient confidence."

---

### Output 1 — Single-Bucket View (Primary)

Each row receives exactly one category. Results shown in a scrollable 
table:

| Column | Notes |
|--------|-------|
| `ticket_id` | |
| `issue_description` | Truncated to 80 chars; full text on hover |
| `current_label` | |
| `new_category` | |
| `confidence` | High / Medium / Low |
| `reasoning` | One sentence, specific to this ticket |

**Confidence filter toggle** — when enabled, table shows only Low 
confidence rows for quick spot-checking before download.

**Bar chart** — horizontal, sorted descending by ticket count, one 
bar per category including Uncategorised. Hover shows count and 
percentage. Rendered with Plotly.

---

### Output 2 — Advanced Multi-Tag View (Secondary)

A button below the primary output: "Run advanced multi-tag view."  
Tooltip on hover: "Each ticket is mapped to all applicable themes, 
not just the primary one. This takes an additional 60–90 seconds."

A second LLM pass runs on all rows. Each row receives a list of all 
applicable categories. A separate bar chart shows total tag occurrences 
by category — the same ticket can contribute to multiple bars.

This answers a different question from Output 1: not "what is the 
primary issue type?" but "which themes appear most often across the 
dataset, including as secondary signals?"

A toggle switches between Single-Bucket and Multi-Tag views. Both 
are retained in the session — switching does not re-run processing.

The downloadable CSV includes a `multi_tags` column (pipe-separated) 
populated only if this view was run.

---

### Output 3 — Post-Run Review Panel

A collapsible panel generated by a single final LLM call after the main 
run completes. This call is structured as a two-stage pass within a 
single prompt:

**Stage A — Generate per-category one-liners**
The LLM first writes a one-sentence summary for every named category, 
based on the ticket sample and category distribution it receives. These 
one-liners serve two purposes: they are surfaced to the user in the 
review panel, and they are used immediately within the same call to 
inform merge decisions. A merge based on one-liners is a merge based 
on what the category actually contains — not just what it is named.

**Stage B — Consolidation decisions**
Using the one-liners, ticket counts, and a sample of 2–3 tickets per 
category as input, the LLM makes consolidation decisions:

*High-confidence merges — auto-applied:*
Where two categories are semantically near-identical based on their 
one-liners and ticket samples, the LLM merges them automatically and 
assigns a consolidated label. The merged result is what the user sees 
in the output — the intermediate state is not exposed. A transparency 
note in the review panel logs every merge that occurred:
*"Merged: 'API latency issue' + 'Slow response time' → 'API / response 
latency' (12 tickets)"*

*Uncertain merges — flagged, not applied:*
Where categories appear related but the LLM is not confident enough to 
merge, a flag is surfaced in the review panel for user awareness. 
Maximum 3 flags. Section hidden if none found.
Example: *"'Payment timeout' and 'Transaction failure' may overlap — 
review before sharing."*

The review panel surfaces four sections in order:

1. **Transparency log** — every auto-merge that occurred, with the 
   consolidated label and ticket count. If no merges occurred, a 
   confirmation note: "No categories were merged — all categories 
   were sufficiently distinct."

2. **Per-category summaries** — one sentence per final category, 
   written in plain language for stakeholder sharing.

3. **Merge flags** *(if any)* — uncertain merge suggestions for 
   user awareness. Shown only when present.

4. **Uncategorised bucket analysis** — top 2–3 themes observed 
   within Uncategorised rows, written as observations.
   Example: *"Several uncategorised tickets describe edge cases in 
   multi-currency settlements — potentially a pattern worth tracking."*
   If the bucket is empty, replaced with: "All tickets were categorised 
   with sufficient confidence."

---

### Download

A single **Download CSV** button. Exported fields:
- All original columns
- `new_category` — single-bucket assignment
- `confidence` — High / Medium / Low  
- `reasoning` — one-line rationale
- `multi_tags` — pipe-separated list (populated only if multi-tag 
  view was run; blank otherwise)

---

## Sample Datasets

Three built-in datasets, 80–100 rows each, hardcoded in `app/samples.py` 
and documented fully in `docs/02b_sample_data_bank.md`.

| Dataset | Domain | Rows | Expected categories |
|---------|--------|------|-------------------|
| UPI Customer Care | Payment failures, KYC issues, transaction disputes, app crashes | 90 | 6–8 |
| Ads System Monitoring | Bid anomalies, creative rejections, delivery failures, latency spikes | 85 | 6–8 |
| Copilot Support | Hallucination complaints, context loss, slow responses, formatting errors | 80 | 6–8 |

Each dataset is designed to include: clean single-category rows, 
ambiguous rows requiring full description to classify, rows that 
legitimately belong to multiple categories, rows that should end up 
Uncategorised, and at least one pair likely to trigger a merge flag.

---

## Success Metrics

| Metric | Target | How measured |
|--------|--------|--------------|
| Categorisation accuracy | ≥80% agreement with expected output on sample datasets | Manual review against `02b_sample_data_bank.md` |
| Processing time (100 rows) | ≤90 seconds | Timed during pre-launch validation |
| Zero-friction path | Full output in ≤2 clicks from page load | Tested during pre-launch validation |
| Uncategorised rate | <15% of rows on sample datasets | Checked during pre-launch validation |
| Merge flag accuracy | All flagged merges are semantically valid | Subjective review at launch |
| Multi-tag additional run time | ≤90 seconds | Timed during pre-launch validation |
| Error rate | Zero unhandled exceptions on valid input | Tested across all edge cases |

---

## Design Principles

**Context-aware categorisation.** The domain description is passed to 
the LLM with every prompt. "Slow response" means something different 
in a payments context vs. an AI assistant context. The tool uses that 
context to make the distinction.

**Auditability over automation.** Every categorisation decision includes 
a reasoning note. The post-run review panel extends this to the category 
level — not just individual rows.

**Thresholds create signal, not noise.** Categories below the minimum 
threshold are collapsed into Uncategorised and analysed separately. 
A chart with 20 categories where 12 have 1–2 rows is not useful. 
Six meaningful categories plus an analysed Uncategorised bucket is.

**Batch processing for speed and consistency.** All LLM calls process 
10 rows per batch. This reduces API call volume by 10x, keeps the run 
within Gemini free tier limits, and ensures category labels are 
consistent across the dataset.

**Fail gracefully, always.** Batch failures skip and continue. All 
errors surface with a plain-English cause and a suggested fix. Partial 
results are always returned.

**Zero friction for the first run.** Sample datasets with pre-filled 
context descriptions exist entirely so a first-time user can reach 
full output without preparing anything.

---

## Open Questions

| Question | What it affects | Resolution target |
|----------|----------------|-------------------|
| For Mode 1 (No Suggestions), should the prompt include a soft target for category count (e.g. "aim for 6–10 categories")? Unbounded risks fragmentation. | Prompt design, output usefulness | Milestone 1 |
| What is the right token budget for the post-run review panel call? Sending full categorisation results for 100 rows may be large — sending category distribution + 3 sample tickets per category may be sufficient. | API cost, prompt design | Milestone 4 |
| Should the multi-tag view show a confirmation step with estimated time before running? | UX clarity | Milestone 3 |
| Should batch size be dynamic (e.g. larger batches for smaller datasets)? | Speed optimisation | Milestone 1 |

---

*Previous: [01 — Problem Statement](./01_problem_statement.md)*  
*Next: [02b — Sample Data Bank](./02b_sample_data_bank.md)*

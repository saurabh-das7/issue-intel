# 08 — Launch & Retrospective

*llm-issue-categorizer · Pre-launch checklist to be completed at Milestone 5 · Retrospective to be written post-launch*

---

## Pre-Launch Checklist

All items below must pass before the live URL is shared. This 
checklist is run on the deployed Streamlit Community Cloud URL — 
not on localhost. A pass on localhost does not count.

---

### 1. Functional Verification — Data Input

- [ ] Landing screen loads with three sample tiles and upload zone visible
- [ ] Clicking UPI tile: highlights tile, auto-fills context field, shows row count badge, enables "Select mode" button
- [ ] Clicking Ads tile: same behaviour with correct row count and context
- [ ] Clicking Copilot tile: same behaviour with correct row count and context
- [ ] Clicking upload zone after sample tile: context field clears completely
- [ ] Uploading a valid CSV: row count shown, columns validated, context field remains blank
- [ ] Uploading a valid Excel (.xlsx) file: same behaviour as CSV
- [ ] Uploading a valid TXT (pipe-delimited) file: same behaviour as CSV
- [ ] Uploading a file with >100 rows: hard block shown with exact row count, no further UI renders
- [ ] Uploading a file with missing required columns: named error lists all missing columns together
- [ ] Uploading a TXT file with wrong format: error shown with correct format example
- [ ] Context field left blank: "Select mode" button stays disabled
- [ ] Context field with fewer than 10 characters: "Select mode" button stays disabled

---

### 2. Functional Verification — Mode Selection

- [ ] Mode selection card renders after "Select mode" is clicked
- [ ] No Suggestions is pre-selected by default
- [ ] Manual List: selecting shows textarea, category counter, run button disabled with <2 categories
- [ ] Manual List: entering 2+ categories enables run button
- [ ] Manual List: entering 16 categories shows block message
- [ ] Auto-Suggest: available and selectable for datasets >50 rows
- [ ] Auto-Suggest: greyed out with correct tooltip for datasets ≤50 rows
- [ ] Auto-Suggest: "Generate suggestions" triggers sample call and returns editable chip list
- [ ] Auto-Suggest: chips can be renamed, deleted, and added before run
- [ ] Auto-Suggest: "Run full categorisation" enabled only when ≥2 chips present

---

### 3. Functional Verification — Processing

- [ ] Progress bar appears on run initiation
- [ ] Batch count label increments correctly (e.g. "Processing batch 3 of 10...")
- [ ] ETA shown beneath progress bar and updates each batch
- [ ] Final message shows "Running final consolidation pass..." at 100%
- [ ] Progress bar does not freeze if a batch fails — continues to next batch
- [ ] Failed rows are marked ⚠ in results table if any batch fails

---

### 4. Functional Verification — Results: Single-Bucket View

- [ ] Completion message shows correct row count
- [ ] Results table renders with all five columns: ticket ID, description, old label, new category, confidence
- [ ] Description truncated to ~80 chars in table
- [ ] Confidence badges colour-coded correctly: High = green, Medium = amber, Low = red
- [ ] "Low confidence only" toggle filters table to Low confidence rows only
- [ ] "Low confidence only" toggle off restores full table
- [ ] Bar chart renders horizontally, sorted descending by ticket count
- [ ] Bar chart includes Uncategorised bucket (even if small)
- [ ] Bar chart hover shows ticket count and percentage
- [ ] Download CSV button present at top of results section
- [ ] Download CSV button present at bottom of results section
- [ ] Downloaded CSV contains all original columns plus new_category, confidence, reasoning

---

### 5. Functional Verification — Review Panel

- [ ] Review panel is collapsed by default
- [ ] Clicking panel header expands it
- [ ] Transparency log shows all auto-merges with label and ticket count
- [ ] Transparency log shows confirmation message if no merges occurred
- [ ] Per-category summaries show one sentence per final category
- [ ] Merge flags section visible when uncertain merges exist
- [ ] Merge flags section hidden when no uncertain merges found
- [ ] Uncategorised bucket analysis shows 2–3 themes when bucket is non-empty
- [ ] Uncategorised bucket analysis shows confirmation note when bucket is empty

---

### 6. Functional Verification — Multi-Tag View

- [ ] Advanced multi-tag card renders below review panel after main run
- [ ] Tooltip on "Run multi-tag analysis" button explains what it does and time cost
- [ ] Second progress bar runs on click
- [ ] On completion: multi-tag card disappears
- [ ] On completion: toggle appears in table header
- [ ] On completion: view switches automatically to multi-tag
- [ ] Toggle switches between single-bucket and multi-tag views correctly
- [ ] Multi-tag bar chart label updated to reflect multi-tag counts
- [ ] Switching views does not re-trigger any API calls
- [ ] Downloaded CSV after multi-tag run contains multi_tags column (pipe-separated)
- [ ] Downloaded CSV before multi-tag run has blank multi_tags column

---

### 7. Sample Dataset Validation (Accuracy Check)

Run each sample in No Suggestions mode. Compare output against 
expected taxonomy in `docs/02b_sample_data_bank.md`.

| Dataset | Expected categories | Accuracy target | Pass? |
|---------|-------------------|----------------|-------|
| UPI Customer Care | Payment Failure, App Crash, Refund/Dispute, KYC, Fraud, Beneficiary Mgmt | ≥80% | [ ] |
| Ads System Monitoring | Bid/Auction, Ad Delivery, Creative/Quality, Budget/Pacing, Measurement, Policy | ≥80% | [ ] |
| Copilot Support | Hallucination, Context Loss, Response Quality, Latency, Integration, Capability | ≥80% | [ ] |

Hard cases from `02b_sample_data_bank.md` — manually verify each:

| Row | Expected behaviour | Pass? |
|-----|--------------------|-------|
| UPI-025 (ambiguous — needs resolution notes) | Payment Failure / Decline | [ ] |
| UPI-049, UPI-074 (low signal) | Uncategorised | [ ] |
| ADS-022, ADS-042, ADS-077 (expected behaviour logged as issue) | Bid / Auction Anomaly or Uncategorised | [ ] |
| ADS-072 (by-design data lag) | Uncategorised | [ ] |
| COP-039, COP-078 (policy questions, no technical issue) | Uncategorised | [ ] |
| COP-065 (potential data isolation failure) | Surfaced in results or Uncategorised analysis | [ ] |

---

### 8. Performance Validation

- [ ] UPI sample (90 rows): run completes in ≤90 seconds — run 3 times, all pass
- [ ] Ads sample (85 rows): run completes in ≤90 seconds — run 3 times, all pass
- [ ] Copilot sample (80 rows): run completes in ≤90 seconds — run 3 times, all pass
- [ ] Multi-tag second pass: completes in ≤90 additional seconds on any sample

---

### 9. Consistency Validation

- [ ] Same sample, same mode, run twice: category names are consistent across runs (no label drift)
- [ ] Consolidation pass on second run: same auto-merges applied as first run
- [ ] Low confidence rows: same rows flagged as Low confidence across two runs of same sample

---

### 10. Edge Case Validation

- [ ] All rows Uncategorised scenario: tested with a minimal custom upload (2 unrelated rows, vague labels, no matching categories)
- [ ] Zero user-defined categories in Manual List: run button stays disabled
- [ ] 15 categories entered in Manual List: run proceeds with warning
- [ ] 16 categories entered in Manual List: hard block shown
- [ ] API key missing from Streamlit secrets: clear error message shown, not a stack trace
- [ ] Session timeout mid-run (simulate by leaving the tab idle): correct behaviour on return

---

### 11. Security and Configuration

- [ ] Gemini API key is in Streamlit secrets manager — not in any committed file
- [ ] `.gitignore` includes `.env` and any local secrets files
- [ ] Data warning shown on upload path before file is processed
- [ ] No PII present in `app/samples.py` — all sample data is fictional
- [ ] `requirements.txt` has pinned versions confirmed working on Streamlit Community Cloud

---

### 12. Documentation and Repo

- [ ] README shows live URL
- [ ] README status updated from "🔨 Actively building" to "✅ Live"
- [ ] All 8 docs in `docs/` are present and linked correctly in README table
- [ ] All screenshot references in `03_ux_flow_wireframe.md` resolve to existing files in `docs/images/`
- [ ] `07_build_log.md` has entries for all milestones
- [ ] `PROJECT_CONTEXT.md` updated with current milestone status
- [ ] No broken links in any doc

---

### 13. Final Smoke Test — Zero-Friction Path

Run this exact sequence on the live URL before sharing:

1. Open the live URL in a fresh browser tab (incognito, no cached state)
2. Click the UPI Customer Care tile — confirm context auto-fills and row count shows
3. Click "Select mode" — confirm mode selection card appears
4. Leave No Suggestions selected — click "Run categorisation"
5. Watch progress bar complete — confirm batch count updates correctly
6. Review results table — confirm 6–7 categories present, confidence badges colour-coded
7. Check bar chart — confirm horizontal bars sorted by volume, hover works
8. Expand review panel — confirm all four sections render
9. Click "Run advanced multi-tag view" — confirm second progress bar runs
10. Confirm toggle appears and view switches to multi-tag
11. Click "Download CSV" — confirm file downloads with correct columns
12. Click "Start over" — confirm full reset to landing state
13. Repeat steps 2–7 for Ads and Copilot tiles

**All 13 steps must pass before the URL is shared.**

---

## Retrospective

*(To be written after the app has been live for at least one week)*

---

### What worked well

---

### What was harder than expected

---

### What I would do differently

*(Specific, not generic — e.g. "I would build the consolidation pass 
prompt before the batch prompt, not after, because the output schema 
needs to be consistent across both" — not "I would plan better")*

---

### Quality assessment

*(Re-run sample dataset validation one week after launch. Note any 
accuracy drift, unexpected Uncategorised rates, or merge flag 
quality issues observed in real-world usage)*

---

### Post-MVP next bets

*(Re-prioritise the post-MVP backlog from `06_roadmap.md` based on 
what was learned during the build and from early usage)*

---

### What this project taught me

*(One honest paragraph)*

---

*Previous: [07 — Build Log](./07_build_log.md)*

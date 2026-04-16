# 05 — Risk & Cost Plan

*llm-issue-categorizer · Last updated: April 2026*

---

## Cost Philosophy

The target monthly cost for this project is ₹0. Every technical 
decision in this stack was made with this constraint as a hard 
requirement, not a preference. The cost plan below confirms that 
the ₹0 target is achievable at expected usage volumes and documents 
the exact conditions under which costs would begin to accrue.

---

## Cost Breakdown

| Line item | Provider | Free tier | Monthly cost at expected usage |
|-----------|----------|-----------|-------------------------------|
| LLM inference (categorisation + consolidation) | Google Gemini 2.5 Flash-Lite | 1,000 req/day, 15 RPM | ₹0 |
| App hosting | Streamlit Community Cloud | 1 public app, unlimited | ₹0 |
| Version control | GitHub | Unlimited public repos | ₹0 |
| Development environment | Personal machine + VS Code | — | ₹0 |
| Domain / custom URL | None — Streamlit subdomain used | — | ₹0 |
| **Total** | | | **₹0** |

---

## Usage Model and Free Tier Fit

A full run on 100 rows requires:
- 10 categorisation batch calls (10 rows per batch)
- 1 consolidation pass call
- 1 multi-tag pass call (if triggered by the user)

**Maximum 12 API calls per full session.**

At 1,000 free requests per day, the app can serve **83 complete 
100-row sessions per day** before approaching the daily limit. 
At realistic portfolio usage — occasional hiring manager demos, 
periodic self-testing during development — daily call volume will 
stay well below 100 requests.

---

## Cost Scenarios

### Expected (normal portfolio usage)
- 2–5 demo sessions per day, mix of sample and uploaded datasets
- Average 8 API calls per session (not all users run multi-tag)
- Daily call volume: 16–40 calls
- Monthly cost: **₹0**

### Elevated (active job search period, shared link circulating)
- 20–30 demo sessions per day
- Daily call volume: 160–360 calls
- Still well within 1,000 req/day free tier
- Monthly cost: **₹0**

### Degraded (free tier reduced or rate limit hit)
- Google reduces Gemini Flash-Lite free tier below ~150 req/day
- At that point, a full day of active demo usage could hit the cap
- Mitigation: switch to Gemini 2.5 Flash paid tier
- Paid cost: $0.10 per 1M input tokens + $0.40 per 1M output tokens
- At ~500 tokens per batch call × 12 calls × 30 sessions/day × 30 days = ~5.4M tokens/month
- Estimated monthly cost at paid tier: **~$0.80 / ₹67**

### Worst case (runaway usage, viral sharing)
- 200+ sessions per day sustained for a month
- Daily call volume: 1,600–2,400 calls — exceeds free tier
- Monthly cost at paid tier: **~$8 / ₹670**
- This scenario requires the app to be shared publicly at significant 
  scale — an outcome that would justify the cost

---

## Spend Controls

- Gemini API key stored in Streamlit secrets manager — never in codebase
- Hard 100-row input limit prevents any single session from making 
  more than 12 API calls regardless of user behaviour
- No background jobs, no scheduled tasks, no calls made outside of 
  an active user session
- Gemini Console spend cap can be set to $0 to block any paid usage 
  entirely — useful during development when only free tier is needed
- No auto-scaling infrastructure that could generate unexpected costs

---

## Risk Register

---

### R1 — Gemini free tier reduced or discontinued

**Category:** API  
**Description:** Google reduces the Gemini 2.5 Flash-Lite free tier 
request limit, rate limit, or discontinues the free tier entirely. 
This has precedent — Google has changed API pricing and free tier 
terms for several products with limited notice.  
**Probability:** Low  
**Impact:** Medium — app continues to function but monthly cost 
increases from ₹0 to ~₹67–670 depending on usage volume  
**Mitigation:** The app uses the standard `google-generativeai` SDK 
with no Flash-Lite-specific configuration beyond the model name. 
Switching to Gemini Flash paid tier or an OpenAI-compatible 
alternative requires changing one line: the model name string. 
No architectural change needed.  
**Escalation trigger:** Free tier daily limit drops below 200 req/day 
or is announced for deprecation

---

### R2 — LLM category label inconsistency across batches

**Category:** Product  
**Description:** In No Suggestions mode, the LLM generates category 
names row by row across batches and produces semantically similar 
but differently worded labels — e.g. "API latency" in batch 1 and 
"Slow API response" in batch 7. The consolidation pass is designed 
to catch this but may miss subtle variations, resulting in a 
fragmented category distribution that is harder to interpret.  
**Probability:** Medium — especially for datasets with diverse issue 
types and no user-provided taxonomy anchor  
**Impact:** Medium — output is still useful but requires more 
manual review before sharing  
**Mitigation:** Each batch prompt explicitly includes the accumulated 
category list from all prior batches, instructing the LLM to 
reuse existing labels where appropriate. The consolidation pass 
provides a second opportunity to merge near-duplicates. Prompt 
engineering in Milestone 1 will test and tune this behaviour.  
**Escalation trigger:** >20% of categories in sample dataset runs 
are near-duplicate labels that the consolidation pass fails to merge

---

### R3 — Consolidation pass merges categories incorrectly

**Category:** Product  
**Description:** The final LLM consolidation pass auto-merges two 
categories that the user would consider distinct. Because merges 
are applied automatically for high-confidence cases, the user 
receives a CSV where categories have been consolidated without 
their explicit approval.  
**Probability:** Low — the one-liner-informed merge logic reduces 
the risk of merging on label similarity alone  
**Impact:** Medium — user must re-run or manually correct the output  
**Mitigation:** The transparency log surfaces every auto-merge 
explicitly: what was merged, into what, and how many tickets were 
affected. A user who disagrees with a merge can see exactly what 
happened and why. Prompt engineering for the consolidation pass 
will include explicit instructions to err on the side of flagging 
rather than merging when confidence is not high.  
**Escalation trigger:** Any merge of categories that share fewer 
than 50% of tickets from a similar root cause in sample dataset 
validation

---

### R4 — Processing time exceeds 90-second target

**Category:** Product  
**Description:** The 90-second target for a 100-row run is based 
on 10 API calls at 15 RPM with minimal latency per call. Under 
real conditions — network variability, Gemini API response time 
fluctuation, Streamlit Community Cloud compute constraints — 
actual run time may exceed this target.  
**Probability:** Medium  
**Impact:** Low — the app still works; user experience degrades  
**Mitigation:** Progress bar with batch count and ETA keeps the 
user informed throughout the run. Graceful batch failure handling 
means a slow batch does not block the overall run. If sustained 
latency is observed, batch size can be increased to 15 rows per 
call (reducing calls from 10 to 7) at the cost of slightly higher 
per-call token count.  
**Escalation trigger:** Median run time for 100-row sample datasets 
exceeds 120 seconds in pre-launch validation

---

### R5 — Streamlit Community Cloud deployment failure

**Category:** Execution  
**Description:** The app fails to deploy on Streamlit Community 
Cloud due to a dependency conflict, an incompatible package 
version, or a resource limit on the free tier.  
**Probability:** Low  
**Impact:** High — the shareable URL is the portfolio deliverable; 
deployment failure means no live demo  
**Mitigation:** Dependency list is minimal (5 packages). Versions 
will be pinned in `requirements.txt` after successful local testing 
in Milestone 1. A Streamlit Community Cloud test deployment will 
be run at Milestone 2 — well before the final launch milestone — 
to catch any environment issues early.  
**Escalation trigger:** Deployment fails after two remediation 
attempts at Milestone 2

---

### R6 — User uploads sensitive or confidential data

**Category:** Security  
**Description:** A user uploads a CSV containing personally 
identifiable information (PII), confidential business data, or 
other sensitive content. This data is sent to the Gemini API 
for processing.  
**Probability:** Medium — operational ticket data frequently 
contains customer names, transaction IDs, or internal system 
details  
**Impact:** Medium — data handling risk for the user; reputational 
risk for the project  
**Mitigation:** A clear data warning is shown on the upload screen 
before any file is processed: *"Do not upload files containing 
personal data, customer PII, or confidential business information. 
Data submitted to this tool is processed by the Google Gemini API 
and subject to Google's data handling policies."* The three built-in 
sample datasets are fictional and serve as the recommended demo 
path for first-time users.  
**Escalation trigger:** Any report of PII being inadvertently 
processed through the tool

---

### R7 — Low categorisation accuracy on real-world data

**Category:** Product  
**Description:** The ≥80% accuracy target is validated against the 
built-in sample datasets, which are curated to include clear, 
ambiguous, and multi-tag cases. Real-world ticket data from a user's 
own ops team may be messier, more domain-specific, or use 
terminology the LLM handles poorly — resulting in accuracy below 
the target on uploaded data.  
**Probability:** Medium  
**Impact:** Low to Medium — the tool is still useful for pattern 
detection at a directional level even if individual row accuracy 
is below 80%; the user can spot-check using the Low confidence 
filter  
**Mitigation:** The context description field (required) gives the 
LLM domain-specific grounding that significantly improves accuracy 
on unfamiliar data types. The Low confidence filter in the results 
table allows the user to identify and review uncertain rows before 
downloading. The uncategorised bucket analysis surfaces patterns 
in rows the LLM could not classify with confidence.  
**Escalation trigger:** Accuracy below 60% on any of the three 
built-in sample datasets in pre-launch validation

---

## Risk Summary

| Risk | Probability | Impact | Primary mitigation |
|------|------------|--------|-------------------|
| R1 — Free tier reduced | Low | Medium | One-line model switch to paid tier |
| R2 — Label inconsistency across batches | Medium | Medium | Accumulated category list in batch prompt + consolidation pass |
| R3 — Incorrect auto-merge | Low | Medium | Transparency log + conservative merge threshold in prompt |
| R4 — Processing time exceeds target | Medium | Low | Progress bar UX + batch size tuning option |
| R5 — Deployment failure | Low | High | Early deployment test at Milestone 2, pinned dependencies |
| R6 — Sensitive data uploaded | Medium | Medium | Pre-upload data warning, sample datasets as default path |
| R7 — Low accuracy on real-world data | Medium | Low–Medium | Context description field + Low confidence filter + uncategorised analysis |

No risks are currently rated High probability + High impact. The 
highest impact risk (R5 — deployment failure) is Low probability 
and is mitigated by early testing. The two Medium probability + 
Medium impact risks (R2, R6) both have structural mitigations built 
into the product design rather than relying on post-launch fixes.

---

*Previous: [04 — Tech Stack Decisions](./04_tech_stack_decisions.md)*  
*Next: [06 — Roadmap](./06_roadmap.md)*

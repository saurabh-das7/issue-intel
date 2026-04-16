# 04 — Tech Stack Decisions

*llm-issue-categorizer · Last updated: April 2026*

---

## Overview

Six decisions govern the technical foundation of this project. Each 
is documented here with the options evaluated, the decision made, 
and the rationale — including why the alternatives were rejected. 
These decisions are locked for v1. Revisiting them is only warranted 
if a constraint changes (cost, hosting, API availability) or a 
post-MVP feature makes the current choice untenable.

---

## Decision 1 — UI Framework

**The question:** What framework renders the user interface and 
orchestrates the application flow?

### Options evaluated

| Option | Pros | Cons |
|--------|------|------|
| Streamlit | Python-native, no frontend skills needed, tab/column layout built-in, one-command deploy to Community Cloud, large ecosystem of data components | Limited UI customisation compared to React; not suitable for production SaaS |
| Flask + HTML/CSS | Full control over UI | Requires frontend development, separate hosting, significantly higher build time |
| Gradio | Fast prototyping for ML demos | Less flexible layout control, designed for model demos not multi-step workflows |
| Dash | Good for data dashboards | More complex setup, heavier than needed for this use case |

### Decision: Streamlit

Streamlit is the only framework in this list where a solo Python 
developer can build a multi-step, file-upload workflow with a 
downloadable output and deploy it to a persistent public URL without 
writing a single line of HTML or managing any infrastructure. The 
progressive disclosure pattern (each step revealing the next) maps 
cleanly to Streamlit's top-to-bottom rendering model and session 
state. Build time saved by not writing frontend code goes directly 
into product quality.

**Why not Flask:** The UI complexity required — file upload, step 
progression, live progress indication, chart rendering, downloadable 
CSV — would require significant frontend work. Not justified for a 
solo portfolio project with a 2-week build target.

**Why not Gradio:** The multi-step workflow with distinct configuration 
stages, an editable taxonomy, and a results table is beyond Gradio's 
natural strengths. Gradio is well-suited to single-input / single-output 
model demos.

---

## Decision 2 — LLM / Categorisation Engine

**The question:** Which LLM API powers the ticket categorisation, 
consolidation pass, and pattern summary?

### Options evaluated

| Option | Cost | Rate limit | Quality | Notes |
|--------|------|-----------|---------|-------|
| Google Gemini 2.5 Flash-Lite (free tier) | ₹0 | 1,000 req/day, 15 RPM | Good for structured classification | No credit card required |
| Google Gemini 2.5 Flash (free tier) | ₹0 | 500 req/day, 10 RPM | Better reasoning, slower | Lower daily limit, tighter RPM |
| Anthropic Claude (API) | Pay per token | No free tier | Excellent | Requires credit card; separate from Claude Pro subscription |
| OpenAI GPT-4o mini | Pay per token | No meaningful free tier | Good | Requires credit card |
| Ollama (local LLM) | ₹0 | Unlimited | Variable | Requires local GPU; not deployable to Streamlit Cloud |

### Decision: Google Gemini 2.5 Flash-Lite (free tier)

The cost constraint for this project is ₹0 per month. Gemini 
2.5 Flash-Lite is the only option that meets this constraint 
while providing sufficient reasoning quality for structured 
classification tasks and being deployable to Streamlit Community 
Cloud via an API key in Streamlit secrets.

At 10 rows per batch, a 100-row run requires 10 categorisation 
calls plus 1 consolidation call — 11 calls total per session. 
At the 15 RPM free tier limit, this fits comfortably within a 
single minute. The 1,000 req/day limit provides headroom for 
multiple test runs and live demo usage without approaching the cap.

**Why not Gemini 2.5 Flash:** Better reasoning quality, but 10 RPM 
and 500 req/day is tighter. For a 100-row run, 10 RPM adds 
meaningful latency. Flash-Lite is sufficient for classification 
tasks that have a well-defined rubric in the prompt — the quality 
gap does not justify the tighter limits.

**Why not Claude API:** Excellent quality but requires a credit card 
and pay-per-token billing. The project's ₹0 cost constraint makes 
this a non-starter for v1. Claude Pro subscription (used for 
building and iterating in chat) does not cover API inference costs 
in deployed applications.

**Why not Ollama:** Local models cannot be deployed to Streamlit 
Community Cloud. A public shareable URL is a hard requirement — 
a locally-run app is not a portfolio asset.

---

## Decision 3 — Batch Size

**The question:** How many rows should be sent to the LLM in each 
API call?

### Options evaluated

| Batch size | API calls for 100 rows | Consistency risk | Latency at 15 RPM |
|-----------|----------------------|-----------------|-------------------|
| 1 row / call | 100 calls | High — no cross-row context | ~7 minutes |
| 5 rows / call | 20 calls | Medium | ~90 seconds |
| 10 rows / call | 10 calls | Low — LLM sees category list from prior batch | ~45 seconds |
| 20 rows / call | 5 calls | Lower still | ~25 seconds |
| 50 rows / call | 2 calls | Best | Large prompt, token cost risk |

### Decision: 10 rows per batch

10 rows per batch is the point where three constraints are 
simultaneously satisfied: latency is within the 90-second target 
(10 calls at 15 RPM = well under 60 seconds with room for the 
consolidation pass), category label consistency is enforced by 
passing the accumulated category list into each subsequent batch 
prompt, and token cost per call stays well within free tier limits.

Each batch prompt includes: the data context description, the full 
accumulated category list from previous batches, the 10 rows to 
classify, and the output schema. At ~500 tokens per batch prompt 
and 10 batches, total input tokens per run is approximately 5,000 
— a negligible fraction of Gemini free tier token limits.

**Batch size may be revisited** if testing in Milestone 1 reveals 
that 10-row batches produce inconsistent category naming. In that 
case, the first option is to reduce to 5 rows and accept longer 
runtime before considering architectural changes.

---

## Decision 4 — Data Handling: CSV / Excel / TXT Parsing

**The question:** Which libraries handle file parsing for the three 
supported input formats?

### Decision: Pandas + openpyxl

| Format | Library | Why |
|--------|---------|-----|
| CSV | `pandas.read_csv()` | Standard, robust, handles encoding edge cases |
| Excel (.xlsx) | `pandas.read_excel()` via `openpyxl` | Reads .xlsx without requiring Excel installed |
| TXT (pipe-delimited) | `pandas.read_csv(sep='|')` | Same API as CSV, just a different separator |

All three input formats are normalised into a single Pandas 
DataFrame immediately after parsing. Everything downstream — 
validation, batching, output assembly — operates on the DataFrame. 
This means format-specific code is limited to the three read calls 
at the entry point; the rest of the application is format-agnostic.

Column name normalisation (lowercase, strip whitespace) is applied 
at parse time so the rest of the application can assume clean column 
names regardless of how the user formatted their file.

**Why not csv module (stdlib):** Pandas adds negligible overhead 
and provides substantially better handling of encoding issues, 
malformed rows, and mixed data types. The project already needs 
Pandas for DataFrame operations — adding it for parsing too 
eliminates a dependency boundary.

---

## Decision 5 — Chart Rendering

**The question:** Which library renders the category distribution 
bar chart?

### Options evaluated

| Option | Interactivity | Streamlit integration | Complexity |
|--------|-------------|----------------------|-----------|
| Plotly | Hover tooltips, zoom | `st.plotly_chart()` — native | Low |
| Altair | Good interactivity | `st.altair_chart()` — native | Medium |
| Matplotlib | Static only | `st.pyplot()` — works | Low |
| Streamlit native `st.bar_chart()` | None | Native, no config | Too limited |

### Decision: Plotly

Plotly is the right choice because hover state is a functional 
requirement — hovering a bar shows the exact ticket count and 
percentage, which is more precise than reading bar labels for 
categories with similar volumes. Plotly's `st.plotly_chart()` 
integration is seamless, the horizontal bar chart is one function 
call with a `go.Bar` trace, and no additional configuration is 
needed for the Streamlit Community Cloud deployment.

**Why not Altair:** Altair requires a different mental model 
(Vega-Lite grammar) and produces equivalent output for this use 
case. Plotly is more widely documented for Streamlit and the 
horizontal bar chart pattern is simpler to implement.

**Why not Matplotlib:** Static only. No hover state. The chart 
would require printed labels for every bar to be usable, which 
becomes crowded with 7–8 categories.

---

## Decision 6 — Hosting and Deployment

**The question:** Where does the app live after build?

### Options evaluated

| Option | Cost | Persistent URL | Auto-deploy from GitHub | Setup effort |
|--------|------|---------------|------------------------|-------------|
| Streamlit Community Cloud | Free | Yes | Yes | Minimal |
| Railway | Free tier available | Yes | Yes | Low |
| Render | Free tier available | Yes | Yes | Low |
| Hugging Face Spaces | Free | Yes | Yes | Low, but Gradio-first |
| Self-hosted (VPS) | ~₹500–1000/month | Yes | Manual | High |

### Decision: Streamlit Community Cloud

Streamlit Community Cloud is the natural deployment target for a 
Streamlit app — no configuration files, no Docker, no server 
management. Connect the GitHub repo, point it at `app/main.py`, 
add the Gemini API key in the Streamlit secrets manager, and the 
app is live at a persistent public URL. Auto-deploys on every push 
to main.

The free tier supports one public app with sufficient compute for 
this use case. The persistent URL is shareable directly with hiring 
managers — a link that opens a live working tool is the intended 
portfolio outcome.

**Why not Railway or Render:** Both are valid alternatives but add 
configuration overhead (environment files, build commands, port 
settings) that Streamlit Community Cloud eliminates entirely. No 
meaningful benefit for this use case.

---

## Dependency List (v1)

```
streamlit
pandas
openpyxl
plotly
google-generativeai
```

Five dependencies. No pinned versions at this stage — pinning 
happens in `requirements.txt` after Milestone 1 confirms 
compatibility across all five packages on Streamlit Community Cloud.

---

## Locked Decisions — Do Not Revisit Unless a Constraint Changes

| Decision | Locked choice | What would reopen it |
|----------|-------------|---------------------|
| UI framework | Streamlit | Post-MVP need for real-time updates or complex UI state |
| LLM engine | Gemini 2.5 Flash-Lite free tier | Gemini free tier discontinued or quality proven insufficient in M1 |
| Batch size | 10 rows | M1 testing shows unacceptable label inconsistency |
| File parsing | Pandas + openpyxl | Addition of a new input format not supported by this stack |
| Chart library | Plotly | Post-MVP drill-down interaction requiring a different rendering model |
| Hosting | Streamlit Community Cloud | Paid tier needed for compute or uptime guarantees |

---

*Previous: [03 — UX Flow & Wireframe](./03_ux_flow_wireframe.md)*  
*Next: [05 — Risk & Cost Plan](./05_risk_and_cost.md)*

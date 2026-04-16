# llm-issue-categorizer

A practical, PM-driven tool for turning noisy operational issue data into structured, 
actionable product intelligence — built from real experience managing vendor operations 
where ticket labels are vague, inconsistent, and useless for pattern detection.

This repo documents both the **thinking** (frameworks, PRDs, risk registers) and the 
**doing** (a working tool you can run on your own data).

---

## Current Tool — Issue Categorisation & Pattern Detection

> **Your vendor team resolves hundreds of issues every week. Your ticket labels tell you nothing.**

The Issue Categorisation & Pattern Detection tool is a Streamlit web application that 
takes a CSV of operational issues — tagged with vague, self-defined labels — and 
re-categorises them against a structured taxonomy using an LLM, then surfaces the 
patterns a PM actually needs to make product decisions.

### Three Outputs From One Upload

**Categorised CSV**
Upload your ticket data. Define your category taxonomy — or let the app suggest one 
based on scanning your data. Get back a clean CSV with a new category label, a 
confidence score, and a one-line reasoning note for every row. Every categorisation 
is auditable, not a black box.

**Pattern Summary**
Top categories by volume. Fastest growing issue types. Top product improvement 
opportunities surfaced automatically from the category distribution. The kind of 
analysis a PM would spend a week doing manually — generated in under 60 seconds.

**Visual Breakdown**
A bar chart of issue distribution by category. Instantly shareable. Immediately 
understandable by a stakeholder who has never seen the raw data.

Every flow is zero-friction — a first-time user can experience the full output 
by uploading the built-in sample dataset with one click.

### What Makes It Smart

The taxonomy is not hardcoded. The app supports two modes:

**Bring your own taxonomy** — you define the category names upfront, the app maps 
every ticket to them with a confidence score.

**Auto-suggest mode** — the app scans a sample of your tickets first, proposes a 
taxonomy based on what it finds, you approve or edit the categories, then the full 
categorisation run begins.

This mirrors how a real PM approaches the problem. You don't know your categories 
until you've seen your data.

**Status:** 🔨 Actively building — follow along via the [build log](./docs/07_build_log.md)

**Live demo:** *(link added at launch)*

---

## Why This Exists

Vendor teams and support operations resolve hundreds of issues every week. But the 
labels they assign — "quality issue", "performance problem", "ad not serving" — 
are written for resolution speed, not for analysis. The result is a graveyard of 
ticket data that no one mines.

Product improvements that should be obvious stay invisible because the signal is 
buried in noise. A PM inheriting this data cannot tell whether they have a 
systematic infrastructure problem, a training gap in the vendor team, or a product 
bug driving the same failure mode repeatedly — because the labels do not distinguish 
between them.

This tool is the missing layer between raw operational data and product decisions: 
consistent categorisation at scale, with the reasoning surfaced alongside every label.

Full problem framing: [docs/01_problem_statement.md](./docs/01_problem_statement.md)

---

## How to Run This Locally

**Requirements:** Python 3.9+, a free API key from [console.anthropic.com](https://console.anthropic.com) 
(no credit card required for free tier)

```bash
# Clone the repo
git clone https://github.com/saurabh-das7/llm-issue-categorizer.git
cd llm-issue-categorizer

# Install dependencies
pip install -r requirements.txt

# Add your API key
export ANTHROPIC_API_KEY=your_key_here

# Run the app
streamlit run app/main.py
```

*Full setup walkthrough and environment notes will be added here as the build progresses.*

---

## Repo Structure

```
llm-issue-categorizer/
├── README.md
├── app/                                  # Streamlit application (coming build stage)
│   ├── main.py
│   └── samples.py                        # Built-in sample ticket dataset
├── docs/                                 # PM documentation — in progress
│   ├── 01_problem_statement.md      ⏳
│   ├── 02a_prd.md                   ⏳
│   ├── 02b_sample_data_bank.md      ⏳
│   ├── 03_ux_flow_wireframe.md      ⏳
│   ├── 04_tech_stack_decisions.md   ⏳
│   ├── 05_risk_and_cost.md          ⏳
│   ├── 06_roadmap.md                ⏳
│   ├── 07_build_log.md              ⏳
│   └── 08_launch_and_retro.md       ⏳
└── requirements.txt                      # Python dependencies (coming build stage)
```

---

## The PM Documentation

Every stage of this build is documented the way a PM would approach it at work — 
problem framing, requirements, UX flows, risk registers, cost plans, and retrospectives.

| Doc | What it covers | Status |
|-----|---------------|--------|
| [01 — Problem Statement](./docs/01_problem_statement.md) | Why this exists, who it's for, cost of the problem | ⏳ Upcoming |
| [02a — PRD](./docs/02a_prd.md) | Two modes, categorisation engine, output specs, success metrics | ⏳ Upcoming |
| [02b — Sample Data Bank](./docs/02b_sample_data_bank.md) | ~30 realistic operational tickets with expected re-categorisation | ⏳ Upcoming |
| [03 — UX Flow & Wireframe](./docs/03_ux_flow_wireframe.md) | Interface structure, user journeys, interaction design | ⏳ Upcoming |
| [04 — Tech Stack Decisions](./docs/04_tech_stack_decisions.md) | Tools evaluated and chosen, with rationale | ⏳ Upcoming |
| [05 — Risk & Cost Plan](./docs/05_risk_and_cost.md) | What could go wrong, API cost breakdown | ⏳ Upcoming |
| [06 — Roadmap](./docs/06_roadmap.md) | Milestone plan from setup to live URL | ⏳ Upcoming |
| [07 — Build Log](./docs/07_build_log.md) | Running journal of what was built and learned | ⏳ Upcoming |
| [08 — Launch & Retrospective](./docs/08_launch_and_retro.md) | Pre-launch QA checklist, post-launch reflection | ⏳ Upcoming |

---

## Who This Is For

**If you manage vendor operations, support queues, or customer feedback pipelines** 
— this tool replaces manual re-labelling and pattern-spotting with a consistent, 
auditable, LLM-powered categorisation layer you can run on any CSV.

**If you're a PM or TPM building AI products** — the docs folder is a worked example 
of how to frame, spec, and ship an LLM-powered tool from scratch, including the 
thinking behind every product and technical decision.

**If you're exploring how to use LLMs for structured data classification** — the 
taxonomy design, confidence scoring, and two-mode input model are directly 
transferable to any domain where you need LLM-powered labelling at scale.

---

## About This Project

Built by [Saurabh Das](https://linkedin.com/in/saurabhdas7) — Senior TPM and 
Designated PM at Microsoft AI, documenting an AI learning journey in public.

Background: I've spent years managing vendor operations where the difference between 
a solvable product problem and invisible noise was entirely determined by the quality 
of the ticket data. This tool is the thing I kept wishing existed — structured, 
consistent categorisation that surfaces what the raw labels never could.

Related tools: [LLM Eval Toolkit](https://github.com/saurabh-das7/llm-eval-toolkit) · 
[PM & TPM Playbooks](https://github.com/saurabh-das7/pm-tpm-playbooks)

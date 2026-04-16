# 01 — Problem Statement

*llm-issue-categorizer · Last updated: April 2026*

---

## The Problem

Every team that runs vendor operations, a support queue, or a customer feedback 
pipeline has the same data problem. Hundreds of issues are logged and resolved 
every week. Every one of them gets a label. And almost none of those labels are 
useful for anything other than closing the ticket.

"Performance issue — resolved." "Quality problem — escalated and fixed." 
"Serving issue — investigated." These labels are written for resolution speed, 
not for analysis. The person logging the ticket is optimising for moving on to 
the next one, not for giving a future PM a clean signal about what's breaking 
and how often.

The result is a graveyard of ticket data that no one mines. Not because the 
data isn't valuable — it is — but because the labels are too vague to aggregate 
meaningfully. A PM inheriting six months of this data cannot tell whether they 
have a systematic infrastructure problem, a training gap in the vendor team, a 
product bug surfacing repeatedly in slightly different forms, or all three. The 
labels do not distinguish between them. Pattern detection is impossible when 
every pattern is obscured by the same handful of catch-all tags.

The problem compounds with scale. A team resolving 50 tickets a week might 
manually re-examine the data quarterly. A team resolving 500 cannot. At volume, 
the graveyard grows faster than any PM can dig through it manually, and the 
product improvements that should be obvious — the systematic fixes that would 
reduce ticket volume at the root — stay invisible.

---

## Who Feels This Acutely

The primary user is a **PM or TPM who owns a vendor operations or support 
function** at a mid-to-large tech company — someone responsible not just for 
keeping the queue moving, but for identifying what the queue is telling them 
about the product.

More specifically: they manage a team of 4–10 people (direct or vendor) who 
log and resolve issues daily. They have inherited a ticket system with months 
or years of historical data. They know the data is valuable. They do not have 
a data scientist embedded in their team to clean and categorise it for them. 
And they have been burned enough times by "let me check the tickets" rabbit 
holes that they have stopped looking.

They are not asking for a BI dashboard. They are not asking for a data 
warehouse. They are asking for one honest answer: *what is actually breaking, 
and how often?*

This user exists at every company with a vendor ops function — Ad Tech 
platforms, marketplace operations, customer support teams, content moderation 
pipelines, logistics and fulfilment ops. The problem is not domain-specific. 
The ticket label quality problem is universal.

---

## The Cost of the Problem

The cost operates at two levels.

**Immediate cost — wasted PM time.** A PM who wants to understand issue 
patterns from raw ticket data has to read through them manually, impose 
a mental taxonomy, tally counts, and write up findings — a process that 
takes days for any meaningful dataset and produces results that are 
non-reproducible, non-transferable, and stale the moment the next week's 
tickets come in. This is time that should be spent on product decisions, 
not data janitorial work.

**Structural cost — invisible product debt.** The more damaging cost is 
what never gets found. When issue labels are too vague to surface patterns, 
systematic product failures stay hidden. A bug that surfaces in 40 different 
tickets as "performance issue" never gets triaged as a product priority 
because no one ever sees it as 40 instances of the same thing — they see 
40 individual resolutions. The product never gets fixed. The tickets keep 
coming. The vendor team keeps absorbing the load. The PM never connects 
the operational signal to the product backlog.

This is not a hypothetical failure mode. It is the default state for any 
operations function that has not deliberately invested in ticket taxonomy 
design — which is most of them.

---

## Why Now

Two things have changed that make this problem newly tractable.

**LLMs can read and reason about unstructured text at scale.** Re-categorising 
a ticket from a vague vendor label into a structured taxonomy is exactly the 
kind of task that would have required a trained classifier, a large labelled 
dataset, and a data scientist to build and maintain it — a resource most ops 
PMs do not have. An LLM can do this with nothing more than a well-designed 
prompt and a taxonomy definition. The accuracy is good enough to surface 
meaningful patterns, which is all a PM actually needs.

**The cost of doing this has dropped to zero.** Free-tier LLM APIs — 
specifically Google Gemini 2.5 Flash-Lite — provide enough capacity for 
the categorisation workloads a typical ops team generates, with no credit 
card required and no monthly bill. A solution that was previously gated 
behind a data science engagement is now within reach of any PM with a 
CSV export and an hour to spare.

The combination of LLM reasoning capability and zero infrastructure cost 
means the only remaining barrier is the tool — and that is what this 
project builds.

---

## What This Is Not

This is not a ticket management system. It does not replace Jira, Zendesk, 
or any existing ops tooling. It sits downstream of whatever system the team 
already uses — it takes a CSV export and returns structured intelligence. 
No integrations, no APIs, no workflow changes required.

This is not a real-time monitoring tool. It is a batch analysis tool — 
designed for periodic review (weekly, monthly, quarterly) of accumulated 
ticket data, not live incident response.

This is not a replacement for a data scientist. It surfaces patterns 
accurately enough to inform product decisions. It does not replace 
rigorous statistical analysis for cases where that bar matters.

---

*Next: [02a — PRD](./02a_prd.md)*

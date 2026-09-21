# iGrow Agents

The standalone AI outreach agent system for **iGrow** — *"the flight simulator for the
conversations that decide your career."* Users rehearse high-stakes conversations out loud
with an AI that pushes back and scores the **substance** of what they said (the ANCHOR
framework), then get a scored report. Records are portable and owned by the individual.

This repo is self-contained and independent: it is not tied to any other project, website,
or deploy pipeline. It holds the six agents, their runnable implementations, the pipeline
that connects them, and the reference data they use.

## The pipeline

```
[1] Lead Scraper & Daily Updater
        │  routes each lead by channel
        ├───────────────┬───────────────┐
[2] University Leads   [3] Partnership Leads   (run in parallel)
        └───────────────┴───────────────┘
                        │  verified, ready-for-email
[4] Email Personalization
                        │  drafted email
[5] QA/QC Check   ← mandatory, non-skippable gate
                        │  approved only
[6] Email Outreach
```

Every stage reads and writes its own Airtable table in the base **iGrow Growth — 90-Day**
(`appREB7S8JsGgk0PZ`), and hands only qualified records to the next stage.

## The 6 agents

### 1. Lead Scraper & Daily Updater
- **Work:** The single front door for every lead. Runs daily to discover leads from Apollo,
  Clay, LinkedIn, GummySearch, Awario, conference lists, competitor audiences, and manual
  research; dedupes; scores each on a 1–10 Fit Score; routes to exactly one channel.
- **Function:** Enforce one intake standard so no lead is lost and no channel agent wastes
  time on bad-fit prospects. Output: a scored, routed lead record.
- **Airtable:** Lead Scraper Inbox (`tblhgCknvXTuyljiv`)
- Definition: [`agents/1-lead-scraper-daily-updater.md`](agents/1-lead-scraper-daily-updater.md) · Code: [`implementations/1-lead-scraper.js`](implementations/1-lead-scraper.js)

### 2. University Leads
- **Work:** Owns the university channel. Verifies contacts, enriches with program-specific
  context (which office, what they measure), assigns priority (P1/P2/P3), and excludes
  bad-fit institutions (R1/Ivy/flagship/>15k enrollment).
- **Function:** Turn a raw university lead into a verified, prioritized, ready-for-email
  record with a documented rationale. Small colleges (500–10k), career services / co-op /
  leadership / student affairs / Greek life / alumni.
- **Airtable:** University Leads (`tblKfBKccmofehC9u`)
- Definition: [`agents/2-university-leads.md`](agents/2-university-leads.md) · Code: [`implementations/2-university-leads.js`](implementations/2-university-leads.js)

### 3. Partnership Leads
- **Work:** Owns the consultant/coaching-firm channel. Verifies contacts, researches each
  firm's methodology, assigns a weighted Fit Score (methodology 30% / client overlap 30% /
  firm size 20% / tech-forward 20%), classifies segment and hook.
- **Function:** Qualify coaching firms as a distribution channel (iGrow = between-sessions
  practice for their clients). Only Tier A/B advance. Output: verified, scored lead + hook.
- **Airtable:** Partnership Leads (`tblAcv3IdVM62WwyB`)
- Definition: [`agents/3-partnership-leads.md`](agents/3-partnership-leads.md) · Code: [`implementations/3-partnership-leads.js`](implementations/3-partnership-leads.js)

### 4. Email Personalization
- **Work:** The single email-writing door. Takes verified leads and composes personalized,
  anti-spam-compliant emails (under 200 words, one CTA, research-first hook) in 4-touch
  sequences, with a competitor-aware differentiation angle when relevant.
- **Function:** Consistent voice and quality across every channel; one place to tune reply
  rates. Output: subject + body per touch, marked ready for QA.
- **Airtable:** Email Personalization (`tblLwgWPmHW0UVQQ9`)
- Definition: [`agents/4-email-personalization.md`](agents/4-email-personalization.md) · Code: [`implementations/4-email-personalization.js`](implementations/4-email-personalization.js)

### 5. QA/QC Check
- **Work:** The last line before send. Reviews every draft against the anti-spam and quality
  rules, tone, and factual accuracy; approves or rejects with reasons.
- **Function:** Nothing reaches a real inbox without passing this gate. Rejected drafts go
  back to Personalization; only approved drafts advance. Output: pass/fail + feedback.
- **Airtable:** QA/QC Reviews (`tblitpaJ9ixPXQNjg`)
- Definition: [`agents/5-qaqc-check.md`](agents/5-qaqc-check.md) · Code: [`implementations/5-qaqc-check.js`](implementations/5-qaqc-check.js)

### 6. Email Outreach
- **Work:** Sends only QA-approved emails, on schedule, respecting per-channel timing rules,
  and records send/response status.
- **Function:** Reliable delivery and status tracking for the approved queue — the only agent
  that actually sends. Output: sent record + outcome.
- **Airtable:** Email Outreach Queue (`tblEfgRhtmx9D6nS0`)
- Definition: [`agents/6-email-outreach.md`](agents/6-email-outreach.md) · Code: [`implementations/6-email-outreach.js`](implementations/6-email-outreach.js)

## Repo layout

| Path | What's in it |
|---|---|
| `agents/` | The 6 agent definitions — the "why", identity, mission, Airtable, rules |
| `implementations/` | The 6 runnable Node.js (OpenAI function-calling) implementations |
| `pipeline/PIPELINE.md` | The fixed-sequence conductor that runs the 6 agents in order |
| `reference/competitor-intel.md` | 55 verified competitor/adjacent companies (lead source + differentiation) |
| `docs/positioning.md` | iGrow positioning, ICP, unit economics, anti-spam rules |

## Running an implementation

```bash
cd implementations
npm install          # openai, dotenv
cp .env.example .env # add OPENAI_API_KEY
node 1-lead-scraper.js
```

## Orchestration

The pipeline can also be driven by the `run-company` orchestrator, which runs these agents
under `/run-company`. This repo is the source of truth for the agents; run-company references
it. See `pipeline/PIPELINE.md`.

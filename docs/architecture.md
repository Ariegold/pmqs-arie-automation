# System Architecture — PMQs Intelligence System

## Overview

Three n8n workflows + one front-end + one Google Sheet operating as a unified intelligence pipeline for No.10 PMQs preparation.

---

## Component Map

```
┌─────────────────────────────────────────────────────────┐
│                   FRONT-END (GitHub Pages)               │
│              index.html — PMQs Briefing UI               │
│         POST → n8n webhook on question submit            │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              n8n Cloud (arie08.app.n8n.cloud)            │
│                                                         │
│  Workflow A: Live Briefing (webhook)                    │
│  POST /pmqs-brief → Tavily → GPT-4.1 → response        │
│                                                         │
│  Workflow B: Hansard Pipeline (Wednesday 14:00)         │
│  Parliament API → classify → extract → FOI flag         │
│                                                         │
│  Workflow C: Batch Answers (Tuesday 09:00)              │
│  Sheets read → Tavily → GPT-4.1 → Sheets write          │
└──────────┬─────────────────────┬───────────────────────┘
           │                     │
           ▼                     ▼
┌──────────────────┐   ┌─────────────────────────────────┐
│   Tavily API     │   │        Google Sheets             │
│  Web research    │   │  📥 Questions (MP intake)        │
│  6 UK sources    │   │  🤖 AI Answers (GPT output)      │
│  per query       │   │  ⚑  FOI Tracker                 │
└──────────────────┘   │  📊 Session Log                 │
                       └─────────────────────────────────┘
           │
           ▼
┌──────────────────┐
│   OpenAI GPT-4.1 │
│  Briefing drafts │
│  Structured JSON │
└──────────────────┘
```

---

## Workflow A — Live Briefing Webhook

**Trigger:** HTTP POST from front-end form  
**Path:** `/pmqs-brief`  
**Response time:** ~8–12 seconds

### Node sequence
1. **Webhook** — receives `{ question, mp_name, constituency, party, domain, priority }`
2. **Tavily** — searches UK sources, returns 6 results with content
3. **Prepare Context** — formats Tavily results into readable prompt context
4. **GPT-4.1** — generates structured briefing (temperature 0.3)
5. **Format Response** — extracts text from OpenAI response shape
6. **Respond to Webhook** — returns `{ output: "..." }` with CORS headers

### Output format (parsed by front-end)
```
THREAT ASSESSMENT
[text]

SUGGESTED PM RESPONSE
[text]

LINES TO TAKE
- item
- item

LINES TO AVOID
- item
- item

KEY FACTS
- item with source
- item with source

CONFIDENCE SCORE: [0-100]
FOI RISK: [High|Medium|Low] — [explanation]
DOMAIN: [domain]

SOURCES
- title — url
- title — url
```

---

## Workflow B — Hansard Auto-Ingest

**Trigger:** Schedule — Wednesday 14:00 (Cron: `0 14 * * 3`)  
**Purpose:** Auto-ingests PMQs transcript after each session

### Node sequence
1. **Schedule Trigger** — fires Wednesday 14:00
2. **Hansard API** — fetches today's debate transcript
3. **Parse & Split** — extracts exchanges by speaker
4. **Claude** — domain classification per exchange
5. **Claude** — figure extraction + confidence scoring
6. **Claude** — FOI risk assessment
7. **IF node** — routes high-risk FOI to immediate alert
8. **Google Sheets** — appends to session log
9. **Gmail** — sends digest (high risk only)

---

## Workflow C — Batch Answer Generator

**Trigger:** Schedule — Tuesday 09:00 (Cron: `0 9 * * 2`)  
**Purpose:** Generates draft answers for all submitted questions

### Node sequence
1. **Schedule / Manual Trigger**
2. **Google Sheets Read** — reads Questions tab (unanswered rows)
3. **Loop** — processes one question at a time
4. **Tavily** — researches each question topic
5. **GPT-4.1** — drafts PM-style answer
6. **Format** — structures output for Sheets
7. **Google Sheets Write** — appends to Answers tab

---

## Data Dictionary

### Questions tab (Google Sheets)
| Column | Type | Source |
|--------|------|--------|
| question_id | Auto | Formula |
| date_received | Date | Manual / Parliament API |
| mp_name | Text | Manual / Parliament API |
| constituency | Text | Manual / Parliament API |
| party | Dropdown | Manual |
| question_text | Text | Manual / Parliament API |
| domain | Dropdown | Manual / Workflow B |
| priority | Dropdown | Manual |
| source | Dropdown | Manual |
| session_date | Date | Manual |
| answered | Yes/No | Workflow C |

### Answers tab (Google Sheets)
| Column | Type | Source |
|--------|------|--------|
| date_run | Date | Workflow C |
| mp_name | Text | From Questions |
| suggested_answer | Text | GPT-4.1 |
| key_facts | Text | GPT-4.1 |
| confidence_score | Number | GPT-4.1 |
| sources | Text | Tavily + GPT |
| risk_flag | Text | GPT-4.1 |
| reviewed | Checkbox | Human |

---

## Security Notes

- Repo is **private** — no credentials stored in code
- API keys stored in n8n credential store only
- Front-end makes no direct API calls — all routed through n8n
- CORS headers set to `*` — restrict to specific domain in production
- Classification banner: RESTRICTED — INTERNAL USE ONLY

# PMQs Intelligence System
### No.10 Downing Street · Cabinet Office Internal Prototype

> AI-assisted briefing system for Prime Minister's Questions. Powered by GPT-4.1 + Tavily web search. Built and maintained by the No.10 Digital BA team.

---

## ⚠ Classification

**RESTRICTED — INTERNAL USE ONLY**
All outputs must be verified by a senior policy adviser before use at the despatch box.

---

## What This Is

A real-time intelligence briefing tool that:

1. Accepts an MP question submitted ahead of Wednesday PMQs
2. Researches the topic using Tavily web search (gov.uk, parliament.uk, ONS, BBC, FT)
3. Passes research to GPT-4.1 with a senior briefing officer system prompt
4. Returns a structured brief including:
   - Threat assessment
   - Suggested PM response
   - Lines to take / lines to avoid
   - Key facts with sources
   - Confidence score
   - FOI risk rating

---

## Repository Structure

```
pmqs-intelligence/
│
├── index.html                          # Front-end — PMQs briefing UI
│
├── workflows/
│   └── n8n-pmqs-webhook-workflow.json  # n8n automation workflow (backup)
│
├── docs/
│   └── architecture.md                 # System architecture documentation
│
└── .github/
    └── workflows/
        └── deploy.yml                  # GitHub Actions — auto-deploy to Pages
```

---

## Live URL

Hosted on GitHub Pages:
```
https://YOUR-GITHUB-USERNAME.github.io/pmqs-intelligence/
```

---

## n8n Workflow Architecture

```
Webhook POST /pmqs-brief
    ↓
Tavily Search API (6 results, UK sources)
    ↓
Prepare Context (Code Node)
    ↓
GPT-4.1 (structured briefing output)
    ↓
Format Response (Code Node)
    ↓
Respond to Webhook → front-end
```

**Webhook URL:** `https://arie08.app.n8n.cloud/webhook/pmqs-brief`

---

## Setup

### Prerequisites
- n8n Cloud account (app.n8n.cloud)
- Tavily API key (app.tavily.com)
- OpenAI API key with GPT-4.1 access (platform.openai.com)

### Deploy n8n Workflow
1. Open n8n Cloud
2. Workflows → ⋯ → Import from file
3. Select `workflows/n8n-pmqs-webhook-workflow.json`
4. Add credentials: Tavily API key, OpenAI API key
5. Toggle **Active**

### Deploy Front-end
Push to `main` branch — GitHub Actions deploys automatically to Pages.

---

## Data Flow

| Step | Tool | Purpose |
|------|------|---------|
| Trigger | Webhook (n8n) | Receives question from front-end |
| Research | Tavily API | Web search across UK government sources |
| Briefing | GPT-4.1 | Generates structured PM brief |
| Output | Webhook response | Returns JSON to front-end |
| Logging | Google Sheets | Session history, confidence scores, FOI tracker |

---

## Related Workflows

| File | Purpose |
|------|---------|
| `n8n-pmqs-webhook-workflow.json` | Live briefing — this repo |
| `n8n-workflow1-pmqs-intelligence.json` | Hansard auto-ingest (Wednesday pipeline) |
| `n8n-workflow2-pm-answer-generator.json` | Batch answer generation (Tuesday schedule) |

---

## Cost Estimate

| Service | Usage | Approx cost |
|---------|-------|-------------|
| GPT-4.1 | ~1,200 tokens/brief | ~£0.003/brief |
| Tavily | 6 searches/brief | Free tier: 1,000/month |
| GitHub Pages | Static hosting | Free |
| n8n Cloud | Workflow execution | Included in plan |

A full PMQs session (20 questions) costs approximately **6p** in API calls.

---

## Changelog

| Date | Change |
|------|--------|
| 26 Feb 2025 | Initial deployment — webhook + Tavily + GPT-4.1 |
| 26 Feb 2025 | Google Sheets integration added (4-tab structure) |
| 26 Feb 2025 | GitHub Pages deployment configured |

---

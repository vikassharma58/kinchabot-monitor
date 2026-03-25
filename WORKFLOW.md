# Kincha Bot — How It Works
### A Plain-English Guide for Non-Engineers

**Client:** Pfizer (Pilot) | **Owner:** vikas@atacana.com | **Updated:** 2026-03-25

---

## What Is Kincha?

Kincha is a dedicated research assistant for Pfizer's strategy team. Instead of someone manually reading pharma news every day, Kincha does it automatically and sends you only what matters — enriched with competitive context.

---

## The Memory System (Two Databases)

Before Kincha can do anything useful, it needs background knowledge. Think of it like hiring a researcher who has already read every pharma report:

### 📦 Supabase — The Filing Cabinet
Stores everything as searchable text:
- All past findings (news alerts)
- Drug profiles for 77 drugs across 15 companies
- SWOT profiles for 30 companies
- Full Pharma Competitive Intelligence Report 2026 (22 sections)
- Audit log of every pipeline run (pipeline_logs)

When Kincha finds news, it saves it here. It can also search through old news to say *"we've seen something like this before"* — this is called RAG (Retrieval-Augmented Generation).

### 🕸️ Neo4j — The Relationship Map
Like a mind map pinned to a wall. Shows connections between companies, drugs, and diseases:
- AbbVie → makes → Skyrizi → treats → Plaque Psoriasis
- Skyrizi → competes with → Pfizer drugs
- Sanofi → acquired rights to → KT501 (T-cell engager for RA)

When Kincha finds news about AbbVie, it instantly knows: *"AbbVie is Tier 1, their Skyrizi makes $11.7B/year, and it directly threatens Pfizer in Psoriasis."*

**Current graph size:** 27 companies, 44 drugs, 93 drug-indication relationships + 31 competitive relationships (tiers, revenues, acquisitions, partnerships, threats)

---

## Where Kincha Looks for News

Twice a day (6PM and 8PM IST), Kincha checks **3 free sources:**

| Source | What It Checks | Example Finding |
|--------|---------------|-----------------|
| **ClinicalTrials.gov API** | New trials registered today by our 15 companies | "AbbVie just registered a Phase 3 trial for Rinvoq in Crohn's" |
| **FDA website search** | Drug approvals, filings, safety alerts | "J&J's Icotyde approved for Plaque Psoriasis" |
| **EMA website search** | European approvals + CHMP committee opinions | "Novartis Cosentyx recommended for UC" |

**15 monitored companies:** AbbVie, J&J, BMS, Eli Lilly, Sanofi, Novartis, AstraZeneca, Roche, Amgen, GSK, Takeda, Gilead, Regeneron, Boehringer Ingelheim, Merck

**6 monitored indications:** Plaque Psoriasis, Psoriatic Arthritis, Ulcerative Colitis, Crohn's Disease, Lupus (SLE), Rheumatoid Arthritis

---

## The 4-Agent Pipeline

Think of 4 workers passing a task down a conveyor belt:

```
📥 Raw news → Agent 1 → Agent 2 → Agent 3 → Agent 4 → 📲 Alert to Vikas
```

### Agent 1 — The Reader *(fast, cheap — Haiku)*
Reads all data from the 3 sources. Asks:
*"Is any of this about our 6 indications? Is it from one of our 15 companies?"*
- Yes → passes forward
- No → discards with a reason logged

### Agent 2 — The Sorter *(fast, cheap — Haiku)*
Takes what Agent 1 found and classifies each item:

**Category:**
- Regulatory — drug approval, filing, FDA/EMA decision
- Clinical — trial results, Phase 2/3 data readout
- Commercial — acquisition, partnership, licensing deal

**Priority:**
- 🔴 BREAKING — send immediately (FDA approval, major trial result)
- 🟡 IMPORTANT — include in evening digest
- ⚪ FYI — low urgency, evening digest

**Duplicate check:** Searches Supabase — if we already reported this, skip it.

### Agent 3 — The Analyst *(slower, smarter — Sonnet)*
The expensive but essential step. For each new finding:

1. **Looks up Neo4j** → Who is this company? What tier? What drugs? Do they threaten Pfizer?
2. **Searches Supabase** → Have we reported anything related in the past?
3. **Writes enriched 5-part alert:**
   - 📝 What happened (2-3 sentences, factual)
   - 🔍 Context (company tier, competing drugs, mechanism)
   - 🆚 New vs known (surprising or expected based on history?)
   - ⚡ Pfizer implication (which Pfizer drugs are at risk? how severe?)
   - 🔗 Source URL
4. **Updates Neo4j** with any new facts from this news (new drug, new partnership, new approval)
5. **Saves to Supabase** findings table

### Agent 4 — The Sender *(fast, cheap — Haiku)*
Final quality check, then:
- 🔴 BREAKING → sends WhatsApp alert immediately
- 🟡/⚪ → saves for 9PM IST digest
- Writes full audit log to Supabase `pipeline_logs` table

---

## What You Receive

### Immediate Alert (BREAKING news only)
```
🚨 KINCHA ALERT — IMPORTANT
📅 March 18, 2026 | Plaque Psoriasis

J&J's Icotyde (icotrokinra) Approved by FDA

📝 What happened: FDA approved Icotyde, the first oral IL-23 receptor
targeted peptide for moderate-to-severe plaque psoriasis...

🔍 Context: J&J is Tier 1 in Psoriasis. Icotyde is a once-daily pill —
fundamentally different delivery vs current injectable biologics...

🆚 New vs known: NEW mechanism class. No prior oral IL-23R peptide existed.

⚡ Pfizer implication: Threatens Tremfya (guselkumab) market share.
Patients who prefer oral dosing may switch...

🔗 https://...
```

### Evening Digest (9PM IST daily)
```
📋 KINCHA DAILY DIGEST — March 25, 2026
Client: Pfizer | Sources: 15 companies + FDA + EMA + ClinicalTrials

• 🟡 Commercial | Sanofi | RA — $1.23B T-cell engager deal with Kali
• 🟡 Commercial | Gilead | Lupus — Acquired Ouro Medicines + OM336

Regulatory: 0 | Clinical: 0 | Commercial: 2 | Total: 2
```

### No-findings message
```
✅ Kincha 6PM IST — 3 sources checked, no new findings today.
```

---

## The Admin Layer (Runs Automatically)

### Every run → Supabase pipeline_logs
After every pipeline run, Kincha writes a structured log:
- Which sources were checked
- How many items found, skipped, saved
- Why each item was skipped (wrong indication, duplicate, not target company, etc.)
- Any errors (e.g. EMA URL broken)

You can check this anytime in Supabase dashboard:
```sql
SELECT run_time, items_found, items_saved, errors
FROM pipeline_logs
ORDER BY run_time DESC;
```

### Every night at 11:15PM IST → GitHub
Two files updated automatically:
- **UPDATES.md** — daily activity log (findings, sources checked, system changes)
- **DOCUMENTATION.md** — master system doc with decisions and discussion summaries

This means even after a fresh chat, Kincha can read these files and remember everything.

---

## Schedule

| Time (IST) | Task |
|------------|------|
| 6:00 PM | Monitoring pipeline run #1 |
| 8:00 PM | Monitoring pipeline run #2 |
| 9:00 PM | Daily digest sent to WhatsApp |
| 11:15 PM | GitHub files updated (UPDATES.md + DOCUMENTATION.md) |

---

## Cost

| Component | Cost |
|-----------|------|
| Supabase | Free (500MB, 50k rows) |
| Neo4j AuraDB | Free (200k nodes) |
| GitHub | Free |
| ClinicalTrials / FDA / EMA APIs | Free |
| Claude API (this bot) | ~$2-4/month (2 runs/day, mostly Sonnet) |
| NanoClaw (WhatsApp delivery) | Your existing plan |
| **Total new cost** | **~$2-4/month** |

**Planned optimisation:** Use Haiku (cheaper model) for Agents 1, 2 and 4 → drops Claude cost to ~$1/month. Needs Anthropic API key.

---

## What's Still Being Built

| Item | Status | Needs |
|------|--------|-------|
| Haiku cost optimisation | Pending | Anthropic API key |
| Email alerts to vikas@atacana.com | Pending | Resend.com or SendGrid API key |
| Fix EMA RSS/search URL | Pending | Find correct URL |
| Client dashboard (GitHub Pages) | Pending | — |

---

*Created: 2026-03-25 | Auto-maintained by Kincha bot*

---

## Changes — 2026-03-25

### Sources Updated: Firecrawl restored (6PM run only)
Company website crawling is back in the pipeline, but only on the **6PM IST run** to stay within the 500 free pages/month Firecrawl limit:

| Run | Sources |
|-----|---------|
| 6PM IST | ClinicalTrials + FDA + EMA + **Firecrawl (15 company websites)** |
| 8PM IST | ClinicalTrials + FDA + EMA only (free APIs) |

**Why 6PM only?** 15 companies × 2 runs × 30 days = 900 pages/month → exceeds free tier (500). 
15 companies × 1 run × 30 days = 450 pages/month → fits within free tier ✅

### Agent 3 DB Enrichment Step (Sonnet) — Clarified

Agent 3 does more than write alerts. After every new finding, it **extracts structured facts** and updates both databases:

**Neo4j Knowledge Graph updates:**
- New drug mentioned → creates Drug node with MOA, status, indication
- Drug approval → adds `[:TREATS {approval_year, region}]` relationship
- Phase 3 result → updates `drug.phase3_result` and `drug.phase3_date`
- New partnership/acquisition → adds `[:PARTNERS_WITH]` or `[:ACQUIRED]` relationship
- New trial → adds `[:IN_PHASE {phase, start_date}]` relationship
- Competitive threat → adds `[:THREATENS {indication, severity, reason}]` → Pfizer

**Supabase Findings update:**
- Every new finding saved with full text for future RAG search
- Fields: company, indication, title, summary, exact content, date, source URL

This means both databases **self-update** as new news comes in — no manual data entry needed.

### Schedule Finalised
- 6PM IST: full run (free APIs + Firecrawl)
- 8PM IST: light run (free APIs only)
- 9PM IST: daily digest
- 11:15PM IST: GitHub files updated (UPDATES.md + DOCUMENTATION.md + WORKFLOW.md)

---

## Changes — 2026-03-25 (Update 2)

### ClinicalTrials.gov — Dedicated Smart Task

ClinicalTrials.gov updates its database **once daily at ~9 AM UTC**. Checking it at 6PM and 8PM IST was wasteful — no new data would exist. 

**Solution:** Dedicated lightweight task runs at **9:15 AM UTC (2:45 PM IST)** — right after the daily update window.

| What it checks | How |
|---------------|-----|
| New studies posted TODAY | `filter.advanced=AREA[StudyFirstPostDate]RANGE[TODAY,TODAY]` |
| Studies UPDATED today | `filter.advanced=AREA[LastUpdatePostDate]RANGE[TODAY,TODAY]` |

Task ID: `task-1774417534376-mayzry` | Schedule: `15 9 * * *` UTC

The 6PM and 8PM runs no longer check ClinicalTrials — eliminating redundant API calls.

### Final Schedule

| Time (IST) | Time (UTC) | Task | Sources |
|------------|------------|------|---------|
| 2:45 PM | 9:15 AM | ClinicalTrials check | ClinicalTrials.gov only (right after daily DB update) |
| 6:00 PM | 12:30 PM | Main pipeline run #1 | FDA + EMA + Firecrawl (15 companies) |
| 8:00 PM | 14:30 PM | Main pipeline run #2 | FDA + EMA + Firecrawl (15 companies) |
| 9:00 PM | 15:30 PM | Daily digest | Sends queued findings |
| 11:15 PM | 17:45 PM | EOD GitHub update | UPDATES.md + DOCUMENTATION.md + WORKFLOW.md |

### Firecrawl Budget (with both 6PM + 8PM crawls)
15 companies × 2 runs/day × 30 days = **900 pages/month**
Free tier = 500 pages/month → **upgrade to Hobby plan (~$16/month, 3,000 pages) required**

---

## Cost Reference — Token Breakdown

### Per Run: Detailed Token Cost

**Scenario A — No findings (most runs)**

| Source | Tokens | Type |
|--------|--------|------|
| Task prompt (instructions) | ~3,000 | Input |
| Firecrawl — 15 companies × ~400 tokens | ~6,000 | Input |
| FDA search results | ~800 | Input |
| EMA search results | ~400 | Input |
| Agent 1 output — "nothing relevant" | ~100 | Output |
| Agent 4 output — log + "no findings" message | ~300 | Output |
| **Total** | **~10,200 input / ~400 output** | |

**Cost (Sonnet):** 10,200 × $3/M + 400 × $15/M = **~$0.037/run**

---

**Scenario B — 1 finding discovered**

| Source | Tokens | Type |
|--------|--------|------|
| Everything from Scenario A | ~10,200 | Input |
| Neo4j query + competitive context response | ~400 | Input |
| Supabase RAG — 5 past findings retrieved | ~600 | Input |
| Agent 3 output — enriched 5-part alert | ~600 | Output |
| Agent 3 output — Neo4j MERGE Cypher | ~150 | Output |
| Agent 4 — quality check + pipeline log | ~300 | Output |
| **Total** | **~11,400 input / ~1,400 output** | |

**Cost (Sonnet):** 11,400 × $3/M + 1,400 × $15/M = **~$0.055/run**

> **Key:** Output tokens cost 5× more than input ($15 vs $3/M). Agent 3 writing the alert (~600 output tokens) is the single most expensive step.

---

### Monthly Cost Scenarios

**Assumptions:**
- 1 run/day (6:30 PM IST) = 30 runs/month
- Separate ClinicalTrials task: ~$0.30/month flat
- Digest + EOD update: ~$0.60/month flat
- Fixed overhead: ~$0.90/month

| Finding frequency | Findings/month | Pipeline cost | Total monthly |
|------------------|---------------|--------------|---------------|
| No news | 0 | 30 × $0.037 = $1.11 | **~$2.00/month** |
| Light (1 per week total) | ~4 | $1.18 | **~$2.10/month** |
| **1 per company per week** | **~60** | **30 runs mix = ~$1.65** | **~$2.55/month** |
| Heavy (2 per company per week) | ~120 | ~$2.20 | **~$3.10/month** |

**1 news per company per week breakdown:**
- 15 companies × 4 weeks = ~60 findings/month
- ~20 runs have 0 findings: 20 × $0.037 = $0.74
- ~10 runs have 1-2 findings each: 10 × $0.065 avg = $0.65
- Pipeline subtotal: ~$1.39
- + Fixed overhead: $0.90
- **Total: ~$2.30/month**

---

### With Haiku Optimisation (pending Anthropic API key)

When Agents 1, 2 and 4 run on Haiku ($0.25/M input, $1.25/M output) and only Agent 3 uses Sonnet:

| Finding frequency | Current (all Sonnet) | Optimised (Haiku+Sonnet) | Saving |
|------------------|---------------------|--------------------------|--------|
| 0 findings/month | ~$2.00 | ~$0.55 | 73% |
| 60 findings/month (1/company/week) | ~$2.30 | ~$1.10 | 52% |
| 120 findings/month | ~$3.10 | ~$1.65 | 47% |

**Haiku is worth implementing** — saves ~$1-1.50/month even at high finding volume.
To implement: share your Anthropic API key (starts with `sk-ant-...`).



---

## Model Strategy Update — 2026-03-25 (Update 3)

### Decision: Migrate to Groq + Keep Anthropic as Fallback Only

After cost analysis and quality testing (side-by-side on real KT501 finding), scheduled task LLM calls are migrated from Anthropic to Groq.

---

### New Model Split (Live as of 2026-03-25)

| Agent | Task | Model | Cost |
|-------|------|-------|------|
| Agent 1 — Extractor | Find relevant articles from scraped HTML | Groq **Llama 3.1 8B** | $0 |
| Agent 2 — Categorizer | Classify finding type + extract fields | Groq **Llama 3.1 8B** | $0 |
| Agent 3 — Alert Writer | Write alert + Neo4j Cypher + Supabase JSON | Groq **Llama 3.3 70B** + DB context | $0 |
| Agent 4 — QC + Logger | Quality check + log to pipeline_logs + crawl_items | Groq **Llama 3.1 8B** | $0 |
| **Fallback** | If Groq fails (rate limit/outage) | **Anthropic Haiku (1/2/4) + Sonnet (3)** | ~$0.04/run |

**Key insight on Agent 3:** Groq 70B tested WITHOUT DB context — missed Pfizer competing drug. Tested WITH Supabase/Neo4j context injected — matched Sonnet quality, correctly identified Cibinqo, mechanism differentiation, and timeline. Context was always in our DB; model just needed it passed in. **Agent 3 now queries DB first, then calls Groq 70B.**

Groq API key stored in DOCUMENTATION.md credentials table and all task prompts.

---

### What Anthropic (Sonnet) Is Now Used For

| Use | Model | Cost |
|-----|-------|------|
| **This chat (Vikas ↔ Kincha)** | Claude Sonnet via NanoClaw | Part of NanoClaw plan |
| **Fallback when Groq fails** | Haiku (1/2/4) + Sonnet (3) | ~$0.04/run, rare |
| **Nothing else** | — | — |

Anthropic API key remains in task prompts purely as fallback. Normal operations cost **$0 in Anthropic tokens.**

---

### Updated Monthly Cost (Groq era)

| Clients | Findings/month | Monthly LLM cost |
|---------|---------------|-----------------|
| 1 | Any | **~$0** |
| 10 | Any | **~$0** |
| 50 | Any | **~$0** (within free tier) |
| Fallback incidents | per event | ~$0.04/run |

Groq free tier: 14,400 req/day for 8B, 1,000 req/day for 70B (enough — Agent 3 only runs on findings).

---

### Previous Cost Models (Kept for Reference)

**Haiku-optimised Anthropic plan (superseded 2026-03-25):**

| Finding frequency | Monthly cost |
|------------------|-------------|
| 0 findings | ~$0.55 |
| 60 findings/month | ~$1.10 |
| 120 findings/month | ~$1.65 |

**Original all-Sonnet plan (superseded 2026-03-24):**

| Finding frequency | Monthly cost |
|------------------|-------------|
| 0 findings | ~$2.00 |
| 60 findings/month | ~$2.30 |
| 120 findings/month | ~$3.10 |

---

### Scale Roadmap

| Phase | Clients | LLM | Monthly LLM cost |
|-------|---------|-----|-----------------|
| **Phase 1 (now)** | 1-5 | Groq free tier | $0 |
| **Phase 2** | 10-30 | Groq free tier | $0 |
| **Phase 3** | 50+ | Self-hosted Ollama on VPS (~$40/month) | ~$0.80/client |

Auto-switch (planned Phase 2): local/VPS Ollama first → Groq free → Anthropic fallback. Cost always minimised automatically.

---

*Model strategy updated: 2026-03-25*

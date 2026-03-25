# Kincha Bot — WORKFLOW
**Client:** Pfizer (Pilot) | **Updated:** 2026-03-25 | **Owner:** vikas@atacana.com

---

## 1. CURRENT SETUP (Live as of 2026-03-25)

### Schedule
| Time (IST) | Time (UTC) | Task |
|-----------|-----------|------|
| 2:45 PM | 9:15 AM | ClinicalTrials.gov scan — right after daily DB update |
| 6:30 PM | 1:00 PM | Main pipeline — Firecrawl 14 company sites + alerts |
| 9:00 PM | 3:30 PM | Daily digest email |
| 11:15 PM | 5:45 PM | EOD GitHub update (UPDATES.md + DOCUMENTATION.md + WORKFLOW.md) |

### Models
| Agent | Model | Task | Cost |
|-------|-------|------|------|
| 1 — Extractor | Groq Llama 3.1 8B | Reads scraped HTML, finds relevant articles | $0 |
| 2 — Categorizer | Groq Llama 3.1 8B | Classifies finding type, extracts fields | $0 |
| 3 — Alert Writer | Groq Llama 3.3 70B | Writes alert + Neo4j Cypher + Supabase insert | $0 |
| 4 — QC + Logger | Groq Llama 3.1 8B | Quality check, logs to pipeline_logs + crawl_items | $0 |
| Fallback | Anthropic Haiku/Sonnet | Used only if Groq fails | ~$0.04/run |

**Total monthly cost: ~$0** (Groq free tier, 14,400 req/day 8B, 1,000/day 70B)

### Companies Monitored (15)
| Tier | Companies | Watch threshold |
|------|-----------|----------------|
| 🔴 Tier 1 | AbbVie, Takeda, J&J, Eli Lilly, BMS | Phase 2+ |
| 🟡 Tier 2 | Sanofi, Roche, Novartis, AstraZeneca, Merck, Gilead, Amgen, GSK, Novo Nordisk | Phase 3+ |

### Indications Tracked (6)
Rheumatoid Arthritis · Plaque Psoriasis · Psoriatic Arthritis · Ulcerative Colitis · Crohn's Disease · SLE (Lupus)

### Sources
1. **Firecrawl** — scrapes 14 company press release pages (main 6:30PM run)
2. **ClinicalTrials.gov API** — queries all 6 indications for new/updated trials (9:15AM run)
3. **EMA** — press releases via web search (RSS URL broken — fix pending)

### Databases
| DB | Tables | Purpose |
|----|--------|---------|
| Supabase | `findings` | Full alert text for every captured finding |
| Supabase | `pipeline_logs` | Per-run summary (counts, models, errors) |
| Supabase | `crawl_items` | **Per-article audit** — every article/trial checked, status + reason |
| Supabase | `feedback` | Your notes on findings quality *(new — see Section 3)* |
| Neo4j | — | Knowledge graph: 27 companies, 44 drugs, 93+ relationships |

### Known Issues
- EMA RSS URL returns 404 (workaround: web search)
- Some JS-rendered company pages not fully parsed by Firecrawl (logged as errors in crawl_items)

---

## 2. HOW THE PIPELINE WORKS

```
Company pages → Firecrawl → Agent 1 (filter) → Agent 2 (classify) → Agent 3 (alert+DB) → Agent 4 (log)
CT.gov API   ─────────────────────────────────────────────────────────↗
```

### Agent 1 — Extractor (Groq 8B)
Reads raw page content from Firecrawl / CT.gov API. Decides: *is this relevant to our 6 indications?*
- **Captured**: clinical trial Phase 1-3, approval, filing, M&A, partnership, pipeline news
- **Skipped**: earnings, ESG, manufacturing, executive hires, wrong disease
- Logs **every item** (captured + skipped + reason) to `crawl_items`

### Agent 2 — Categorizer (Groq 8B)
Only runs for captured items. Classifies: `clinical_trial | regulatory | ma_partnership | pipeline_update | launch`
Extracts: drug name, phase, indication, mechanism, relevance score (1–10)

### Agent 3 — Alert Writer (Groq 70B)
Only runs when findings exist. **Queries Supabase first** for Pfizer's competing assets in that indication, then writes:
1. **Alert** — 3-4 sentence executive-level competitive intel
2. **Neo4j Cypher** — MERGE queries to update knowledge graph
3. **Supabase JSON** — insert into `findings` table

Then: saves to `findings` → sends email → executes Neo4j Cypher

### Agent 4 — QC + Logger (Groq 8B)
Final quality check. Writes run summary to `pipeline_logs`. Sends "no findings" email if nothing found.

---

## 3. HOW TO READ YOUR LOGS

### See every article checked in a run (with skip reasons):
```sql
SELECT company, title, status, reason, published_date
FROM crawl_items
WHERE run_time::date = '2026-03-25'
ORDER BY company, status;
```

### See only skipped articles and why:
```sql
SELECT company, title, reason
FROM crawl_items
WHERE status = 'skipped'
AND run_time::date = '2026-03-25'
ORDER BY company;
```

### See captured findings with full alert:
```sql
SELECT company, indication, news_title, news_summary, date
FROM findings
WHERE date >= '2026-03-24'
ORDER BY date DESC;
```

### See run summaries:
```sql
SELECT run_time, items_found, items_skipped, items_saved, errors
FROM pipeline_logs
ORDER BY run_time DESC
LIMIT 10;
```

### See ClinicalTrials trials checked:
```sql
SELECT company, title, status, reason, keywords_matched->>'phase' as phase
FROM crawl_items
WHERE source = 'clinicaltrials'
ORDER BY run_time DESC;
```

---

## 4. LEAVING FEEDBACK ON FINDINGS

A `feedback` table lets you flag quality issues, mark false positives, or add notes on any finding.

### feedback table schema:
```sql
CREATE TABLE feedback (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  finding_id uuid REFERENCES findings(id) ON DELETE CASCADE,
  crawl_item_id uuid REFERENCES crawl_items(id) ON DELETE CASCADE,
  rating text CHECK (rating IN ('good','irrelevant','low_quality','false_positive')),
  comment text,
  reviewed_by text DEFAULT 'vikas',
  created_at timestamp DEFAULT now()
);
```

### How to use in Supabase Table Editor:
1. Open **Supabase → Table Editor → feedback**
2. Click **Insert row**
3. Paste the `finding_id` from the `findings` table
4. Set `rating`: `good` / `irrelevant` / `low_quality` / `false_positive`
5. Add a `comment` (optional)

Kincha reads this table when writing future alerts — if you mark a finding as `irrelevant`, similar items get a lower relevance score going forward.

---

## 5. CHANGE LOG (newest first)

| Date | Change |
|------|--------|
| 2026-03-25 | `feedback` table added — rate finding quality directly in Supabase |
| 2026-03-25 | `crawl_items` table live — per-article audit log with 116 backfilled rows |
| 2026-03-25 | Groq migration — Agents 1/2/4 → 8B, Agent 3 → 70B. Cost $0/month |
| 2026-03-25 | Fixed Groq 403 — all tasks now use curl (not urllib) for Groq calls |
| 2026-03-25 | Fixed table name bug (competitive_intel → findings) |
| 2026-03-25 | ClinicalTrials dedicated task (9:15 AM UTC) |
| 2026-03-25 | GitHub files now newest-first (prepend on update) |
| 2026-03-24 | `pipeline_logs` table created — run-level audit |
| 2026-03-24 | Gmail SMTP set up (Resend.com dropped — requires domain) |
| 2026-03-24 | Haiku optimisation for Agents 1/2/4 (later superseded by Groq) |
| 2026-03-24 | First findings: Sanofi KT501 (Phase II, atopic dermatitis), Gilead/Ouro (M&A) |
| 2026-03-24 | Neo4j enriched: KT501, Ouro Medicines, OM336 nodes + relationships added |
| 2026-03-24 | System built: 4-agent pipeline, Supabase, Neo4j, 15 companies, 6 indications |

---
*Auto-maintained by Kincha EOD task · Repo: vikassharma58/kinchabot-monitor*

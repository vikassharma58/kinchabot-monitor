# Kincha Bot - Master Documentation

**Client:** Pfizer (Pilot) | **Owner:** vikas@atacana.com | **Updated:** 2026-03-25

---

## 1. What Is Kincha?

Pharma competitive intelligence monitor. Watches 15 competitor companies across 6 disease indications for Pfizer. Sends enriched WhatsApp alerts when significant news breaks. Maintains dual database (Supabase + Neo4j) for RAG memory and knowledge graph.

---

## 2. Architecture

```
KINCHA BOT (runs every 2 hrs, 8AM-9PM IST — CURRENTLY PAUSED)
  |-- ClinicalTrials.gov API --> company-sponsored trials (free, unlimited)
  |-- FDA web search --> drug approvals + safety news (free)
  |-- EMA web search --> CHMP opinions + approvals (free, RSS broken)
  |-- [Firecrawl --> 15 company press release pages (500 pages/month free — paused)]
  |
  |-- Agent 1 Haiku --> extract relevant findings
  |-- Agent 2 Haiku --> categorize + route (BREAKING/IMPORTANT/FYI)
  |-- Agent 3 Sonnet --> write enriched 5-part alert + update both DBs
  |-- Agent 4 Haiku --> quality check + send + log to pipeline_logs
  |
  |-- Supabase (PostgreSQL + pgvector) --> findings + RAG memory + pipeline_logs
  |-- Neo4j AuraDB --> knowledge graph (company/drug/indication relationships)
  |-- WhatsApp (NanoClaw) --> enriched alert to Vikas
  |-- Email vikas@atacana.com --> PENDING (need email API key)
```

---

## 3. Credentials

| Service | Value |
|---------|-------|
| Supabase URL | https://oamaeosqshspohodjihv.supabase.co |
| Supabase Key | sb_publishable_xfEJyUdzk8gZi1h8Qu5PLA_4tI2gOpS |
| Neo4j HTTP API | https://bb84eae8.databases.neo4j.io/db/bb84eae8/query/v2 |
| Neo4j User | bb84eae8 |
| Neo4j Password | XtvZWHyr3yB_sWPLjbBVolnog8fwzBNJFrx6SNqaHlY |
| Firecrawl Key | fc-bf4e535572894061aa9d21f93da1619f |
| GitHub Repo | https://github.com/vikassharma58/kinchabot-monitor |
| GitHub Token | [stored securely - ask Kincha bot] |

---

## 4. Scheduled Tasks

| Task ID | Purpose | Schedule | Status |
|---------|---------|----------|--------|
| task-1774095367777-k9j0ix | Main monitoring pipeline | Every 2hrs, 8AM-9PM IST | PAUSED |
| task-1774345190505-a5ksd1 | Daily digest (9PM IST) | 9:00 PM IST daily | PAUSED |
| task-1774342525878-p39wz5 | EOD doc update (11:15PM IST) | 11:15 PM IST daily | PAUSED |

**All tasks paused on Mar 24 pending cost optimisation.**

---

## 5. Supabase Schema

```sql
companies     (id, name, website, created_at)
indications   (id, name, category, created_at)
findings      (id, client, company, indication,
               news_title, news_summary, exact_content,
               date, source_url, created_at)           -- ~165 records
pipeline_logs (id, run_time, sources_checked jsonb,
               items_found, items_skipped, items_saved,
               errors jsonb, raw_log jsonb)             -- audit trail per run
-- pgvector + RLS enabled on all tables
```

**Supabase data loaded:**
- 77 drug-indication profiles (from Drug_Profiles.csv)
- 30 SWOT company profiles (from SWOT_and_Earnings_Intel.csv)
- 22 sections from Pharma Competitive Report 2026 (Word doc)
- 2 legend/MOA reference records
- 5 live alert findings (from monitoring runs)
- 26 companies + 6 indications

---

## 6. Neo4j Knowledge Graph

**Current state:** 27 companies, 44 drugs, 93 drug-indication relationships + 31 enriched relationships

```
Enriched relationships loaded:
- Company tiers (Tier 1/2/3) + strategic focus
- Drug revenues (e.g. Skyrizi $11.7B, Rinvoq $10.9B)
- THREATENS relationships (competitor → Pfizer, by indication + severity)
- ACQUIRED relationships (Merck→Prometheus, Pfizer→Arena, etc.)
- PARTNERS_WITH relationships (Sanofi↔Teva duvakitug, Sanofi↔Regeneron, etc.)
- New from monitoring: Sanofi→KT501 (licensed from Kali Therapeutics)
                       Gilead→OM336/gamgertamig (acquired Ouro Medicines)
```

---

## 7. Monitoring Config

| Parameter | Value |
|-----------|-------|
| Companies | 15 (top pharma excl. Pfizer) |
| Indications | Plaque Psoriasis, Psoriatic Arthritis, Ulcerative Colitis, Crohns Disease, Lupus (SLE), Rheumatoid Arthritis |
| Schedule | Every 2hrs, 11:00-15:30 UTC (8AM-9PM IST) |
| Sources | ClinicalTrials.gov API + FDA search + EMA search (Firecrawl paused) |
| Status | PAUSED — resume after cost optimisation |

---

## 8. Alert Format (5-part enriched)

```
[PRIORITY] CATEGORY | COMPANY | INDICATION
TITLE
What happened: 2-3 sentence factual summary
Context: company tier + competing drugs from Neo4j
New vs known: comparison to past Supabase findings (RAG)
Pfizer implication: THREATENS relationships + Pfizer competing drugs
Source URL
```

---

## 9. Findings Reported

| Date | Company | Indication | Finding |
|------|---------|-----------|---------|
| Mar 18 2026 | J&J | Plaque Psoriasis | Icotyde (icotrokinra) FDA approved — first oral IL-23R peptide |
| Mar 16 2026 | Sun Pharma | Psoriatic Arthritis | Ilumya sBLA submitted |
| Mar 6 2026 | BMS | Psoriatic Arthritis | Sotyktu FDA approved for PsA |
| Mar 6 2026 | Roche | Lupus (SLE) | Gazyva Phase 3 NEJM — 76.7% SRI-4 response |
| Mar 2026 | Takeda | Plaque Psoriasis | Zasocitinib Phase 3 all endpoints met |
| Mar 23 2026 | Sanofi | Rheumatoid Arthritis | Licensed KT501 from Kali Therapeutics — $1.23B T-cell engager deal |
| Mar 23 2026 | Gilead | Lupus (SLE) | Acquired Ouro Medicines + OM336/gamgertamig BCMAxCD3 T-cell engager |

---

## 10. Pending Items

| Priority | Task | Notes |
|----------|------|-------|
| HIGH | Cost optimisation — implement Haiku for Agents 1/2/4 | Need Anthropic API key in task prompts; saves ~$25/month |
| HIGH | Reduce monitoring runs: 10/day → 5/day | Change cron from 0,30 to 0 only |
| HIGH | Resume all paused tasks after optimisation | Currently all 3 tasks paused |
| MEDIUM | Email alerts to vikas@atacana.com | Need Resend.com or SendGrid API key |
| MEDIUM | Fix EMA RSS URL (returning 404) | Find correct current EMA RSS URL |
| MEDIUM | Backfill 12 months historical findings into Supabase | For RAG "new vs known" to work well |
| LOW | Client dashboard UI (GitHub Pages) | Free, I build it once token available |

---

## 11. Decisions Log

| Date | Decision | Reason |
|------|----------|--------|
| Mar 22 2026 | Started 12-company monitor, Psoriasis + Ovarian Cancer | Initial prototype |
| Mar 23 2026 | Expanded: 15 companies, 6 indications, Pfizer as client | Pilot scope defined |
| Mar 23 2026 | Supabase + Neo4j dual database architecture | RAG (Supabase) + relationships (Neo4j) |
| Mar 23 2026 | Enriched 5-part alert format | More actionable than raw news |
| Mar 24 2026 | Firecrawl for web crawling (500 pages/month free) | Better than RSS (many 404s) |
| Mar 24 2026 | Added pipeline_logs table to Supabase | Audit trail so Vikas can verify what was checked |
| Mar 24 2026 | All tasks PAUSED | Cost optimisation needed before resuming |
| Mar 24 2026 | Haiku for Agents 1/2/4, Sonnet only for Agent 3 | ~$1/month vs ~$26/month |

---

## 12. Discussion Summary Log

### Mar 22, 2026
- First session. Set up hourly pharma monitor for 12 companies (Psoriasis + Ovarian Cancer)
- Connected Google Sheet via Apps Script webhook
- Ran 30-day lookback scan, posted 4 findings to Google Sheet (J&J, Sun Pharma, BMS, GSK)
- Troubleshot: Apps Script deployment URLs, HTML entities in pasted code, curl redirect issue

### Mar 23, 2026
**Built:**
- Supabase: findings + companies + indications tables, pgvector, RLS policies
- Neo4j: 27 companies, 44 drugs, 6 indications, 31 enriched relationships (tiers, revenues, THREATENS, ACQUIRED, PARTNERS_WITH)
- Loaded Drug_Profiles.csv (77 drugs), SWOT_and_Earnings_Intel.csv (30 SWOTs), Legend_and_Notes.csv, Pharma_Competitive_Report_2026.docx (22 sections) into Supabase
- Built 4-agent pipeline: Haiku Extractor → Haiku Categorizer → Sonnet Alert Writer → Haiku QC+Send
- Launched monitoring: every 2hrs, 15 companies, 6 indications
- First live findings: Roche Gazyva (Lupus) + Takeda Zasocitinib (Psoriasis)

**Key decisions:**
- Firecrawl for company pages (free tier, 500 pages/month = 15/day fits)
- Fresh chats periodically to save context tokens
- GitHub DOCUMENTATION.md as persistent memory across sessions

### Mar 24, 2026
**Findings discovered:**
- Sanofi × Kali Therapeutics: $1.23B license for KT501 tri-specific T-cell engager (RA)
- Gilead × Ouro Medicines: acquisition of OM336/gamgertamig BCMAxCD3 T-cell engager (autoimmune/Lupus)
- Trend: Two T-cell engager M&A deals in same day = sector pivot toward BCMA B-cell depletion

**Built:**
- pipeline_logs Supabase table: per-run audit trail (sources checked, items found/skipped/saved, errors)
- UPDATES.md GitHub file: daily activity log
- EOD documentation update task (11:15PM IST)

**Cost analysis:**
- Current (all Sonnet): ~$26/month
- Optimised (Haiku for Agents 1/2/4, Sonnet only for Agent 3): ~$1/month
- Action: need Anthropic API key to implement true Haiku sub-agent calls

**Decisions:**
- Remove Firecrawl from pipeline (use free APIs only for now — revisit)
- All tasks PAUSED until cost optimisation done
- GitHub token restored for file updates

**Pending from Vikas:**
- Anthropic API key (for Haiku sub-agent calls)
- Email API key (Resend.com or SendGrid) for vikas@atacana.com alerts

---

*Auto-updated daily by Kincha bot*

### Mar 25, 2026
**Findings:** 0 new items today
**Decisions:**
- Monitoring schedule finalised: 2 runs/day at 6PM + 8PM IST (down from 10 runs/day)
- All tasks resumed after cost optimisation discussion
- GitHub token ghp_... stored in task prompts (not in committed files)

**Built/Changed:**
- UPDATES.md created as daily activity log
- DOCUMENTATION.md fully updated
- EOD task updated to write both files nightly
- Monitoring cron changed to `30 12,14 * * *`

**Pending:**
- Anthropic API key → implement Haiku for Agents 1/2/4 (~$1/month vs ~$26/month)
- Email alerts → need Resend.com or SendGrid API key
- Fix EMA RSS URL (404)
- Client dashboard UI (GitHub Pages)


---

*Auto-updated daily by Kincha bot*

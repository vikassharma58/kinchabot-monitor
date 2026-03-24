# Kincha Bot - Master Documentation

**Client:** Pfizer (Pilot) | **Owner:** vikas@atacana.com | **Updated:** 2026-03-24

---

## 1. What Is Kincha?

Pharma competitive intelligence monitor. Watches 25 competitor companies across 6 disease indications for Pfizer. Sends enriched WhatsApp alerts when significant news breaks.

---

## 2. Architecture

```
KINCHA BOT (runs every 30 min, 5-9 PM IST)
  |-- Firecrawl --> crawls company press release pages
  |-- Supabase (PostgreSQL + pgvector) --> stores findings + RAG memory
  |-- Neo4j AuraDB --> knowledge graph (company/drug/indication map)
  |-- WhatsApp --> enriched alert to Vikas
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
| GitHub Repo | https://github.com/vikassharma58/kinchabot-monitor |
| GitHub Token | [stored securely - ask Kincha bot] |
| Apps Script | https://script.google.com/macros/s/AKfycbwowCwmAoiR2nVVFHV0Ybg9fhA3ryibc2iryU5AX-em7Ghoka7CpVb5_Jc35QGNxXbS/exec |
| Scheduled Task | task-1774095367777-k9j0ix (PAUSED) |

---

## 4. Supabase Schema

```sql
companies   (id, name, website, created_at)           -- empty, needs populating
indications (id, name, category, created_at)          -- empty, needs populating
findings    (id, client, company, indication,          -- ACTIVE, findings stored here
             news_title, news_summary, exact_content,
             date, source_url, created_at)
-- pgvector + RLS enabled
```

---

## 5. Neo4j Knowledge Graph

- 27 Company nodes, 19 Drug nodes, 6 Indication nodes
- Schema: (Company)-[:MAKES]->(Drug)-[:TREATS]->(Indication)
- Drugs loaded: Skyrizi, Rinvoq, Humira, Sotyktu, Icotyde, Tremfya, Stelara,
  Cosentyx, Taltz, Olumiant, Enbrel, Otezla, Dupixent, Entyvio,
  Zasocitinib, Xeljanz, Cibinqo, Ilumya, Filgotinib
- STILL NEEDED: ~100 more drugs (Vikas running research prompt)

---

## 6. Monitoring Config

| Parameter | Value |
|-----------|-------|
| Companies | 25 (top pharma excl. Pfizer) |
| Indications | Plaque Psoriasis, Psoriatic Arthritis, Ulcerative Colitis, Crohns Disease, Lupus (SLE), Rheumatoid Arthritis |
| Schedule | Every 30 min, 11:00-15:30 UTC (5-9 PM IST) |
| Status | PAUSED - resume after pilot build complete |

---

## 7. Alert Format (5-part enriched)

```
COMPANY | INDICATION - DATE
TITLE
What happened: factual 2-3 sentence summary
Context: background from Neo4j graph
New vs known: compare to Supabase past findings (RAG)
Implications for Pfizer: which Pfizer drugs are affected
Source URL
```

---

## 8. Findings Reported

| Date | Company | Indication | Finding |
|------|---------|-----------|---------|
| Mar 18 2026 | J&J | Plaque Psoriasis | Icotyde FDA approved |
| Mar 16 2026 | Sun Pharma | Psoriatic Arthritis | Ilumya sBLA submitted |
| Mar 6 2026 | BMS | Psoriatic Arthritis | Sotyktu FDA approved |
| Mar 6 2026 | Roche | Lupus (SLE) | Gazyva Phase 3 NEJM - 76.7% SRI-4 |
| Mar 2026 | Takeda | Plaque Psoriasis | Zasocitinib Phase 3 all endpoints met |

---

## 9. Pending Items

| Priority | Task | Blocker |
|----------|------|---------|
| HIGH | Firecrawl crawlers for 25 press release pages | Need Firecrawl API key from Vikas |
| HIGH | Populate Supabase companies + indications tables | Ready to do now |
| HIGH | Load full drug profiles into Neo4j + Supabase | Vikas running research on Claude.ai |
| HIGH | Backfill 12 months historical findings | For RAG to work well |
| MEDIUM | Email alerts to vikas@atacana.com | Need SendGrid or SMTP |
| MEDIUM | Neo4j visual explorer (GitHub Pages) | Ready to build |
| MEDIUM | Haiku crawling + Sonnet alerts (cost saving) | After crawlers built |
| GOAL | Full pilot go-live | After all HIGH items done |

---

## 10. Decisions Log

| Date | Decision |
|------|----------|
| Mar 22 2026 | Started 12-company monitor, Psoriasis + Ovarian Cancer |
| Mar 23 2026 | Expanded: 25 companies, 6 indications, Pfizer as client |
| Mar 23 2026 | Supabase + Neo4j + enriched alert format built |
| Mar 23 2026 | Decided Firecrawl over RSS for web crawling |
| Mar 23 2026 | Monitor paused - building full pilot first |
| Mar 24 2026 | Token-saving strategy: fresh chats + Haiku for crawling |

---

*Maintained by Kincha bot*

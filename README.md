# 🤖 Kincha Pharma Monitor — Project Board

**Client:** Pfizer (Pilot) | **Bot:** Kincha | **Updated:** 2026-03-23 12:05 UTC

---

## ✅ Done (5)

| # | Task | Completed |
|---|------|-----------|
| 1 | Set up hourly pharma monitor (now daily) | Mar 22, 2026 |
| 2 | 30-day lookback scan (Mar 14–22, 2026) | Mar 22, 2026 |
| 3 | Connect Google Sheet via Apps Script webhook | Mar 22, 2026 |
| 4 | Post 4 findings to Google Sheet | Mar 22, 2026 |
| 5 | Set up GitHub Kanban board (this board) | Mar 23, 2026 |

---

## ⏳ Blocked — Waiting on Vikas (2)

| # | Task | Waiting For |
|---|------|------------|
| 6 | Set up Supabase database | Create free account → supabase.com → share URL + API key |
| 7 | Set up Neo4j AuraDB knowledge graph | Create free account → neo4j.com/cloud/aura-free → share URI + password |

---

## 📋 Backlog (9)

| # | Task | Priority |
|---|------|----------|
| 8 | Set up Supabase schema + pgvector (RAG) | 🔴 High |
| 9 | Build RAG layer — embed competitor drug profiles | 🔴 High |
| 10 | Set up Neo4j knowledge graph schema | 🔴 High |
| 11 | Build competitor drug profiles (6 indications × 25 companies) | 🔴 High |
| 12 | Build enriched alert format (context + so-what for Pfizer) | 🔴 High |
| 13 | Update monitor: 30-min, 5–9PM IST, 25 companies, 6 indications | 🟡 Medium |
| 14 | Set up email alerts to vikas@atacana.com | 🟡 Medium |
| 15 | Set up direct website crawling (FDA, EMA, 25 company pages) | 🟡 Medium |
| 16 | Full Pfizer pilot live | 🏁 Goal |

---

## 📊 Monitoring Setup

| Parameter | Current | Target |
|-----------|---------|--------|
| Companies | 12 | 25 |
| Indications | 2 (Psoriasis, Ovarian Ca) | 6 (+UC, Crohn's, Lupus, RA) |
| Frequency | Daily digest | Every 30 min, 5–9PM IST |
| Alert type | Simple | Enriched (RAG + Knowledge Graph) |
| Storage | Google Sheets | Supabase (Postgres + pgvector) |
| Client | — | Pfizer |
| Email | — | vikas@atacana.com |

---

*Auto-updated by Kincha bot*

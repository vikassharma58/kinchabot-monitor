# 🤖 Kincha Pharma Monitor — Task Board

**Client:** Pfizer (Pilot) | **Bot:** Kincha | **Updated:** 2026-03-23

| # | Task | Status | Priority | Notes |
|---|------|--------|----------|-------|
| 1 | Set up hourly pharma monitor (now daily) | ✅ Done | — | Runs daily at 6AM UTC |
| 2 | 30-day lookback scan (Mar 14–22, 2026) | ✅ Done | — | 4 findings found |
| 3 | Connect Google Sheet via Apps Script webhook | ✅ Done | — | Webhook working |
| 4 | Post 4 findings to Google Sheet | ✅ Done | — | J&J, Sun Pharma, BMS, GSK |
| 5 | Set up GitHub task board | ✅ Done | — | This board |
| 6 | Set up Supabase database | ⏳ Blocked | 🔴 High | Vikas to create account → supabase.com |
| 7 | Set up Neo4j AuraDB knowledge graph | ⏳ Blocked | 🔴 High | Vikas to create account → neo4j.com/cloud/aura-free |
| 8 | Set up Supabase schema + pgvector (RAG) | 📋 Backlog | 🔴 High | Needs #6 first |
| 9 | Build RAG — embed competitor drug profiles | 📋 Backlog | 🔴 High | Needs #8 first |
| 10 | Set up Neo4j knowledge graph schema | 📋 Backlog | 🔴 High | Needs #7 first |
| 11 | Build competitor profiles (6 indications × 25 companies) | 📋 Backlog | 🔴 High | — |
| 12 | Build enriched alert format (context + so-what) | 📋 Backlog | 🔴 High | Needs #9 + #10 |
| 13 | Update monitor: 30-min, 5–9PM IST, 25 companies, 6 indications | 📋 Backlog | 🟡 Medium | — |
| 14 | Set up email alerts to vikas@atacana.com | 📋 Backlog | 🟡 Medium | — |
| 15 | Set up direct website crawling (FDA, EMA, 25 company pages) | 📋 Backlog | 🟡 Medium | — |
| 16 | Full Pfizer pilot live | 📋 Backlog | 🏁 Goal | All above complete |

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

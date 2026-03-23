# 🤖 Kincha Pharma Monitor — Task Board

**Client:** Pfizer (Pilot) | **Bot:** Kincha | **Updated:** 2026-03-23

| # | Task | Status | Priority | Notes |
|---|------|--------|----------|-------|
| 1 | Set up hourly pharma monitor (now daily) | ✅ Done | — | Runs daily at 6AM UTC |
| 2 | 30-day lookback scan (Mar 2026) | ✅ Done | — | 4 findings found |
| 3 | Connect Google Sheet via Apps Script webhook | ✅ Done | — | Webhook working |
| 4 | Post findings to Google Sheet | ✅ Done | — | J&J, Sun Pharma, BMS, GSK |
| 5 | Set up GitHub task board | ✅ Done | — | This board |
| 6 | Set up Supabase database | ✅ Done | 🔴 High | oamaeosqshspohodjihv.supabase.co |
| 7 | Set up Neo4j AuraDB knowledge graph | ✅ Done | 🔴 High | bb84eae8.databases.neo4j.io |
| 8 | Set up Supabase schema + pgvector (RAG) | ✅ Done | 🔴 High | findings table + RLS + vector |
| 9 | Load 19 drug profiles into Neo4j + Supabase | ✅ Done | 🔴 High | 27 companies, 19 drugs, 6 indications |
| 10 | Build enriched alert format (context + so-what) | ✅ Done | 🔴 High | RAG + KG + Pfizer implications |
| 11 | Update monitor: 30-min, 5–9PM IST, 25 companies, 6 indications | ✅ Done | 🔴 High | Runs 11:00–15:30 UTC |
| 12 | Load full competitor drug profiles (25 companies × 6 indications) | 🔄 In Progress | 🔴 High | Vikas running research prompt on Claude.ai |
| 13 | Neo4j visual explorer (no-code) | 📋 Backlog | 🟡 Medium | GitHub Pages HTML option ready to build |
| 14 | Set up email alerts to vikas@atacana.com | 📋 Backlog | 🟡 Medium | Need SendGrid API or SMTP |
| 15 | Set up direct website crawling (FDA, EMA, 25 company pages) | 📋 Backlog | 🟡 Medium | Firecrawl / Apify |
| 16 | Sub-agent architecture (Haiku research + Sonnet alerts) | 📋 Backlog | 🟡 Medium | Cost optimisation |
| 17 | Seed vector DB with 3–6 months historical news | 📋 Backlog | 🟡 Medium | Needs #12 first |
| 18 | Full Pfizer pilot live | 📋 Backlog | 🏁 Goal | All above complete |

---

## 📊 Monitoring Setup — LIVE

| Parameter | Current |
|-----------|---------|
| Companies | 25 (top pharma, excl. Pfizer) |
| Indications | 6 (Plaque Psoriasis, Psoriatic Arthritis, UC, Crohn's, Lupus, RA) |
| Frequency | Every 30 min, 5–9PM IST (11:00–15:30 UTC) |
| Alert type | Enriched: What happened / Context / New vs Known / Pfizer implications / Source |
| Storage | Supabase (Postgres + pgvector) + Neo4j (knowledge graph) |
| Client | Pfizer |
| WhatsApp | ✅ Active |
| Email | vikas@atacana.com (pending) |

---

## 🧠 Infrastructure

| Component | Status | Details |
|-----------|--------|---------|
| Supabase (RAG / vector DB) | ✅ Live | findings table, pgvector, RLS enabled |
| Neo4j AuraDB (knowledge graph) | ✅ Live | 27 companies, 19 drugs, 6 indications |
| Google Sheet | ✅ Live | Apps Script webhook, 4 findings |
| GitHub Board | ✅ Live | This README |
| WhatsApp alerts | ✅ Live | Enriched format |
| Email alerts | ⏳ Pending | Need SMTP/SendGrid |

---

*Auto-updated by Kincha bot • Last run: 2026-03-23*

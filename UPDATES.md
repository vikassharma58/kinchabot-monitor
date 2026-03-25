# Kincha Bot — Daily Activity Log

---

## 2026-03-25

**Findings:** 0 new items
**Sources checked:** Pipeline paused until 05:32 UTC; runs at 12:30 + 14:30 UTC (6PM + 8PM IST) resumed — no new findings
**Pipeline runs:** 2 runs scheduled, pipeline_logs not yet writing (prompt update pending)
**System changes:**
- All tasks resumed (were paused Mar 24 for cost optimisation)
- Monitoring schedule changed: 10 runs/day → 2 runs/day (6PM + 8PM IST)
- GitHub token restored and stored in task prompts
- UPDATES.md created as new daily activity log
- DOCUMENTATION.md fully updated with Mar 24 summary
**Pending:**
- Anthropic API key (for Haiku sub-agent calls → drops cost from ~$26 to ~$1/month)
- Email alerts setup (Resend.com or SendGrid API key)
- Fix EMA RSS URL (404)

---

## 2026-03-24

**Findings:** 2 new items saved to Supabase + Neo4j
- 🟡 Sanofi | RA — Licensed KT501 tri-specific T-cell engager from Kali Therapeutics ($1.23B upfront $180M + $1.05B milestones)
- 🟡 Gilead | Lupus/Autoimmune — Acquired Ouro Medicines + OM336/gamgertamig (BCMAxCD3 T-cell engager, Phase 1/2)
- ⚠️ Trend: Two T-cell engager M&A deals same day = sector-wide pivot to BCMA B-cell depletion

**Sources checked:**
- ClinicalTrials.gov: 6 trials found, 0 from target companies
- FDA search: no new immunology approvals
- EMA RSS: 404 error (URL broken)
- Firecrawl: 10 company pages crawled (AbbVie, J&J, BMS, Lilly, Amgen, GSK, Gilead, Sanofi, Novartis, Regeneron)

**Pipeline runs:** 4 runs, 1 error (EMA RSS 404)

**System changes:**
- Added pipeline_logs table to Supabase (audit trail per run)
- Added UPDATES.md to GitHub (this file)
- EOD documentation update task created (11:15PM IST)
- All scheduled tasks PAUSED for cost optimisation

**Cost analysis completed:**
- Current (all Sonnet): ~$26/month
- Optimised (Haiku for Agents 1/2/4): ~$1/month
- Needs: Anthropic API key to implement

**Pending carried forward:**
- Resume tasks after cost optimisation (need Anthropic API key)
- Email alerts setup (need Resend.com or SendGrid key)
- Fix EMA RSS URL

---

*Prepended daily by Kincha bot at 11:15PM IST*

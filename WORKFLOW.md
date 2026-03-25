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

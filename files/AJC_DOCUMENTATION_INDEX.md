# AJC Captain's Log: Complete Documentation Package

**Status:** READY FOR DEVELOPMENT  
**Last Updated:** May 2026  
**All Specs Locked:** ✅ YES

---

## 📋 WHAT'S INCLUDED

This package contains **6 documents** with everything needed to build the AJC Captain's Log app.

---

## 📚 DOCUMENTS (IN ORDER TO READ)

### 1. **AJC_CLAUDE_CODE_PROMPT.md** ⭐ START HERE
**What:** Complete prompt to give to Claude Code (or any developer)
**Contains:** Project overview, build phases, specifications summary, testing checklist, success criteria
**Read time:** 10 minutes
**Use:** Copy this entire prompt and paste into Claude Code to start building

---

### 2. **AJC_MASTER_SUMMARY.md**
**What:** Quick reference guide with all locked decisions
**Contains:** Feature checklist, database tables, API endpoints, calculations, timeline removed, testing checklist, rollout plan
**Read time:** 15 minutes
**Use:** Check this first if you want quick answers about any feature

---

### 3. **AJC_CALCULATIONS_SPEC.md**
**What:** Every metric formula + 8 test cases for verification
**Contains:** Individual metrics, group metrics, test sets with expected outputs, data entry notes, currency handling
**Read time:** 20 minutes
**Use:** Reference for all calculations, run test cases before launch to verify correctness

---

### 4. **AJC_DATABASE_SCHEMA.md**
**What:** Complete PostgreSQL schema with all tables, fields, constraints
**Contains:** 7 tables with SQL, access patterns, cohort isolation, calculated views, seed data example
**Read time:** 20 minutes
**Use:** Copy the SQL and run in Supabase to create database

---

### 5. **AJC_API_SPECIFICATION.md**
**What:** Every API endpoint with request/response formats
**Contains:** 20+ endpoints (auth, profiles, daily logs, group logs, dashboard, admin), error responses, rate limiting
**Read time:** 25 minutes
**Use:** Reference for building backend, implement each endpoint exactly as specified

---

### 6. **AJC_FINAL_LOCKED_SPECS.md**
**What:** Deep dive into three critical decisions
**Contains:** Currency input validation, invite link & cohort registration flow, notification preferences, complete registration flow diagram
**Read time:** 15 minutes
**Use:** Understand cohort isolation and registration flow in detail before building frontend

---

## 🚀 HOW TO USE THIS PACKAGE

### Option A: You're Building It (Recommended)
1. Read **AJC_CLAUDE_CODE_PROMPT.md** first (copy entire content)
2. Paste prompt into Claude Code
3. Let Claude Code follow the specification
4. Reference other documents as needed during development

### Option B: You're Hiring a Developer
1. Give them **AJC_CLAUDE_CODE_PROMPT.md** as their task
2. Give them all 6 documents for reference
3. They can build without asking clarifying questions

### Option C: You're Just Reviewing
1. Read **AJC_MASTER_SUMMARY.md** first (high-level overview)
2. Reference specific documents as needed
3. All decisions are locked in — no changes needed

---

## ✅ KEY FEATURES

**Group Captain's Log**
- Editable table with all users' rows visible
- Each user edits only their own data
- Manual GBP entry with £ symbol pre-filled
- Period selector (pre-school, school, monthly)

**Individual Dashboard**
- Click on any person to see their metrics
- Totals, projections, rankings, sales growth
- ICF global comparison (% vs £41,713)
- Current month totals

**Individual Profile**
- 12 form fields (editable)
- Form answer history with dates (expandable)
- Connected to daily log

**Daily Log**
- Backfillable entries (date picker)
- Auto-sums to monthly (reference only, not source of truth)
- User can batch entries or log daily

**Admin Features**
- Create cohorts (name + year)
- Generate invite links
- Export CSV with timestamp

**Cohort Isolation**
- Users see ONLY their cohort's data
- Invite link pre-fills cohort
- JWT enforces isolation
- Zero cross-cohort leakage

---

## 💾 DATABASE

7 tables, all with cohort_id isolation:
- `cohorts` — AJC 2026, AJC 2027, etc.
- `users` — Coaches in cohorts
- `profiles` — 12 form fields + history
- `profile_history` — Audit trail of form submissions
- `daily_logs` — Backfillable entries
- `group_logs` — Source of truth for group dashboard
- `monthly_periods` — For UI date pickers

All currency stored as **pence (integers)**, displayed as **GBP with commas**.

---

## 🔌 API

20+ endpoints covering:
- Auth (register with invite link, login)
- Profiles (get, update, history)
- Daily logs (create, fetch)
- Group logs (fetch all, update own row)
- Dashboard (totals, tiers, growth, individual)
- Admin (create cohort, export CSV)

Every endpoint filters by `cohort_id` from JWT — **no cross-cohort access possible**.

---

## 📊 CALCULATIONS

All metrics locked in:
- Average monthly income/sales
- Projected annual income/sales
- % vs ICF global average (£41,713)
- Sales growth (pre-school vs school)
- Rankings (conversations, proposals, sales, income, referrals, renewals)
- Sales tier buckets (10k, 25k, 50k, 75k, 100k, 250k, 500k)

**8 test cases provided** to verify every calculation before launch.

---

## 🔒 SECURITY & ISOLATION

**Cohort Isolation (Critical):**
- Database: Foreign key + WHERE clause on every query
- API: JWT includes cohort_id, backend filters by it
- Frontend: Route guards check cohort_id matches
- Registration: Invite link pre-fills cohort (read-only)

**User Data Protection:**
- Only users in same cohort see each other's data
- Admins can only access their own cohort
- CSV export is cohort-specific
- No cross-cohort data leakage possible

---

## 📱 WHAT'S NOT INCLUDED

❌ Notifications or emails  
❌ Real-time updates (refresh to see changes is OK)  
❌ Mobile app (responsive web only)  
❌ Two-factor authentication  
❌ Role-based access (admin vs user only)  
❌ Bulk user import  
❌ User in multiple cohorts  

---

## 🎯 SUCCESS CRITERIA

Before launch:
✅ All 8 calculation test cases pass  
✅ Zero cross-cohort data leakage  
✅ Users register via invite link  
✅ All features working in QA  
✅ Charts display correctly  
✅ Currency formats properly  
✅ Admin export works  
✅ App deployed  

---

## 📖 READING GUIDE

**If you want to...**

**...understand the whole project quickly**
→ Read: AJC_MASTER_SUMMARY.md (15 min)

**...build it with Claude Code**
→ Use: AJC_CLAUDE_CODE_PROMPT.md (copy entire content)

**...verify calculations are correct**
→ Read: AJC_CALCULATIONS_SPEC.md, run 8 test cases

**...create the database**
→ Read: AJC_DATABASE_SCHEMA.md, copy SQL

**...build the API**
→ Read: AJC_API_SPECIFICATION.md, implement each endpoint

**...understand cohort isolation**
→ Read: AJC_FINAL_LOCKED_SPECS.md

**...get a quick feature checklist**
→ Read: AJC_MASTER_SUMMARY.md (top section)

---

## 🚀 NEXT STEPS

1. **If building:** Copy AJC_CLAUDE_CODE_PROMPT.md and start in Claude Code
2. **If hiring:** Give AJC_CLAUDE_CODE_PROMPT.md to developer + all 6 docs
3. **If reviewing:** Read AJC_MASTER_SUMMARY.md first, drill into other docs as needed

**Everything is locked in. No more decisions. Just build!**

---

## 📝 VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | May 2026 | Initial complete specification package |

**Status:** LOCKED FOR DEVELOPMENT ✅

All specifications finalized. No changes after this point.

---

## 📄 FILE CHECKLIST

```
✅ AJC_CLAUDE_CODE_PROMPT.md — Prompt for developers
✅ AJC_MASTER_SUMMARY.md — Quick reference guide
✅ AJC_CALCULATIONS_SPEC.md — Formulas + 8 test cases
✅ AJC_DATABASE_SCHEMA.md — PostgreSQL schema
✅ AJC_API_SPECIFICATION.md — 20+ endpoints
✅ AJC_FINAL_LOCKED_SPECS.md — Currency, registration, isolation
✅ AJC_DOCUMENTATION_INDEX.md — This file
```

**7 documents total. All specifications complete.**

---

**Ready to build? Start with AJC_CLAUDE_CODE_PROMPT.md!** 🚀

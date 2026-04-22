# AJC Captain's Log: Complete Build Prompt for Claude Code

You are tasked with building a complete full-stack web application for AJC Coaching Career School. This prompt contains all specifications needed to build without asking clarifying questions.

---

## PROJECT OVERVIEW

**Application:** AJC Captain's Log - A coaching cohort management and metrics dashboard

**What it does:**
- Coaches log daily conversations, proposals, referrals, renewals, and income
- Group captain's log shows all cohort members' metrics in an editable table
- Individual dashboards show projections, rankings, sales growth, and ICF global comparison
- Admin can create cohorts and export data

**Timeline:** Build as fast as possible (no deadline constraints)

**Users:** Ankush (admin), coaches in cohorts (regular users)

---

## COMPLETE SPECIFICATIONS

You have been provided with 5 specification documents. Read them in this order:

1. **AJC_MASTER_SUMMARY.md** — Start here. Quick reference of all decisions.
2. **AJC_CALCULATIONS_SPEC.md** — Every metric formula + test cases
3. **AJC_DATABASE_SCHEMA.md** — Complete PostgreSQL schema
4. **AJC_API_SPECIFICATION.md** — All 20+ endpoints
5. **AJC_FINAL_LOCKED_SPECS.md** — Currency input, registration flow, cohort isolation details

Read all 5 documents completely before writing any code.

---

## TECH STACK

**Frontend:** React + TypeScript + TailwindCSS + Recharts (for charts)
**Backend:** Node.js + Express + PostgreSQL (via Supabase)
**Authentication:** Supabase Auth (email/password)
**Hosting:** Vercel (frontend + backend)
**Database:** Supabase PostgreSQL

---

## BUILD PHASES (No timeline pressure, go fast)

### Phase 1: Backend Foundation
1. Set up Supabase project (PostgreSQL database)
2. Create all 7 tables (cohorts, users, profiles, profile_history, daily_logs, group_logs, monthly_periods)
3. Implement auth endpoints (register with cohort_id, login)
4. Implement profile endpoints (GET, PUT, history)
5. Test all endpoints with Postman/Insomnia

**Deliverable:** Backend running, auth working, database fully functional

### Phase 2: Core API Endpoints
1. Daily log endpoints (POST, GET)
2. Group log endpoints (GET all, PUT own row)
3. Dashboard endpoints (cohort summary, sales tiers, sales growth, individual dashboard)
4. Admin endpoints (create cohort, export CSV)
5. Test cohort isolation on every endpoint

**Deliverable:** All 20+ API endpoints working, tested

### Phase 3: Frontend Scaffold
1. Initialize React app with TailwindCSS
2. Set up auth context (login state, token storage)
3. Create page layout (navigation, auth pages)
4. Implement login flow (email + password)
5. Implement register flow (invite link pre-fills cohort)

**Deliverable:** Users can register via invite link and log in

### Phase 4: Core Features
1. Group Captain's Log page (fetch all users, editable table, update rows)
2. Individual Profile page (fetch profile, edit form, show history)
3. Daily Log page (date picker, form, save entries)
4. Individual Dashboard modal (click person → see their metrics)

**Deliverable:** All three main views functional

### Phase 5: Dashboard & Calculations
1. Implement all calculation functions (average income, projections, growth, rankings)
2. Build dashboard metric cards (totals, averages, coaches above global avg)
3. Build all charts (top 10, sales tiers, sales growth, comparisons)
4. Hook up individual dashboard with correct metrics

**Deliverable:** Full dashboard with all metrics and charts working

### Phase 6: Admin & Polish
1. Admin panel (create cohort, generate invite link)
2. CSV export with timestamp
3. Full QA and edge case testing
4. Deploy to production

**Deliverable:** App live and fully functional

---

## CRITICAL SPECIFICATIONS

### Database
- Read **AJC_DATABASE_SCHEMA.md** completely
- 7 tables with specific fields, constraints, indexes
- All currency stored as **pence (integers)**, not pounds
- **Cohort isolation:** Every table has cohort_id, every query filters by cohort_id
- Foreign keys and UNIQUE constraints as specified

### API
- Read **AJC_API_SPECIFICATION.md** completely
- 20+ endpoints with exact request/response formats
- JWT authentication: token includes user_id + cohort_id
- Every endpoint filters by cohort_id from JWT
- Error handling: 400, 401, 403, 404, 500 responses
- Rate limiting: 100 requests/minute per user

### Calculations
- Read **AJC_CALCULATIONS_SPEC.md** completely
- **8 test cases provided** — verify every calculation before launching
- Average monthly income = total_income / months_logged
- Projected annual = average_monthly × 12
- % vs ICF = ((projected - 41713) / 41713) × 100
- Sales growth = ((school_sales - preschool_sales) / preschool_sales) × 100
- All tier buckets: 10k, 25k, 50k, 75k, 100k, 250k, 500k

### Registration & Cohort Isolation
- Read **AJC_FINAL_LOCKED_SPECS.md** completely
- **Invite link format:** `https://ajccoach.app/register?cohort=AJC-2026`
- Registration form pre-fills cohort from URL (read-only)
- User created with cohort_id from URL
- JWT issued at login contains cohort_id
- Every API call must filter by cohort_id from JWT
- **No cross-cohort data leakage under any circumstances**

### Currency
- Input: `£ [input]` field — user types "5000" or "5000.50" only
- Storage: Integer pence (5000 → 500000 pence)
- Display: "£5,000.00" with commas and 2 decimals
- Validation: Numbers and decimals only, reject negatives

### Features (Must Have)
- ✅ Group captain's log (editable table, all users' rows visible)
- ✅ Individual profile (12 form fields, editable, history visible)
- ✅ Daily log (backfillable, auto-sums to group log reference)
- ✅ Individual dashboard (totals, projections, rankings, growth, ICF comparison)
- ✅ Dashboard metrics (8 cards, 5 charts, tier buckets)
- ✅ Admin cohort creation and CSV export
- ✅ Cohort isolation at every layer (database, API, JWT, frontend)

### Features (Not Included - Out of Scope)
- ❌ Notifications or emails
- ❌ Real-time updates (refresh to see changes is OK)
- ❌ Mobile app (responsive web only)
- ❌ Two-factor auth
- ❌ Role-based access (admin vs user only)
- ❌ Bulk user import
- ❌ User in multiple cohorts

---

## TESTING CHECKLIST

### Before Launch
- [ ] Run all 8 calculation test cases (match expected outputs exactly)
- [ ] Create 2 cohorts, register users in each, verify no data leakage
- [ ] Edit group log row, verify update works and only own row editable
- [ ] Add daily log entries, verify they don't auto-populate group log (reference only)
- [ ] Update profile, verify history shows previous versions with dates
- [ ] Click person in group log → individual dashboard shows correct metrics
- [ ] View all 5 charts on dashboard, verify data accuracy
- [ ] Test currency with decimals (£5,000.50), negative rejection, zero values
- [ ] Test invite link with invalid cohort → error message
- [ ] Test sales growth with zero pre-school baseline → "N/A"
- [ ] Export CSV, verify no sensitive data leakage, timestamp in filename
- [ ] Test on Chrome, Safari, Firefox
- [ ] Mobile responsive check (TailwindCSS)

---

## KEY IMPLEMENTATION NOTES

### Backend Priorities
1. Get database schema correct first (hardest to change later)
2. Implement cohort isolation filters on EVERY query (no exceptions)
3. Test calculations against spec before UI
4. Use pence (integers) for all currency, never floats
5. Return 403 Forbidden if user tries to access wrong cohort

### Frontend Priorities
1. Set up auth context and JWT token storage first
2. Create route guard that checks cohort_id matches
3. Make group log editable (inline editing or modal)
4. Build individual dashboard as reusable modal component
5. Implement all calculations as pure functions (testable, no side effects)

### Database Priorities
1. Create all tables with correct constraints
2. Add indexes on frequently queried columns (cohort_id, user_id, period, period_type)
3. Seed test data (1 cohort, 5 test users, sample daily/group log entries)
4. Test cohort isolation: write queries that try to cross cohorts, verify they fail

---

## FILE STRUCTURE (Suggested)

```
ajc-coaching-app/
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── profiles.js
│   │   │   ├── daily-logs.js
│   │   │   ├── group-logs.js
│   │   │   ├── dashboard.js
│   │   │   └── admin.js
│   │   ├── middleware/
│   │   │   └── auth.js (JWT verification)
│   │   ├── utils/
│   │   │   ├── calculations.js (all metric formulas)
│   │   │   └── db.js (Supabase client)
│   │   └── index.js (Express server)
│   ├── .env (Supabase credentials)
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── GroupLog.jsx
│   │   │   ├── Profile.jsx
│   │   │   ├── DailyLog.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── IndividualDashboard.jsx
│   │   │   └── Admin.jsx
│   │   ├── pages/
│   │   │   ├── LoginPage.jsx
│   │   │   ├── RegisterPage.jsx
│   │   │   └── DashboardPage.jsx
│   │   ├── context/
│   │   │   └── AuthContext.jsx
│   │   ├── hooks/
│   │   │   ├── useAuth.js
│   │   │   └── useFetch.js
│   │   ├── utils/
│   │   │   ├── api.js (fetch wrapper with auth)
│   │   │   ├── calculations.js (mirror of backend)
│   │   │   └── formatting.js (currency, dates)
│   │   └── App.jsx
│   └── package.json
└── README.md
```

---

## DEPLOYMENT

**Backend:** Deploy Express server to Vercel
**Frontend:** Deploy React app to Vercel
**Database:** Use Supabase (PostgreSQL hosted)
**Environment Variables:** Store in Vercel secrets

---

## SUCCESS CRITERIA

✅ All 8 calculation test cases pass
✅ Zero cross-cohort data leakage
✅ Users can register via invite link and see cohort pre-filled
✅ Users can only edit their own group log row
✅ Individual dashboard shows correct metrics and rankings
✅ All 5 dashboard charts render correctly
✅ Currency displays with commas and decimals
✅ Admin can export CSV with timestamp
✅ App deployed and live
✅ Full QA completed with zero critical bugs

---

## NOTES

- **No scope creep:** Only build what's in the specs
- **Simplicity first:** If something takes > 4 hours, simplify or ask
- **Test as you go:** Don't wait until end to test
- **Cohort isolation is critical:** Spend extra time verifying this
- **Calculations must be exact:** Use the 8 test cases to verify before launch
- **Currency handling:** Use integers (pence), never floats, to avoid rounding errors

---

## START HERE

1. Read all 5 specification documents (takes ~30 minutes)
2. Set up Supabase and create PostgreSQL tables
3. Write and test calculation functions against 8 test cases
4. Implement backend auth endpoints
5. Build React scaffold and auth flow
6. Implement each feature phase by phase
7. Test thoroughly before deploying

**You have all the information you need. Build fast and well!**

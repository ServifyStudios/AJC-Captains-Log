# Project Context

**Last Updated:** 2026-04-22

## What We've Built So Far

Single-file HTML prototype: `prototype.html` — a coaching business tracking app for the AJC (Ankush Jain Coaching) 2026 cohort of 5 coaches.

### Pages
- **Login** — email + password, quick-fill buttons for demo users, forgot password link
- **Register** — invite-link cohort onboarding
- **Forgot Password** — email form + confirmation state
- **Main App** (4 tabs):
  - **Captain's Log** — editable monthly data table (conversations, proposals, sales, income, referrals, renewals) with period switcher (pre-school Mar–Aug / school year Sep–Feb) and group filter
  - **Dashboard** — cohort view (summary stats, 6 ranking charts, Sales Board, cohort sales growth) + individual view (hero acknowledgement, school year totals, 4 metric cards, sales growth chart, ICF comparison, "Learn more" profile modal)
  - **Daily Log** — personal entry form with ⓘ info tooltips on each field
  - **My Profile** — 10-question coaching practice profile, version history (current / Nov / Sep), "Ask AJC group" copy-to-clipboard button
  - **Admin** (admin only) — cohort management, mentoring groups, invite link generator, CSV export

### Key Features
- Period switcher: pre-school (6 months) vs school year (6 months), month-by-month selection
- Mentoring groups: coaches grouped under a mentor, captain's log filterable by group
- Sales Board: 7 tier columns (£10k → £500k+) based on projected annual sales
- Rankings: 6 horizontal bar charts (conversations, proposals, sales, income, referrals, renewals)
- Profile modal: "Learn more about [Coach]" opens polished modal with AJC logo + formatted answers
- ICF comparison: only shown when coach is above the £41,713/yr global average
- Weekly rotating acknowledgement in individual hero banner

## Tech Stack

- **Single HTML file** — no build step, no backend
- **Tailwind CSS** via CDN
- **Chart.js** via CDN
- **No framework** — vanilla JS
- All data is mock/in-memory (MOCK_USERS, LOG_DATA, SCHOOL_TOTALS, COACH_PROFILES, PROFILE_DATA)

## Brand

- Navy: `#020e1d`
- Gold: `#bea566`
- Blue-grey: `#637480` / `#92a6b3`
- AJC logo: inline SVG (4 paths)
- Icons: minimal stroke SVGs (stroke-width 1.5–2, no fill)
- Nav active tab: white background + navy text + subtle shadow

## Data Model

- Currency stored in **pence** (integers), displayed via `p2gbp()` helper
- `MONTHS_IN_SCHOOL = 6` (Sep–Feb)
- `ICF_AVG = 41713` (£/year, used for comparison)
- `preSales` field on SCHOOL_TOTALS = 6-month pre-school sales total

## Key Functions

- `renderCaptainsLog()` — renders editable table for current period/month
- `renderIndividual(userId)` — populates individual dashboard, updates Chart.js instance
- `initCharts()` — creates 6 ranking charts + cohort sales growth chart + sales board (runs once on first dashboard visit)
- `renderSalesBoard()` — places coaches in highest qualifying tier column
- `openProfileModal()` — opens coach profile modal from `learn-more-btn._userId`
- `showColInfo(event, key)` / `showDailyInfo(event, key)` / `showIcfInfo(event)` — shared tooltip system using `#col-tooltip`
- `copyAskMsg()` — copies question to clipboard, opens confirmation modal
- `loadProfileVersion(ver)` — fills profile form from PROFILE_DATA[ver]

## Current Focus

Mobile responsiveness — ensuring all views work well across phone, tablet, and desktop.

## Key Technical Decisions

- Single-file prototype for easy sharing and iteration (no deployment needed initially)
- Chart.js instances persisted in variables (`_indSalesChart`) to avoid "canvas already in use" errors
- ICF card hidden when coach is below average (only shown as positive achievement)
- Sales Board placement: coach goes in highest qualifying tier (not lowest)
- Period banner removed to reduce clutter — period context shown inline in pills only
- Backdrop click on modals only closes when clicking the dark overlay itself (not elements outside inner)

## Next Steps

1. **Mobile polish** — test and fix layout on 375px, 768px, 1280px breakpoints
2. **Upload to GitHub** — create repo, push single-file prototype
3. **Deploy to Vercel** — connect GitHub repo, auto-deploy on push
4. **Real backend** — replace mock data with Supabase or similar (auth, database, real-time updates)
5. **Coach onboarding flow** — invite link → register → profile completion

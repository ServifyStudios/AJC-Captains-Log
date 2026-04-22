# AJC Captain's Log: MASTER LOCKED SPECIFICATIONS
**Status: READY FOR DEVELOPMENT**

---

## Quick Reference

| Item | Decision |
|------|----------|
| **Timeline** | 5 weeks (May 1 - June 5, 2026) |
| **Tech Stack** | React + Node.js + PostgreSQL (Supabase) |
| **Cohort Naming** | Name tag (e.g., "AJC") + Year dropdown (2026, 2027...) → "AJC 2026" |
| **User Registration** | Invite link with cohort in URL (e.g., `?cohort=AJC-2026`) |
| **Cohort Isolation** | Every user sees ONLY their cohort's data, JWT enforces this |
| **Currency Input** | £ symbol pre-filled, user types numbers only (e.g., "5000") |
| **Storage** | All GBP in pence (integers), displayed with commas and decimal |
| **Notifications** | None (no emails, no reminders) |
| **Pre-School Data** | Manually entered starting in April, shown in separate view |
| **Group Log Editing** | Each user edits only their own row, others can't edit their data |
| **Individual Dashboard** | Click person in group log → see their totals, projections, rankings, growth |
| **Admin Export** | CSV with timestamp, admin-only, cohort-specific |

---

## Complete Feature List

### **Group Captain's Log (Editable Table)**
- ✅ All users' rows visible to everyone in cohort
- ✅ Each user edits only their own row (conversations, proposals, sales, income, referrals, renewals)
- ✅ Manual GBP entry: `£ [number]` format
- ✅ Period selector: Apr-Sep (pre-school), Oct-Mar (school), monthly (May, June, etc.)
- ✅ Auto-calculation: Click on person → individual dashboard modal

### **Individual Dashboard (Modal)**
- ✅ Totals: conversations, proposals, sales, income, referrals, renewals
- ✅ Projections: average monthly, projected annual
- ✅ ICF comparison: % above/below £41,713 global average
- ✅ Sales growth: pre-school total vs school total (actual, not estimated)
- ✅ Rankings: rank in each category among cohort
- ✅ Current month totals shown on their profile page

### **Individual Profile Page**
- ✅ 12 form fields (coaching experience, ideal clients, challenge, etc.)
- ✅ Editable by user only
- ✅ Form answer history: expandable card showing previous submissions with dates
- ✅ Timestamp of last edit displayed
- ✅ Current month totals from daily log shown below form

### **Daily Log**
- ✅ Date picker (can backfill past entries)
- ✅ Fields: conversations, proposals (amount in GBP), referrals, renewals, income
- ✅ Saves to daily_logs table
- ✅ Displayed as reference only (group log is source of truth)
- ✅ User can enter daily or batch multiple entries

### **Group Dashboard (Metrics)**
- ✅ Total conversations, proposals, sales, income, referrals, renewals
- ✅ Average cohort income
- ✅ Coaches above ICF global average (£41,713)
- ✅ Top 10 charts: conversations, proposals, income, referrals, renewals
- ✅ Sales tiers: count in each bucket (10k, 25k, 50k, 75k, 100k, 250k, 500k)
- ✅ Hover tooltip on tiers: "£X left for next tier"
- ✅ Sales growth comparison: pre-school vs school with % growth
- ✅ Individual dashboard accessible by clicking any person's row

### **Admin Features**
- ✅ Create new cohort (name + year)
- ✅ Generate invite links: `https://ajccoach.app/register?cohort=AJC-2026`
- ✅ Export cohort data as CSV with timestamp
- ✅ Admin-only access (one email per cohort)

### **Authentication & Cohort Isolation**
- ✅ Email/password registration via invite link
- ✅ Cohort pre-filled in registration URL (read-only)
- ✅ JWT token includes cohort_id
- ✅ Every API query filters by cohort_id from JWT
- ✅ User cannot access other cohorts
- ✅ No cross-cohort data leakage

---

## Database Tables (Final)

```sql
cohorts (id, name, year, admin_email, is_active, created_at, updated_at)
users (id, cohort_id, email, first_name, last_name, is_admin, created_at, updated_at)
profiles (id, user_id, cohort_id, 12_form_fields, filled_at, created_at, updated_at)
profile_history (id, user_id, cohort_id, 12_form_fields, filled_at, created_at)
daily_logs (id, user_id, cohort_id, entry_date, conversations, proposals_pence, income_pence, referrals, renewals, created_at, updated_at)
group_logs (id, user_id, cohort_id, period, period_type, conversations, proposals_pence, sales_pence, income_pence, referrals, renewals, created_at, updated_at)
monthly_periods (id, cohort_id, month, year, month_number, period_type, created_at)
```

---

## API Endpoints (Final)

### Auth
- POST /auth/register (cohort_id from invite link)
- POST /auth/login

### Profiles
- GET /profiles/me
- PUT /profiles/me
- GET /profiles/{user_id}
- GET /profiles/{user_id}/history

### Daily Logs
- POST /daily-logs
- GET /daily-logs

### Group Logs
- GET /group-logs (all users' rows for cohort)
- PUT /group-logs/{id} (edit own row only)
- GET /group-logs/{user_id}/individual (individual dashboard)

### Dashboard
- GET /dashboard/cohort-summary (totals, averages, top 10)
- GET /dashboard/sales-tiers (tier buckets with counts)
- GET /dashboard/sales-growth (pre-school vs school)

### Admin
- POST /admin/cohorts (create cohort)
- GET /admin/export (CSV export)

---

## Calculations (Locked)

### Individual Metrics
- **Average monthly income** = total_income / months_logged
- **Projected annual income** = average_monthly_income × 12
- **% vs ICF** = ((projected_annual - 41713) / 41713) × 100
- **Average monthly sales** = total_proposals / months_logged
- **Projected annual sales** = average_monthly_sales × 12
- **Sales growth** = ((school_sales - preschool_sales) / preschool_sales) × 100
- **Amount left for tier** = next_tier_min - projected_annual_sales

### Group Metrics
- **Total conversations** = SUM(all users' conversations)
- **Total proposals** = SUM(all users' proposals_pence)
- **Total income** = SUM(all users' income_pence)
- **Average cohort income** = total_income / total_users
- **Coaches above global avg** = COUNT(users WHERE projected_annual > 41713)
- **Sales tiers** = COUNT(users in each bracket)

---

## Currency Handling (Final)

**Input:** `£ [input]` — user types "5000" or "5000.50"
**Storage:** Integer pence (500000 or 500050)
**Display:** "£5,000.00" with commas and 2 decimals
**Validation:** Numbers and decimals only, reject negatives

---

## Registration & Cohort Flow (Final)

```
1. Admin creates cohort: name="AJC", year=2026
2. System generates: display_name="AJC 2026", invite_link="...?cohort=AJC-2026"
3. Admin sends link to coaches
4. Coach clicks link → registration page pre-fills cohort
5. Coach registers with email, password, name
6. User created with cohort_id in database
7. Coach logs in with email + password
8. Backend issues JWT with user_id + cohort_id
9. Frontend stores JWT, loads dashboard for that cohort only
10. Every API call includes JWT, backend filters by cohort_id
11. User cannot see other cohorts or cross-cohort data
```

---

## Testing Checklist (Pre-Launch)

### Calculations
- [ ] Test set 1: Average income/sales calculations
- [ ] Test set 2: Zero income edge case
- [ ] Test set 3: Sales growth calculation
- [ ] Test set 4: Sales growth with zero baseline
- [ ] Test set 5: Sales tier bucket assignments
- [ ] Test set 6: Amount left for next tier (hover)
- [ ] Test set 7: Coaches above global average
- [ ] Test set 8: Group dashboard totals match spreadsheet

### User Flows
- [ ] Registration with invite link
- [ ] Login and cohort isolation
- [ ] Edit own group log row
- [ ] Try to edit someone else's row → should fail
- [ ] View own profile
- [ ] Edit profile and see history
- [ ] Enter daily log entries
- [ ] Backfill past daily entries
- [ ] Click person in group log → individual dashboard
- [ ] View all charts on dashboard

### Cohort Isolation
- [ ] Create two cohorts
- [ ] Register users in each cohort
- [ ] User A cannot see cohort B's data
- [ ] API returns 403 if wrong cohort_id
- [ ] Database contains no cross-cohort leakage

### Admin
- [ ] Create cohort and copy invite link
- [ ] Send invite link (manual copy-paste)
- [ ] User registers via link
- [ ] Export CSV with correct data
- [ ] Export includes timestamp in filename
- [ ] Only admin email can export

### Edge Cases
- [ ] Currency with decimals (£5,000.50)
- [ ] Negative currency → rejected
- [ ] User with zero income
- [ ] No pre-school baseline → sales growth shows "N/A"
- [ ] Null values handled as zero
- [ ] Very large numbers (£999,999,999)
- [ ] Missing daily log entry for a day (total still calculates)

---

## Rollout Plan

### Week 1-4: Development
- Database, API, frontend features built and tested locally

### Week 5: Deployment & Testing
- Deploy to production
- Admin creates test cohort
- Internal team tests full flow
- Fix any bugs

### Week 5 (Day 5): Launch
- Send invite links to early cohort
- Users register and start logging data
- Monitor for issues and provide live support

### Post-Launch
- Monitor API logs for errors
- Gather user feedback
- Plan Phase 2 improvements

---

## What's NOT Included (Out of Scope)

- ❌ Mobile app (responsive web only)
- ❌ Notifications or reminders
- ❌ Real-time updates (refresh page to see changes)
- ❌ Role-based access (only admin vs user)
- ❌ Bulk user import
- ❌ Two-factor authentication
- ❌ User can be in multiple cohorts
- ❌ Custom report builder
- ❌ Integrations (Stripe, HubSpot, etc.)

---

## Key Decisions (Finalized)

✅ **Cohort naming:** Simple (name + year)
✅ **Registration:** Invite links with cohort in URL
✅ **Isolation:** JWT-based, enforced at API layer
✅ **Currency:** Pre-filled £ symbol, numbers only
✅ **Notifications:** None
✅ **Data entry:** Manual (no auto-import or sync)
✅ **Pre-school data:** Separate view, manually entered

---

## Files to Implement

1. **AJC_CALCULATIONS_SPEC.md** — All formulas + 8 test cases
2. **AJC_DATABASE_SCHEMA.md** — Complete PostgreSQL schema
3. **AJC_API_SPECIFICATION.md** — 20+ endpoints, request/response
4. **AJC_5WEEK_TIMELINE.md** — Day-by-day breakdown
5. **AJC_FINAL_LOCKED_SPECS.md** — Currency, registration, cohort isolation

---

## Ready to Build

✅ All specs finalized and locked
✅ No ambiguity remaining
✅ Ready for developer(s) to start
✅ 5-week timeline achievable
✅ Zero scope creep after development starts

**Next Step:** Hand these 5 documents to your development team and begin Week 1.

---

**Last Updated:** May 2026
**Status:** LOCKED FOR DEVELOPMENT
**Approval:** ✅ All final decisions made and verified

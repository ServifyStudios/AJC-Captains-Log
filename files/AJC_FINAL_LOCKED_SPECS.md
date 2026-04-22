# AJC Captain's Log: Final Locked Specifications

## 1. Currency Input Validation

**Implementation:**
- All GBP input fields display as: `£ [input box]`
- User can only type numbers (0-9, decimal point)
- No manual £ symbol entry allowed
- Frontend regex: `/^[\d.]*$/` (numbers and decimals only)
- Backend validation: Reject any non-numeric characters

**Example UI:**
```
Proposals Amount: £ [5000.50]
Sales Amount:     £ [10000]
Income:           £ [3500.75]
```

**Storage:**
- Parse "5000.50" → 500050 pence (integer)
- Display 500050 pence → "£5,000.50" with commas

**Edge Cases:**
- User types "5000." → accept (trailing decimal)
- User types ".5" → accept (leading decimal, convert to 0.5)
- User types "5000.5.5" → reject (multiple decimals)
- User types "-5000" → reject (negative)

---

## 2. Invite Link & Cohort Registration Flow

**URL Structure:**
```
https://ajccoach.app/register?cohort=AJC-2026
```

**How It Works:**

### Step 1: Admin Creates Cohort
- Ankush logs in with admin@example.com
- Clicks "Create New Cohort"
- Enters: name="AJC", year=2026
- System creates cohort with display name "AJC 2026"
- Admin gets invite link to copy/send: `https://ajccoach.app/register?cohort=AJC-2026`

### Step 2: User Clicks Invite Link
- Coach receives email with link: "Join AJC 2026: https://ajccoach.app/register?cohort=AJC-2026"
- Clicks link → registration page pre-populates cohort "AJC 2026" (read-only display)
- Fills in: email, password, first_name, last_name
- Submits → creates user in cohort_id associated with "AJC 2026"
- Redirected to login page

### Step 3: User Logs In
- Any email can try to login
- If email not found → "User not found" error
- If email found → check cohort_id
- Load dashboard for THAT cohort only
- User sees **only their cohort's data** (group log, other users in same cohort, etc.)
- User **cannot see other cohorts** even if they try to hack the URL

**Cohort Isolation After Login:**
```javascript
// Frontend route guard example
if (user.cohort_id !== currentCohortId) {
  redirect('/cohort-selector'); // or back to their own cohort
}

// Backend API filter
SELECT * FROM group_logs 
WHERE cohort_id = $1 AND user_id = $2; 
// where $1 = user.cohort_id from JWT
```

**Database Layer:**
- JWT token includes `cohort_id`
- Every API query auto-filters by `WHERE cohort_id = decoded_token.cohort_id`
- User cannot specify different cohort_id in API calls

**URL Parameter Validation:**
```javascript
// On registration page load
const cohortParam = new URLSearchParams(window.location.search).get('cohort');
// Example: cohortParam = "AJC-2026"

// Convert to cohort_id via API call
const cohortData = await fetch(`/api/cohorts/lookup?name=${cohortParam}`);
const { cohort_id } = cohortData;

// If cohort not found or malformed → show error
if (!cohortData.ok) {
  show("Invalid invite link. Contact admin.");
}
```

**API Endpoint for Registration:**
```
POST /auth/register
{
  "email": "john@example.com",
  "password": "password123",
  "first_name": "John",
  "last_name": "Smith",
  "cohort_id": "uuid_of_ajc_2026" // Provided by invite link lookup
}
```

**Invite Link Format:**
- `cohort` parameter = `${name}-${year}` (e.g., "AJC-2026", "Spring-2026")
- Simple, human-readable, no UUIDs in URL
- Backend looks up cohort by name + year

---

## 3. Notifications

**No email notifications.**

What is turned OFF:
- ❌ Email when someone updates their profile
- ❌ Daily reminders to log conversations
- ❌ Admin alerts for missing data
- ❌ Weekly digests
- ❌ Any transactional email beyond registration/password reset

**Only notifications sent:**
- ✅ Account creation confirmation (Supabase Auth standard)
- ✅ Invite link email (admin manually sends, or app generates link for copy-paste)
- ✅ Password reset (if user requests)

---

## Complete Registration & Login Flow

```
REGISTRATION FLOW:
1. Admin creates cohort "AJC 2026"
2. Admin copies invite link: https://ajccoach.app/register?cohort=AJC-2026
3. Admin sends link to coach (email, Slack, etc.)
4. Coach clicks link
5. Frontend parses cohort parameter "AJC-2026"
6. Frontend calls GET /api/cohorts/lookup?name=AJC&year=2026 → returns cohort_id
7. Registration form displays "Joining: AJC 2026" (read-only)
8. Coach enters email, password, first_name, last_name
9. Coach submits → POST /auth/register with cohort_id
10. Backend creates user in that cohort
11. Redirect to /login
12. Coach logs in with email + password
13. Backend issues JWT containing user_id + cohort_id
14. Frontend stores token in localStorage
15. Coach sees dashboard for AJC 2026 only

LOGIN FLOW (repeat visits):
1. Coach goes to https://ajccoach.app
2. Frontend checks if token in localStorage
3. If token exists and not expired → decode cohort_id → load dashboard for that cohort
4. If no token or expired → show login page
5. Coach enters email + password
6. Backend verifies credentials + issues JWT
7. Frontend loads dashboard for cohort in JWT
8. User navigates within AJC 2026 only

COHORT ISOLATION:
- Every API call includes Authorization: Bearer {jwt}
- Backend decodes JWT → extracts cohort_id
- Every query filters: WHERE cohort_id = jwt.cohort_id
- User cannot access another cohort's data via URL hacking
- If user tries to access /dashboard?cohort_id=xyz, backend rejects (wrong cohort)
```

---

## Admin Invite Link Management

### Option A: Manual Copy-Paste (Simplest)
```
1. Admin clicks "Create Cohort"
2. Enters: name="AJC", year=2026
3. System shows: "Invite link: https://ajccoach.app/register?cohort=AJC-2026"
4. Admin copies link
5. Admin pastes into email, Slack, Google Sheets, etc.
```

### Option B: Built-in Email (More Polished)
```
1. Admin clicks "Create Cohort"
2. Enters: name="AJC", year=2026
3. Admin clicks "Send Invite" button
4. Modal appears: "Enter email addresses (comma-separated)"
5. Admin pastes: john@example.com, mary@example.com, lara@example.com
6. System sends email with pre-filled invite link
7. Coaches click link → pre-populated cohort
```

**Recommendation:** Start with Option A (simpler, zero email infrastructure), add Option B in Phase 2 if needed.

---

## Implementation Notes

### Database
- No changes needed — cohort_id already the foreign key
- Cohort lookup by (name, year) fast with index

### Frontend
- Registration page: Read cohort from URL, display it, prevent user from changing it
- Login page: Standard (email + password only)
- After login: All routes check `user.cohort_id` matches `currentCohortId`
- If mismatch: Show "Invalid cohort access" or redirect to correct cohort

### Backend
- New endpoint: `GET /api/cohorts/lookup?name=${name}&year=${year}` → returns cohort_id
- Update: `POST /auth/register` to accept cohort_id (required, validated against URL)
- JWT now includes cohort_id (already in schema)
- Auth middleware: Extract cohort_id from JWT, use in every query filter
- Error responses for invalid cohort lookups (not found, malformed URL, etc.)

### Security
- URL parameter is not sensitive (cohort names are public)
- No authentication token in URL (only in JWT after login)
- Each user can only see data for cohorts they're registered in
- Admin email restriction: Only that email can create cohorts, export data, manage that cohort

---

## Example URLs

**Invite Link:**
- `https://ajccoach.app/register?cohort=AJC-2026`
- `https://ajccoach.app/register?cohort=Spring-2026`
- `https://ajccoach.app/register?cohort=Fall-2026`

**Login (no cohort in URL):**
- `https://ajccoach.app/login`

**Dashboard (after login, cohort determined by JWT):**
- `https://ajccoach.app/dashboard`
- User sees only their cohort's data, based on JWT

**Cohort Selection (if user is in multiple cohorts):**
- Future feature, not in Phase 1
- For now: one email = one cohort

---

## Testing Checklist

- [ ] Create cohort "AJC 2026"
- [ ] Copy invite link: https://ajccoach.app/register?cohort=AJC-2026
- [ ] Visit link in new browser (incognito)
- [ ] Verify cohort name displayed as "AJC 2026" (read-only)
- [ ] Register with test email + password
- [ ] Verify user created in correct cohort_id
- [ ] Login with that email + password
- [ ] Verify JWT contains correct cohort_id
- [ ] Verify dashboard shows only AJC 2026 data
- [ ] Try to manually change URL cohort parameter → verify not allowed
- [ ] Create second cohort "AJC 2027"
- [ ] Register different user in "AJC 2027"
- [ ] Verify both users see only their cohort's data
- [ ] Verify no cross-cohort data leakage in API responses
- [ ] Test invalid cohort link → show error "Invalid invite link"
- [ ] Test malformed cohort parameter → show error

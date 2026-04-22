# AJC Captain's Log: Database Schema

## Overview
- **Database:** PostgreSQL (via Supabase)
- **Auth:** Email/password (Supabase Auth)
- **Cohort isolation:** Every table includes `cohort_id` as a foreign key filter

---

## Tables

### `cohorts`
Represents a school cohort (e.g., "AJC 2026")

```sql
CREATE TABLE cohorts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL, -- e.g., "AJC"
  year INTEGER NOT NULL, -- 2026, 2027, etc.
  admin_email VARCHAR(255) NOT NULL UNIQUE,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(name, year)
);

-- Indexes
CREATE INDEX idx_cohorts_admin_email ON cohorts(admin_email);
CREATE INDEX idx_cohorts_year ON cohorts(year);
CREATE INDEX idx_cohorts_is_active ON cohorts(is_active);
```

**Fields:**
- `id`: Unique identifier
- `name`: "AJC" (or whatever tag, e.g., "Spring", "Fall")
- `year`: 2026, 2027, 2028, etc. (dropdown selector in UI)
- `admin_email`: Only this email can export all cohort data
- `is_active`: Soft delete (archive cohorts without losing data)
- `created_at`, `updated_at`: Audit timestamps

**Display:** "AJC 2026" (name + year automatically combined in UI)

---

### `users`
Coaches in the school

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT auth.uid(), -- Links to Supabase Auth
  cohort_id UUID NOT NULL REFERENCES cohorts(id) ON DELETE CASCADE,
  email VARCHAR(255) NOT NULL,
  first_name VARCHAR(255),
  last_name VARCHAR(255),
  is_admin BOOLEAN DEFAULT false, -- true if email = cohorts.admin_email
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(cohort_id, email)
);

-- Indexes
CREATE INDEX idx_users_cohort_id ON users(cohort_id);
CREATE INDEX idx_users_email ON users(email);
```

**Fields:**
- `id`: Supabase Auth UID
- `cohort_id`: Which cohort they belong to
- `email`: For login (unique per cohort)
- `first_name`, `last_name`: Display name
- `is_admin`: Can export all cohort data if true

---

### `profiles`
Coach's personal coaching business information (12 form fields)

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  cohort_id UUID NOT NULL REFERENCES cohorts(id) ON DELETE CASCADE,
  
  -- 12 Form Fields
  coaching_experience VARCHAR(255), -- "How long have you been a professional coach?"
  ideal_clients TEXT, -- "Who are your ideal clients?"
  current_challenge TEXT, -- "What's your current challenge?"
  num_current_clients INTEGER, -- "How many clients do you currently have?"
  client_types TEXT, -- "Paying/non-paying breakdown"
  full_practice_target INTEGER, -- "How many clients make your practice full?"
  work_format TEXT, -- "How do people work with you?" (Session types, in-person, online, etc.)
  pricing_amount INTEGER, -- In pence (£X * 100)
  client_sources TEXT, -- "Where do your clients come from?" (Ranking/breakdown)
  form_insights TEXT, -- "Has completing this form revealed anything..."
  six_month_goals TEXT, -- "What are your main goals for 6-12 months?"
  support_questions TEXT, -- "Do you have any questions?"
  
  filled_at TIMESTAMP, -- Timestamp of when this version was filled
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_profiles_user_id ON profiles(user_id);
CREATE INDEX idx_profiles_cohort_id ON profiles(cohort_id);
CREATE INDEX idx_profiles_filled_at ON profiles(filled_at);
```

**Fields:**
- `user_id`: Link to user
- `cohort_id`: Redundant but for fast filtering
- 12 form fields (all nullable initially)
- `filled_at`: Timestamp of this response set
- Multiple rows possible (user can re-fill form) — query by `filled_at` DESC LIMIT 1 for latest

---

### `profile_history`
Complete history of all form answers

```sql
CREATE TABLE profile_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  cohort_id UUID NOT NULL REFERENCES cohorts(id) ON DELETE CASCADE,
  
  -- Copy all 12 fields + metadata
  coaching_experience VARCHAR(255),
  ideal_clients TEXT,
  current_challenge TEXT,
  num_current_clients INTEGER,
  client_types TEXT,
  full_practice_target INTEGER,
  work_format TEXT,
  pricing_amount INTEGER,
  client_sources TEXT,
  form_insights TEXT,
  six_month_goals TEXT,
  support_questions TEXT,
  
  filled_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_profile_history_user_id ON profile_history(user_id);
CREATE INDEX idx_profile_history_cohort_id ON profile_history(cohort_id);
CREATE INDEX idx_profile_history_filled_at ON profile_history(filled_at DESC);
```

**Purpose:** Immutable log of every form submission. When user updates their profile, insert old data here, update profiles table.

---

### `daily_logs`
Individual daily entries (conversations, proposals, referrals, renewals, income)

```sql
CREATE TABLE daily_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  cohort_id UUID NOT NULL REFERENCES cohorts(id) ON DELETE CASCADE,
  
  entry_date DATE NOT NULL, -- Can be backdated
  num_conversations INTEGER DEFAULT 0,
  num_proposals INTEGER DEFAULT 0,
  proposals_amount_pence INTEGER DEFAULT 0, -- In pence (£X * 100)
  num_referrals INTEGER DEFAULT 0,
  num_renewals INTEGER DEFAULT 0,
  income_pence INTEGER DEFAULT 0, -- In pence
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, entry_date) -- One entry per user per day
);

-- Indexes
CREATE INDEX idx_daily_logs_user_id ON daily_logs(user_id);
CREATE INDEX idx_daily_logs_cohort_id ON daily_logs(cohort_id);
CREATE INDEX idx_daily_logs_entry_date ON daily_logs(entry_date);
```

**Fields:**
- `entry_date`: User-specified date (can backfill)
- All numeric fields default to 0
- `proposals_amount_pence` and `income_pence` stored as integers (pence, not pounds)

---

### `group_logs`
Monthly summaries per user (source of truth for dashboard)

```sql
CREATE TABLE group_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  cohort_id UUID NOT NULL REFERENCES cohorts(id) ON DELETE CASCADE,
  
  period VARCHAR(50) NOT NULL, -- e.g., "May 2026" or "April-September 2026"
  period_type VARCHAR(50) NOT NULL, -- "monthly", "preschool", "school"
  
  num_conversations INTEGER DEFAULT 0,
  num_proposals INTEGER DEFAULT 0,
  proposals_amount_pence INTEGER DEFAULT 0, -- Editable by user
  sales_amount_pence INTEGER DEFAULT 0, -- Editable by user
  income_pence INTEGER DEFAULT 0, -- Editable by user
  num_referrals INTEGER DEFAULT 0, -- Editable by user
  num_renewals INTEGER DEFAULT 0, -- Editable by user
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, cohort_id, period, period_type)
);

-- Indexes
CREATE INDEX idx_group_logs_user_id ON group_logs(user_id);
CREATE INDEX idx_group_logs_cohort_id ON group_logs(cohort_id);
CREATE INDEX idx_group_logs_period ON group_logs(period);
CREATE INDEX idx_group_logs_period_type ON group_logs(period_type);
```

**Fields:**
- `period`: "May 2026" or "April-September 2026" (for pre-school/school)
- `period_type`: "monthly" (May), "preschool" (Apr-Sep), "school" (Oct-Mar)
- All numeric fields **manually entered by user** (editable textbox in UI)
- These are the source of truth for dashboard calculations

**Note:** Daily logs feed into this as a *helper* reference, but users can edit group_logs directly.

---

### `monthly_periods`
Metadata for which months belong to which periods (for UI date range pickers)

```sql
CREATE TABLE monthly_periods (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cohort_id UUID NOT NULL REFERENCES cohorts(id) ON DELETE CASCADE,
  
  month VARCHAR(20) NOT NULL, -- "April", "May", "June", etc.
  year INTEGER NOT NULL,
  month_number INTEGER NOT NULL, -- 1-12
  period_type VARCHAR(50) NOT NULL, -- "preschool", "school", or specific label
  
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(cohort_id, month, year)
);

-- Indexes
CREATE INDEX idx_monthly_periods_cohort_id ON monthly_periods(cohort_id);
```

**Purpose:** Quick lookup of which months belong to pre-school vs school without hardcoding dates in app.

---

## Calculated Views (for Dashboard Queries)

These aren't tables, but SQL queries that aggregate data:

### Monthly Cohort Totals (for Group Dashboard)
```sql
SELECT 
  period,
  SUM(num_conversations) as total_conversations,
  SUM(proposals_amount_pence) as total_proposals_pence,
  SUM(sales_amount_pence) as total_sales_pence,
  SUM(income_pence) as total_income_pence,
  SUM(num_referrals) as total_referrals,
  SUM(num_renewals) as total_renewals,
  COUNT(user_id) as total_users
FROM group_logs
WHERE cohort_id = $1 AND period = $2
GROUP BY period;
```

### Per-User Totals (for Individual Dashboard)
```sql
SELECT 
  user_id,
  SUM(num_conversations) as total_conversations,
  SUM(proposals_amount_pence) as total_proposals_pence,
  SUM(sales_amount_pence) as total_sales_pence,
  SUM(income_pence) as total_income_pence,
  SUM(num_referrals) as total_referrals,
  SUM(num_renewals) as total_renewals,
  COUNT(DISTINCT period) as months_logged
FROM group_logs
WHERE cohort_id = $1 AND period_type IN ('monthly', 'preschool', 'school')
GROUP BY user_id;
```

### Sales Growth (Pre-School vs School)
```sql
SELECT 
  user_id,
  SUM(CASE WHEN period_type = 'preschool' THEN sales_amount_pence ELSE 0 END) as preschool_sales_pence,
  SUM(CASE WHEN period_type = 'school' THEN sales_amount_pence ELSE 0 END) as school_sales_pence
FROM group_logs
WHERE cohort_id = $1
GROUP BY user_id;
```

### Rankings (for Dashboard)
```sql
SELECT 
  user_id,
  RANK() OVER (ORDER BY SUM(num_conversations) DESC) as conversations_rank,
  RANK() OVER (ORDER BY SUM(proposals_amount_pence) DESC) as proposals_rank,
  RANK() OVER (ORDER BY SUM(sales_amount_pence) DESC) as sales_rank,
  RANK() OVER (ORDER BY SUM(income_pence) DESC) as income_rank,
  RANK() OVER (ORDER BY SUM(num_referrals) DESC) as referrals_rank,
  RANK() OVER (ORDER BY SUM(num_renewals) DESC) as renewals_rank
FROM group_logs
WHERE cohort_id = $1 AND period_type = $2 -- e.g., period_type = 'monthly'
GROUP BY user_id;
```

---

## Data Flow

```
User fills daily log
  ↓
INSERT into daily_logs
  ↓
(Optional) Frontend displays daily entries as reference
  ↓
User edits group_logs row (conversations, proposals, sales, income, referrals, renewals)
  ↓
UPDATE group_logs
  ↓
Dashboard calculates from group_logs (not daily_logs)
```

---

## Constraints & Validation

1. **Cohort Isolation:** Every query includes `WHERE cohort_id = $1`
2. **Currency:** All currency stored in **pence** (integers), displayed as GBP
3. **Unique Email per Cohort:** `UNIQUE(cohort_id, email)` — same person can't join twice
4. **One Entry per Day:** `UNIQUE(user_id, entry_date)` in daily_logs
5. **Immutable History:** profile_history is insert-only
6. **Timestamps:** All tables have `created_at`, most have `updated_at`

---

## Seed Data Example

```sql
INSERT INTO cohorts (name, year, admin_email)
VALUES ('AJC', 2026, 'ankush@example.com');

INSERT INTO users (cohort_id, email, first_name, last_name, is_admin)
VALUES 
  ($1, 'ankush@example.com', 'Ankush', 'Jain', true),
  ($1, 'john@example.com', 'John', 'Smith', false),
  ($1, 'mary@example.com', 'Mary', 'Jane', false);

INSERT INTO group_logs (user_id, cohort_id, period, period_type, num_conversations, proposals_amount_pence, sales_amount_pence, income_pence, num_referrals, num_renewals)
VALUES 
  ($2, $1, 'May 2026', 'monthly', 10, 760000, 625700, 535700, 2, 1),
  ($3, $1, 'May 2026', 'monthly', 35, 6526300, 3814000, 3289000, 8, 2);
```

---

## Migration Strategy

1. **Week 1:** Create all tables + indexes
2. **Week 1-2:** Backfill sample cohort + users
3. **Week 2-3:** Frontend populates data via API
4. **Week 4:** Manual testing with real cohort data
5. **Week 5:** Go live, production data entry begins

---

## Access Patterns

**Admin (Ankush):**
- Export: `SELECT * FROM group_logs WHERE cohort_id = $1`
- View all users: `SELECT * FROM users WHERE cohort_id = $1`
- Create new cohort: `INSERT INTO cohorts (...)`

**Individual User:**
- View own profile: `SELECT * FROM profiles WHERE user_id = $1`
- Edit own profile: `UPDATE profiles WHERE user_id = $1`
- View own daily log: `SELECT * FROM daily_logs WHERE user_id = $1`
- View own group log row: `SELECT * FROM group_logs WHERE user_id = $1 AND cohort_id = $2`
- Edit own group log row: `UPDATE group_logs WHERE user_id = $1 AND cohort_id = $2`
- View all group dashboard: `SELECT * FROM group_logs WHERE cohort_id = $1` (all users' rows)

---

## Notes

- All calculations happen in **application code**, not in database triggers. This keeps the DB simple and calculations testable.
- Currency stored as **pence** to avoid float rounding errors (£5,000.50 = 500050 pence).
- **profile_history** is a complete audit trail of all form responses — useful for "show what they answered on [date]" feature.
- **group_logs** can have multiple rows per user per cohort if they span multiple periods (e.g., May monthly + April-Sep preschool + Oct-Mar school). The `period_type` column disambiguates.

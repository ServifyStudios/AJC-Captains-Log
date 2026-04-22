# AJC Captain's Log: API Specification

## Base URL
```
https://api.ajccoach.app/v1
```

## Authentication
All endpoints (except `/auth/login` and `/auth/register`) require:
```
Authorization: Bearer {jwt_token}
```
JWT issued by Supabase Auth on login.

---

## Endpoints

### **AUTH**

#### POST /auth/register
Create a new user account

**Request:**
```json
{
  "email": "john@example.com",
  "password": "secure_password_123",
  "first_name": "John",
  "last_name": "Smith",
  "cohort_id": "uuid_of_cohort"
}
```

**Response (201):**
```json
{
  "user_id": "uuid",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Smith",
  "cohort_id": "uuid_of_cohort",
  "token": "jwt_token",
  "created_at": "2026-05-01T10:00:00Z"
}
```

**Error (400):**
```json
{
  "error": "Email already exists in this cohort"
}
```

---

#### POST /auth/login
Login with email and password

**Request:**
```json
{
  "email": "john@example.com",
  "password": "secure_password_123"
}
```

**Response (200):**
```json
{
  "user_id": "uuid",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Smith",
  "cohort_id": "uuid_of_cohort",
  "is_admin": false,
  "token": "jwt_token",
  "token_expires_at": "2026-05-02T10:00:00Z"
}
```

**Error (401):**
```json
{
  "error": "Invalid email or password"
}
```

---

### **PROFILES**

#### GET /profiles/me
Get current user's profile (latest form submission)

**Request:**
```
GET /profiles/me
Authorization: Bearer {token}
```

**Response (200):**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "first_name": "John",
  "last_name": "Smith",
  "coaching_experience": "5 years",
  "ideal_clients": "Female entrepreneurs",
  "current_challenge": "Growing to 20 clients",
  "num_current_clients": 10,
  "client_types": "8 paying, 2 non-paying",
  "full_practice_target": 20,
  "work_format": "1-1 online, 3-month packages",
  "pricing_amount_pence": 500000, -- £5,000
  "client_sources": "Referrals: 5, LinkedIn: 3, Word of mouth: 2",
  "form_insights": "Need to create no-brainer packages",
  "six_month_goals": "Fill practice to 20 clients",
  "support_questions": "How to raise prices?",
  "filled_at": "2026-05-10T14:30:00Z",
  "created_at": "2026-05-10T14:30:00Z",
  "updated_at": "2026-05-10T14:30:00Z"
}
```

---

#### PUT /profiles/me
Update current user's profile (creates new version in profile_history)

**Request:**
```json
{
  "coaching_experience": "5 years",
  "ideal_clients": "Female entrepreneurs",
  "current_challenge": "Growing to 20 clients",
  "num_current_clients": 12,
  "client_types": "10 paying, 2 non-paying",
  "full_practice_target": 20,
  "work_format": "1-1 online, 3-month packages",
  "pricing_amount_pence": 500000,
  "client_sources": "Referrals: 6, LinkedIn: 4, Word of mouth: 2",
  "form_insights": "Need to create no-brainer packages",
  "six_month_goals": "Fill practice to 20 clients, raise prices",
  "support_questions": "How to position price increases?"
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "first_name": "John",
  "last_name": "Smith",
  ...
  "filled_at": "2026-05-15T09:00:00Z",
  "updated_at": "2026-05-15T09:00:00Z"
}
```

---

#### GET /profiles/{user_id}
Get any user's profile (visible to all cohort members)

**Request:**
```
GET /profiles/uuid_of_john
Authorization: Bearer {token}
```

**Response (200):**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "first_name": "John",
  "last_name": "Smith",
  ... (same as GET /profiles/me)
}
```

---

#### GET /profiles/{user_id}/history
Get all previous profile submissions for a user

**Request:**
```
GET /profiles/uuid_of_john/history
Authorization: Bearer {token}
```

**Response (200):**
```json
[
  {
    "id": "uuid_v1",
    "coaching_experience": "5 years",
    "ideal_clients": "Female entrepreneurs",
    ... (all 12 fields),
    "filled_at": "2026-05-10T14:30:00Z",
    "created_at": "2026-05-10T14:30:00Z"
  },
  {
    "id": "uuid_v2",
    "coaching_experience": "5 years",
    "ideal_clients": "Female entrepreneurs",
    ... (all 12 fields),
    "filled_at": "2026-05-15T09:00:00Z",
    "created_at": "2026-05-15T09:00:00Z"
  }
]
```

---

### **DAILY LOGS**

#### POST /daily-logs
Create or update a daily entry

**Request:**
```json
{
  "entry_date": "2026-05-15",
  "num_conversations": 5,
  "num_proposals": 2,
  "proposals_amount_pence": 1500000, -- £15,000
  "num_referrals": 1,
  "num_renewals": 0,
  "income_pence": 500000 -- £5,000
}
```

**Response (201 or 200):**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "entry_date": "2026-05-15",
  "num_conversations": 5,
  "num_proposals": 2,
  "proposals_amount_pence": 1500000,
  "num_referrals": 1,
  "num_renewals": 0,
  "income_pence": 500000,
  "created_at": "2026-05-15T10:00:00Z",
  "updated_at": "2026-05-15T10:00:00Z"
}
```

---

#### GET /daily-logs
Get current user's daily log entries (for a date range or all)

**Request:**
```
GET /daily-logs?start_date=2026-05-01&end_date=2026-05-31
Authorization: Bearer {token}
```

**Query Params:**
- `start_date` (optional): ISO date, defaults to cohort start
- `end_date` (optional): ISO date, defaults to today

**Response (200):**
```json
[
  {
    "id": "uuid",
    "entry_date": "2026-05-01",
    "num_conversations": 3,
    "num_proposals": 1,
    "proposals_amount_pence": 800000,
    "num_referrals": 0,
    "num_renewals": 0,
    "income_pence": 300000,
    "created_at": "2026-05-01T10:00:00Z",
    "updated_at": "2026-05-01T10:00:00Z"
  },
  {
    "id": "uuid",
    "entry_date": "2026-05-15",
    "num_conversations": 5,
    "num_proposals": 2,
    "proposals_amount_pence": 1500000,
    "num_referrals": 1,
    "num_renewals": 0,
    "income_pence": 500000,
    "created_at": "2026-05-15T10:00:00Z",
    "updated_at": "2026-05-15T10:00:00Z"
  }
]
```

---

### **GROUP LOGS**

#### GET /group-logs
Get all users' group log rows for the cohort (full captain's log table)

**Request:**
```
GET /group-logs?period=May%202026&period_type=monthly
Authorization: Bearer {token}
```

**Query Params:**
- `period` (optional): e.g., "May 2026" or "April-September 2026"
- `period_type` (optional): "monthly", "preschool", or "school"
- If not provided, returns current month

**Response (200):**
```json
[
  {
    "id": "uuid",
    "user_id": "uuid_john",
    "first_name": "John",
    "last_name": "Smith",
    "num_conversations": 10,
    "num_proposals": 2,
    "proposals_amount_pence": 760000,
    "sales_amount_pence": 625700,
    "income_pence": 535700,
    "num_referrals": 2,
    "num_renewals": 1,
    "created_at": "2026-05-01T10:00:00Z",
    "updated_at": "2026-05-10T14:00:00Z"
  },
  {
    "id": "uuid",
    "user_id": "uuid_mary",
    "first_name": "Mary",
    "last_name": "Jane",
    "num_conversations": 35,
    "num_proposals": 7,
    "proposals_amount_pence": 6526300,
    "sales_amount_pence": 3814000,
    "income_pence": 3289000,
    "num_referrals": 8,
    "num_renewals": 2,
    "created_at": "2026-05-01T10:00:00Z",
    "updated_at": "2026-05-12T09:00:00Z"
  }
]
```

---

#### PUT /group-logs/{group_log_id}
Update a group log row (editable textbox in UI)

**Request:**
```json
{
  "num_conversations": 12,
  "num_proposals": 3,
  "proposals_amount_pence": 900000,
  "sales_amount_pence": 700000,
  "income_pence": 600000,
  "num_referrals": 3,
  "num_renewals": 1
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "user_id": "uuid_john",
  "first_name": "John",
  "last_name": "Smith",
  "num_conversations": 12,
  "num_proposals": 3,
  "proposals_amount_pence": 900000,
  "sales_amount_pence": 700000,
  "income_pence": 600000,
  "num_referrals": 3,
  "num_renewals": 1,
  "updated_at": "2026-05-15T10:30:00Z"
}
```

**Authorization:** User can only update their own row OR is admin. Other users get 403 Forbidden.

---

#### GET /group-logs/{user_id}/individual
Get individual dashboard for a specific user (totals, projections, rankings)

**Request:**
```
GET /group-logs/uuid_john/individual?period_type=monthly
Authorization: Bearer {token}
```

**Query Params:**
- `period_type` (optional): "monthly", "preschool", or "school". Defaults to current.

**Response (200):**
```json
{
  "user_id": "uuid_john",
  "first_name": "John",
  "last_name": "Smith",
  
  "totals": {
    "total_conversations": 39,
    "total_proposals_pence": 1500000,
    "total_sales_pence": 1200000,
    "total_income_pence": 1000000,
    "total_referrals": 5,
    "total_renewals": 2,
    "months_logged": 3
  },
  
  "projections": {
    "average_monthly_income_pence": 333333,
    "projected_annual_income_pence": 4000000,
    "average_monthly_sales_pence": 500000,
    "projected_annual_sales_pence": 6000000,
    "percentage_vs_icf": 87.0, -- +87% above global average
    "coaches_above_global_avg": 5
  },
  
  "sales_growth": {
    "preschool_sales_pence": 2000000,
    "school_sales_pence": 3000000,
    "sales_growth_percentage": 50.0,
    "has_preschool_baseline": true
  },
  
  "rankings": {
    "conversations_rank": 2,
    "proposals_rank": 1,
    "sales_rank": 2,
    "income_rank": 1,
    "referrals_rank": 3,
    "renewals_rank": 2,
    "total_cohort_members": 10
  }
}
```

---

### **DASHBOARD**

#### GET /dashboard/cohort-summary
Get group dashboard metrics (totals, averages, counts)

**Request:**
```
GET /dashboard/cohort-summary?period=May%202026&period_type=monthly
Authorization: Bearer {token}
```

**Query Params:**
- `period` (optional): e.g., "May 2026"
- `period_type` (optional): "monthly", "preschool", "school"

**Response (200):**
```json
{
  "period": "May 2026",
  "period_type": "monthly",
  
  "totals": {
    "total_conversations": 119,
    "total_proposals_pence": 23379900,
    "total_sales_pence": 9186800,
    "total_income_pence": 5195700,
    "total_referrals": 19,
    "total_renewals": 6
  },
  
  "averages": {
    "average_cohort_income_pence": 1298925,
    "coaches_above_global_avg_count": 1
  },
  
  "top_10": {
    "conversations": [
      {
        "user_id": "uuid",
        "first_name": "Mary",
        "last_name": "Jane",
        "value": 41,
        "rank": 1
      },
      {
        "user_id": "uuid",
        "first_name": "John",
        "last_name": "Smith",
        "value": 39,
        "rank": 2
      }
    ],
    "proposals_pence": [ ... ],
    "sales_pence": [ ... ],
    "income_pence": [ ... ],
    "referrals": [ ... ],
    "renewals": [ ... ]
  }
}
```

---

#### GET /dashboard/sales-tiers
Get sales tier bucket counts

**Request:**
```
GET /dashboard/sales-tiers?period_type=monthly
Authorization: Bearer {token}
```

**Response (200):**
```json
{
  "period_type": "monthly",
  "tiers": [
    {
      "tier_name": "10k",
      "min_pence": 1000000,
      "max_pence": 2500000,
      "count": 2,
      "users": [
        {
          "user_id": "uuid",
          "first_name": "John",
          "last_name": "Smith",
          "projected_annual_sales_pence": 1200000,
          "amount_left_for_next_tier_pence": 1300000
        }
      ]
    },
    {
      "tier_name": "25k",
      "min_pence": 2500000,
      "max_pence": 5000000,
      "count": 1,
      "users": [ ... ]
    },
    {
      "tier_name": "50k",
      "min_pence": 5000000,
      "max_pence": 7500000,
      "count": 0,
      "users": []
    },
    ...
  ]
}
```

---

#### GET /dashboard/sales-growth
Get sales growth comparison (pre-school vs school)

**Request:**
```
GET /dashboard/sales-growth
Authorization: Bearer {token}
```

**Response (200):**
```json
[
  {
    "user_id": "uuid_john",
    "first_name": "John",
    "last_name": "Smith",
    "preschool_sales_pence": 2000000,
    "school_sales_pence": 3000000,
    "sales_growth_percentage": 50.0,
    "has_preschool_baseline": true
  },
  {
    "user_id": "uuid_mary",
    "first_name": "Mary",
    "last_name": "Jane",
    "preschool_sales_pence": 5000000,
    "school_sales_pence": 7500000,
    "sales_growth_percentage": 50.0,
    "has_preschool_baseline": true
  }
]
```

---

### **ADMIN**

#### POST /admin/cohorts
Create a new cohort (admin only)

**Request:**
```json
{
  "name": "AJC",
  "year": 2026,
  "admin_email": "ankush@example.com"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "name": "AJC",
  "year": 2026,
  "display_name": "AJC 2026",
  "admin_email": "ankush@example.com",
  "is_active": true,
  "created_at": "2026-04-01T10:00:00Z"
}
```

---

#### GET /admin/export
Export all cohort data as CSV

**Request:**
```
GET /admin/export?cohort_id=uuid
Authorization: Bearer {admin_token}
```

**Query Params:**
- `cohort_id` (required): Which cohort to export

**Response (200):**
```
Content-Type: text/csv
Content-Disposition: attachment; filename="cohort_export_2026-05-15_143000.csv"

user_email,first_name,last_name,period,period_type,num_conversations,proposals_amount,sales_amount,income,referrals,renewals,last_updated
john@example.com,John,Smith,May 2026,monthly,10,7600,6257,5357,2,1,2026-05-10T14:00:00Z
mary@example.com,Mary,Jane,May 2026,monthly,35,65263,38140,32890,8,2,2026-05-12T09:00:00Z
...
```

---

## Error Responses

### 400 Bad Request
```json
{
  "error": "Invalid request body",
  "details": "Field 'entry_date' is required"
}
```

### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "details": "Missing or invalid token"
}
```

### 403 Forbidden
```json
{
  "error": "Forbidden",
  "details": "You cannot edit another user's profile"
}
```

### 404 Not Found
```json
{
  "error": "Not found",
  "details": "User not found in this cohort"
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal server error",
  "details": "An unexpected error occurred"
}
```

---

## Rate Limiting
- 100 requests per minute per user
- Admin exports limited to 5 per day per cohort

---

## Notes

1. **All currency values in pence** (integers, no decimals)
   - Frontend parses £5,000 → 500000 pence
   - API returns 500000 pence → Frontend displays £5,000.00

2. **Dates are ISO 8601** (YYYY-MM-DD)

3. **Timestamps are ISO 8601** with timezone (2026-05-15T10:00:00Z)

4. **Cohort isolation:** All endpoints automatically filter by the user's cohort_id from their JWT

5. **Idempotency:** POST /daily-logs with same entry_date updates instead of creating duplicate

6. **Rankings:** RANK() over PARTITION, so ties get same rank and next person gets skipped rank

---

## Frontend Usage Example

```javascript
// Login
const loginRes = await fetch('/v1/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'john@example.com',
    password: 'password123'
  })
});
const { token } = await loginRes.json();
localStorage.setItem('authToken', token);

// Fetch group log for current month
const logsRes = await fetch('/v1/group-logs', {
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('authToken')}`
  }
});
const logs = await logsRes.json();

// Update own group log row
await fetch(`/v1/group-logs/${logId}`, {
  method: 'PUT',
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('authToken')}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    num_conversations: 12,
    proposals_amount_pence: 900000
  })
});
```

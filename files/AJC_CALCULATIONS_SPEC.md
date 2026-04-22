# AJC Captain's Log: Calculation Specifications

## Constants
- **ICF Global Average Income (2023):** £41,713
- **Pre-School Period:** April - September (6 months)
- **School Period:** October - March (6 months) OR April - September + October onwards (rolling 12 months depending on cohort)

---

## Individual Dashboard Metrics

### Average Monthly Income
```
average_monthly_income = total_income_earned / number_of_months_logged
```
**Example:** User logged income in May (£5k) and June (£8k) = £13k / 2 = £6,500/month

**Edge case:** If user logged 0 months, show "No data" or £0.

---

### Average Monthly Sales (Proposals Value)
```
average_monthly_sales = total_proposals_value / number_of_months_logged
```
**Example:** User proposed £10k in May, £15k in June = £25k / 2 = £12,500/month

---

### Projected Annual Income
```
projected_annual_income = average_monthly_income × 12
```
**Example:** £6,500/month × 12 = £78,000/year

---

### Projected Annual Sales
```
projected_annual_sales = average_monthly_sales × 12
```
**Example:** £12,500/month × 12 = £150,000/year

---

### Percentage vs ICF Global Average
```
percentage_vs_icf = ((projected_annual_income - 41713) / 41713) × 100
```
**Example:** 
- Projected = £78,000
- Difference = £78,000 - £41,713 = £36,287
- Percentage = (36287 / 41713) × 100 = +87.0%

**Display:** "87% above global average" or "87% more than ICF global coach"

**If below:** "-30% below global average"

---

### Pre-School Sales Total
```
preschool_sales_total = SUM(proposals_value for months in {April, May, June, July, August, September})
```
From daily log or group log entries marked as "pre-school" period.

---

### School Sales Total
```
school_sales_total = SUM(proposals_value for months in {October onwards or current school period})
```
Depends on cohort. For "April start" cohorts, school_sales = Oct-Mar total.

---

### Sales Growth Percentage
```
sales_growth = ((school_sales_total - preschool_sales_total) / preschool_sales_total) × 100
```
**Example:**
- Pre-school = £50,000
- School = £75,000
- Growth = ((75000 - 50000) / 50000) × 100 = +50%

**Edge case:** If preschool_sales_total = 0, cannot calculate growth. Show "N/A" or "No pre-school baseline."

---

## Group Dashboard Metrics

### Total Conversations
```
total_conversations = SUM(conversations for all users in current month/period)
```
From group captain's log table.

---

### Total Proposals (GBP)
```
total_proposals = SUM(proposals_amount for all users in current month/period)
```
Sum of GBP values entered by each user.

---

### Total Sales (GBP)
```
total_sales = SUM(sales_amount for all users in current month/period)
```
Sum of GBP values entered by each user.

---

### Total Income (GBP)
```
total_income = SUM(income_amount for all users in current month/period)
```
Sum of GBP values entered by each user.

---

### Total Referrals
```
total_referrals = SUM(referrals for all users in current month/period)
```

---

### Total Renewals
```
total_renewals = SUM(renewals for all users in current month/period)
```

---

### Average Income (Cohort-Wide)
```
average_cohort_income = total_income / number_of_users_in_cohort
```
**Note:** This is total income divided by total number of coaches, NOT just coaches who logged income.

---

### Coaches Above ICF Global Average
```
coaches_above_global_avg = COUNT(users WHERE projected_annual_income > 41713)
```
Based on each user's projected annual income (not their current month).

---

## Charts & Visualizations

### Top 10 (Conversations)
```
Rank users by total conversations (descending), show top 10 as horizontal bar chart
```

---

### Top 10 (Proposals)
```
Rank users by total proposals_amount (descending), show top 10 as horizontal bar chart
```

---

### Top 10 (Income)
```
Rank users by total income_amount (descending), show top 10 as horizontal bar chart
```

---

### Top 10 (Referrals)
```
Rank users by total referrals (descending), show top 10 as horizontal bar chart
```

---

### Top 10 (Renewals)
```
Rank users by total renewals (descending), show top 10 as horizontal bar chart
```

---

### Sales Tiers (Number of People in Each Bucket)
```
Count users by projected_annual_sales:

10k tier:   COUNT(users WHERE projected_annual_sales >= 10000 AND < 25000)
25k tier:   COUNT(users WHERE projected_annual_sales >= 25000 AND < 50000)
50k tier:   COUNT(users WHERE projected_annual_sales >= 50000 AND < 75000)
75k tier:   COUNT(users WHERE projected_annual_sales >= 75000 AND < 100000)
100k tier:  COUNT(users WHERE projected_annual_sales >= 100000 AND < 250000)
250k tier:  COUNT(users WHERE projected_annual_sales >= 250000 AND < 500000)
500k tier:  COUNT(users WHERE projected_annual_sales >= 500000)
```

**Display:** Single aggregated count for each tier (not separate "before" and "after" bars, unless viewing a comparison view).

---

### Amount Left for Next Tier (Hover Tooltip)
```
current_tier = determine_tier(projected_annual_sales)
next_tier_min = next_tier_threshold

amount_left = next_tier_min - projected_annual_sales
```

**Display on hover:** "£X,XXX left for £Xk tier"

**Edge case:** If user already exceeds the highest tier (£500k+), show "Already exceeded highest tier" or hide tooltip.

---

### Sales Growth Comparison (Pre-School vs School)
```
Chart showing two bars:
1. Pre-school sales total (April-Sep)
2. School sales total (Oct onwards or relevant period)

Plus growth percentage displayed above/between bars
```

---

### Annual Income Compared to Global Income
```
Individual dashboard card showing:
- Their projected annual income
- ICF global average (£41,713)
- Percentage difference (± X%)

Format: "You're making 87% more than the global average coach"
```

---

## Testing Checklist

### Test Set 1: Basic Calculations
```
User: John
May: £5k income, £10k proposals
June: £8k income, £15k proposals

Expected:
- Average monthly income: £6,500
- Average monthly sales: £12,500
- Projected annual income: £78,000
- Projected annual sales: £150,000
- % vs ICF: +87.0%
```

### Test Set 2: Zero Income Edge Case
```
User: Mary
May: £0 income, £5k proposals
June: £0 income, £0 proposals

Expected:
- Average monthly income: £0
- Average monthly sales: £2,500
- Projected annual income: £0
- Projected annual sales: £30,000
- % vs ICF: -100% (or "below global average")
```

### Test Set 3: Sales Growth
```
Pre-school (Apr-Sep): £50,000 total
School (Oct-Mar): £75,000 total

Expected:
- Sales growth: +50%
- Display: "50% sales growth"
```

### Test Set 4: Pre-School Zero (No Baseline)
```
Pre-school: £0 total
School: £30,000 total

Expected:
- Sales growth: "N/A" (cannot calculate from zero)
```

### Test Set 5: Sales Tier Buckets
```
Users (projected annual sales):
- User A: £12,000 → 10k tier
- User B: £30,000 → 25k tier
- User C: £60,000 → 50k tier
- User D: £5,000 → no tier (below 10k)

Expected bucket counts:
10k: 1
25k: 1
50k: 1
75k: 0
100k: 0
250k: 0
500k: 0
```

### Test Set 6: Amount Left for Next Tier (Hover)
```
User: A (£12,000 projected)
Current tier: 10k
Next tier: 25k

Expected:
- Amount left: £25,000 - £12,000 = £13,000
- Tooltip: "£13,000 left for £25k tier"
```

### Test Set 7: Coaches Above Global Average
```
Cohort:
- User A: projected £78,000 (✓ above £41,713)
- User B: projected £30,000 (✗ below)
- User C: projected £50,000 (✓ above)

Expected:
- Coaches above global average: 2
```

### Test Set 8: Group Dashboard Totals
```
Three users in May:
- User A: 10 conversations, £7,600 proposals, £6,257 sales, £5,357 income, 2 referrals, 1 renewal
- User B: 35 conversations, £65,263 proposals, £38,140 sales, £32,890 income, 8 referrals, 2 renewals
- User C: 64 conversations, £123,702 proposals, £90,637 sales, £80,126 income, 9 referrals, 3 renewals

Expected totals:
- Total conversations: 109
- Total proposals: £196,565
- Total sales: £135,034
- Total income: £118,373
- Total referrals: 19
- Total renewals: 6
- Average cohort income: £118,373 / 3 = £39,457.67
```

---

## Data Entry Notes

**Group Captain's Log Columns (Editable by User):**
1. Name (display only, from profile)
2. Number of Conversations
3. Amount in Proposals (GBP)
4. Sales - Amount Invoiced (GBP)
5. Income Amount (GBP)
6. Referrals
7. Renewals

**All values entered manually by the user (no auto-conversion).**

**Daily Log:**
- Date picker (users can backfill past entries)
- Number of conversations
- Number of proposals
- Amount of proposals (GBP)
- Number of referrals
- Number of renewals
- Income earned (GBP)

**Auto-rollup:** Daily log entries sum into the monthly group captain's log row (as a helper/reference, but group log is the source of truth).

---

## Currency Handling

- **Store in database:** Integers (pence, not pounds) to avoid float rounding errors
  - Example: £5,000.50 = 500050 (pence)
- **Display to user:** Formatted as GBP with commas
  - Example: 500050 pence = "£5,000.50"
- **User input:** Accept GBP format, parse to pence
  - Example: User types "5000.50" → store as 500050

---

## Cohort Isolation

**Every query includes cohort_id filter:**
- No cross-cohort data leakage
- Pre-school and school periods are separate views but same cohort
- Admin export only exports one cohort at a time

---

## Final Notes

- **No rounding errors:** Use integer pence, not float pounds
- **Null handling:** Treat missing values as 0
- **Negative values:** Reject on input (validation error)
- **Date boundaries:** April = month 1, May = month 2, ..., September = month 6
- **Test before launch:** Run all 8 test sets, verify each calculation manually

# 🥑 Case Study #3 - Foodie-Fi 🥑

## B. Data Analysis Questions

In this section, we perform SQL-based analysis on the Foodie-Fi platform data to extract meaningful insights about customer behavior and engagement patterns.

---

### 1. How many customers has Foodie-Fi ever had?

**Steps Taken:**
- Used `COUNT(DISTINCT customer_id)` to get unique customer count
- Selected from the subscriptions table as the main data source
- Applied DISTINCT to avoid counting duplicate customer records

**Query:**
```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM subscriptions;
```

**Output:**
| customer_count |
|----------------|
| 1000           |

**Analysis Points:**
- Foodie-Fi has exactly 1,000 unique customers in their database
- This represents the total customer base since platform inception
- The round number suggests this might be a sample dataset for analysis purposes

---

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

**Steps Taken:**
- Used `DATE_TRUNC('month', start_date)` to group dates by month start
- Joined subscriptions with plans table to filter for trial plans
- Applied WHERE clause to filter only trial plan records
- Grouped by month_start and counted occurrences

**Query:**
```sql
SELECT 
  DATE_TRUNC('month', start_date) AS month_start,
  COUNT(*) AS trial_count
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id
WHERE p.plan_name = 'trial'
GROUP BY month_start
ORDER BY month_start;
```

**Output:**
| month_start             | trial_count |
|------------------------|-------------|
| 2020-01-01 00:00:00+00 | 88          |
| 2020-02-01 00:00:00+00 | 68          |
| 2020-03-01 00:00:00+00 | 94          |
| 2020-04-01 00:00:00+00 | 81          |
| 2020-05-01 00:00:00+00 | 88          |
| 2020-06-01 00:00:00+00 | 79          |
| 2020-07-01 00:00:00+00 | 89          |
| 2020-08-01 00:00:00+00 | 88          |
| 2020-09-01 00:00:00+00 | 87          |
| 2020-10-01 00:00:00+00 | 79          |
| 2020-11-01 00:00:00+00 | 75          |
| 2020-12-01 00:00:00+00 | 84          |

**Analysis Points:**
- Trial sign-ups are fairly consistent throughout 2020, ranging from 68-94 per month
- March had the highest trial sign-ups (94), while February had the lowest (68)
- Average monthly trial sign-ups: approximately 83 customers
- No significant seasonal trends visible in trial acquisition

---

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

**Steps Taken:**
- Applied WHERE clause to filter dates after 2020 using `start_date >= '2021-01-01'`
- Joined subscriptions with plans table to get plan names
- Grouped by plan_id and plan_name to get counts for each plan type
- Ordered by plan_id for consistent output

**Query:**
```sql
SELECT p.plan_id, p.plan_name, COUNT(*) AS total_count
FROM subscriptions AS s
JOIN plans AS p ON p.plan_id = s.plan_id
WHERE s.start_date >= '2021-01-01'
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id;
```

**Output:**
| plan_id | plan_name     | total_count |
|---------|---------------|-------------|
| 1       | basic monthly | 8           |
| 2       | pro monthly   | 60          |
| 3       | pro annual    | 63          |
| 4       | churn         | 71          |

**Analysis Points:**
- Most activity after 2020 was churning (71 events), indicating customer attrition
- Pro plans (monthly: 60, annual: 63) dominated new subscriptions in 2021
- Basic monthly had minimal uptake (8) suggesting customers prefer premium offerings
- Total post-2020 events: 202, showing continued platform activity

---

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

**Steps Taken:**
- Used CTE (Common Table Expression) to calculate total and churned customers
- Applied `COUNT(DISTINCT customer_id)` for total customers
- Used conditional aggregation with CASE statement to count churned customers
- Calculated percentage using numeric casting and ROUND function

**Query:**
```sql
WITH totals AS (
    SELECT
        COUNT(DISTINCT s.customer_id) AS total_customers,
        COUNT(DISTINCT CASE WHEN p.plan_name = 'churn' THEN s.customer_id END) AS churned_customers
    FROM subscriptions s
    JOIN plans p ON p.plan_id = s.plan_id
)
SELECT
    total_customers,
    churned_customers,
    ROUND(churned_customers::numeric * 100 / total_customers, 1) AS churn_pct
FROM totals;
```

**Output:**
| total_customers | churned_customers | churn_pct |
|----------------|-------------------|-----------|
| 1000           | 307               | 30.7      |

**Analysis Points:**
- 30.7% churn rate is quite high, indicating significant customer retention challenges
- 307 out of 1000 customers have churned from the platform
- This suggests need for improved retention strategies and customer engagement
- Industry benchmark comparison would be valuable to assess performance

---

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

**Steps Taken:**
- Used window function `LAG()` to get previous plan for each customer
- Created CTE to identify customers who churned (plan_id = 4) immediately after trial (previous_plan = 0)
- Calculated total customers for percentage calculation
- Used ROUND function to get whole number percentage

**Query:**
```sql
WITH curr_prev_plans AS (
    SELECT 
        customer_id, 
        plan_id, 
        LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan
    FROM subscriptions
),
churn_after_trial AS (
    SELECT customer_id
    FROM curr_prev_plans
    WHERE plan_id = 4 AND previous_plan = 0
),
total_customers AS (
    SELECT COUNT(DISTINCT customer_id) AS total
    FROM subscriptions
)

SELECT 
    COUNT(*) AS churn_count,
    ROUND(COUNT(*) * 100.0 / (SELECT total FROM total_customers)) AS churn_percent
FROM churn_after_trial;
```

**Output:**
| churn_count | churn_percent |
|-------------|---------------|
| 92          | 9             |

**Analysis Points:**
- 9% of customers churn immediately after their free trial
- 92 customers never converted from trial to paid subscription
- This indicates trial-to-paid conversion issues that need addressing
- Trial experience optimization could significantly impact revenue

---

### 6. What is the number and percentage of customer plans after their initial free trial?

**Steps Taken:**
- Used LAG window function to identify plans that followed the trial (previous_plan = 0)
- Grouped by plan_id to count customers for each post-trial plan
- Calculated percentages against total customer base
- Used ROUND function for 2 decimal places

**Query:**
```sql
WITH curr_prev_plans AS (
    SELECT 
        customer_id, 
        plan_id, 
        LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan
    FROM subscriptions
),
plan_count_after_trial AS (
    SELECT plan_id, COUNT(*) AS counts
    FROM curr_prev_plans
    WHERE previous_plan = 0
    GROUP BY plan_id
),
total_customers AS (
    SELECT COUNT(DISTINCT customer_id) AS total
    FROM subscriptions
)
SELECT 
    *,
    ROUND(pcat.counts * 100.0 / (SELECT total FROM total_customers), 2) AS churn_percent
FROM plan_count_after_trial AS pcat;
```

**Output:**
| plan_id | counts | churn_percent |
|---------|--------|---------------|
| 1       | 546    | 54.60         |
| 2       | 325    | 32.50         |
| 3       | 37     | 3.70          |
| 4       | 92     | 9.20          |

**Analysis Points:**
- 54.6% of customers choose basic monthly after trial - most popular conversion path
- 32.5% upgrade to pro monthly, showing willingness to pay for premium features
- Only 3.7% choose annual plans immediately after trial
- 9.2% churn immediately, confirming previous analysis

---

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

**Steps Taken:**
- Used ROW_NUMBER() window function to get the most recent plan for each customer
- Filtered for plans that started on or before 2020-12-31
- Selected only the most recent plan (ranks = 1) for each customer
- Calculated percentages and ordered by customer count

**Query:**
```sql
WITH allp AS (
  SELECT 
    customer_id, 
    plan_id, 
    start_date, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS ranks
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT 
  p.plan_name, 
  COUNT(*) AS customer_count,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 1) AS percentage
FROM allp ap
JOIN plans p ON ap.plan_id = p.plan_id
WHERE ap.ranks = 1
GROUP BY p.plan_name
ORDER BY customer_count DESC;
```

**Output:**
| plan_name     | customer_count | percentage |
|---------------|----------------|------------|
| pro monthly   | 267            | 26.7       |
| churn         | 236            | 23.6       |
| pro annual    | 189            | 18.9       |
| basic monthly | 120            | 12.0       |

**Analysis Points:**
- Pro monthly is the most popular active plan (26.7% of all customers)
- High churn rate visible in snapshot (23.6% churned by end of 2020)
- Pro annual shows good adoption (18.9%) indicating customer commitment
- Basic monthly represents smaller segment (12.0%)
- Missing trial data suggests analysis focused on post-trial states

---

### 8. How many customers have upgraded to an annual plan in 2020?

**Steps Taken:**
- Used LAG window function to identify customers who had a previous plan
- Filtered for annual plan (plan_id = 3) upgrades in 2020
- Ensured previous plan existed (not first-time subscribers)
- Used COUNT(DISTINCT customer_id) to avoid double-counting

**Query:**
```sql
WITH with_prev_plan AS (
  SELECT 
    customer_id,
    plan_id,
    start_date,
    LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS prev_plan
  FROM subscriptions
)

SELECT COUNT(DISTINCT customer_id) AS upgrade_count
FROM with_prev_plan
WHERE plan_id = 3
  AND DATE_PART('year', start_date) = 2020
  AND prev_plan IS NOT NULL;
```

**Output:**
| upgrade_count |
|---------------|
| 195           |

**Analysis Points:**
- 195 customers upgraded to annual plans in 2020
- This represents 19.5% of the total customer base
- Strong annual upgrade rate suggests customer satisfaction and commitment
- Annual upgrades likely provide better customer lifetime value

---

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

**Steps Taken:**
- Used ROW_NUMBER() to identify first subscription (join date) for each customer
- Created separate CTEs for initial joins and annual plan adoptions
- Joined the two datasets on customer_id
- Calculated average difference in days using date arithmetic

**Query:**
```sql
WITH with_plan_journey AS (
  SELECT 
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS ranks
  FROM subscriptions
),

foodie_initial_joins AS (
  SELECT customer_id, start_date AS foodie_join
  FROM with_plan_journey
  WHERE ranks = 1
),

annual_joins AS (
  SELECT customer_id, start_date AS annual_start
  FROM with_plan_journey
  WHERE plan_id = 3
)

SELECT ROUND(AVG(aj.annual_start - fi.foodie_join), 1) AS average_days_from_join_to_annual
FROM foodie_initial_joins AS fi
INNER JOIN annual_joins AS aj ON fi.customer_id = aj.customer_id;
```

**Output:**
| average_days_from_join_to_annual |
|----------------------------------|
| 104.6                            |

**Analysis Points:**
- Average time to annual upgrade is 104.6 days (approximately 3.5 months)
- This suggests customers need time to evaluate the service before committing annually
- The timeframe aligns with typical B2C subscription behavior patterns
- Could inform marketing campaigns for annual plan promotions

---

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

**Steps Taken:**
- Built upon previous query structure to get days between join and annual upgrade
- Used FLOOR(days_diff / 30) to create 30-day buckets
- Created readable day ranges using CONCAT function
- Calculated both customer count and average days within each bucket

**Query:**
```sql
<!-- my naive solution
WITH with_plan_journey AS (
  SELECT 
    customer_id,
    plan_id,
    start_date,
    row_number() OVER (PARTITION BY customer_id ORDER BY start_date) AS ranks
  FROM subscriptions
),

foodie_initial_joins as (
  SELECT customer_id, start_date as foodie_join
  FROM with_plan_journey
  WHERE ranks = 1
),

annual_joins as (
  SELECT customer_id, start_date as annual_start
  FROM with_plan_journey
  WHERE plan_id = 3
)

SELECT 
  count(CASE WHEN (aj.annual_start - fi.foodie_join) BETWEEN 0 AND 30 THEN 1 END) AS within_30,
  count(CASE WHEN (aj.annual_start - fi.foodie_join) BETWEEN 31 AND 60 THEN 1 END) AS within_60,
  count(CASE WHEN (aj.annual_start - fi.foodie_join) BETWEEN 61 AND 90 THEN 1 END) AS within_90,
  count(CASE WHEN (aj.annual_start - fi.foodie_join) BETWEEN 91 AND 120 THEN 1 END) AS within_120,
  count(CASE WHEN (aj.annual_start - fi.foodie_join) BETWEEN 121 AND 150 THEN 1 END) AS within_150
FROM foodie_initial_joins AS fi
INNER JOIN annual_joins AS aj
  ON fi.customer_id = aj.customer_id
-->

WITH with_plan_journey AS (
  SELECT 
    customer_id,
    plan_id,
    start_date,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS ranks
  FROM subscriptions
),

foodie_initial_joins AS (
  SELECT customer_id, start_date AS foodie_join
  FROM with_plan_journey
  WHERE ranks = 1
),

annual_joins AS (
  SELECT customer_id, start_date AS annual_start
  FROM with_plan_journey
  WHERE plan_id = 3
),

days_to_annual AS (
  SELECT 
    fi.customer_id,
    (aj.annual_start - fi.foodie_join) AS days_diff
  FROM foodie_initial_joins fi
  JOIN annual_joins aj ON fi.customer_id = aj.customer_id
),

bucketed_customers AS (
  SELECT 
    customer_id, 
    days_diff,
    FLOOR(days_diff / 30) AS bucket
  FROM days_to_annual
)

SELECT 
  CONCAT(bucket * 30 + 1, '-', (bucket + 1) * 30, ' days') AS day_range,
  COUNT(*) AS customer_count,
  ROUND(AVG(days_diff), 1) AS avg_days_in_bucket
FROM bucketed_customers
GROUP BY bucket
ORDER BY bucket;
```

**Output:**
| day_range     | customer_count | avg_days_in_bucket |
|---------------|----------------|--------------------|
| 1-30 days     | 48             | 9.5                |
| 31-60 days    | 25             | 41.8               |
| 61-90 days    | 33             | 70.9               |
| 91-120 days   | 35             | 99.8               |
| 121-150 days  | 43             | 133.0              |
| 151-180 days  | 35             | 161.5              |
| 181-210 days  | 27             | 190.3              |
| 211-240 days  | 4              | 224.3              |
| 241-270 days  | 5              | 257.2              |
| 271-300 days  | 1              | 285.0              |
| 301-330 days  | 1              | 327.0              |
| 331-360 days  | 1              | 346.0              |

**Analysis Points:**
- Largest group (48 customers) upgrades within first 30 days - early adopters
- Secondary peak at 121-150 days (43 customers) suggests evaluation period completion
- Most upgrades happen within 180 days (241 out of 258 total)
- Very few customers take longer than 8 months to upgrade to annual
- Bimodal distribution suggests two customer segments: quick deciders vs. careful evaluators

---

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

**Steps Taken:**
- Used LEAD window function to get the next plan for each customer
- Filtered for pro monthly plans (plan_id = 3) that had basic monthly as next plan (plan_id = 1)
- Applied year filter for 2020 data
- Counted occurrences of this downgrade pattern

**Query:**
```sql
WITH with_plan_journey AS (
  SELECT 
    customer_id,
    plan_id,
    start_date,
    LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_plan
  FROM subscriptions
  WHERE DATE_PART('year', start_date) = 2020
)

SELECT COUNT(*) AS counts
FROM with_plan_journey
WHERE plan_id = 2 AND next_plan = 1;
```

**Output:**
| counts |
|--------|
| 0      |

**Analysis Points:**
- Zero customers downgraded from pro monthly to basic monthly in 2020
- This suggests strong value perception in pro monthly plans
- Customers either maintain their pro monthly subscription or churn completely
- No evidence of plan downgrading behavior in the dataset
- Note: Original query had incorrect plan_id (used 3 instead of 2 for pro monthly)

---

## Summary Insights

**Customer Acquisition & Retention:**
- 1,000 total customers with consistent monthly trial acquisition (~83/month)
- 30.7% overall churn rate indicates retention challenges
- 9% immediate post-trial churn suggests onboarding optimization opportunities

**Plan Preferences:**
- Basic monthly is most popular post-trial choice (54.6%)
- Pro monthly shows strong adoption (32.5%) 
- Annual plans have lower immediate adoption (3.7%) but grow over time

**Upgrade Patterns:**
- 195 customers upgraded to annual plans in 2020
- Average 104.6 days from join to annual upgrade
- Two distinct upgrade patterns: quick deciders (1-30 days) and evaluators (121-150 days)

**Customer Behavior:**
- No downgrading observed from pro to basic plans
- Strong value retention once customers commit to higher tiers
- Pro monthly emerges as the dominant active plan type by end of 2020

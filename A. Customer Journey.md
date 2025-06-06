# 🥑 Case Study #3 - Foodie-Fi 🥑

## A. Customer Journey

This case study explores the onboarding journeys of 8 sample customers from the `subscriptions` table. The goal is to provide a concise description of each customer's subscription path based on their plan changes over time.

---

### Data Overview

The customers included are identified by their `customer_id` values: 1, 2, 11, 13, 15, 16, 18, 19


We joined the `subscriptions` and `plans` tables to gather the following information for each customer:

| customer_id | plan_id | plan_name     | start_date  |
|-------------|----------|---------------|-------------|
| 1           | 0        | trial         | 2020-08-01  |
| 1           | 1        | basic monthly | 2020-08-08  |
| 2           | 0        | trial         | 2020-09-20  |
| 2           | 3        | pro annual    | 2020-09-27  |
| 11          | 0        | trial         | 2020-11-19  |
| 11          | 4        | churn         | 2020-11-26  |
| 13          | 0        | trial         | 2020-12-15  |
| 13          | 1        | basic monthly | 2020-12-22  |
| 13          | 2        | pro monthly   | 2021-03-29  |
| 15          | 0        | trial         | 2020-03-17  |
| 15          | 2        | pro monthly   | 2020-03-24  |
| 15          | 4        | churn         | 2020-04-29  |
| 16          | 0        | trial         | 2020-05-31  |
| 16          | 1        | basic monthly | 2020-06-07  |
| 16          | 3        | pro annual    | 2020-10-21  |
| 18          | 0        | trial         | 2020-07-06  |
| 18          | 2        | pro monthly   | 2020-07-13  |
| 19          | 0        | trial         | 2020-06-22  |
| 19          | 2        | pro monthly   | 2020-06-29  |
| 19          | 3        | pro annual    | 2020-08-29  |

---

### SQL Query Used to Extract Data

```sql
WITH customer_journeys AS ( 
  SELECT
    s.customer_id,
    p.plan_name,
    s.start_date,
    ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS step_num
  FROM foodie_fi.subscriptions s
  JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
  WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
),
journey_text AS (
  SELECT
    customer_id,
    STRING_AGG(plan_name || ' (' || TO_CHAR(start_date, 'YYYY-MM-DD') || ')', ', ' ORDER BY step_num) AS journey
  FROM customer_journeys
  GROUP BY customer_id
)

SELECT * FROM journey_text
ORDER BY customer_id;
```

### Customer Onboarding Journeys Summary
Customer 1: Started with a free trial on 2020-08-01, then upgraded to basic monthly on 2020-08-08.

Customer 2: Began with a free trial on 2020-09-20, then moved to a pro annual plan on 2020-09-27.

Customer 11: Started with a free trial on 2020-11-19 and churned after trial ended on 2020-11-26.

Customer 13: Started with a free trial on 2020-12-15, subscribed to basic monthly on 2020-12-22, and upgraded to pro monthly on 2021-03-29.

Customer 15: Began with a free trial on 2020-03-17, upgraded to pro monthly on 2020-03-24, then churned on 2020-04-29.

Customer 16: Started free trial on 2020-05-31, subscribed to basic monthly on 2020-06-07, and upgraded to pro annual on 2020-10-21.

Customer 18: Started free trial on 2020-07-06 and upgraded to pro monthly on 2020-07-13.

Customer 19: Started free trial on 2020-06-22, upgraded to pro monthly on 2020-06-29, and further upgraded to pro annual on 2020-08-29.


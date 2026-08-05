# Data validation
---
Before calculating revenue, I check the basic properties of the data: period boundaries, grain, the presence of duplication on joins, and the completeness of key relationships. The goal is to establish which conclusions the data can support before any figure reaches a report.

## 1. Data period
The dataset contains three distinct time boundaries, and it is important to separate them from the outset.
```sql
SELECT MIN(date) as min
    , MAX(date) as max
FROM `DA.session`;
-- 2020-11-01 … 2021-01-31
```
<img width="745" height="462" alt="image" src="https://github.com/user-attachments/assets/3cdac01f-255f-4958-b22a-5ded4a5a856c" />

```sql
SELECT MIN(s.date) as min
    , MAX(s.date) as max
FROM `DA.order` o
JOIN `DA.session` s
ON o.ga_session_id = s.ga_session_id;
-- 2020-11-01 … 2021-01-27
```
<img width="707" height="508" alt="image" src="https://github.com/user-attachments/assets/93438f2e-95b5-437b-85e0-1af85717a5f2" />

Sessions run until January 31, orders until the 27th. The four-day gap is sessions with no purchases at the end of the period. Since the project is built around revenue, I take the order period as the working range: **2020-11-01 – 2021-01-27**.

The key limitation is set here. The window covers roughly three months: November, December and January, i.e. Black Friday, the pre-Christmas season and the post-holiday slump. This is a peak trading period, not a slice of the year. I therefore frame every subsequent finding as a description of revenue structure within this window, not as a judgement about year-over-year growth or decline. The data does not support conclusions of that kind.

---

## 2. Volume and grain

```sql
SELECT COUNT(*) as cnt_order
FROM `DA.order`;  -- 33,538
```
<img width="430" height="358" alt="image" src="https://github.com/user-attachments/assets/1729042b-8279-497f-b487-8541da10d082" />

```sql
SELECT COUNT(DISTINCT ga_session_id)
FROM `DA.order`;  -- 33,538
```
<img width="617" height="411" alt="image" src="https://github.com/user-attachments/assets/bc9a8e9b-527f-42c9-8d1f-e7f725b0fe7d" />

The row count equals the number of unique sessions.
```sql
SELECT ga_session_id, COUNT(*) as items
FROM `DA.order`
GROUP BY ga_session_id
HAVING COUNT(*) >= 2;
-- 0 rows
```
<img width="435" height="495" alt="image" src="https://github.com/user-attachments/assets/00cd95c6-8ca2-4f92-876c-266b08304e07" />

No session has more than one row. On this data, then, one session corresponds to one order and one purchased unit. 
Joining `order` to `session` does not multiply rows, so the revenue sum is not inflated and `DISTINCT` is not needed on aggregation. 

## 3. Countries

```sql
SELECT DISTINCT country
FROM `DA.session_params`;
-- 108 (with "(not set)")
```
<img width="1215" height="740" alt="image" src="https://github.com/user-attachments/assets/6651638b-6b9b-4d48-be54-0255d9935bbe" />

The data contains 107 countries. I exclude `(not set)`: it is a sessions with an undetermined country, not a distinct market. Left unfiltered, it is counted as one more "country" and inflates the result.
Country data is based on session IPs rather than billing addresses. This means users on a VPN or traveling get assigned to the wrong market. Since I can't fix this with the data on hand, it somewhat undermines the reliability of any country-specific claims I can make.

## 4. Order-to-account linkage

First I count the accounts that have at least one order:

```sql
SELECT COUNT(DISTINCT ass.account_id)
FROM `DA.account_session` ass
JOIN `DA.order` o
ON ass.ga_session_id = o.ga_session_id;
-- 2781
```
<img width="440" height="398" alt="image" src="https://github.com/user-attachments/assets/33a9db26-6199-4f8a-ad91-04e189e0cd26" />

I check how many orders have an account attached at all:

```sql
SELECT COUNT(DISTINCT o.ga_session_id)
FROM `DA.order` o
JOIN `DA.account_session` ass
ON o.ga_session_id = ass.ga_session_id;
-- 2781
```
<img width="446" height="413" alt="image" src="https://github.com/user-attachments/assets/ce8572a2-4fbe-4910-9099-b0ca3c038dc0" />

To verify the possibility of analyzing repeat purchases, we also determine whether there are any accounts associated with more than one order:

```sql
SELECT
    ass.account_id,
    COUNT(*) as orders
FROM `DA.order` o
JOIN `DA.account_session` ass
ON o.ga_session_id = ass.ga_session_id
GROUP BY ass.account_id
HAVING COUNT(*) > 1;
-- 0
```
<img width="472" height="570" alt="image" src="https://github.com/user-attachments/assets/47cc92db-06c3-4ce4-a2b0-fc7c03803008" />

| Metric | Value |
|---|---|
| Total orders | 33,538 |
| Orders with an account | 2,781 |
| Accounts with order | 2,781 |

The results show that only 2,781 out of 33,538 orders (approximately 8%) can be linked to identified accounts. For the remaining orders, no account information is available, making it impossible to reliably identify the customer. An additional validation also confirmed that none of the linked accounts is associated with more than one order.

These findings indicate that the dataset is not suitable for customer-level analysis, such as customer retention, repeat purchase behavior, or LTV. Therefore, the subsequent analysis is performed at the order and session levels rather than at the customer level.

---

## Summary

| Checked | Result |
|---|---|
| Order period | 2020-11-01 – 2021-01-27 (~3 months, seasonal peak) |
| Orders | 33,538 |
| Grain | 1 session = 1 order = 1 purchased unit |
| Duplication on join | none |
| Countries | 107 (excluding `(not set)`) |
| Accounts with orders | 2,781 (~8% of orders) |
| Unit of analysis | order / session |

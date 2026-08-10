# Data validation
---
Before calculating revenue, I check the basic properties of the data: period boundaries, grain, the presence of duplication on joins, and the completeness of key relationships. The goal is to establish which conclusions the data can support before any figure reaches a report.

## 1. Data period
The dataset contains three distinct time boundaries, and it is important to separate them from the outset.
```sql
SELECT MIN(date) AS min
    , MAX(date) AS max
FROM `DA.session`;
-- 2020-11-01 … 2021-01-31
```
<img width="396" height="402" alt="image" src="https://github.com/user-attachments/assets/a26d6bf4-7bba-4c8a-a793-74aa028922ad" />

```sql
SELECT MIN(s.date) AS min
    , MAX(s.date) AS max
FROM `DA.order` AS o
JOIN `DA.session` AS s
ON o.ga_session_id = s.ga_session_id;
-- 2020-11-01 … 2021-01-27
```
<img width="400" height="437" alt="image" src="https://github.com/user-attachments/assets/a7eea8b1-becb-427a-b21a-633887a84c6f" />

Sessions run until January 31, orders until the 27th. The four-day gap is sessions with no purchases at the end of the period. Since the project is built around revenue, I take the order period as the working range: **2020-11-01 – 2021-01-27**.

The key limitation is set here. The window covers roughly three months: November, December and January, i.e. Black Friday, the pre-Christmas season and the post-holiday slump. This is a peak trading period, not a slice of the year. I therefore frame every subsequent finding as a description of revenue structure within this window, not as a judgement about year-over-year growth or decline. The data does not support conclusions of that kind.

---

## 2. Volume and grain

```sql
SELECT COUNT(*) AS cnt_order
FROM `DA.order`;  -- 33,538
```
<img width="425" height="357" alt="image" src="https://github.com/user-attachments/assets/38fd0080-b899-4383-b925-22532ee92cbc" />

```sql
SELECT COUNT(DISTINCT ga_session_id)
FROM `DA.order`;  -- 33,538
```
<img width="617" height="411" alt="image" src="https://github.com/user-attachments/assets/bc9a8e9b-527f-42c9-8d1f-e7f725b0fe7d" />

The row count equals the number of unique sessions.
```sql
SELECT ga_session_id, COUNT(*) AS items
FROM `DA.order`
GROUP BY ga_session_id
HAVING COUNT(*) >= 2;
-- 0 rows
```
<img width="411" height="417" alt="image" src="https://github.com/user-attachments/assets/083d67be-3267-443e-825b-1c6c75b15b89" />

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
FROM `DA.account_session` AS ass
JOIN `DA.order` AS o
ON ass.ga_session_id = o.ga_session_id;
-- 2781
```
<img width="410" height="411" alt="image" src="https://github.com/user-attachments/assets/d3e389cf-dd2c-454b-81bd-6ea38fc45270" />

I check how many orders have an account attached at all:

```sql
SELECT COUNT(DISTINCT o.ga_session_id)
FROM `DA.order` AS o
JOIN `DA.account_session` AS ass
ON o.ga_session_id = ass.ga_session_id;
-- 2781
```
<img width="407" height="400" alt="image" src="https://github.com/user-attachments/assets/b6e0061b-a781-49c7-b0df-d4e013608ab5" />

To verify the possibility of analyzing repeat purchases, we also determine whether there are any accounts associated with more than one order:

```sql
SELECT
    ass.account_id,
    COUNT(*) AS orders
FROM `DA.order` AS o
JOIN `DA.account_session` AS ass
ON o.ga_session_id = ass.ga_session_id
GROUP BY ass.account_id
HAVING COUNT(*) > 1;
-- 0
```
<img width="430" height="520" alt="image" src="https://github.com/user-attachments/assets/cf7537d8-77ec-4aed-905d-7168b563c159" />

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

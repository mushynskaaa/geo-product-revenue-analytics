# Geo-Product-Revenue-Analytics
## What this project is about
This project looks at customer behavior and revenue for an e-commerce business: where customers are located, what they buy, how they respond to marketing, and how revenue moves over time. The point is to take raw transaction data and figure out what's actually going on with the business.

## Why it matters
A lot of business decisions get made without really looking at the data behind them. Without knowing which markets or products actually deserve attention, teams end up relying on assumptions instead of evidence. This project works through some of those questions using SQL and logic.

## Project structure
The analysis is split into a few stages:

1. **Geography & Revenue Overview** - which countries bring in the most revenue, and what people are actually buying there
2. **Product ABC Analysis** - which products carry most of the revenue, and which ones barely matter
3. **Traffic Channels** - which channels bring paying customers, and whether traffic volume actually matches revenue
4. **Revenue Trends** - what the shape of revenue looks like over the period

Every stage has its own folder: the SQL query, a screenshot of it running, and the findings.

## Key findings

Revenue is concentrated on one market, not a group. The United States alone accounts for 43.98% of revenue, the second market, India, five times less. 80% of revenue is reached only at the 14th of 107 countries: one dominant market plus a long tail. That's both a clear priority and a heavy dependence on a single market.

The 80/20 rule doesn't hold for products, it's closer to 80/34. It takes 851 products (34% of the catalogue), not 20%, to reach 80% of revenue, so there are no standout hit products. At the other end, group C (970 products) plus 434 products with zero sales, roughly half the catalogue, contribute almost nothing.

Traffic volume equals revenue, there's no vanity channel. Every channel converts at almost exactly the same rate (~9.58%) with near-identical order values, so a channel's revenue depends only on how much traffic it brings. Organic Search leads (35.8%), ahead of Paid Search,  free search, not advertising, is the largest revenue source.

The revenue trend can't be measured, only its shape. With three months of one peak season and no prior year to compare against, there's no basis for a growth conclusion. The weekly view shows a mild peak in early December and, notably, no dramatic holiday effects.

## What the data turned out to be

The most important finding isn't about the business but about the data itself. Three independent observations point the same way: conversion is identical across completely different traffic channels; Black Friday produces no spike and the December peak is under 2× the off-weeks; and there's no January slump despite normal post-holiday behavior. Individually each is odd; together they indicate an educational dataset rather than a real business. The business conclusions above are drawn honestly within the data.

## Data

- **Period:** 2020-11-01 - 2021-01-27 (orders; sessions run to 2021-01-31), a single three-month seasonal peak
- **Volume:** 33,538 orders, 107 countries, 2,962 products, 2,781 accounts
- **Unit of analysis:** order / session - only ~8% of orders link to an account, so 92% are anonymous and customer-level metrics aren't available

Before the analysis I validated the data against its documentation and found, that one session equals one order. Full detail in [`data-validation.md`](data-validation.md).

### Dataset structure
---

The dataset describes an online business: user sessions, orders, products, subscriptions and email communication. The session identifier is `ga_session_id`.

#### Tables used in the project

**`order`** - orders. One row = one purchased unit.

| Column | Description |
|---|---|
| `ga_session_id` | Session in which the purchase happened |
| `item_id` | Product identifier |

**`session`** - user sessions.

| Column | Description |
|---|---|
| `date` | Session date |
| `ga_session_id` | Session identifier |

**`session_params`** - additional session information.

| Column | Description |
|---|---|
| `ga_session_id` | Session identifier |
| `device` | Device type (desktop, mobile, tablet) |
| `mobile_model_name` | Mobile device model |
| `operating_system` | Operating system |
| `language` | Browser language |
| `browser` | Browser |
| `continent` | Continent |
| `country` | Country by IP |
| `medium` | Traffic source identifier |
| `name` | Additional source information |
| `channel` | General traffic channel |

**`product`** - product catalogue.

| Column | Description |
|---|---|
| `item_id` | Product identifier |
| `name` | Product name |
| `category` | Product category |
| `price` | Price, USD |
| `short_description` | Short product description |

**`account`** - site subscribers (users who left an email).

| Column | Description |
|---|---|
| `id` | Subscriber identifier |
| `send_interval` | Email sending interval |
| `is_verified` | Email verified (0/1) |
| `is_unsubscribed` | Unsubscribed (0/1) |

**`account_session`** - link between a subscriber and a session.

| Column | Description |
|---|---|
| `account_id` | Subscriber identifier |
| `ga_session_id` | Session identifier |

## Tools
SQL (BigQuery), Looker Studio, and this repo for keeping it organized.

## Get in touch
Feel free to connect on [LinkedIn](https://www.linkedin.com/in/yuliia-mushynska-a31141346), or check out more dashboards on [Tableau Public](https://public.tableau.com/app/profile/yuliia.mushynska).

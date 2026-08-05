# Geo-Product-Revenue-Analytics
## What this project is about
This project looks at customer behavior and revenue for an e-commerce business - where customers are located, what they buy, how they respond to marketing, and how revenue moves over time. The point is to take raw transaction data and figure out what's actually going on with the business.

## Why it matters
A lot of business decisions get made without really looking at the data behind them. Without knowing which markets or products actually deserve attention, teams end up relying on assumptions instead of evidence. This project works through some of those questions using SQL and logic.

## Project structure
The analysis is split into a few stages:

1. **Geography & Revenue Overview** — which countries bring in the most revenue, and what people are actually buying there
2. **Devices & Traffic Channels** — does it matter whether someone shops from mobile or desktop, and which channels bring in paying customers
3. **Revenue Trends Over Time** — is the business actually growing, or does it just look that way month to month
4. **Product ABC Analysis** — which products are carrying 
   most of the revenue, and which ones barely matter

Every stage has its own folder - the SQL query, a screenshot of it running, the dashboard, and the findings.

### Dataset structure

The dataset describes an online business: user sessions, orders, products,
subscriptions and email communication. The session identifier is `ga_session_id`.

#### Tables used in the project

**`order`** — orders. One row = one purchased unit.

| Column | Description |
|---|---|
| `ga_session_id` | Session in which the purchase happened |
| `item_id` | Product identifier |

**`session`** — user sessions.

| Column | Description |
|---|---|
| `date` | Session date |
| `ga_session_id` | Session identifier |

**`session_params`** — additional session information.

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

**`product`** — product catalogue.

| Column | Description |
|---|---|
| `item_id` | Product identifier |
| `name` | Product name |
| `category` | Product category |
| `price` | Price, USD |
| `short_description` | Short product description |

**`account`** — site subscribers (users who left an email).

| Column | Description |
|---|---|
| `id` | Subscriber identifier |
| `send_interval` | Email sending interval |
| `is_verified` | Email verified (0/1) |
| `is_unsubscribed` | Unsubscribed (0/1) |

**`account_session`** — link between a subscriber and a session.

| Column | Description |
|---|---|
| `account_id` | Subscriber identifier |
| `ga_session_id` | Session identifier |

## Tools
SQL (BigQuery), Looker Studio, and this repo for keeping it organized.

## Get in touch
Feel free to connect on [LinkedIn](www.linkedin.com/in/yuliia-mushynska-a31141346), or check out more dashboards on [Tableau Public](https://public.tableau.com/app/profile/yuliia.mushynska).

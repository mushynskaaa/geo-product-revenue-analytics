## Questions

1. Which countries generate the most revenue?
2. How concentrated is revenue, how many countries form its main share?
3. What is bought in the top countries, does the product assortment differ by market?

## Hypothesis

The majority of revenue is concentrated in 5-7 countries, accounting for 80% of total revenue.

## Part 1. Revenue by country

### Query

```sql
WITH
  country_revenue AS (
    SELECT sp.country AS country
        , SUM(p.price) AS revenue
    FROM `DA.session_params` AS sp
    JOIN `DA.order` AS o
      ON sp.ga_session_id = o.ga_session_id
    JOIN `DA.product` AS p
      ON o.item_id = p.item_id
    WHERE country != '(not set)'
    GROUP BY sp.country
  ),
  cml_item AS (
    SELECT country
        , revenue
        , revenue / SUM(revenue) OVER () * 100 AS revenue_pct
        , SUM(revenue) OVER (ORDER BY revenue DESC) AS cumulative_revenue
        , SUM(revenue) OVER (ORDER BY revenue DESC) / SUM(revenue) OVER () * 100 AS cumulative_pct
    FROM country_revenue
  )


SELECT country
    , revenue
    , revenue_pct
    , cumulative_revenue
    , cumulative_pct
FROM cml_item
ORDER BY revenue DESC;
```

### Results

<img width="883" height="627" alt="image" src="https://github.com/user-attachments/assets/1c508088-57d2-4d89-a77a-ce878306c37f" />

The first 14 countries account for 80% of total revenue, while the remaining 93 countries contribute the remaining 20%.

<img width="1248" height="817" alt="image" src="https://github.com/user-attachments/assets/692d0e82-9935-4f80-8068-7c2ebd4345c5" />

### Findings

My hypothesis was not supported. The 80% threshold falls not at 5-7 countries but at the fourteenth. Revenue is concentrated not across a group of fourteen countries but primarily in a single one: the United States alone accounts for almost half of all revenue (43.98%).

The second-largest market, India, trails the leader almost fivefold (8.86%), and every country after that contributes progressively less. So the structure is not an even spread across fourteen markets but one dominant market followed by a long tail of more than ninety countries that together make up only the final 20%.

In the end the concentration turned out weaker than expected by the number of countries, but far stronger by dependence on a single leader. That reframes the conclusion: the concentration is driven by the dominance of the United States, not by a small group of comparable markets.

---

## Part 2. Product categories in the top-5 countries

Revenue is used as the primary ranking metric, while the number of orders is shown for context to distinguish high-revenue markets from high-order-volume markets.

### Query

```sql

WITH
top_5_countries_by_revenue AS (
  SELECT country
      , SUM(p.price) AS country_revenue
  FROM `DA.session_params` AS sp
  JOIN `DA.order` AS o
  ON sp.ga_session_id = o.ga_session_id
  JOIN `DA.product` AS p
  ON o.item_id = p.item_id
  GROUP BY country
  ORDER BY country_revenue DESC
  LIMIT 5
),

country_by_category AS (
  SELECT country
      , category
      , SUM(p.price) AS category_revenue
      , ROW_NUMBER() OVER (PARTITION BY country ORDER BY SUM(p.price) DESC) AS rank
      , COUNT(*) AS cnt_orders
FROM `DA.session_params` AS sp
JOIN `DA.order` AS o
ON sp.ga_session_id = o.ga_session_id
JOIN `DA.product` AS p
ON o.item_id = p.item_id
GROUP BY country, category
)

SELECT country_by_category.country AS country
      , category
      , category_revenue
      , cnt_orders
FROM top_5_countries_by_revenue AS top_5_countries
JOIN country_by_category
ON top_5_countries.country = country_by_category.country
WHERE rank <= 3
ORDER BY country_revenue DESC
```

### Results (top-3 categories per country)

<img width="887" height="642" alt="image" src="https://github.com/user-attachments/assets/d89394f1-1895-47ff-832c-f44bf6e98cd7" />

### Findings

The top two categories are consistent across all five largest markets. "Sofas & armchairs" generate the highest revenue, while "Chairs" rank first by order volume, which suggests that the main product mix is similar across all analyzed markets.

A consistent relationship exists between price and demand: the lower-priced category ("Chairs") leads in order volume, whereas the higher-priced category ("Sofas & armchairs") generates the highest revenue. For example, in the United States, 2,576 chair orders exceed 1,903 sofa orders, yet sofas generate more revenue due to a substantially higher average order value. The same pattern is observed across all five markets. So while "Sofas & armchairs" are the highest-revenue category, "Chairs" receive the highest number of orders in almost every market.

India differs from the overall pattern. Unlike the other four markets, where "Beds" rank third by revenue, India's third position is occupied by "Bookcases & shelving units". Moreover, this category records the highest order volume in India (734 orders), exceeding both "Chairs" and "Sofas & armchairs". But its relatively low average price keeps it in third place by revenue.

## Caveats

Unit of analysis is the order rather than the customer. Therefore, the statement "The United States is the largest market by revenue" is fully supported by the data, whereas the conclusion "The United States has the largest customer base" cannot be drawn because most orders cannot be linked to individual customers.

Country data is determined from session IP addresses rather than billing addresses. VPN usage, travel, and other factors may introduce inaccuracies in geographic attribution. But, the substantial lead of the United States (43.98% of total revenue) is unlikely to be explained solely by geolocation errors, making this conclusion robust.

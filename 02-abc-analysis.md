## Questions

1. How concentrated is revenue within a small group of products?
2. How many products form each group: A (main revenue), B (middle), C (losses)?

## Hypothesis

Before running the analysis I expected a distribution close to the Pareto principle: roughly **20% of products (~593)** forming group A and generating 80% of total revenue.

## Analysis frame

Classic ABC analysis is built on annual data, but the available period covers only three months of the seasonal peak (November 2020 - January 2021). For that reason I treat this as an analysis of **revenue concentration for a specific period**, not a product classification - a product that lands in group A here is a leader for this time period, not a permanently important one.

A validation of the product identifiers confirmed that this distinction is important: there are 2,962 distinct item_id values but only 607 distinct product names. Grouping by product name would therefore silently combine different products under the same name, distorting the ABC classification and subsequent results.

## Query

```sql
WITH
    item_revenue AS (
    SELECT p.item_id as item_id
        , name
        , category
        , SUM(p.price) AS revenue
    FROM `DA.product` AS p
    JOIN `DA.order` AS o
      ON p.item_id = o.item_id
    GROUP BY item_id, name, category
  ),
  cml_item AS (
    SELECT item_id
        , name
        , category
        , revenue
        , revenue / SUM(revenue) OVER () * 100 AS revenue_pct
        , SUM(revenue) OVER (ORDER BY revenue DESC) AS cumulative_revenue
        , SUM(revenue) OVER (ORDER BY revenue DESC) / SUM(revenue) OVER () * 100 AS cumulative_pct
    FROM item_revenue
  ),


abc_result AS (
  SELECT item_id
    , name
    , category
    , revenue
    , revenue_pct
    , cumulative_revenue
    , cumulative_pct
    , CASE WHEN cumulative_pct <= 80 THEN 'A'
    WHEN cumulative_pct <= 95 THEN 'B'
    ELSE 'C' END AS ABC
FROM cml_item
ORDER BY revenue DESC
)

SELECT abc
    , COUNT(*) as cnt_items
    , COUNT(*) / SUM(COUNT(*)) OVER () * 100 as items_pct
    , SUM(revenue_pct) as revenue_pct_it
FROM abc_result
GROUP BY abc
ORDER BY abc
```

## Results

<img width="866" height="285" alt="image" src="https://github.com/user-attachments/assets/0b551891-5df2-4852-9a16-6d44ea525641" />

Only 2,528 products entered the analysis - those sold at least once. The gap to the full catalogue (2,962 - 2,528 = 434 products) represents items with zero sales during the period.

## Findings

The hypothesis was not supported. Group A is formed not by approximately 20% of products, but by 851 products (34% of the catalogue). In other words, generating 80% of total revenue requires roughly one-third of all products, making the observed distribution closer to “80/34” than to the classic 80/20 pattern.

This indicates that revenue is less concentrated at the product level than the Pareto principle would suggest. The business is not dependent on a small handful of individual SKUs. Instead, a relatively broad group of products contributes to the majority of revenue.

At the other end of the distribution, nearly half of the catalogue contributes very little. Group C contains 970 products that together generate only 5% of total revenue, while an additional 434 products recorded no sales during the analyzed period. Combined, these approximately 1,400 products represent around half of the catalogue while contributing little or no revenue.

The contrast with the geographic analysis is particularly notable. Revenue is highly concentrated geographically, with the United States alone accounting for 44% of total revenue. At the product level, however, concentration is weaker: 34% of products are required to generate 80% of revenue. This suggests two different concentration patterns: the business is strongly dependent on one major market, while revenue within the product catalogue is distributed across a relatively broad range of products.

## Recommendation

Group A (851 products) drives the majority of revenue during the analyzed period and should therefore be given priority in inventory planning and promotional activity, while keeping the seasonal nature of the observation window in mind.

Group C, together with the 434 products with no recorded sales, represents the main area for assortment review. But, low or zero sales alone should not automatically lead to delisting. Before taking action, the underlying reason needs to be investigated, as a product may show low sales because it is newly introduced, temporarily out of stock, or affected by other factors not captured in the available data.

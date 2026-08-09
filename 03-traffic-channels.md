## Questions

1. Which channel brings in the main share of revenue, and how do conversion and average order value differ across channels?
2. Does the ranking of channels by revenue match their ranking by traffic volume or are there channels that draw a lot of visits but little money?

## Hypothesis

Before running the analysis I expected revenue to split roughly as: Paid Search ~30% (the leader), Direct ~25%, Organic Search ~25%, Undefined ~10%, Social Search ~10%. I also expected the ranking by traffic to match the ranking by revenue only partially, that some high-traffic channel would underperform on money.

## Query

```sql
WITH channel_metrics AS (
    SELECT channel
           , COUNT(DISTINCT sp.ga_session_id) AS total_sessions
           , COUNT(DISTINCT o.ga_session_id) AS paid_sessions
           , SUM(p.price) AS revenue
           , COUNT(o.item_id) AS orders
    FROM `DA.session` s
    JOIN `DA.session_params` sp
    ON s.ga_session_id = sp.ga_session_id
    LEFT JOIN `DA.order` o
    ON s.ga_session_id = o.ga_session_id
    LEFT JOIN `DA.product` p
    ON o.item_id = p.item_id
    GROUP BY channel
)

SELECT channel
    , total_sessions
    , SAFE_DIVIDE(paid_sessions, total_sessions) * 100 AS conversion
    , revenue
    , SAFE_DIVIDE(revenue, orders) AS average_order_value
    , SAFE_DIVIDE(revenue, SUM(revenue) OVER()) * 100 AS revenue_share
FROM channel_metrics
ORDER BY revenue DESC;
```

## Results

<img width="1025" height="303" alt="image" src="https://github.com/user-attachments/assets/0e5b4fbc-80ff-4bf3-80e5-f8c3b466260d" />

## Findings

My hypothesis was half right. The revenue split across channels came out close to what I expected in proportion, but I got the leader wrong. I predicted Paid Search would top the list, whereas the actual leader is Organic Search at 35.8%. Free search traffic, not paid advertising, is the largest source of revenue, with Paid Search second at 26.6% and Direct third at 23.4%.

The part of the hypothesis I expected to hold a only partial match between traffic and revenue did not hold, the match is in fact complete. The ranking of channels by number of sessions reproduces the ranking by revenue exactly. There is no channel that pulls in a lot of traffic but little money.

The reason becomes clear from the other two metrics, conversion is practically identical across channels (around 9.58% everywhere) and average order value sits in a band of roughly 930–970. Because every channel converts equally well and every order is worth about the same, a channel's revenue depends almost entirely on how much traffic it brings.

## Caveat

The near-identical conversion is too clean to be a coincidence, so I checked it: `session_params` has exactly one row per session, which rules out row multiplication on the join. The uniformity is therefore real in this data. That said, in a real business different channels almost always convert differently, so this uniformity more likely reflects educational nature of the dataset. 

## Question

1. How does revenue behave over the period, where are the peaks and dips, and is there a discernible seasonal shape?
I don`t ask "is the business growing", because the data cannot answer that, with a single three-month period and no prior year to compare against, there is no baseline for a year-over-year trend. So this slice describes revenue within the time period, not its long-term direction.

## Approach

I look at the series at two granularities. The daily level is meant to catch specific anomalies (individual high or low days such as Black Friday or holiday spikes). The weekly level reveals the overall shape: build-up, peak, decline.

## Queries

```sql
-- daily
SELECT date
    , SUM(price) AS revenue
FROM `DA.session` AS s
JOIN `DA.order` AS o
ON s.ga_session_id = o.ga_session_id
JOIN `DA.product` AS p
ON o.item_id = p.item_id
GROUP BY date
ORDER BY date;

-- weekly
SELECT DATE_TRUNC(date, WEEK) AS week
    , SUM(price) AS revenue
FROM `DA.session` AS s
JOIN `DA.order` AS o
ON s.ga_session_id = o.ga_session_id
JOIN `DA.product` AS p
ON o.item_id = p.item_id
GROUP BY week
ORDER BY week;
```

## Results (weekly)

<img width="497" height="527" alt="image" src="https://github.com/user-attachments/assets/d1fffce1-bbe9-4620-a459-f99470fb5f27" />

<img width="1480" height="381" alt="image" src="https://github.com/user-attachments/assets/2063c281-c24f-41f0-9276-b2f10458a8d6" />

## Findings

At the daily level the series is surprisingly even. There is no pronounced holiday spike and no expected January slump, the days fluctuate without a clear rise or fall, and the best-selling days are scattered across December plus one day in early January. Black Friday does not stand out at all.

I also checked whether the weak start in November was a real low season or data collection beginning. It is not an second: revenue on 1 November (244k, 281 orders) is already a full day, and 3 November is one of the highest days of the whole period. The data is complete from day one, so November's figures can be trusted.

The weekly view gives a clearer shape, a gradual build-up through November, a peak in the first week of December (3.54M), a decline toward the end of December, and a renewed rise in January. I disregard the final week (starting 24 January) as a comparison point, the data ends on 27 January, so that week contains only four days and its lower figure is artificial, not a drop in demand.

The most telling detail is the size of the peak. The December peak (3.54M) is less than twice the low weeks (~2M). In e-commerce the December peak is typically three to five times the off-season, here it is under 2×. Taken together with the absent Black Friday spike and the absence of a January slump, this points the same way as the uniform conversion found in the channels slice - the data most likely reflects educational dataset rather than a real seasonal pattern.

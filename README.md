# product-analyst-portfolio--ga4-ecommerce-analysis
# Product Analyst Portfolio

## GA4 E-commerce Case Study: Google Merchandise Store

GA4 dataset analysis in BigQuery:
- CR = 1.25%
- Retention D1 = 6.4%
- Bottleneck: 70% drop in add_to_cart
- Growth hypotheses: +12–18% revenue

Full case in PDF: [https://github.com/Bohdan-Borysenko/product-analyst-portfolio--ga4-ecommerce-analysis/blob/main/PA1.pdf]

Link to Notion: [https://tundra-leaf-b86.notion.site/GA4-E-commerce-Analysis-Google-Merchandise-Store-2c90c0bce00a80fc8c3fd275494eefa3]

##
--1. Overall funnel --

- - Total funnel for all days

```sql
SELECT 
  COUNT(DISTINCT user_pseudo_id) AS unique_users,
  COUNTIF(ecommerce.purchase_revenue IS NOT NULL) AS buyers,
  ROUND(100.0 * COUNTIF(ecommerce.purchase_revenue IS NOT NULL) / COUNT(DISTINCT user_pseudo_id), 2) AS conversion_rate_percent
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`;
```

--[<img width="431" height="52" alt="Снимок экрана 2025-12-14 в 23 41 16" src="https://github.com/user-attachments/assets/16a98bed-9d1c-45a7-992e-e9adb66312eb" />]--

Result:
~280k users, ~3.5k purchases, CR = 1.25%

-- 2. Event funnel (page_view → add_to_cart → purchase) --
```sql
SELECT
event_name,
COUNT(*) AS events
FROM bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*
WHERE event_name IN ('page_view', 'add_to_cart', 'purchase')
GROUP BY event_name
ORDER BY events DESC;
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`;
```
--[<img width="382" height="106" alt="Снимок экрана 2025-12-14 в 23 42 39" src="https://github.com/user-attachments/assets/98f7260c-223c-4fbc-9767-3140c1b9e77a" />]--

-- 3. Conversion by device --
```sql
SELECT 
  device.category,
  COUNT(DISTINCT user_pseudo_id) AS users,
  COUNTIF(ecommerce.purchase_revenue IS NOT NULL) AS buyers,
  ROUND(100.0 * COUNTIF(ecommerce.purchase_revenue IS NOT NULL) / COUNT(DISTINCT user_pseudo_id), 2) AS cr
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
GROUP BY device.category;
```
--[<img width="632" height="105" alt="Снимок экрана 2025-12-14 в 23 44 26" src="https://github.com/user-attachments/assets/2cd97e0f-7151-4144-95dc-4361c96fe489" />]--

-- 4. Top 10 products by revenue --
```sql
SELECT
items.item_name,
SUM(ecommerce.purchase_revenue) AS revenue
FROM bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*,
UNNEST(items) AS items
WHERE ecommerce.purchase_revenue IS NOT NULL
GROUP BY items.item_name
ORDER BY revenue DESC
LIMIT 10;
```
--[<img width="388" height="296" alt="Снимок экрана 2025-12-14 в 23 47 15" src="https://github.com/user-attachments/assets/112598a6-bd8a-4e89-b0dd-34b429ec42a6" />]--

-- 5 Retention D0 / D1 / D7 / D30 --
```sql
WITH first AS (
SELECT
user_pseudo_id,
MIN(PARSE_DATE('%Y%m%d', event_date)) AS first_date
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
GROUP BY user_pseudo_id
)

SELECT
DATE_DIFF(PARSE_DATE('%Y%m%d', e.event_date), f.first_date, DAY) AS days_since_first,
COUNT(DISTINCT e.user_pseudo_id) AS returning_users
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` e
JOIN first f USING (user_pseudo_id)
WHERE DATE_DIFF(PARSE_DATE('%Y%m%d', e.event_date), f.first_date, DAY) IN (0, 1, 7, 30)
GROUP BY days_since_first
ORDER BY days_since_first;
```
--[<img width="310" height="138" alt="Снимок экрана 2025-12-14 в 23 48 30" src="https://github.com/user-attachments/assets/e40d8e72-0e46-476f-9c54-534e63cd5f4d" />]--

## Insights

- CR = 1.25% (low for e-commerce).
- Retention D1 = 6.4% (norm 10–15%).
- Mobile CR lover desktop на 40%.
- 70% drop between view_item и add_to_cart.

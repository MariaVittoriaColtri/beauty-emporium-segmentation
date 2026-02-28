# Customer Segmentation & E-commerce Analysis - The Beauty Emporium

## Overview

This project analyses 12 months of transactional data from a beauty e-commerce brand. The analysis follows the OSEMN framework (Obtain, Scrub, Explore, Model, Interpret) and addresses three business questions from the Global Marketing Manager: how did the business perform in year one, which product should be recommended per city and who are the customers.

The framework was not strictly linear in practice. I returned to the scrub phase twice during exploration. The mixed casing in the city column only surfaced when a groupby operation returned double the expected number of distinct cities.

## Tools

Python: pandas, numpy, scikit-learn, matplotlib  
Tableau for dashboards (three CSV files exported from the notebook)

## Methodology

The dataset has two tables: orders (12,482 rows) and products (15,600 rows). I joined them on `order_id` using a left join, so that all orders were kept even when there was no matching product record. An inner join would have silently removed those rows and understated total revenue. I also checked for duplicates in the products table before merging, because an undetected duplicate key would have inflated row counts without producing any error.

For missing values, 128 rows had a null in `order_price`. The distribution was right-skewed (mean 23.90 euro vs. median 15 euro), so I imputed using the median. The mean was pulled up by a small number of high-value orders and was not a reliable estimate of the typical order value.

For duplicates, 600 fully duplicated rows were identified (same customer ID, timestamp, and price) and removed. These were treated as data entry errors rather than genuine repeat purchases.

City and product name columns contained mixed casing. I applied `.str.title()` to standardise both before any aggregation.

## Results

A- **E-commerce performance overview**

Total revenue over 12 months was approximately 240,000 euro, with an average of 20,000 euro per month. The dataset contains 11,482 unique customers and 14,496 unique orders, which gives an average of 1.26 orders per customer. This means that most customers placed only one order during the year, with a small group of repeat buyers pulling the average up.

About 17% of orders received a discount. This was calculated by reconstructing the theoretical price for each order using the business rule (number of products x 4 euro + 3 euro shipping fee) and comparing it to the actual order price. Revenue was stable across all 12 months with no significant seasonal variation.

B- **Product recommendation by city**

For each city, I ranked products by total quantity sold and selected the top-ranked item as the recommended gift for reactivation campaigns. The assumption is that the product with the highest existing purchase volume in a given city is the safest recommendation for driving repeat engagement, because it is already proven to sell there.

C- **Customer segmentation (K-Means, k=3)**

I calculated three RFM metrics per customer: Recency (days since last purchase), Frequency (number of orders), and Medium Cart (average order value). I then normalised them with MinMaxScaler before clustering. Normalisation was necessary because the features operate on very different scales: recency ranges from 1 to 365 days, while frequency ranges from 1 to 14. Without normalisation, K-Means would have weighted recency disproportionately just because its numeric range is larger.

Cluster labels were assigned after fitting by examining the mean RFM values per group, since K-Means assigns numbers arbitrarily. The three segments are a high-value group with low recency, high frequency and high average cart; a mid-tier group of 5,009 customers with medium recency and predominantly one-time purchase behaviour; and a dormant group with an average recency of 291 days.

## Recommendations for the Marketing Team

- **Focus re-engagement campaigns on the one-time buyers first.** With 5,009 customers, this is the largest group and the most actionable one. They have already converted once and have not yet lapsed, which means the barrier to a second purchase is lower than for dormant customers. A targeted campaign with a personalised product recommendation based on their city could be enough to bring them back.

- **Personalise reactivation offers based on what sells best in each city.** The analysis identified the top-selling product for each city. Using a familiar product in a campaign reduces friction because the customer has already shown interest in that category, so the recommendation feels relevant rather than generic.

- **Treat the high-value segment as a retention priority** These customers already buy frequently and spend more per order. The goal is to protect that behaviour, not to push harder. Loyalty programmes or early access to new products are more appropriate than discount-driven campaigns for this group.

- **Use discounts selectively and only for dormant customers.** The discount rate in this dataset is already at 17%. Given that the average order value is between 15 and 23 euro, a discount needs to generate a significant volume increase just to break even. Discounts make the most sense for dormant customers where the alternative is losing them permanently, not as a default tool across all segments.

- **Invest in increasing purchase frequency before acquiring new customers.** The average of 1.26 orders per customer shows that retention is the main opportunity here. Acquiring new customers is more expensive than reactivating existing ones, and the current customer base is largely untapped from a repeat purchase perspective.

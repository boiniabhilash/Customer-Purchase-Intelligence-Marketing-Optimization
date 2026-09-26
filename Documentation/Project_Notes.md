# Customer Purchase Intelligence & Marketing Optimization

## 1. Project Scope

This project analyzes customer purchasing behavior and product relationships using the Dunnhumby "The Complete Journey" dataset.

The objective is to bridge customer analytics and marketing decision-making by identifying:

- Customer purchase frequency and value patterns
- Basket size and purchasing behavior
- Product and category performance
- Frequently purchased product combinations
- Cross-sell opportunities
- Bundle opportunities
- Customer purchase segments
- Campaign assignment and coupon redemption patterns
- Marketing opportunities based on observed purchasing behavior

The final analytical solution was built entirely in Power BI using:

- Power Query for data preparation
- DAX for analytical calculations and business metrics
- Power BI data modeling for relationships
- Power BI visuals for analysis and decision support

The project is observational. Product associations and marketing patterns are used to generate business hypotheses and testable opportunities rather than claims of causal impact.

## 2. Data Preparation & Cleaning

### Transaction Data

The transaction table contains approximately 2.6 million transaction-line records.

The following cleaning rule was applied to create the analytical transaction table:

- `QUANTITY > 0`
- `QUANTITY <= 100`

This removed 37,602 records, approximately 1.45% of the original transaction rows.

Zero-sales records were retained because they may represent valid transaction activity such as discounts, adjustments, or other non-revenue purchase records.

### Analytical Transaction Table

After cleaning:

- Transaction rows: 2,558,130
- Analytical baskets: 253,605
- Customers: 2,500
- Unique purchased products: 91,991
- Total sales: $7,453,105.40

A basket was considered an analytical basket when it contained at least one valid transaction line after applying the quantity filter.

### Product Data

The product master was linked to transaction data using `PRODUCT_ID`.

Product attributes used for analysis include:

- Department
- Commodity
- Sub-commodity
- Brand
- Product ID

This product hierarchy supports analysis from broad product departments down to individual products.

### Customer Data

Customer-level analysis was based on the 2,500 households present in the cleaned transaction data.

The demographic table contains information for only 801 of these households. Therefore, demographic information was not treated as representative of the complete customer population.

### Data Model

The Power BI model uses a dimensional structure with transaction data as the primary fact table and product, household, customer, and date dimensions supporting analysis.

Relationships were configured to allow filtering from dimensions into transaction data while avoiding unnecessary bidirectional relationships.


## 3. Analytical Methodology

The analysis follows a structured progression from customer behavior to product relationships and marketing opportunities.

### 3.1 Customer & Basket Analysis

Customer-level metrics were calculated using household purchase history.

Key measures include:

- Number of baskets per customer
- Products purchased
- Units purchased
- Total customer sales
- Average basket value

Customers were segmented by purchase frequency:

- Low Frequency: 1–10 baskets
- Occasional: 11–50 baskets
- Regular: 51–100 baskets
- Frequent: 101–200 baskets
- Very Frequent: 201+ baskets

Basket-level analysis was also performed by grouping baskets according to the number of distinct products purchased.

### 3.2 Product Performance

Product performance was evaluated using:

- Sales
- Units
- Number of baskets containing the product
- Number of customers purchasing the product
- Basket penetration

Product performance was analyzed across department, commodity, sub-commodity, brand, and product levels.

### 3.3 Market Basket Analysis

Market basket analysis was used to identify products that frequently occur together within the same shopping basket.

To keep the association analysis commercially meaningful and computationally manageable, candidate products were restricted to products appearing in at least 500 baskets.

This produced:

- 590 candidate products
- 181,929 candidate baskets
- 164,648 unique product pairs

For each product pair, the following association metrics were calculated:

- Support
- Confidence
- Lift

### 3.4 Association Rules

A pair was considered eligible for stronger association analysis when it appeared in at least 50 baskets.

A strong association was defined using:

- Pair baskets ≥ 50
- Lift ≥ 2

Under these business rules:

- 5,843 strong eligible product associations were identified
- Strong association rate: 3.55%

These thresholds are project-specific business rules and should not be interpreted as universal statistical standards.

### 3.5 Cross-Sell & Bundle Opportunities

Association rules were translated into marketing opportunities.

A pair was classified as a:

**Bundle Opportunity** when:

- Pair baskets ≥ 50
- Lift ≥ 2
- Confidence A → B ≥ 10%
- Confidence B → A ≥ 10%

A pair was classified as a:

**Cross-Sell Opportunity** when:

- Pair baskets ≥ 50
- Lift ≥ 2
- The pair did not satisfy the two-way confidence threshold for bundling

This resulted in:

- 183 Bundle Opportunities
- 5,660 Cross-Sell Opportunities

These classifications represent observed purchase patterns and proposed marketing opportunities, not proven incremental sales effects.

## 4. Marketing & Promotion Analysis

The project includes campaign assignment and coupon redemption data to connect observed customer purchasing behavior with available marketing activity.

### 4.1 Campaign Analysis

Campaign assignment data was linked to customers using `household_key`.

The dataset contains:

- 30 campaigns
- 7,208 campaign assignments
- 1,584 customers assigned to at least one campaign
- Campaign customer coverage: 63.36%

Campaign analysis examines:

- Campaign reach
- Number of households assigned to each campaign
- Campaign type
- Observed sales among households associated with campaigns
- Sales per campaign-associated household
- Campaign duration

Campaign-associated sales represent the total observed sales of households assigned to a campaign.

They are **not treated as campaign revenue**, because:

- A household may be assigned to multiple campaigns
- Purchases cannot be uniquely attributed to a specific campaign
- Campaign cost data is not available
- The dataset is observational

Therefore, campaign ROI and causal campaign effectiveness were not calculated.

### 4.2 Campaign Type Analysis

Campaigns were grouped into their available campaign descriptions, including TypeA, TypeB, and TypeC.

The analysis compares:

- Campaign reach
- Associated household counts
- Observed household sales
- Sales per associated household

These comparisons are descriptive and are not interpreted as evidence that one campaign type caused higher customer spending.

### 4.3 Coupon Redemption Analysis

Coupon redemption data contains:

- 2,318 redemption records
- 434 households with recorded redemptions
- 30 campaigns

The overall coupon redemption household rate is approximately 17.36% of the 2,500 transaction households.

Campaign-level redemption rates were calculated only for campaigns reaching at least 50 households to reduce instability from very small campaign populations.

### 4.4 Coupon Product Mapping Limitation

Coupon redemption records contain `COUPON_UPC`, while the coupon master contains product mappings.

During data preparation, 1,088 coupon UPCs were found to map to more than one product.

A direct coupon-to-product merge therefore produced a many-to-many expansion and was not used in the final analytical model.

Coupon redemption was consequently analyzed at the household and campaign levels rather than assigning a specific redeemed product.

### 4.5 Campaign Duration

Campaign duration was calculated from campaign start and end days.

The 30 campaigns have an average duration of approximately 47 days.

Campaigns were grouped into:

- Short: ≤30 days
- Medium: 31–60 days
- Long: 61–90 days
- Extended: >90 days

The duration analysis is descriptive and does not establish that campaign length caused differences in customer behavior.

## 5. Key Business Interpretation

The analysis produced several observations that can be translated into marketing and customer strategy hypotheses.

### 5.1 Customer Concentration

Frequent and Very Frequent customers represent 36.56% of the customer base but contribute approximately 65.49% of total customer sales.

This indicates that customer value is concentrated among a relatively smaller portion of the observed customer base.

Potential business implication:

- Retention initiatives can focus on high-frequency customers.
- Personalized recommendations can be prioritized for customers with established purchase histories.
- High-value customers may provide a stronger opportunity for cross-sell and basket-expansion strategies.

### 5.2 Basket Size and Customer Value

Average basket value increases as the number of distinct products in a basket increases.

Observed average basket values range from approximately:

- $5.38 for 1-product baskets
- $8.38 for 2–3 products
- $13.95 for 4–5 products
- $22.47 for 6–10 products
- $41.59 for 11–20 products
- $98.98 for 21+ products

This pattern suggests that increasing the number of relevant products purchased within a basket is associated with higher basket value.

The relationship is observational and does not establish that adding products will automatically increase customer spending.

### 5.3 Product Affinity

The association analysis identified 5,843 product pairs satisfying the project's strong-association criteria.

However, high Lift alone was not treated as sufficient evidence for a marketing recommendation.

Commercial relevance should consider multiple factors together:

- Pair basket volume
- Support
- Confidence
- Lift
- Product sales
- Product basket penetration
- Product category
- Direction of the association

This reduces the risk of recommending extremely rare product combinations simply because they have high Lift.

### 5.4 Cross-Sell Opportunities

The analysis identified 5,660 pairs classified as Cross-Sell Opportunities.

These represent product combinations with meaningful observed association but insufficient two-way confidence to qualify as bundle opportunities under the project's rules.

Potential applications include:

- Product recommendation modules
- "Frequently bought together" placements
- Checkout recommendations
- Category-level cross-sell campaigns

### 5.5 Bundle Opportunities

The analysis identified 183 Bundle Opportunities.

These pairs satisfy the project's stronger two-way confidence criteria in addition to the minimum basket-volume and Lift requirements.

Potential applications include:

- Bundle design
- Multi-product promotions
- Store merchandising
- Digital product recommendations
- Promotional package testing

These opportunities should be validated through controlled business experiments before assuming incremental revenue impact.

### 5.6 Marketing Interpretation

Campaign and coupon analysis provides context around observed customer marketing activity.

However, the available data does not support reliable causal conclusions about campaign effectiveness.

Therefore, the project treats campaign-related findings as descriptive evidence and uses them to generate hypotheses for future testing rather than claiming campaign-driven sales uplift.

### 5.7 Recommended Validation Approach

Potential recommendations derived from the analysis should be validated through controlled tests such as:

- A/B testing
- Holdout groups
- Incremental sales measurement
- Conversion-rate comparison
- Basket-value comparison
- Repeat-purchase measurement

This would allow the business to distinguish correlation from incremental marketing impact.

## 6. Power BI Implementation Notes

The final analytical solution was developed entirely in Power BI.

### 6.1 Power Query

Power Query was used for:

- Loading source CSV files
- Data type validation
- Transaction filtering
- Removing invalid transaction quantities
- Creating reference queries
- Removing unnecessary columns
- Creating basket-product datasets
- Creating candidate product datasets
- Generating product-pair combinations
- Grouping product pairs by basket occurrence
- Preparing supporting analytical tables

The transaction cleaning rule was implemented as:

`QUANTITY > 0 AND QUANTITY <= 100`

### 6.2 Data Model

The core model uses:

- `Fact_Transaction`
- `Dim_Product`
- `Dim_Household`
- `Dim_Customer`
- `Dim_Date`
- `Fact_Campaign`
- `campaign_desc`

Additional analytical tables were created for:

- Customer summaries
- Basket summaries
- Basket-product relationships
- Candidate product analysis
- Association pairs
- Candidate product basket counts

### 6.3 DAX

DAX was used to create:

- Executive KPIs
- Customer metrics
- Basket metrics
- Product performance measures
- Support
- Confidence
- Lift
- Association eligibility
- Cross-sell classification
- Bundle classification
- Customer frequency segmentation
- Campaign coverage metrics
- Coupon redemption metrics

### 6.4 Association Rule Calculation

For each product pair:

**Support**

Measures the proportion of candidate baskets containing both products.

**Confidence A → B**

Measures how frequently product B appears when product A is present.

**Confidence B → A**

Measures the reverse relationship.

**Lift**

Measures how much more frequently the products occur together compared with what would be expected from their individual occurrence rates.

A Lift value above 1 indicates a positive association within the analyzed dataset.

### 6.5 Dashboard Design

The Power BI solution is organized into seven analytical pages:

1. Executive Overview
2. Customer & Basket Analysis
3. Product Performance
4. Product Affinity & Association Rules
5. Customer Segments & Purchase Patterns
6. Marketing & Promotion Analysis
7. Cross-Sell & Bundling Opportunities

The dashboard follows a progression from:

**Business Overview → Customer Behavior → Product Performance → Product Affinity → Customer Segmentation → Marketing Activity → Business Opportunities**

This structure is designed to move from descriptive analysis toward actionable marketing opportunities.

## 7. Validation & Quality Checks

Validation was performed throughout the project to ensure that the Power BI results were consistent with the underlying dataset and analytical definitions.

### 7.1 Data Quality Checks

The source data was checked for:

- Missing values
- Duplicate records
- Invalid transaction quantities
- Invalid product references
- Invalid campaign references
- Invalid dates
- Basket-to-household consistency

The core source tables contained no missing values or exact duplicate rows in the completed audit.

### 7.2 Transaction Validation

Transaction cleaning was validated against the original dataset.

Key checks included:

- Original transaction rows
- Clean transaction rows
- Removed transaction rows
- Total sales
- Total units
- Number of analytical baskets
- Number of customers
- Number of purchased products

The final Power BI transaction table contains 2,558,130 valid transaction rows.

### 7.3 Basket Validation

Basket-level checks confirmed that:

- Each basket maps to one household
- Each basket maps to one transaction day
- Transaction days map consistently to calendar dates
- Analytical baskets contain at least one valid transaction line

The final analytical basket count is 253,605.

### 7.4 Product Validation

All transaction product IDs were checked against the product master.

The transaction data successfully maps to the available product dimension.

The final analytical dataset contains 91,991 unique purchased products.

### 7.5 Association Analysis Validation

Market basket calculations were validated using independent benchmark calculations before being reproduced in Power BI.

The Power BI results were checked for:

- Candidate product count
- Candidate basket count
- Association pair count
- Pair basket counts
- Support
- Confidence
- Lift

The final association universe contains:

- 590 candidate products
- 181,929 candidate baskets
- 164,648 unique product pairs
- 5,843 strong eligible associations

### 7.6 Business Rule Validation

The following rules were explicitly validated:

- Candidate product threshold: ≥500 baskets
- Strong pair threshold: ≥50 baskets
- Strong association threshold: Lift ≥2
- Bundle confidence threshold: ≥10% in both directions

These thresholds are analytical business rules selected for this project and should be recalibrated for a different business or dataset.

### 7.7 Marketing Data Validation

Campaign and coupon analysis was validated separately from transaction analysis.

Important checks included:

- Campaign count
- Campaign assignment count
- Campaign-assigned households
- Coupon redemption records
- Coupon redemption households
- Campaign duration
- Campaign-level redemption calculations

Campaign-associated sales were not treated as campaign-attributed revenue.

### 7.8 Final QA

The completed dashboard was reviewed for:

- Correct KPI values
- Correct filters and slicers
- Appropriate aggregation settings
- Correct sorting
- Consistent metric definitions
- Appropriate visual titles
- Avoidance of misleading causal interpretations
- Consistency between dashboard results and analytical benchmarks

The final Power BI file represents the validated analytical version of the project.

## 8. Limitations & Assumptions

### 8.1 Observational Data

The dataset records observed customer transactions and marketing activity.

Therefore, the analysis identifies associations and patterns but does not establish causal relationships.

### 8.2 Campaign Attribution

Campaign-assigned household sales cannot be interpreted as campaign-generated revenue because:

- Customers may receive multiple campaigns
- Individual purchases cannot be uniquely attributed to a campaign
- Campaign costs are unavailable
- No experimental control group is provided

Campaign findings are therefore descriptive.

### 8.3 Customer Demographics

The demographic table contains information for only 801 of the 2,500 households in the transaction dataset.

Demographic findings should therefore be described as applying to the **available demographic households only** and should not be generalized to all customers.

### 8.4 Coupon Product Mapping

Some coupon UPCs map to multiple products in the coupon master.

Because a direct merge created many-to-many duplication, coupon redemption was analyzed at the household and campaign levels rather than assigning redeemed coupons to individual products.

### 8.5 Association Thresholds

The market basket analysis uses project-specific thresholds:

- Minimum product basket count: 500
- Minimum pair basket count: 50
- Minimum Lift: 2
- Bundle confidence: 10% in both directions

These thresholds were selected to balance computational efficiency and commercial relevance. They are not universal statistical standards.

### 8.6 Rare High-Lift Associations

Lift can become very high for relatively rare product combinations.

For this reason, Lift was not used as the sole recommendation criterion.

Pair volume, support, confidence, product sales, basket penetration, and category context should be considered together.

### 8.7 Business Recommendations

The cross-sell and bundling recommendations represent **testable business hypotheses** based on observed purchasing behavior.

They should be validated through controlled experiments before being treated as evidence of incremental sales or marketing effectiveness.

### 8.8 Dataset Scope

The analysis reflects the customers, products, transactions, campaigns, and time period available in the Dunnhumby "The Complete Journey" dataset.

Results should therefore be interpreted within the scope of this dataset rather than assumed to represent a broader retail population.


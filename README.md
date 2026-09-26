# Customer Purchase Intelligence & Marketing Optimization

**Market Basket Analysis for Cross-Sell, Bundling & Promotion Strategy**

## 1. Project Overview

This project analyzes customer purchase behavior from a large-scale retail transaction dataset to identify product purchasing patterns, customer segments, product affinities, and marketing opportunities.

The analysis was designed to bridge **business analytics and marketing decision-making** by moving beyond basic product and sales reporting. The project combines customer and basket analysis with market basket analysis to identify products that are frequently purchased together and translate those associations into potential cross-sell and bundling opportunities.

The final solution was built entirely in **Power BI**, using Power Query for data preparation, DAX for analytical calculations and business rules, Power BI's data model for integrating the datasets, and interactive dashboards for analysis and decision support.

The analysis covers **2.56 million cleaned transaction records, 253,605 analytical baskets, 2,500 customers, and 91,991 purchased products**.

## 2. Business Problem

Retail businesses generate large volumes of transaction data, but transaction records alone do not clearly show:

* Which customer groups contribute the most sales?
* How frequently do customers purchase?
* How does basket size relate to basket value?
* Which products and product categories drive sales?
* Which products are commonly purchased together?
* Which product combinations have meaningful association strength?
* Where are potential cross-sell and bundle opportunities?
* How can observed customer and purchase patterns support marketing decisions?

The objective of this project is to transform transaction-level retail data into **actionable customer, product, and marketing insights**.

Rather than treating market basket analysis as a purely statistical exercise, the project connects association metrics such as **support, confidence, and lift** with basket volume and customer/product context. This allows product associations to be evaluated from a practical marketing perspective.

The resulting Power BI solution provides an end-to-end view of:

## 3. Business Objectives

The project was designed around the following business objectives:

1. **Understand customer purchasing behavior**

   * Measure customer purchase frequency, basket count, units purchased, and sales contribution.
   * Identify differences between low-frequency and high-frequency customers.

2. **Understand basket behavior**

   * Analyze basket size and basket value.
   * Identify how basket composition varies across different basket-size groups.

3. **Evaluate product performance**

   * Analyze sales, units, customers, and basket penetration across departments, commodities, sub-commodities, and individual products.

4. **Identify product associations**

   * Discover products that frequently occur together within the same basket.
   * Evaluate associations using support, confidence, and lift.

5. **Translate associations into marketing opportunities**

   * Identify potential cross-sell opportunities.
   * Identify stronger two-way associations that may support bundle opportunities.

6. **Analyze customer segmentation**

   * Group customers according to observed purchase frequency.
   * Compare sales contribution and basket behavior across customer segments.

7. **Analyze campaign and coupon activity**

   * Examine campaign reach, campaign assignment, campaign timing, and coupon redemption patterns.
   * Connect observed marketing activity with customer purchase behavior without making causal claims.

8. **Create actionable marketing insights**

   * Convert observed purchase patterns into potential applications such as cross-selling, bundling, personalization, and promotion planning.

## 4. Dataset

### Source

The project uses the **Dunnhumby — The Complete Journey** retail dataset.

The dataset contains household-level transactions together with product, campaign, coupon, and demographic information.

### Files Used

The Power BI model uses the following core files:

| File                   | Purpose                                                                      |
| ---------------------- | ---------------------------------------------------------------------------- |
| `transaction_data.csv` | Customer transactions, baskets, products, quantities, sales and discounts    |
| `product.csv`          | Product master data including department, commodity, sub-commodity and brand |
| `hh_demographic.csv`   | Household demographic information                                            |
| `campaign_table.csv`   | Campaign-to-household assignments                                            |
| `campaign_desc.csv`    | Campaign descriptions and campaign dates                                     |
| `coupon.csv`           | Coupon and product/campaign reference information                            |
| `coupon_redempt.csv`   | Household-level coupon redemption records                                    |

### Dataset Scale

The original transaction dataset contains approximately **2.60 million transaction records**.

After applying the analytical transaction-quality rule:

> `QUANTITY > 0 AND QUANTITY <= 100`

the final transaction table contains approximately **2.56 million valid transaction records**.

The resulting analytical dataset contains:

* **2,500 households/customers**
* **253,605 analytical baskets**
* **91,991 unique purchased products**
* **711 days of transaction activity**
* **582 stores**

### Transaction Cleaning Rule

Rows with zero or negative quantity were excluded from the analytical transaction table. Extremely high quantity values above 100 were also excluded as potential data-quality anomalies.

Zero-sales records were retained when they satisfied the quantity rule because removing them solely on sales value could alter the underlying transaction structure.

A basket was considered an **analytical basket** when it contained at least one valid transaction line after this cleaning step.

### Market Basket Analysis Universe

For computational efficiency and business relevance, the association analysis was performed using products appearing in at least **500 analytical baskets**.

This produced:

* **590 candidate products**
* **181,929 candidate baskets**
* **164,648 unique product pairs**

The candidate-basket universe is used as the denominator for association support calculations.

### Important Dataset Limitations

The demographic table contains information for only **801 of the 2,500 transaction households**. Therefore, demographic observations should be interpreted as applying to the available demographic records rather than being treated as representative of all customers.

Campaign and coupon analysis is observational. Campaign-associated sales are not interpreted as campaign-attributed revenue or ROI because the dataset does not establish a controlled causal relationship between campaign exposure and purchase outcomes.

The market basket analysis identifies **purchase associations, not causal relationships**. A high lift value indicates a strong observed association within the analytical universe; it does not prove that purchasing one product causes customers to purchase another.


## 5. Data Model

The Power BI solution uses a dimensional model to separate transaction facts from descriptive dimensions and analytical tables.

### Core Model

```text
                    ┌─────────────────┐
                    │   Dim_Product   │
                    │   Product_ID    │
                    └────────┬────────┘
                             │
                             │ 1 : *
                             ▼
┌─────────────────┐    ┌─────────────────────┐    ┌─────────────────┐
│  Dim_Customer   │───▶│  Fact_Transaction   │◀───│    Dim_Date     │
│ Household Key   │    │                     │    │      Date       │
└───────┬─────────┘    │ Basket / Product   │    └─────────────────┘
        │              │ Quantity / Sales   │
        │              └─────────────────────┘
        │
        ▼
┌─────────────────┐
│  Fact_Campaign  │
│ Household/Camp. │
└─────────────────┘

campaign_desc ─────────▶ Fact_Campaign
```

The primary transaction model contains:

* `Fact_Transaction`
* `Dim_Product`
* `Dim_Customer`
* `Dim_Date`
* `Fact_Campaign`
* `campaign_desc`

Additional analytical tables were created for customer, basket, and market-basket analysis.

### Analytical Tables

| Table                            | Purpose                                                         |
| -------------------------------- | --------------------------------------------------------------- |
| `Customer_Summary`               | Customer-level purchase frequency, products, units, and sales   |
| `Basket_Summary`                 | Basket-level products, units, and sales                         |
| `Basket_Products`                | Unique product-basket combinations                              |
| `Candidate_Basket_Products`      | Product-basket combinations restricted to candidate products    |
| `Candidate_Product_Basket_Count` | Number of candidate baskets containing each product             |
| `Association_Pairs`              | Unique product pairs with basket counts and association metrics |
| `Product_A_Details`              | Product attributes for Product A in an association              |
| `Product_B_Details`              | Product attributes for Product B in an association              |

The analytical tables were intentionally kept separate where appropriate to avoid introducing unnecessary relationships into the main transaction model.

## 6. Data Preparation

Data preparation was performed using **Power Query**.

### Transaction Preparation

The original transaction table contained approximately **2.60 million rows**.

The analytical transaction table was created as a reference query from the raw transaction data.

The following rule was applied:

```text
QUANTITY > 0 AND QUANTITY <= 100
```

This removed **37,602 rows**, resulting in approximately **2.56 million analytical transaction records**.

Zero-sales rows were retained when their quantity satisfied the validation rule.

### Product Preparation

The product master was loaded and used as the primary product dimension.

Product attributes used throughout the analysis include:

* Product ID
* Manufacturer
* Department
* Brand
* Commodity
* Sub-commodity
* Product size

### Customer Preparation

A distinct customer dimension was created from the transaction data using household keys.

This provided a common customer reference for connecting transaction behavior with campaign assignments.

### Basket Preparation

A basket-product reference table was created from the cleaned transaction table.

Duplicate combinations of:

```text
BASKET_ID + PRODUCT_ID
```

were removed so that each product was represented once within a basket for market basket analysis.

This distinction is important because market basket analysis measures **product co-occurrence**, rather than the number of units purchased.

### Candidate Product Selection

Products appearing in at least **500 analytical baskets** were selected as candidate products.

This produced:

* 590 candidate products
* 181,929 candidate baskets

The threshold was used as a practical business rule to reduce computational complexity and focus the association analysis on products with meaningful basket presence.

### Product Pair Generation

Candidate products were self-joined by `BASKET_ID` to identify products occurring together in the same basket.

Only unique unordered pairs were retained by applying:

```text
PRODUCT_A < PRODUCT_B
```

This prevents the same relationship from appearing twice as:

```text
A → B
B → A
```

The resulting association table contains **164,648 unique product pairs**.

### Association Metrics

Each product pair was evaluated using:

* **Support** — proportion of candidate baskets containing both products.
* **Confidence A → B** — proportion of baskets containing Product A that also contain Product B.
* **Confidence B → A** — proportion of baskets containing Product B that also contain Product A.
* **Lift** — strength of the observed association relative to what would be expected from the individual product frequencies.

These metrics were calculated using **DAX** inside Power BI.

### Analytical Workflow

```text
Raw Retail Data
      ↓
Power Query Cleaning
      ↓
Dimensional Data Model
      ↓
Customer & Basket Summaries
      ↓
Candidate Product Selection
      ↓
Product Pair Generation
      ↓
Support / Confidence / Lift
      ↓
Customer & Product Analysis
      ↓
Marketing Opportunity Rules
      ↓
Power BI Dashboards
```


## 7. Analytical Framework

The analysis follows a structured path from transaction behavior to marketing opportunity identification.

### 7.1 Customer & Basket Analysis

Customer-level and basket-level analytical tables were created to understand purchasing behavior.

Key measures include:

* Total customers
* Total baskets
* Total units
* Total sales
* Average baskets per customer
* Average basket value
* Average products per basket
* Customer purchase frequency
* Basket size distribution

Customers were grouped into five descriptive frequency segments:

* Low Frequency
* Occasional
* Regular
* Frequent
* Very Frequent

These segments are based on observed basket counts and are intended for descriptive customer profiling rather than predictive or causal classification.

### 7.2 Product Performance

Product performance was analyzed across multiple levels of the product hierarchy:

```text id="3yk8p1"
Department
    ↓
Commodity
    ↓
Sub-Commodity
    ↓
Product
```

The analysis evaluates:

* Sales
* Units
* Customer reach
* Basket presence
* Basket penetration

Product Basket Penetration measures the proportion of analytical baskets containing a particular product.

This provides a behavioral measure of product presence beyond total sales alone.

### 7.3 Market Basket Analysis

Market basket analysis was used to identify products that occur together within the same shopping basket.

The analysis uses products appearing in at least 500 analytical baskets and generates unique product pairs.

For each pair, the following metrics are calculated:

**Support**

Measures how frequently the product pair occurs within the candidate-basket universe.

**Confidence A → B**

Measures how frequently Product B appears when Product A appears.

**Confidence B → A**

Measures the reverse relationship.

**Lift**

Measures the strength of the observed co-occurrence relative to the expected co-occurrence based on individual product frequencies.

A lift value above 1 indicates that the pair occurs together more frequently than would be expected under an independence assumption.

### 7.4 Customer Segmentation

Customer purchase frequency was used to create descriptive customer segments.

The analysis compares segments using:

* Customer count
* Total sales
* Average customer sales
* Median customer sales
* Average basket value
* Average products per basket

The segmentation is based on historical transaction behavior and is not intended to predict future customer behavior.

### 7.5 Marketing & Promotion Analysis

Campaign and coupon data were analyzed to understand:

* Campaign reach
* Campaign assignment
* Campaign type
* Campaign timing
* Campaign duration
* Coupon redemption
* Coupon redemption by campaign

Campaign-assigned and non-campaign-assigned customer groups were compared descriptively.

Campaign-associated sales are treated as **observed household sales among customers assigned to campaigns**, rather than revenue directly caused by campaigns.

Because households can be associated with multiple campaigns, campaign-level sales figures should not be added together to calculate total campaign revenue.

### 7.6 Cross-Sell & Bundling Opportunities

Association rules were translated into business opportunity categories using explicit business rules.

A pair was considered a **strong eligible association** when:

```text id="p5l1tr"
Pair Baskets ≥ 50
AND
Lift ≥ 2
```

This produced **5,843 strong eligible product pairs**.

A pair was classified as a **Bundle Opportunity** when it additionally satisfied:

```text id="c5p9ue"
Confidence A → B ≥ 10%
AND
Confidence B → A ≥ 10%
```

This produced:

* **183 Bundle Opportunities**
* **5,660 Cross-Sell Opportunities**

The thresholds are business rules created for this project. They are not universal statistical standards.

The recommendation direction is determined by comparing the two confidence measures:

* Higher A → B confidence → **Promote B with A**
* Higher B → A confidence → **Promote A with B**
* Equal confidence → **Mutual Association**

These recommendations represent potential marketing applications of observed purchasing associations. They should be validated through controlled tests before being treated as causal evidence.

### 7.7 Marketing Interpretation

The project connects analytical findings to potential marketing applications such as:

* Cross-sell recommendations
* Bundle design
* Product placement
* Personalized product suggestions
* Promotion planning
* Basket-building strategies

The analysis intentionally distinguishes between **observed customer behavior** and **causal marketing effects**.

## 8. Key Metrics

| Metric                                      |        Result |
| ------------------------------------------- | ------------: |
| Total Sales                                 | $7,453,105.40 |
| Total Customers                             |         2,500 |
| Total Analytical Baskets                    |       253,605 |
| Total Units                                 |     3,356,057 |
| Unique Products Purchased                   |        91,991 |
| Average Basket Value                        |        $29.39 |
| Average Baskets per Customer                |        101.44 |
| Average Products per Basket                 |         10.09 |
| Average Units per Basket                    |         13.23 |
| Candidate Products for Association Analysis |           590 |
| Candidate Baskets                           |       181,929 |
| Unique Association Pairs                    |       164,648 |
| Strong Eligible Associations                |         5,843 |
| Strong Association Rate                     |         3.55% |
| Bundle Opportunities                        |           183 |
| Cross-Sell Opportunities                    |         5,660 |
| Campaign-Assigned Customers                 |         1,584 |
| Campaign Customer Coverage                  |        63.36% |
| Coupon Redemption Households                |           434 |
| Overall Coupon Redemption Rate              |        17.36% |

### Association Analysis Thresholds

The association opportunity metrics use the following project-specific rules:

| Rule               |              Threshold |
| ------------------ | ---------------------: |
| Candidate Product  |          ≥ 500 baskets |
| Strong Association |      ≥ 50 pair baskets |
| Strong Association |               Lift ≥ 2 |
| Bundle Opportunity | Confidence A → B ≥ 10% |
| Bundle Opportunity | Confidence B → A ≥ 10% |

These thresholds were established as **business screening rules** to balance association strength, basket volume, and practical marketing relevance.

They should not be interpreted as universal industry benchmarks.

## 9. Key Findings

### Customer Value Is Concentrated Among Higher-Frequency Customers

The customer frequency analysis shows substantial differences in sales contribution across customer segments.

The **Frequent** and **Very Frequent** segments contain **914 of the 2,500 customers (36.56%)** but account for approximately **65.49% of customer sales**.

Average customer sales also increase substantially across the frequency segments, from approximately **$224 for Low Frequency customers** to approximately **$7,460 for Very Frequent customers**.

This indicates that purchase frequency is strongly associated with observed customer value in the dataset.

### Larger Baskets Have Higher Observed Basket Values

Average basket value increases consistently with basket size.

Observed average basket values range from approximately:

* **$5.38** for one-product baskets
* **$8.38** for 2–3 products
* **$13.95** for 4–5 products
* **$22.47** for 6–10 products
* **$41.59** for 11–20 products
* **$98.98** for 21+ products

This pattern provides a basis for exploring basket-building and cross-sell strategies.

The relationship is descriptive and does not establish that adding products to a basket will necessarily cause basket value to increase.

### Product Performance Can Be Viewed Beyond Sales Alone

The product analysis evaluates sales together with units, customers, baskets, and basket penetration.

This provides multiple ways to assess product importance:

* Sales contribution
* Unit movement
* Customer reach
* Basket presence
* Basket penetration

This helps distinguish products with high sales from products that have broad participation across shopping baskets.

### Strong Product Associations Exist Within the Candidate Product Universe

The market basket analysis generated **164,648 unique product pairs** from **590 candidate products**.

Using the project screening rules of at least **50 pair baskets** and **lift ≥ 2**, **5,843 pairs** qualified as strong eligible associations.

This represents a **3.55% strong association rate** across the analyzed product-pair universe.

The results show that association strength alone is not sufficient for business prioritization because high lift can occur for relatively infrequent product combinations.

### Most Strong Associations Are Classified as Cross-Sell Opportunities

Applying the additional two-way confidence threshold for bundle opportunities resulted in:

* **183 Bundle Opportunities**
* **5,660 Cross-Sell Opportunities**

The distinction is based on the project's business rules rather than an industry-standard classification.

The opportunities provide a structured shortlist for evaluating potential product recommendations, cross-sell placements, and bundle concepts.

### Campaign and Coupon Activity Shows Broad but Uneven Reach

Campaign assignments cover **1,584 of the 2,500 customers**, equivalent to **63.36% customer coverage**.

Coupon redemption records involve **434 customers**, representing approximately **17.36% of all customers**.

Campaign and coupon results provide useful descriptive context for understanding marketing participation, but they should not be interpreted as evidence that campaign exposure or coupon redemption caused the observed purchase behavior.

### Customer and Product Analysis Can Be Connected to Marketing Decisions

The combined analysis creates several potential decision areas:

```text id="y7i4w5"
Customer Frequency
       ↓
Customer Value

Basket Size
       ↓
Basket-Building Opportunity

Product Affinity
       ↓
Cross-Sell / Bundle Opportunity

Campaign & Coupon Activity
       ↓
Marketing Context

Combined Evidence
       ↓
Targeted Marketing Hypotheses
```

The primary value of the project is therefore not simply identifying frequently purchased products, but creating a framework that connects **customer behavior, basket composition, product relationships, and marketing activity** into a single decision-support environment.

## 10. Business Recommendations

The analysis can be translated into the following marketing and merchandising opportunities.

### 1. Develop Cross-Sell Recommendations

Use strong product associations to recommend complementary products during the customer journey.

Potential applications include:

* Product-page recommendations
* Cart recommendations
* Checkout recommendations
* "Frequently bought together" placements
* Personalized product suggestions

The association rules should be prioritized using a combination of **pair basket volume, confidence, lift, product sales, and product penetration**, rather than lift alone.

### 2. Test Product Bundles

The 183 identified Bundle Opportunities provide a starting point for testing product combinations with relatively strong two-way association.

Potential applications include:

* Fixed-price bundles
* Multi-product offers
* Starter packs
* Complementary product bundles
* "Buy together" promotions

These combinations should be experimentally tested before being treated as proven bundle strategies.

### 3. Use Basket-Building Strategies

The observed relationship between basket size and basket value provides a basis for testing strategies designed to increase the number of products purchased per basket.

Potential tactics include:

* Complementary-product recommendations
* Add-on suggestions
* Threshold-based offers
* Multi-item promotions
* Bundle discounts

The objective should be to test whether these interventions increase basket value or product count without creating excessive discount costs.

### 4. Differentiate Customer Engagement by Purchase Frequency

The customer segmentation analysis shows substantial differences in observed sales across purchase-frequency groups.

Potential marketing applications include:

**Low Frequency**

* Re-engagement campaigns
* Product discovery recommendations
* Relevant category promotions

**Occasional / Regular**

* Cross-sell recommendations
* Basket-building offers
* Category-based personalization

**Frequent / Very Frequent**

* Personalized recommendations
* Loyalty-oriented engagement
* New-product discovery
* High-value bundle testing

These segments are based on historical purchase frequency and should be treated as descriptive customer groups.

### 5. Prioritize High-Volume Associations

A high lift value does not automatically mean an association is commercially important.

For practical prioritization, product pairs should be evaluated across several dimensions:

```text id="i7t4qf"
Association Strength
        +
Pair Basket Volume
        +
Confidence
        +
Product Sales
        +
Basket Penetration
        ↓
Commercial Opportunity
```

This helps prevent very rare product combinations from being prioritized simply because their lift is high.

### 6. Use Campaign and Coupon Data as Marketing Context

Campaign assignment and coupon redemption patterns can be used to understand customer participation in promotional activity.

However, the available data does not establish causal campaign effectiveness.

A stronger next step would be to use controlled experiments such as:

* A/B testing
* Holdout groups
* Treatment vs control comparisons
* Incrementality measurement

These approaches could test whether a specific marketing intervention actually changes purchasing behavior.

### 7. Build a Test-and-Learn Marketing Framework

The analysis can serve as a hypothesis-generation layer:

```text id="s4j2pz"
Observed Purchase Pattern
          ↓
Marketing Hypothesis
          ↓
Target Customer / Product Pair
          ↓
Marketing Intervention
          ↓
Controlled Experiment
          ↓
Incremental Impact
          ↓
Scale or Refine
```

This separates **descriptive analytics** from **causal marketing measurement** and provides a practical path from historical transaction analysis to evidence-based marketing optimization.

## 11. Power BI Dashboard

The final Power BI solution is organized into seven analytical pages:

| Dashboard Page                            | Purpose                                                                                               |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Executive Overview**                    | High-level view of sales, customers, baskets, product performance, and marketing opportunities        |
| **Customer & Basket Analysis**            | Customer frequency, basket size, basket value, and customer-level purchase detail                     |
| **Product Performance**                   | Department, commodity, sub-commodity, and product-level performance                                   |
| **Product Affinity & Association Rules**  | Product-pair relationships using support, confidence, and lift                                        |
| **Customer Segments & Purchase Patterns** | Customer frequency segments, sales contribution, basket behavior, and customer value                  |
| **Marketing & Promotion Analysis**        | Campaign reach, campaign structure, campaign timing, coupon redemption, and promotional participation |
| **Cross-Sell & Bundling Opportunities**   | Translation of product associations into potential cross-sell and bundle opportunities                |

### Dashboard Design Approach

The dashboard follows a decision-oriented structure:

```text id="6a5prx"
Executive Summary
      ↓
Customer & Basket Behavior
      ↓
Product Performance
      ↓
Product Affinity
      ↓
Customer Segmentation
      ↓
Marketing & Promotion Context
      ↓
Cross-Sell & Bundling Actions
```

The detailed pages allow users to move from high-level business performance into product relationships and potential marketing actions.

## 12. Tools & Skills

### Power BI

* Power BI Desktop
* Power Query
* DAX
* Data modeling
* Relationships
* Calculated columns
* Measures
* Interactive dashboards
* Conditional formatting
* Slicers and filtering
* Business-oriented data visualization

### Analytics

* Customer segmentation
* Transaction analysis
* Basket analysis
* Product performance analysis
* Market basket analysis
* Association-rule analysis
* Support, confidence, and lift
* Customer value analysis
* Product penetration analysis
* Marketing opportunity identification

### Marketing Analytics

* Cross-sell analysis
* Bundle opportunity identification
* Customer purchase-frequency segmentation
* Campaign reach analysis
* Coupon redemption analysis
* Promotion analysis
* Marketing hypothesis generation
* Test-and-learn framework

### Business Skills Demonstrated

* Translating business questions into analytical requirements
* Designing an analytical data model
* Creating reusable analytical measures
* Converting analytical results into business recommendations
* Distinguishing descriptive association from causal impact
* Communicating insights through executive dashboards

## 13. Data Limitations & Assumptions

This project is based on historical observational retail data. The following limitations should be considered when interpreting the results.

### Observational Analysis

The project identifies patterns and associations in historical transactions. It does not establish causal relationships.

For example, a high-lift product pair indicates strong observed co-occurrence, but it does not prove that purchasing one product causes the purchase of the other.

### Campaign Attribution

Campaign-assigned customer sales are treated as **observed sales from households associated with campaigns**.

They are not treated as campaign-attributed revenue or campaign ROI because the available data does not provide a controlled experimental framework for measuring incremental impact.

A household can also appear in multiple campaign assignments, so campaign-associated sales should not be summed across campaigns to calculate total company revenue.

### Demographic Coverage

The demographic dataset contains information for only **801 of the 2,500 transaction households**.

Therefore, demographic analysis, if performed, should be interpreted only for households with available demographic records and should not be treated as representative of the complete customer population.

### Coupon Mapping

Coupon redemption analysis is maintained at the coupon/campaign level.

The coupon reference data contains cases where the same coupon UPC maps to multiple products, so coupon redemption records were not force-mapped to a single product when such mapping was ambiguous.

### Association Thresholds

The market basket candidate threshold and opportunity thresholds are project-specific business rules.

They were selected to create a practical analytical screening process and should be recalibrated for a real business based on category economics, product margins, inventory, promotion costs, and experimentation results.

### Rare Associations

High lift values can occur for relatively rare product combinations.

For this reason, lift should be evaluated together with pair basket volume, confidence, support, product sales, and basket penetration.

## 14. Project Files

The project repository can contain the following artifacts:

```text id="yqv3hr"
Customer-Purchase-Intelligence/
│
├── README.md
│
├── PowerBI/
│   └── Customer_Purchase_Intelligence.pbix
│
├── Data/
│   └── Source_Dataset_Information.md
│
└── Documentation/
    └── Project_Notes.md
```

The original Dunnhumby dataset is not redistributed with the project repository. Users should obtain the dataset from its original source and follow the applicable dataset terms.

## 15. How to Use

### Step 1 — Obtain the Dataset

Obtain the Dunnhumby **The Complete Journey** dataset from its original distribution source.

### Step 2 — Load the Data

Open the Power BI project and ensure the required source tables are available.

### Step 3 — Refresh the Model

Refresh Power BI after configuring the source data location if required.

### Step 4 — Explore the Dashboard

Navigate through the seven dashboard pages:

1. Executive Overview
2. Customer & Basket Analysis
3. Product Performance
4. Product Affinity & Assoc


**Customers → Baskets → Products → Product Associations → Customer Patterns → Promotions → Marketing Opportunities**


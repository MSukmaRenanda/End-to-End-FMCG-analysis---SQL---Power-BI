# 📊 End-to-End FMCG B2B Sales & Inventory Analysis
### SQL (MySQL) + Power BI | 30,000 Transactions | 7 Countries | 4 Sales Channels
### End-to-end B2B FMCG analysis using MySQL &amp; Power BI — covering data cleaning, RFM store segmentation, ABC-XYZ inventory classification, and promotion effectiveness across 30K transactions in 7 countries.

---

## 🧩 Business Context & Problem Statement

This analysis is conducted from the perspective of an **FMCG distributor** supplying products to **13 retail store partners** (Hypermarket, Supermarket, Convenience, E-commerce) across 7 countries over 3 years (2021–2023).

The business faced three critical operational challenges:

| # | Challenge | Business Impact |
|---|---|---|
| 1 | Stock imbalance risking out-of-stock on high-value SKUs | Direct profit loss per day of stockout |
| 2 | Declining order frequency from high-value store partners | Risk of permanent revenue loss from key B2B accounts |
| 3 | Unclear whether current promotion strategy drives profit or erodes margin | Negative ROI on discount budget |

**Objective:** Deliver an end-to-end data analysis — from raw data cleaning to interactive dashboard — to support supply chain decisions, B2B store retention strategy, and promotion effectiveness evaluation.

---

## 🔍 Key Findings & Business Impact

### 🔴 Inventory Risk
- **3 critical SKUs** identified with out-of-stock risk across **9 distribution points**
- If these SKUs experience stockout for only 7 days → estimated profit loss of **±$26K** based on average daily profit
- **Class A products** (Brand A Cleaner leading at $37K profit) require immediate restock prioritization

### 👥 B2B Store Segmentation (RFM)
- **5 Loyal Customer stores** generating stable revenue of **$2.1M**
- **3 High Value Lost stores** carrying **$3.3M** in revenue — but order frequency is declining
- If High Value Lost stores return to Loyal Customer average frequency → estimated **recovery potential of $800K–$1.2M/year**

### 📉 Promotion Efficiency
- **Promo Uplift: -0.89%** — promotional transactions generate lower net sales compared to non-promo after discount is applied
- Current discount strategy erodes margin without driving sufficient volume to compensate
- Recommendation: cap discount at 15% for Class A products

### 🏪 Channel Performance
- **Hypermarket dominates** at **54.73%** of total sales with GPM of 38.56%
- Berlin ($2.4M) and Rome ($1.6M) are top revenue cities
- Allocating 60% of stock budget to Hypermarket-Berlin and Hypermarket-Rome can secure ~45% of total company profit

---

## 🛠️ Technical Approach

### Tools
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/PowerBI-F2C811?style=flat&logo=powerbi&logoColor=black)

### SQL Pipeline
All analysis was built using **layered SQL VIEWs** to ensure consistency across Power BI reports:

```
fmcg_raw
    └── vw_enriched_sales       ← Data cleaning + business metric calculation
            ├── vw_daily_store_fact     ← Daily aggregation per store & category
            ├── vw_summary_channel      ← Channel-level profitability
            ├── vw_rfm_analysis         ← B2B store segmentation
            └── vw_abc_xyz_analysis     ← Inventory classification
```

### SQL Techniques Used
| Technique | Purpose |
|---|---|
| `CTE (WITH)` | Multi-step layered data transformation |
| `SQL VIEW` | Reusable virtual tables as pipeline |
| `NTILE(4)` Window Function | RFM quartile scoring |
| `STDDEV / AVG` | Coefficient of Variation for XYZ classification |
| `SUM() OVER()` | Cumulative profit for Pareto/ABC analysis |
| `COALESCE` | NULL handling for financial metrics |
| `CASE WHEN` | Conditional segmentation logic |
| Multi-table `JOIN` | Enriching transactions with product, store & supplier master |

### Power BI Dashboard Structure
| Page | Content |
|---|---|
| Executive Summary | KPI cards, sales trend, channel distribution, promo analysis |
| Product & Store Analysis | ABC-XYZ matrix, store performance, geographic map |
| Channel & Customer Strategic | RFM scatter plot, loyalty matrix, promotion efficiency |
| Inventory Health & Supply Chain | Stock-out risk, restock priority list, sales velocity |

**Data Model:** Star schema — transaction fact table connected to product master, store master, and supplier master via unique IDs cleaned in SQL.

---

## 📁 Repository Structure

```
📦 fmcg-b2b-analysis/
├── 📂 sql/
│   └── fmcg_analysis.sql             ← All queries (cleaning, RFM, ABC-XYZ)
│                                        organized by section with headers
├── 📂 dashboard/
│   ├── fmcg_dashboard.pbix           ← Power BI Desktop file
│   └── 📂 screenshots/
│       ├── 01_executive_summary.png
│       ├── 02_product_store.png
│       ├── 03_channel_customer.png
│       └── 04_inventory_supply.png
├── 📂 dataset/
│   └── fmcg_raw.csv                  ← Raw dataset
└── README.md
```

> 💡 **Note for SQL file:** The single `.sql` file is structured into 5 sections using comment headers:
> `-- SECTION 1: Data Cleaning & Pre-processing`
> `-- SECTION 2: Daily Store Fact`
> `-- SECTION 3: Summary Channel`
> `-- SECTION 4: RFM Analysis`
> `-- SECTION 5: ABC-XYZ Inventory Classification`

---

## 📊 Dataset Overview

| Attribute | Detail |
|---|---|
| Period | 2021 – 2023 |
| Total Transactions | 30,000 rows |
| Total SKUs | 100+ |
| Store Partners | 13 stores (B2B) |
| Countries | 7 (Germany, Italy, France, Netherlands, etc.) |
| Sales Channels | Hypermarket, Supermarket, E-commerce, Convenience |
| Suppliers | 60 suppliers |

---

## 💡 Business Recommendations Summary

1. **Restock immediately** — prioritize Class A SKUs at 9 critical distribution points to prevent ±$26K/week profit loss
2. **Re-engage High Value Lost stores** — $3.3M at-risk revenue requires personalized B2B outreach based on historical order patterns
3. **Restructure promotion strategy** — Promo Uplift is negative (-0.89%); cap discounts at 15% for Class A, redirect budget to E-commerce
4. **Double down on Hypermarket-Berlin & Rome** — allocate 60% of stock budget here to secure ~45% of total profit with minimal distribution risk

---

## 📬 Contact

**Muhamad Sukma Renanda**
📧 muhamadsukmarenanda@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/muhamad-sukma-renanda-14022232b)
🌐 [Portfolio Website](https://muhamadsukmarenanda.wixsite.com/portofolio-da)

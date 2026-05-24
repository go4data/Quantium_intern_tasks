# 🛒 Quantium Virtual Internship — Customer Analytics
### Task 1: Chip Category Analysis | Strategic Report for Julia (Category Manager)

[![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)](https://python.org)
[![pandas](https://img.shields.io/badge/pandas-2.x-150458?logo=pandas)](https://pandas.pydata.org)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-visualization-orange)](https://matplotlib.org)
[![Seaborn](https://img.shields.io/badge/Seaborn-statistical%20plots-4C72B0)](https://seaborn.pydata.org)
[![scipy](https://img.shields.io/badge/scipy-hypothesis%20testing-8CAAE6)](https://scipy.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 📌 Project Overview

This project is **Task 1** of the [Quantium Data Analytics Virtual Internship](https://www.theforage.com/virtual-internships/prototype/NkaC7knWtjSbi6aYv/Data-Analytics) — a real-world simulation of data analyst work at one of Australia's leading data science companies.

The goal was to analyze **12 months of chip category transaction data** across **21 customer segments**, uncover the key drivers of chip sales, and deliver **strategic, data-driven recommendations** to Julia, the Category Manager, ahead of her category review.

---

## 🎯 Business Question

> **What customer segments drive chip sales — and what should Julia do about it?**

---

## 📊 Dataset Summary

| Dataset | Description |
|---------|-------------|
| `QVI_purchase_behaviour.csv` | Customer loyalty card data — life stage and spending tier per customer |
| `QVI_transaction_data.csv` | 12 months of chip transactions — store, product, quantity, sales value |

| Metric | Value |
|--------|-------|
| Dataset Period | July 2018 – June 2019 (12 months) |
| Total Transactions (after cleaning) | ~264,834 |
| Customer Segments | 7 Life Stages × 3 Spending Tiers = **21 segments** |
| Unique Brands (after cleaning) | 21 brands |
| Pack Sizes | 21 unique sizes (70g – 380g) |

---

## 🔧 What I Did — Step by Step

### 1. Data Understanding
Loaded and explored two datasets linked via `LYLTY_CARD_NBR` (loyalty card number):
- Inspected column types, null values, unique values
- Identified data quality issues before any analysis

### 2. Data Cleaning & Preparation

#### Date Conversion
The `DATE` column was stored as an **Excel serial integer** — converted to proper datetime:
```python
data["DATE"] = pd.to_datetime(data["DATE"], origin="1899-12-30", unit="D")
```
✅ Dataset confirmed to span exactly **July 2018 – June 2019**
✅ One missing date found: **Christmas Day (25 Dec 2018)** — stores confirmed closed

#### Outlier Removal
Inspection of `PROD_QTY` revealed **2 transactions** where a customer purchased **200 packets** in one visit (total: $650) — identified as wholesale or erroneous entries and removed:
```python
df_merged = df_merged[df_merged["PROD_QTY"] != 200]
```

#### Feature Engineering — Brand & Pack Size
Extracted brand and pack size from the raw `PROD_NAME` string:
```python
data["brand"] = data["PROD_NAME"].str.split().str[0]
data["packet_size"] = data["PROD_NAME"].str.extract(r"(\d+)g", flags=re.IGNORECASE)
```
Standardized **8 brand name abbreviations** to clean names:
```python
clean_brand = {
    'Smith': 'Smiths', 'Snbts': 'Sunbites', 'Infzns': 'Infuzions',
    'Dorito': 'Doritos', 'GrnWves': 'Grain', 'NCC': 'Natural',
    'RRD': 'Red', 'WW': 'Woolworths'
}
```

#### Product Validation
Identified and removed **5 non-chip products** (dips/salsas) that were incorrectly included:
- Old El Paso Salsa Dip Tomato Med 300g
- Old El Paso Salsa Dip Tomato Mild 300g
- Old El Paso Salsa Dip Chnky Tom Ht300g
- Woolworths Medium Salsa 300g
- Woolworths Mild Salsa 300g

---

### 3. Customer Segment Analysis

#### Top 3 Segments by Total Sales

| Rank | Segment | Total Sales | Primary Driver |
|------|---------|-------------|----------------|
| #1 | Older Families + Budget | $168,363 | High frequency + family size |
| #2 | Young Singles/Couples + Mainstream | $157,622 | Largest customer base |
| #3 | Retirees + Mainstream | $155,677 | Large & consistent base |

#### What Drives Spending? (3 metrics examined)

| Segment | Customers | Avg Spend/Txn | Frequency/Year | Primary Driver |
|---------|-----------|---------------|----------------|----------------|
| Older Families + Budget | 4,675 | $7.27 | 4.95x | **Frequency + Volume** |
| Young Singles + Mainstream | 8,088 | $7.56 | 2.58x | **Customer Volume** |
| Retirees + Mainstream | 6,479 | $7.25 | 3.31x | **Customer Volume** |

#### 💡 Counterintuitive Finding — Statistically Proven
**Premium customers pay LESS per chip unit than Mainstream customers** — proven with independent t-tests:

```python
from scipy import stats
t_stat, p_value = stats.ttest_ind(mainstream_prices, premium_prices)
```

| Comparison | t-statistic | p-value | Conclusion |
|------------|-------------|---------|------------|
| Mainstream vs Premium | 30.09 | 0.0000 | Mainstream pays significantly MORE |
| Mainstream vs Budget | 33.35 | 0.0000 | Mainstream pays significantly MORE |
| Budget vs Premium | -2.70 | 0.0070 | Budget pays slightly less |
| Mainstream vs Budget+Premium combined | 38.77 | 0.0000 | Significant — Mainstream pays most |

> All differences statistically significant at **p < 0.05**. This finding directly challenges the assumption that premium labeling drives premium price tolerance in the chip category.

---

### 4. Brand & Pack Size Affinity Analysis

Affinity = how much a segment over- or under-indexes on a brand vs the total population.
> Affinity > 1.0 means the segment buys this brand MORE than average.

#### Brand Affinity — Mainstream Young Singles/Couples

| Brand | Affinity | Interpretation |
|-------|----------|----------------|
| Tyrrells | **1.22** | Buys 22% MORE than average |
| Twisties | **1.21** | Buys 21% MORE than average |
| Tostitos | **1.19** | Buys 19% MORE than average |
| Kettle | **1.18** | Buys 18% MORE than average |
| Pringles | **1.17** | Buys 17% MORE than average |
| Woolworths | 0.51 | Buys 49% LESS than average |
| Sunbites | 0.54 | Buys 46% LESS than average |

#### Pack Size Affinity — Mainstream Young Singles/Couples

| Pack Size | Affinity | Interpretation |
|-----------|----------|----------------|
| 270g | 1.25 | Strongly preferred — party size |
| 380g | 1.24 | Strongly preferred — party size |
| 330g | 1.21 | Preferred — party size |
| 110g | 1.16 | Preferred — on-the-go |
| 160g–200g | 0.51–0.61 | Strongly avoided |

---

### 5. Monthly Sales Trend

| Month | Total Sales | Notes |
|-------|-------------|-------|
| December 2018 | $167,913 | 📈 Peak month — Christmas |
| March 2019 | $166,265 | Second highest |
| January 2019 | $150,665 | 📉 Lowest month of the year |

> Sales are remarkably stable year-round — within a ~$17,000 range. December peaks due to holiday shopping. January dips are common post-holiday.

---

## 📈 Visualizations

All charts generated and saved in the `/outputs` folder:

| Chart | File |
|-------|------|
| Total Sales by Customer Segment | `Total_sales_per_customer_segment.png` |
| Number of Customers per Segment | `Number_of_customers_in_each_segment.png` |
| Average Transactions per Customer | `average_transaction_per_customer_by_life_stage_and_premium_tier.png` |
| Average Price per Unit Sold | `average_price_per_unit_sold.png` |
| Top 5 Brand Preferences by Life Stage | `Top_5_brands_preferences_by_life_stage.png` |
| Top 5 Pack Size Preferences | `Top_5_pack_size_preferences_by_life_stage.png` |
| Number of Customers per Packet Size | `Number_of_customers_for_each_packet_size.png` |
| Monthly Sales Trend | `trend_of_monthly_sales.png` |

---

## 💼 Strategic Recommendations for Julia

### Priority 1 — Protect & Grow Top 3 Segments
| Segment | Recommended Action |
|---------|-------------------|
| Older Families + Budget | Ensure strong availability of value-sized packs. Promote Kettle brand deals. These customers return frequently — retention is key. |
| Young Singles/Couples + Mainstream | Stock Tyrrells, Twisties, Tostitos, Pringles prominently. Offer large party packs (270g–380g) and on-the-go sizes (110g–134g). |
| Retirees + Mainstream | Focus on Kettle and Smiths. Standard 175g packs preferred. Consistent shelf placement matters more than promotions. |

### Priority 2 — Shelving & Ranging Strategy
| Decision | Recommendation |
|----------|----------------|
| ✅ Increase shelf space | **Kettle** — universally preferred across ALL segments, commands higher price |
| ✅ Feature prominently | **Tyrrells, Twisties, Tostitos, Pringles** — key for highest-value segment |
| ❌ Reduce shelf space | **Woolworths generic** and **Sunbites** — consistently under-indexed |
| 📦 Pack size focus | Prioritize 175g (universal) + large party packs (270g–380g) + small snack packs (110g–134g) |

### Priority 3 — Pricing & Promotions
| Insight | Action |
|---------|--------|
| Mainstream pays most per unit | Target Mainstream Young/Midage Singles with mid-to-premium priced products |
| Budget families are high-frequency | Run bulk-buy promotions (buy 2 get 1 free) for Budget Older/Young Families |
| December is peak month | Increase stock of party-size packs and Kettle, Pringles, Doritos in November–December |
| February is lowest month | Run targeted promotion in February to lift seasonal dip |

---

## 🗂️ Project Structure

```
Quantium_Intern_Task1_for_chips_analysis/
│
├── qunatium.ipynb                          # Main analysis notebook
├── QVI_purchase_behaviour.csv             # Customer segment data
├── QVI_transaction_data(in).csv          # Transaction data
├── Quantium_Task1_Report.pdf             # Full strategic report
│
└── outputs/
    ├── Total_sales_per_customer_segment.png
    ├── Number_of_customers_in_each_segment.png
    ├── average_transaction_per_customer_by_life_stage_and_premium_tier.png
    ├── average_price_per_unit_sold.png
    ├── Top_5_brands_preferences_by_life_stage.png
    ├── Top_5_pack_size_preferences_by_life_stage.png
    ├── Number_of_customers_for_each_packet_size.png
    └── trend_of_monthly_sales.png
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Python 3.x** | Core analysis language |
| **pandas** | Data wrangling, merging, feature engineering, groupby analysis |
| **NumPy** | Numerical operations |
| **Matplotlib** | Chart creation and customization |
| **Seaborn** | Statistical visualizations |
| **scipy.stats** | Independent t-tests for hypothesis testing |
| **re (Regex)** | Feature extraction from raw product name strings |
| **Jupyter Notebook** | Interactive analysis environment |

---

## ⚙️ How To Run

```bash
# 1. Clone the repository
git clone https://github.com/go4data/quantium-customer-analytics.git
cd quantium-customer-analytics

# 2. Install dependencies
pip install pandas numpy matplotlib seaborn scipy jupyter

# 3. Add the data files to the project folder
# QVI_purchase_behaviour.csv
# QVI_transaction_data(in).csv

# 4. Open the notebook
jupyter notebook qunatium.ipynb
```

---

## 🔑 Key Takeaways

1. **Premium label ≠ Premium price tolerance** — Mainstream customers pay MORE per unit than Premium customers across every life stage. Statistically proven at p < 0.0001.

2. **Older Families drive sales through frequency** — not volume per visit or high prices. They keep coming back. Retention and availability matter most for this segment.

3. **Kettle is the undisputed category leader** — the only brand that consistently ranks as the top preference across ALL customer segments simultaneously.

4. **Young Singles/Couples (Mainstream) are the highest-value target** — they pay more per unit AND prefer premium brands like Tyrrells and Twisties. They should be the primary target for premium product placement and marketing investment.

5. **Pack size strategy matters** — 175g is the universal best-seller, but the highest-value segment (Young Mainstream) strongly prefers party sizes (270g–380g) and avoids medium sizes entirely.

---

## 👤 Author

**Mohamed Gouda**
Final Year — Computer & Systems Engineering, Mansoura University

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mohamed%20Gouda-blue?logo=linkedin)](https://www.linkedin.com/in/mohamed-goda-36abb7227/)
[![GitHub](https://img.shields.io/badge/GitHub-go4data-black?logo=github)](https://github.com/go4data)
📧 mohamedgoda123200@gmail.com

---

## 📄 License

This project is for educational and portfolio purposes as part of the Quantium Virtual Internship program on Forage.

---

*If you found this project useful or have questions about the methodology, feel free to connect on LinkedIn or open an issue on GitHub.*

# Customer Cohort & Retention Analysis

## 📌 Project Overview

This project performs a **Customer Cohort Analysis** using sales transaction data from `sales.csv`.

The analysis focuses on understanding customer purchasing behavior over time through:

* Monthly cohort analysis
* Weekly cohort analysis
* Customer retention
* Customer churn
* Revenue analysis
* ARPU (Average Revenue Per User)
* Cohort lifetime analysis
* Retention and churn heatmaps

The main objective is to identify **how customers behave after their first purchase**, how quickly they return, and which customer cohorts demonstrate stronger long-term retention.

---

## 📊 Dataset

The analysis uses a sales transaction dataset containing order, customer, product, payment, geographic, and demographic information.

### Dataset characteristics

* **Source:** `sales.csv`
* **Analysis period:** October 2020 – September 2021
* **Granularity:** Order-level transaction data
* **Customer identifier:** `cust_id`
* **Order identifier:** `order_id`

Important fields used in the analysis include:

| Column            | Description                   |
| ----------------- | ----------------------------- |
| `order_id`        | Unique order identifier       |
| `order_date`      | Date of the order             |
| `status`          | Original order status         |
| `cust_id`         | Customer identifier           |
| `qty_ordered`     | Quantity ordered              |
| `price`           | Product price                 |
| `discount_amount` | Discount applied to the order |
| `category`        | Product category              |
| `payment_method`  | Payment method                |
| `Customer Since`  | Customer registration date    |
| `Gender`          | Customer gender               |
| `age`             | Customer age                  |
| `City`            | Customer city                 |
| `State`           | Customer state                |
| `Region`          | Customer region               |

---

# 🧹 1. Data Overview & Cleaning

The notebook begins by loading the dataset and inspecting:

* Dataset dimensions
* Column names
* Data types
* Sample records
* Missing values
* Duplicate records
* Date fields
* Order statuses

### Date processing

The `order_date` field is converted into a proper datetime format:

```python
df['order_date'] = pd.to_datetime(
    df['order_date'],
    format="%d-%m-%Y"
)
```

The analysis covers orders from:

**October 2020 → September 2021**

---

## Order Status Classification

The original `status` field is transformed into three broader order outcomes:

### Successful orders

```text
complete
closed
paid
cod
received
```

### Cancelled orders

```text
canceled
holded
payment_review
pending
pending_paypal
processing
```

### Refunded orders

```text
order_refunded
refund
```

A new `order_outcome` variable is created to classify each order.

For cohort and sales analysis, the notebook uses only successful transactions:

```python
df_sales = df[
    df['order_outcome'] == 'successful'
].copy()
```

This ensures that cancelled and refunded transactions are not treated as successful customer purchases.

---

# 💰 2. Revenue Calculation

Revenue is calculated for each transaction using:

```python
revenue = qty_ordered × price − discount_amount
```

Python implementation:

```python
df['revenue'] = (
    df['qty_ordered'] * df['price']
    - df['discount_amount']
)
```

The resulting `revenue` field is used throughout the cohort and ARPU analysis.

---

# 📅 3. Monthly Cohort Analysis

## First Order Date

For each customer, the first successful purchase date is identified:

```python
first_order = df_sales.groupby(
    'cust_id'
)['order_date'].min().reset_index()
```

This date is then used to determine the customer's cohort.

---

## Cohort Month

The customer's first purchase month becomes their cohort:

```python
df_sales['cohort_month'] = (
    df_sales['first_order_date']
    .dt.to_period('M')
)
```

The month in which each individual order occurred is also calculated:

```python
df_sales['order_month'] = (
    df_sales['order_date']
    .dt.to_period('M')
)
```

This allows customer activity to be compared across different cohorts.

---

# 📈 4. Monthly Cohort Summary

For every cohort, the analysis calculates:

* Unique orders
* Unique customers
* Total revenue

```python
cohort_monthly = df_sales.groupby(
    'cohort_month'
).agg(
    unique_orders=('order_id', 'nunique'),
    unique_customers=('cust_id', 'nunique'),
    total_revenue=('revenue', 'sum')
).reset_index()
```

The results are visualized using charts for:

* Unique orders by cohort
* Unique customers by cohort
* Revenue by cohort

---

# 👥 5. Customer Activity Matrix

A cohort activity pivot table is created where:

* **Rows:** Customer's first purchase month
* **Columns:** Order month
* **Values:** Unique active customers

```python
cohort_pivot = df_sales.pivot_table(
    index='cohort_month',
    columns='order_month',
    values='cust_id',
    aggfunc='nunique'
)
```

This matrix makes it possible to observe how many customers from each cohort continued purchasing in subsequent months.

---

# 💵 6. Monthly Cohort Metrics & ARPU

For every combination of cohort month and order month, the project calculates:

* Unique customers
* Total revenue

```python
monthly_cohort_metric = df_sales.groupby(
    ['cohort_month', 'order_month']
).agg(
    unique_customers=('cust_id', 'nunique'),
    total_revenue=('revenue', 'sum')
).reset_index()
```

## ARPU

ARPU is calculated as:

```python
ARPU = Total Revenue / Unique Customers
```

Implementation:

```python
monthly_cohort_metric['ARPU'] = (
    monthly_cohort_metric['total_revenue']
    / monthly_cohort_metric['unique_customers']
)
```

The results are presented as an ARPU pivot table.

---

# ⏳ 7. Monthly Cohort Lifetime

`cohort_lifetime` measures how many months have passed since the customer's first purchase.

The first purchase month is defined as:

```text
Lifetime 0
```

The next month is:

```text
Lifetime 1
```

and so on.

```python
df_sales['cohort_lifetime'] = (
    df_sales['order_month'].astype(int)
    - df_sales['cohort_month'].astype(int)
)
```

This allows revenue and customer behavior to be analyzed relative to the customer's lifecycle rather than calendar dates.

---

# 🔥 8. ARPU Heatmap

The project creates a cohort-lifetime ARPU matrix using:

* Rows → Monthly cohorts
* Columns → Cohort lifetime
* Values → ARPU

A heatmap is then used to visualize changes in customer value over time.

This helps identify whether customers who remain active generate increasing or decreasing revenue as their relationship with the business develops.

---

# 📆 9. Weekly Retention Analysis

The project then moves from monthly cohort analysis to **weekly retention analysis**.

## Weekly Order Period

Each order is assigned to the beginning of its respective week:

```python
df_sales['order_week'] = (
    df_sales['order_date']
    .dt.to_period('W')
    .dt.start_time
)
```

Each customer is assigned to the week of their first successful purchase.

This becomes their `cohort_week`.

---

# ⏱️ 10. Weekly Cohort Lifetime

Weekly cohort lifetime measures the number of weeks between:

* The customer's first purchase week
* The week of subsequent purchases

```python
df_sales['weekly_cohort_lifetime'] = (
    df_sales['order_week']
    - df_sales['cohort_week']
).dt.days // 7
```

Therefore:

```text
Lifetime 0 → First purchase week
Lifetime 1 → One week after first purchase
Lifetime 2 → Two weeks after first purchase
...
```

---

# 👤 11. Weekly Active Customers

The number of unique active customers is calculated for each cohort and lifetime:

```python
weekly_active_customers = df_sales.groupby(
    ['cohort_week', 'weekly_cohort_lifetime']
).agg(
    active_customers=('cust_id', 'nunique')
).reset_index()
```

This forms the basis for the retention analysis.

---

# 🎯 12. Initial Cohort Size

The number of customers at `Lifetime 0` represents the initial cohort size.

```python
initial_size = weekly_active_customers[
    weekly_active_customers[
        'weekly_cohort_lifetime'
    ] == 0
]
```

This initial cohort size is merged back into the weekly activity table.

---

# 📊 13. Retention Rate

Retention rate is calculated as:

```text
Retention Rate =
Active Customers / Initial Cohort Size
```

Python:

```python
weekly_active_customers['retention_rate'] = (
    weekly_active_customers['active_customers']
    / weekly_active_customers['initial_cohort_size']
)
```

The resulting retention data is transformed into a matrix and visualized using a heatmap.

---

# 🔥 14. Retention Heatmap

The retention matrix contains:

* **Rows:** Cohort week
* **Columns:** Weekly cohort lifetime
* **Values:** Retention percentage

The heatmap makes it possible to quickly identify:

* Early customer drop-off
* Stronger or weaker cohorts
* Long-term retained customer groups
* Changes in retention quality over time

---

# 📉 15. Cumulative Churn

Cumulative churn is derived directly from retention:

```text
Cumulative Churn = 1 − Retention
```

Implementation:

```python
weekly_active_customers[
    'cumulative_churn_rate'
] = (
    1
    - weekly_active_customers['retention_rate']
)
```

A separate churn matrix and heatmap are created to visualize customer loss across cohort lifetimes.

---

# 🔎 Key Findings

### 1. Strong seasonal variation in cohort size

Cohort sizes are heavily affected by seasonality.

The **December 2020 cohort** is particularly large, with approximately:

* **64K orders**
* **19.8K customers**

This cohort is approximately **3–4 times larger** than many other cohorts.

---

### 2. Major customer loss occurs after the first purchase

The largest retention drop occurs between:

```text
Lifetime 0 → Lifetime 1
```

Across cohorts, approximately **80–95% of customers are lost during the first repeat-purchase period**.

This indicates a strong **one-time buyer pattern**, suggesting that many customers do not return after their initial purchase.

---

### 3. Retention quality decreases over time

The notebook's retention analysis indicates that earlier cohorts demonstrated stronger early retention than later cohorts.

Early cohorts showed first-week retention reaching approximately:

**15–40%**

while later 2021 cohorts were generally around:

**5–10%**

This may indicate declining new-customer quality or reduced effectiveness of loyalty/retention initiatives.

---

### 4. A small retained customer core exists

After the first several weeks, retention stabilizes around:

**1–3%**

for some cohorts.

This suggests that a small core group of customers continues purchasing over a longer period.

Although this group represents only a small proportion of the total customer base, it can provide a relatively stable source of recurring revenue.

---

### 5. Later-lifetime ARPU is highly volatile

ARPU becomes less reliable at later cohort lifetimes because the number of remaining customers becomes very small.

As a result, a small number of high-value orders can produce significant ARPU spikes.

Some later-lifetime cells exceed **30K**, demonstrating the effect of outlier transactions.

Therefore, **Lifetime 0–1 ARPU provides a more reliable basis for cohort comparison** than later-lifetime values.

---

# 💼 Business Applications

The analysis can support several business decisions:

### Customer Retention

Identify the critical period immediately after the first purchase and develop campaigns aimed at increasing the probability of a second purchase.

### Loyalty Programs

Target customers during the first week after acquisition with:

* Personalized offers
* Loyalty incentives
* Product recommendations
* Follow-up communication

### Cohort Comparison

Compare newer and older customer cohorts to identify changes in customer quality and retention.

### Revenue Optimization

Use cohort ARPU to understand how customer value develops over the customer lifecycle.

### Churn Reduction

Identify the point where most customers become inactive and focus retention campaigns around that period.

---

# 🛠️ Technologies Used

* **Python**
* **Pandas**
* **NumPy**
* **Matplotlib**
* **Seaborn**
* **Google Colab**
* **Jupyter Notebook**

---

# 📁 Project Structure

```text
customer-cohort-retention-analysis/
│
├── cohort_analiz.ipynb
├── sales.csv
└── README.md
```

---

# ▶️ How to Run

### 1. Clone the repository

```bash
git clone https://github.com/Aslan934/customer-cohort-retention-analysis.git
```

### 2. Open the notebook

Open:

```text
cohort_analiz.ipynb
```

in:

* Google Colab
* Jupyter Notebook
* JupyterLab

### 3. Provide the dataset

Place `sales.csv` in the project directory or update the dataset path in the notebook.

### 4. Run the notebook

Execute the cells sequentially to reproduce:

* Data cleaning
* Revenue calculations
* Monthly cohorts
* ARPU analysis
* Weekly retention
* Churn analysis
* Visualizations

---

# 📚 Analytical Workflow

```text
Raw Sales Data
       ↓
Data Cleaning
       ↓
Order Status Classification
       ↓
Successful Orders
       ↓
Revenue Calculation
       ↓
First Customer Purchase
       ↓
Monthly Cohorts
       ↓
Monthly Activity & ARPU
       ↓
Cohort Lifetime
       ↓
Weekly Cohorts
       ↓
Active Customers
       ↓
Retention Rate
       ↓
Churn Rate
       ↓
Business Insights
```

---

# 🎯 Skills Demonstrated

This project demonstrates practical skills in:

* Data cleaning
* Pandas data manipulation
* Datetime processing
* Customer segmentation
* Cohort analysis
* Retention analysis
* Churn analysis
* ARPU calculation
* GroupBy aggregation
* Pivot tables
* Data visualization
* Heatmap analysis
* Business interpretation
* Customer lifecycle analysis

---

# 🚀 Future Improvements

Potential extensions include:

* Automating the cohort reporting process
* Adding customer Lifetime Value (LTV)
* Segmenting cohorts by acquisition channel
* Comparing customer retention by product category
* Analyzing retention by payment method
* Adding customer demographic segmentation
* Building an interactive Power BI dashboard
* Adding statistical testing between cohorts
* Creating automated retention alerts
* Developing predictive churn models

---

# 📌 Limitations

The analysis is based on historical transaction data and therefore describes observed customer behavior rather than proving causal relationships.

In addition:

* Large seasonal fluctuations can affect cohort size.
* Later-lifetime ARPU is sensitive to small sample sizes.
* Outlier transactions can significantly influence revenue-based metrics.
* Retention percentages should be interpreted together with cohort size.
* The dataset covers a limited historical period, so long-term customer behavior beyond the available observation window cannot be evaluated.

---

# 🏁 Conclusion

This project demonstrates how cohort analysis can be used to understand **customer acquisition, repeat purchasing, retention, churn, and revenue generation over time**.

The analysis shows that the most significant challenge is the large customer drop-off after the first purchase. At the same time, a small group of customers remains active for longer periods and represents an important potential source of recurring revenue.

The results highlight the importance of focusing on **second-purchase conversion, early customer engagement, loyalty initiatives, and retention optimization**.

---

## 👤 Author

**Aslan Rustamov**

Data Analyst

[GitHub](https://github.com/Aslan934)

🛍️ Customer Shopping Behavior Analysis

📌 Overview

This project analyzes customer shopping behavior using transactional data from 3,900 purchases across multiple product categories. The goal is to uncover insights into spending patterns, customer segments, product preferences, and subscription behavior to support strategic business decisions.

End-to-end workflow: Data Loading → Cleaning & Feature Engineering (Python) → SQL Analysis (MySQL) → Power BI Dashboard → Business Recommendations


📁 Dataset

DetailDescriptionRows3,900Columns18Missing Data37 values in Review Rating (imputed)DemographicsAge, Gender, Location, Subscription StatusPurchase DetailsItem Purchased, Category, Purchase Amount, Season, Size, ColorShopping BehaviorDiscount Applied, Previous Purchases, Frequency of Purchases, Review Rating, Shipping Type


🛠️ Tools & Technologies

CategoryTools UsedProgrammingPython (Pandas, SQLAlchemy)DatabaseMySQLVisualizationPower BIEnvironmentJupyter Notebook


🔍 Project Workflow

1️⃣ Data Loading & Exploration (EDA)

The dataset (3,900 rows × 18 columns) was loaded with Pandas and explored before any cleaning:


Previewed structure and data types using df.head(), df.info(), and df.describe(include='all')
Checked for duplicate rows — none found
Checked for missing values — 37 missing entries in Review Rating, the only column with gaps


2️⃣ Data Cleaning & Feature Engineering

Missing Value Handling — Imputed missing Review Rating values using the median rating per product category to minimize outlier impact:

pythondf["review_rating"] = df.groupby("category")["review_rating"] \
    .transform(lambda x: x.fillna(x.median()))

Column Standardization — Converted all column names to snake_case for consistency:

pythondf.columns = df.columns.str.lower().str.replace(" ", "_")
df = df.rename(columns={"purchase_amount_(usd)": "purchase_amount"})

Feature Engineering — Created an age_group column (4 quantile-based segments) and converted purchase frequency text into numeric purchase_frequency_days:

pythonlabels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)

frequency_mapping = {
    'Weekly': 7, 'Fortnightly': 14, 'Bi-Weekly': 14,
    'Monthly': 30, 'Quarterly': 90,
    'Every 3 Months': 90, 'Annually': 365
}
df['purchase_frequency_days'] = df['frequency_of_purchases'].map(frequency_mapping)

Data Consistency Check — Verified discount_applied and promo_code_used were identical and dropped the redundant column:

python(df['discount_applied'] == df['promo_code_used']).all()  # True
df = df.drop('promo_code_used', axis=1)

The cleaned dataset was exported to cleaned_customer_data.csv and loaded into MySQL using SQLAlchemy:

pythonfrom sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:****@localhost:3306/customer_behavior_db")
df.to_sql(name="customers", con=engine, if_exists="replace", index=False)


3️⃣ SQL Analysis (MySQL)

Ten business questions were answered directly in MySQL. A few highlights:

Revenue by Gender

sqlSELECT gender, SUM(purchase_amount) AS total_revenue
FROM customers
GROUP BY gender;


💡 Male customers generated $157,890 in revenue vs. $75,191 from female customers.



Customer Segmentation (New / Returning / Loyal)

sqlWITH customer_type AS (
    SELECT customer_id, previous_purchases,
        CASE WHEN previous_purchases = 1 THEN 'New'
             WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
             ELSE 'Loyal' END AS customer_segment
    FROM customers
)
SELECT customer_segment, COUNT(*) AS number_of_customers
FROM customer_type
GROUP BY customer_segment;


💡 Loyal: 3,116 | Returning: 701 | New: 83 — the customer base is overwhelmingly loyal.



Discount-Dependent Products

sqlSELECT item_purchased,
    ROUND(100.0 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_rate
FROM customers
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;


💡 Hats rely on discounts most (50% of purchases), followed by Sneakers (49.66%) and Coats (49.07%).



📄 The full set of 10 queries (revenue by gender, high-spending discount users, top-rated products, shipping type comparison, subscriber spend, discount-dependent products, customer segmentation, top products per category, repeat buyers vs. subscriptions, and revenue by age group) is available in customer_behavior_sql_queries.sql.


4️⃣ Power BI Dashboard

An interactive dashboard was built to present the findings visually, with filters for subscription status, gender, category, and shipping type.



Key metrics at a glance:


3.9K total customers | $59.76 average purchase amount | 3.75 average review rating
73% of customers are non-subscribers vs. 27% subscribers
Clothing leads both revenue and sales volume across all categories
Young Adults and Middle-aged segments drive the highest revenue and sales


📊 A summarized version of these findings is also available as a presentation: Customer-Shopping-Behavior-Analysis.pptx


✅ Key Insights


Male customers drive ~68% of total revenue ($157,890 vs $75,191 from female customers)
Loyal customers (3,116 of 3,900) form the bulk of the base, but only 958 of repeat buyers (>5 purchases) are subscribers — a large untapped conversion opportunity
Subscribers and non-subscribers spend almost identically per order ($59.49 vs $59.87 average), meaning subscription value isn't currently tied to higher basket size
Express shipping customers spend slightly more on average ($60.48 vs $58.46 for Standard)
Hats, Sneakers, and Coats are the most discount-dependent products, with ~50% of their sales relying on discounts — a potential margin risk
Gloves, Sandals, and Boots have the highest average review ratings (3.78–3.86), making them strong candidates for promotional campaigns
Clothing dominates revenue and order volume across all categories, followed by Accessories, Footwear, and Outerwear



📈 Business Recommendations


Boost Subscriptions — Promote exclusive benefits for subscribers to grow the current 27% subscription rate.
Customer Loyalty Programs — Reward repeat buyers to convert "Returning" customers into "Loyal" status.
Review Discount Policy — Re-evaluate heavy discounting on Hats, Sneakers, and Coats to protect margins.
Product Positioning — Feature top-rated products (Gloves, Sandals, Boots) in marketing campaigns.
Targeted Marketing — Prioritize high-revenue age groups (Young Adult, Middle-aged) and express-shipping users.



📂 Project Structure

├── Business Problem Document.pdf
├── Customer Shopping Behavior Analysis.pdf
├── Customer-Shopping-Behavior-Analysis.pptx
├── Customer_Shopping_Behavior_Analysis.ipynb
├── customer_behavior_dashboard.pbix
├── customer_behavior_sql_queries.sql
├── customer_shopping_behavior.csv
└── README.md

FileDescriptionBusiness Problem Document.pdfDefines the business problem, objectives, and scope of the analysisCustomer Shopping Behavior Analysis.pdfFull written report covering EDA, cleaning, SQL analysis, dashboard, and recommendationsCustomer-Shopping-Behavior-Analysis.pptxStakeholder presentation summarizing key findingsCustomer_Shopping_Behavior_Analysis.ipynbPython notebook — data loading, EDA, cleaning, and feature engineeringcustomer_behavior_dashboard.pbixPower BI dashboard filecustomer_behavior_sql_queries.sqlSQL queries used for business analysis in MySQLcustomer_shopping_behavior.csvRaw dataset (3,900 rows × 18 columns)


📬 Contact

Hareendran
📧 hareendran2003@gmail.com | 💻 GitHub | 💼 LinkedIn

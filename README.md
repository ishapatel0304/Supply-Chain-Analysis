
# **Supply Chain Analysis for E-commerce Retail Platform**
## **IMPORT LIBRARIES**
pandas (pd) → used to handle tabular data (like Excel)
numpy (np) → used for random number generation & calculations
matplotlib.pyplot (plt) → used for plotting graphs
seaborn (sns) → advanced visualization library

👉 sns.set_style("whitegrid")

Adds background grid to graphs → makes charts cleaner and readable
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_style("whitegrid")
## **LOAD DATA**
pd.read_csv(...)
Loads CSV file into DataFrame (df)
df.head()
Displays first 5 rows
👉 Used to visually check if data loaded correctly
df.info()

Shows:

Total rows → 27,555
Columns → 10
Data types (int, float, object)
Missing values

👉 Important observation:

rating has missing values
product and brand also have some nulls
df = pd.read_csv("BigBasket Products.csv")

df.head()
df.info()
## **DATA CLEANING**
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")

df['rating'] = df['rating'].fillna(df['rating'].mean())

df = df.drop_duplicates()
## **FEATURE ENGINEERING**
* Fixes randomness → same output every time
*   Creates unique ID for each row (order simulation)
*   Random values between 1–9
* Simulates number of items per order
* Creates realistic order dates across a year
* Simulates geographic demand
* Business metric
* Simulates stock available
* Converts numeric stock → business category






np.random.seed(42)

df['order_id'] = range(1000, 1000 + len(df))
df['quantity'] = np.random.randint(1, 10, len(df))

df['order_date'] = pd.to_datetime('2023-01-01') + pd.to_timedelta(np.random.randint(0, 365, len(df)), unit='d')

df['city'] = np.random.choice(['Surat','Mumbai','Delhi','Bangalore'], len(df))

df['total_sales'] = df['quantity'] * df['sale_price']

df['delivery_days'] = np.random.randint(1, 8, len(df))

df['inventory_level'] = np.random.randint(0, 100, len(df))

def stock_status(x):
    if x == 0:
        return "Out of Stock"
    elif x < 20:
        return "Low Stock"
    else:
        return "In Stock"

df['stock_status'] = df['inventory_level'].apply(stock_status)
## **CATEGORY SALES VISUAL**
* Groups data by category
* Calculates total revenue per category
* Sorts values
* Creates horizontal bar chart
* Displays chart
category_sales = df.groupby('category')['total_sales'].sum().sort_values()

plt.figure()
category_sales.plot(kind='barh')
plt.title("Category-wise Sales")
plt.xlabel("Total Sales")
plt.ylabel("Category")
plt.show()
## **CITY DEMAND**
* Calculates total quantity sold per city
* Bar chart
city_demand = df.groupby('city')['quantity'].sum()

plt.figure()
city_demand.plot(kind='bar')
plt.title("City-wise Demand")
plt.ylabel("Total Quantity Sold")
plt.show()
city_demand = df.groupby('city')['quantity'].sum()

plt.figure()
city_demand.plot(kind='bar')
plt.title("City-wise Demand")
plt.ylabel("Total Quantity Sold")
plt.show()
## **INVENTORY PIE CHART**
Counts:

* In Stock
* Low Stock
* Out of Stock
stock_counts = df['stock_status'].value_counts()

plt.figure()
stock_counts.plot(kind='pie', autopct='%1.1f%%')
plt.title("Inventory Status Distribution")
plt.ylabel("")
plt.show()
## **TOP PRODUCTS**
* Top 10 products by revenue



top_products = df.groupby('product')['total_sales'].sum().sort_values(ascending=False).head(10)

plt.figure()
top_products.plot(kind='bar')
plt.title("Top 10 Products by Sales")
plt.xticks(rotation=45)
plt.show()
## **KPI SECTION**
* Total revenue
* Avg delivery time
* Total orders
* Stockout rate
print("Total Revenue:", df['total_sales'].sum())
print("Average Delivery Time:", df['delivery_days'].mean())
print("Total Orders:", df['order_id'].nunique())
print("Stockout Rate:", len(df[df['stock_status']=='Out of Stock'])/len(df))
## **SQL SETUP**
* Creates temporary SQL database
* Converts DataFrame → SQL table
import sqlite3

conn = sqlite3.connect(':memory:')
df.to_sql('sales_data', conn, index=False, if_exists='replace')
## **QUERY 1**
* Top 5 products by revenue
query1 = """
SELECT product, SUM(total_sales) AS revenue
FROM sales_data
GROUP BY product
ORDER BY revenue DESC
LIMIT 5;
"""

pd.read_sql(query1, conn)

## **QUERY 2**
* Category-wise revenue
query2 = """
SELECT category, SUM(total_sales) AS total_revenue
FROM sales_data
GROUP BY category
ORDER BY total_revenue DESC;
"""

pd.read_sql(query2, conn)
## **QUERY 3**
* Category-wise revenue
query3 = """
SELECT city, COUNT(order_id) AS total_orders
FROM sales_data
GROUP BY city;
"""

pd.read_sql(query3, conn)
## **EXPORT**
* Saves final dataset
df.to_csv("final_supply_chain_data.csv", index=False)

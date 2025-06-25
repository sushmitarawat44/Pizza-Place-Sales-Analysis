Created on Wed Apr 23 01:04:44 2025
@author: sushmita

"""
PIZZA SALES ANALYSIS PROJECT
Author: SUSHMITA RAWAT
Date: 6 may 2025

--- 1. Import Required Libraries ---
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from collections import Counter

sns.set(style="whitegrid")
plt.rcParams['figure.figsize'] = (12, 6)

--- 2. Load Datasets ---
orders = pd.read_csv('orders.csv', encoding='windows-1252')
order_details = pd.read_csv('order_details.csv', encoding='windows-1252')
pizzas = pd.read_csv('pizzas.csv', encoding='windows-1252')
pizza_types = pd.read_csv('pizza_types.csv', encoding='windows-1252')

 --- 3. Data Preprocessing ---
Fix datetime columns
orders['date'] = pd.to_datetime(orders['date'])
orders['time'] = pd.to_datetime(orders['time'], format='%H:%M:%S').dt.time

- Standardize pizza IDs for merging
pizzas['pizza_id'] = pizzas['pizza_id'].str.replace('-', '_')
pizza_types['pizza_type_id'] = pizza_types['pizza_type_id'].str.replace('-', '_')
pizzas = pd.merge(pizzas, pizza_types, on='pizza_type_id')

- Merge all datasets
data = pd.merge(order_details, orders, on='order_id')
data = pd.merge(data, pizzas, on='pizza_id')

--- 4. Analysis and Visualization ---

Q1: How many customers do we have each day?
daily_customers = orders.groupby('date')['order_id'].nunique()
daily_customers.plot(title='Number of Customers per Day')
plt.ylabel('Customers')
plt.xlabel('Date')
plt.tight_layout()
plt.show()


Q2: Are there any peak hours?
orders['hour'] = pd.to_datetime(orders['time'], format='%H:%M:%S').dt.hour
orders['hour'].value_counts().sort_index().plot(kind='bar', title='Peak Hours')
plt.xlabel('Hour of Day')
plt.ylabel('Number of Orders')
plt.tight_layout()
plt.show()

Q3: How many pizzas are typically in an order?
pizzas_per_order = order_details.groupby('order_id')['quantity'].sum()
pizzas_per_order.plot(kind='hist', bins=20, title='Number of Pizzas per Order')
plt.xlabel('Pizzas per Order')
plt.tight_layout()
plt.show()
print(pizzas_per_order.describe())

Q4: Do we have any bestsellers?
bestsellers = data.groupby('name')['quantity'].sum().sort_values(ascending=False)
bestsellers.head(10).plot(kind='bar', title='Top 10 Bestselling Pizzas')
plt.ylabel('Total Sold')
plt.tight_layout()
plt.show()

Q5: How much money did we make this year?
data['total_price'] = data['price'] * data['quantity']
annual_revenue = data.groupby(data['date'].dt.year)['total_price'].sum()
print(f"Total Revenue This Year: ${annual_revenue.values[0]:,.2f}")


Q6: Can we identify any seasonality in the sales?
monthly_revenue = data.groupby(data['date'].dt.to_period('M'))['total_price'].sum()
monthly_revenue.index = monthly_revenue.index.to_timestamp()  # Convert PeriodIndex to Timestamp
monthly_revenue.plot(title='Monthly Revenue')
plt.ylabel('Revenue')
plt.xlabel('Month')
plt.tight_layout()
plt.show()

worst_selling = data.groupby('name')['quantity'].sum().sort_values()
print("Worst-Selling Pizzas:")
print(worst_selling.head(10))  # Show 10 lowest-selling pizzas
worst_selling.head(10).plot(kind='barh', title='Worst-Selling Pizzas')
plt.xlabel('Total Quantity Sold')
plt.ylabel('Pizza Name')
plt.tight_layout()
plt.show()

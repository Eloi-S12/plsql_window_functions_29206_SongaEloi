# Problem Definition

## Business Context

The company operates in several regions and sells its products. The company would like to know how the sales are performing and the behavior of the customers when it comes to purchasing the product. This task is carried out by the Sales Analytics Department.

## Data Challenge

The company lacks clear visibility about the top-performing products, inactive customers, sales trends over a period of time, and customer segmentation based on revenue. The reports that are being generated are not giving any analytical insights about the sales transactions.

## Expected Outcome

The company aims to achieve analytical insights that would help the company in identifying the top products, sales trends, identifying inactive customers, and customer segmentation.

## Database Schema Design and related tables with primary and foreign keys.

### Customers table

```
CREATE TABLE customers (
 Customer_id NUMBER PRIMARY KEY,
 Customer_name VARCHAR(250),
 Region VARCHAR(250)
);
```
for example: [![Customers Table](/Tables/Customer%20Table.png)](/Tables/Customer%20Table.png)

### Products table

```
CREATE TABLE Products (
 Product_id NUMBER PRIMARY KEY,
 Product_name VARCHAR(250),
 Category VARCHAR(250),
 Price NUMBER(15)
);
```
for example [![Products Table](/Tables/Product%20table.png)](/Tables/Product%20table.png)

### Orders table

```
CREATE TABLE Orders (
 Order_id NUMBER PRIMARY KEY,
 Customer_id NUMBER,
 Order_date DATE,
 CONSTRAINT fk_orders_customer
 FOREIGN KEY (Customer_id) REFERENCES customers (Customer_id)
);
```
for example [![Orders Table](/Tables/Orders%20Table.png)](/Tables/Orders%20Table.png)

### Order_items table

```
CREATE TABLE Order_items (
 Order_items_id NUMBER PRIMARY KEY,
 Order_id NUMBER,
 Product_id NUMBER,
 Quantity NUMBER,
 CONSTRAINT fk_items_order
 FOREIGN KEY (Order_id) REFERENCES Orders (Order_id),
 CONSTRAINT fk_items_product
 FOREIGN KEY (Product_id) REFERENCES Products (Product_id)
 );
```
for example [![Order_items](/Tables/Order_items.png)](/Tables/Order_items.png)


## Part A — SQL JOINs Implementation

### INNER JOIN: Retrieve transactions with valid customers and products

The query only displays confirmed sales transactions for which customers, orders, and products exist. This query enables management to analyze what customers have bought, where they are located, and how much revenue is being generated.

```
SELECT 
 c.Customer_name,
 c.Region,
 p.Product_name,
 oi.Quantity,
 p.price,
 (oi.quantity * p.price) AS total_amount
FROM customers c
INNER JOIN Orders o
 ON c.Customer_id = o.Customer_id
INNER JOIN Order_items oi
 ON o.Order_id = oi.Order_id
INNER JOIN Products p
 ON oi.Product_id = p.Product_id;
```
 for example [![INNER JOIN](/Joins/1%20Inner%20join.png)](/Joins/1%20Inner%20join.png)


### LEFT JOIN: Identify customers who have never made a transaction

This query shows customers who have never ordered from the business. They may be contacted with offers.

```
SELECT 
 c.Customer_id,
 c.Customer_name,
 c.Region 
FROM customers c
LEFT JOIN Orders o
 ON c.Customer_id = o.Customer_id
 WHERE o.Order_id IS NULL
;
```
for example [![LEFT JOIN](/Joins/2%20left%20join.png)](/Joins/2%20left%20join.png)

### RIGHT JOIN: Detect products with no sales activity

The query shows products that have zero sales, helping management make a decision whether to withdraw such products or not.

```
SELECT 
 p.Product_id,
 p.Product_name,
 p.Category
FROM Order_items oi
RIGHT JOIN Products p
 ON oi.Product_id = p.Product_id
WHERE oi.Order_items_id IS NULL;
```
for example [![RIGHT JOIN](/Joins/3%20Right%20join.png)](/Joins/3%20Right%20join.png)

### FULL OUTER JOIN: Compare customers and products including unmatched records

This query will provide a whole picture of the business, including customers who have not made any purchases and products that are not selling.

```
SELECT 
 c.customer_name,
 p.product_name
FROM customers c
FULL OUTER JOIN Orders o
 ON c.Customer_id = o.Customer_id
FULL OUTER JOIN Order_items oi
 ON o.Order_id = oi.Order_id
FULL OUTER JOIN Products p
 ON oi.Product_id = p.Product_id;
```
for example [![FULL OUTER JOIN](/Joins/4%20Full%20outer%20join.png)](/Joins/4%20Full%20outer%20join.png)

### SELF JOIN: Compare customers within the same region

This self-join query can help compare customers based on regions, which can help the business understand customer concentration in different regions.

```
SELECT 
 c1.Customer_name AS customer_1,
 c2.Customer_name AS customer_2,
 c1.region
FROM customers c1
JOIN customers c2
 ON c1.region = c2.region
 AND c1.customer_id < c2.customer_id;
```
for example [![SELF JOIN](/Joins/5%20Self%20join.png)](/Joins/5%20Self%20join.png)


# Success Criteria 
## Rank products by revenue per region: 

```
SELECT
    c.Region,
    p.Product_name,
    SUM(oi.Quantity * p.Price) AS total_revenue,
    ROW_NUMBER() OVER (
        PARTITION BY c.Region
        ORDER BY SUM(oi.Quantity * p.Price) DESC
    ) AS row_num,
    RANK() OVER (
        PARTITION BY c.Region
        ORDER BY SUM(oi.Quantity * p.Price) DESC
    ) AS rank_num,
    DENSE_RANK() OVER (
        PARTITION BY c.Region
        ORDER BY SUM(oi.Quantity * p.Price) DESC
    ) AS dense_rank_num,
    PERCENT_RANK() OVER (
        PARTITION BY c.Region
        ORDER BY SUM(oi.Quantity * p.Price) DESC
    ) AS percent_rank
FROM customers c
JOIN Orders o ON c.Customer_id = o.Customer_id
JOIN Order_items oi ON o.Order_id = oi.Order_id
JOIN Products p ON oi.Product_id = p.Product_id
GROUP BY c.Region, p.Product_name;
```
example: [![Rank products by revenue per region](/Window%20Functions/1%20Rank%20products%20by%20revenue%20per%20region.png)](/Window%20Functions/1%20Rank%20products%20by%20revenue%20per%20region.png)

**This query ranks products by revenue within each region, allowing management to identify top-performing and low-performing products. The ranking functions handle ties differently, making them flexible for use in performance evaluation.**

## (a) Running Monthly Sales Total (ROWS):
```
SELECT
    c.Region,
    TRUNC(o.Order_date, 'MM') AS sales_month,
    SUM(oi.Quantity * p.Price) AS monthly_sales,
    SUM(SUM(oi.Quantity * p.Price)) OVER (
        PARTITION BY c.Region
        ORDER BY TRUNC(o.Order_date, 'MM')
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM customers c
JOIN Orders o ON c.Customer_id = o.Customer_id
JOIN Order_items oi ON o.Order_id = oi.Order_id
JOIN Products p ON oi.Product_id = p.Product_id
GROUP BY c.Region, TRUNC(o.Order_date, 'MM');
```
example: [![Running Monthly Sales Total (ROWS)](/Window%20Functions/2%20Running%20Monthly%20Sales%20Total%20(ROWS).png)](/Window%20Functions/2%20Running%20Monthly%20Sales%20Total%20(ROWS).png)

**This query calculates cumulative sales over time for each region, which helps management analyze sales growth trends.**

## (b) Three-Month Moving Average (RANGE):
```
SELECT
    TRUNC(o.Order_date, 'MM') AS sales_month,
    SUM(oi.Quantity * p.Price) AS monthly_sales,
    AVG(SUM(oi.Quantity * p.Price)) OVER (
        ORDER BY TRUNC(o.Order_date, 'MM')
        RANGE BETWEEN INTERVAL '2' MONTH PRECEDING AND CURRENT ROW
    ) AS moving_avg_3_months
FROM Orders o
JOIN Order_items oi ON o.Order_id = oi.Order_id
JOIN Products p ON oi.Product_id = p.Product_id
GROUP BY TRUNC(o.Order_date, 'MM');
```
example: [![Three-Month Moving Average (RANGE)](/Window%20Functions/5%20Three-Month%20Moving%20Average%20(RANGE).png)](/Window%20Functions/5%20Three-Month%20Moving%20Average%20(RANGE).png)

**This query helps management smooth out short-term variations by calculating a moving average of sales over three months.**

## Month-over-month growth comparison:
```
SELECT
    sales_month,
    monthly_sales,
    LAG(monthly_sales, 1) OVER (
        ORDER BY sales_month
    ) AS previous_month_sales,
    (monthly_sales - LAG(monthly_sales, 1) OVER (
        ORDER BY sales_month
    )) AS month_growth
FROM (
    SELECT
        TRUNC(o.Order_date, 'MM') AS sales_month,
        SUM(oi.Quantity * p.Price) AS monthly_sales
    FROM Orders o
    JOIN Order_items oi ON o.Order_id = oi.Order_id
    JOIN Products p ON oi.Product_id = p.Product_id
    GROUP BY TRUNC(o.Order_date, 'MM')
);
```

example: [![Month-over-month growth comparison](/Window%20Functions/3%20Month-over-month%20growth%20comparison.png)](/Window%20Functions/3%20Month-over-month%20growth%20comparison.png)

**This query calculating a month-to-month sales growth. This helps management to quickly identify sales performance increases or decreases.**

## Customer revenue segmentation
```
SELECT
    customer_name,
    total_spent,
    NTILE(4) OVER (
        ORDER BY total_spent DESC
    ) AS revenue_quartile,
    CUME_DIST() OVER (
        ORDER BY total_spent DESC
    ) AS cumulative_distribution
FROM (
    SELECT
        c.Customer_name,
        SUM(oi.Quantity * p.Price) AS total_spent
    FROM customers c
    JOIN Orders o ON c.Customer_id = o.Customer_id
    JOIN Order_items oi ON o.Order_id = oi.Order_id
    JOIN Products p ON oi.Product_id = p.Product_id
    GROUP BY c.Customer_name
);
```

example: [![Customer revenue segmentation](/Window%20Functions/4%20Customer%20revenue%20segmentation.png)](/Window%20Functions/4%20Customer%20revenue%20segmentation.png)

**The query helps segment customers by their revenue contribution, enables businesses to focus marketing efforts on their high-value customers.**


